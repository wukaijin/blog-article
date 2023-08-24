搭建个人网站过程中, Article 相关接口，如何使用`TypeORM`查询数据库, 踩了不少坑，记录下。

# 环境

|        Env | Vision   |
| ---------: | :------- |
|      macOS | v13.1    |
|       Node | v16.13.1 |
|       pnpm | v7.25.0  |
| @nestjs/\* | v9.0.0   |

# Article Entity

```typescript
import { ApiProperty } from '@nestjs/swagger'
import { Category } from 'src/blog/category/entities/category.entity'
import { Tag } from 'src/blog/tag/entities/tag.entity'
import {
  Column,
  CreateDateColumn,
  Entity,
  JoinTable,
  ManyToMany,
  ManyToOne,
  PrimaryGeneratedColumn,
  UpdateDateColumn,
} from 'typeorm'

export enum ArticleState {
  UN_PUBLISHED,
  PUBLISHED,
}

@Entity()
export class Article {
  @ApiProperty({ name: 'id', description: 'ID', required: true })
  @PrimaryGeneratedColumn('uuid')
  id: string

  @ApiProperty({ name: 'text', description: '标题', required: true })
  @Column({ type: 'varchar', length: 255 })
  title: string

  @ApiProperty({ name: 'poster', description: '封面' })
  @Column({ type: 'varchar', length: 255 })
  poster: string

  @ApiProperty({ name: 'description', description: '描述' })
  @Column({ type: 'varchar', length: 1000 })
  description: string

  @ApiProperty({ name: 'tags', description: '标签' })
  @ManyToMany(() => Tag, (t) => t.articles)
  @JoinTable()
  tags: Tag[]

  @ApiProperty({ name: 'category', description: '分类', required: true })
  @ManyToOne(() => Category, (c) => c.id)
  category: Category

  @ApiProperty({ name: 'content', description: '内容', required: true })
  @Column({ type: 'longtext' })
  content: string

  @ApiProperty({ name: 'state', description: '状态 [0/1]', required: true })
  @Column({ type: 'int', default: 0 })
  state: ArticleState // 0 | 1

  @CreateDateColumn({ type: 'timestamp' })
  createAt: Date

  @UpdateDateColumn({ type: 'timestamp' })
  updateAt: Date
}
```

# Article Controller

```typescript
import {
  Controller,
  Get,
  Post,
  Body,
  Patch,
  Param,
  Delete,
} from '@nestjs/common'
import { ArticleService } from './article.service'
import { CreateArticleDto } from './dto/create-article.dto'
import { UpdateArticleDto } from './dto/update-article.dto'

@Controller('article')
export class ArticleController {
  constructor(private readonly articleService: ArticleService) {}

  @Post()
  create(@Body() createArticleDto: CreateArticleDto) {
    return this.articleService.create(createArticleDto)
  }

  @Get()
  findAll() {
    return this.articleService.findAll()
  }

  @Get('search/:keyword')
  search(@Param('keyword') keyword: string) {
    return this.articleService.search(keyword)
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.articleService.findOne(id)
  }

  @Get('relative/:id')
  findRelativeById(@Param('id') id: string) {
    return this.articleService.findRelativeById(id)
  }

  @Get('findByCategoryId/:id')
  findByCategoryId(@Param('id') id: string) {
    return this.articleService.findByCategoryId(id)
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateArticleDto: UpdateArticleDto) {
    return this.articleService.update(id, updateArticleDto)
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.articleService.remove(id)
  }
}
```

===
以上是前置信息，先从最简单的开始
===

# 如何查找关联外键数据

内置的 find、findAll 在 `findOptions`中的关联字段`relations`就可以实现，比较简单  
比如 findAll 示例：

```typescript
export class ArticleService {
  findAll() {
    return this.articleRepo.find({
      relations: ['category', 'tags'],
      // relations: {
      //   category: true,
      //   tags: true
      // },
    })
  }
}
```

# 模糊查询

typeorm 中的 `Like` 配合 SQL 中的通配符 `%`

```typescript
export class ArticleService {
  search(keyword: string) {
    return this.articleRepo.find({
      where: {
        title: Like(`%${keyword}%`),
      },
    })
  }
}
```

# 筛选匹配项和输出项

`findOptions`中的 `where` 和 `select`

```typescript
const LIST_KEYS: (keyof Article)[] = [
  'id',
  'description',
  'state',
  'poster',
  'title',
  'category',
  'tags',
]
export class ArticleService {
  findByCategoryId(id: string) {
    return this.articleRepo.find({
      where: {
        category: { id },
      },
      select: LIST_KEYS,
    })
  }
}
```

# 多重关联

这个比较有意思， 需求是查找单篇文章详情，需要关联的`category`以及`category`关联的`belongs`  
通过之前`find`做不到，relations 只能到一层表，所以要自己用`builder`实现

```typescript
export class ArticleService {
  findOne(id: string) {
    return this.articleRepo
      .createQueryBuilder('article')
      .select('article')
      .leftJoinAndSelect('article.category', 'category') // key point 1
      .leftJoinAndSelect('category.belongs', 'belongs') // key point 2
      .leftJoinAndSelect('article.tags', 'tags')
      .where({ id })
      .getOne()
  }
}
```

# 关联数据组的匹配

最后这个更有意思， 有点后端味道了。  
需求是查找单篇文章详情之后，查询改文章相关的其他文章  
文章之间是没有关联字段的，我想法是先找到同个`category`，再筛选标签组完全匹配的文章。  
搞到头秃，我 tm 用 JS 查全部再 filter 一下花不了 10 分钟，用 orm 硬是试了 2 小时

```typescript
export class ArticleService {
  async findRelativeById(id: string) {
    const target = await this.findOne(id)
    const cates = await this.articleRepo
      .createQueryBuilder('article')
      .select(['id', 'state', 'title'].map((key) => `article.${key}`))
      .leftJoin('article.category', 'category')
      .leftJoin('article.tags', 'tag')
      .where('article.category = :category', { category: target.category.id })
      .andWhere('article.id != :articleId', { articleId: target.id })
      .andWhere('tag.id in (:...tagIds)', {
        tagIds: target.tags.map((t) => t.id),
      })
      .getMany()
    return cates
  }
}
```

看最终代码挺简单的，但翻遍了官网，找不到 **.andWhere('tag.id in (:...tagIds)', { tagIds })** 这种写法，还是在 stack overflow 上看到的

```sql
SELECT 
  `article`.`id` AS `article_id`, 
  `article`.`title` AS `article_title`, 
  `article`.`state` AS `article_state`
FROM `article` `article` 
LEFT JOIN `category` `category` ON `category`.`id`=`article`.`categoryId`
LEFT JOIN `article_tags_tag` `article_tag` ON `article_tag`.`articleId`=`article`.`id`
LEFT JOIN `tag` `tag` ON `tag`.`id`=`article_tag`.`tagId`
WHERE `article`.`categoryId` = ? AND `article`.`id` != ? AND `tag`.`id` in (?)
```

[TypeORM](https://typeorm.io/) 官网
