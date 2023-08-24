搭建个人网站过程中，希望博客模块在架构上是动态的，分类、标签、文章希望由接口获取。  
这就需要一个后台管理系统，相应地，又需要后台服务以及数据库配合 CURD  
选型没纠结，直接`NestJS`，之前就想玩玩这个久负盛名的 node 服务架构了  
本文**流水记录**首次使用`NestJS`, 不会对内容做详细解释和介绍。

# 环境

|        Env | Vision   |
| ---------: | :------- |
|      macOS | v13.1    |
|       Node | v16.13.1 |
|       pnpm | v7.25.0  |
| @nestjs/\* | v9.0.0   |

# 安装

我倾向与用`pnpm`，因为是个人电脑，为了节省硬盘。

```bash
$ pnpm install @nestjs/cli
$ nest new project-name
$ cd project-name
$ pnpm run start:dev
```

不用再 install 了，cli 在 new 的时候已经做了。

# prettier

进去后随便加了点东西，发现 prettier 报红，和我编码风格不太一样。  
改了`.prettierrc`，可以格式化了，但是还是不能去掉报错。  
查了下还需要改 `.eslintrc.js`中的 rule，配置两次 😅

```javascript
{
  rules: {
    'prettier/prettier':base
      'error',
      {
        printWidth: 100,
        tabWidth: 2,
        useTabs: false,
        semi: false,
        ...
      }
    ],
    '@typescript-eslint/interface-name-prefix': 'off',
    ...
  }
}
```

# 路由示例

构建路由很简单哇 🤩

```typescript
@Controller('/cat')
export class CatController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getCat(): string {
    return 'Cats!'
  }
  @Get('one')
  findCat(@Query('id') id: number): string {
    return `${id} one cat found!`
  }
  @Post('feed/:id')
  feedCat(@Body() body: { food: string }, @Param('id') id: number) {
    return `Cat ${id} is eating ${body.food}.`
  }
}
```

# 结构

博客模块，分类、标签、文章分别创建目录, 用 cli 快捷创建模块

```bash
$ nest g module blog
$ nest g controller blog/category
$ nest g service blog/category
```

├── app.controller.spec.ts  
├── app.controller.ts  
├── app.module.ts  
├── app.service.ts  
├── blog  
│   ├── blog.module.ts  
│   └── category  
│   ├── category.controller.spec.ts  
│   ├── category.controller.ts  
│   ├── category.service.spec.ts  
│   └── category.service.ts  
├── cat.controller.ts  
└── main.ts

牛的牛的...就是文件风格跟我的不一致，要 fix 一下  
以下是 `nest -v` 出来的可以模版生成的功能

| name          | alias       | description                                  |
| :------------ | :---------- | :------------------------------------------- |
| application   | application | Generate a new application workspace         |
| class         | cl          | Generate a new class                         |
| configuration | config      | Generate a CLI configuration file            |
| controller    | co          | Generate a controller declaration            |
| decorator     | d           | Generate a custom decorator                  |
| filter        | f           | Generate a filter declaration                |
| gateway       | ga          | Generate a gateway declaration               |
| guard         | gu          | Generate a guard declaration                 |
| interceptor   | itc         | Generate an interceptor declaration          |
| interface     | itf         | Generate an interface                        |
| middleware    | mi          | Generate a middleware declaration            |
| module        | mo          | Generate a module declaration                |
| pipe          | pi          | Generate a pipe declaration                  |
| provider      | pr          | Generate a provider declaration              |
| resolver      | r           | Generate a GraphQL resolver declaration      |
| service       | s           | Generate a service declaration               |
| library       | lib         | Generate a new library within a monorepo     |
| sub-app       | app         | Generate a new application within a monorepo |
| resource      | res         | Generate a new CRUD resource                 |

我只认识 module 、controller 和 service 🙃  
看到 `resource` 有 CURD，突然很感兴趣!
好的，百度完。再巴拉[官网](https://docs.nestjs.com/)  
回来了，懂了，开工

# Resource

```bash
$ nest g resource blog/tag
```

- ? What transport layer do you use? REST API
- ? Would you like to generate CRUD entry points? Yes

啊，知识盲区，REST API 没得选, 第二个问题，我要的不就是这个？

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
import { TagService } from './tag.service'
import { CreateTagDto } from './dto/create-tag.dto'
import { UpdateTagDto } from './dto/update-tag.dto'

@Controller('tag')
export class TagController {
  constructor(private readonly tagService: TagService) {}

  @Post()
  create(@Body() createTagDto: CreateTagDto) {
    return this.tagService.create(createTagDto)
  }

  @Get()
  findAll() {
    return this.tagService.findAll()
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.tagService.findOne(id)
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateTagDto: UpdateTagDto) {
    return this.tagService.update(id, updateTagDto)
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.tagService.remove(id)
  }
}
```

功能齐全，我哭了。 赶紧删了 category 目录，重来一遍...

# Swagger

## 配置下 swagger 和 ui

```bash
$ pnpm install @nestjs/swagger swagger-ui-express
```

安装完后报了个错  
├─┬ ts-loader 9.4.2  
│ └── ✕ missing peer webpack@^5.0.0  
└─┬ swagger-ui-express 4.6.0  
└── ✕ missing peer express@>=4.0.0  
看了下，/build 下是有编译打包文件的，webpack 应该是有的，这个报错也应该算是版本不符，可以不管

## 修改`main.ts`

```typescript
import { NestFactory } from '@nestjs/core'
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger'
import { AppModule } from './app.module'

async function bootstrap() {
  const config = new DocumentBuilder()
    .setTitle('NestJS Portal Swagger')
    .setDescription('NestJS Portal Swagger')
    .setVersion('1.0')
    .build()
  const app = await NestFactory.create(AppModule)
  const document = SwaggerModule.createDocument(app, config)
  SwaggerModule.setup('swagger', app, document)

  await app.listen(3000)
}
bootstrap()
```

起飞！  
![swagger snapshot](https://i.ibb.co/B2dqCSW/swagger-snapshot.png)

`@nestjs/swagger`提供了很多 Api 开头的装饰，用于文档归类，以及接口，接口参数描述。

# Interceptor

创建一个全局拦截器，将接口返回信息包裹下, `Rxjs`格式化数据

```typescript
// wrapper.interceptor.ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common'
import { Observable } from 'rxjs'
import { map } from 'rxjs/operators'

interface Data<T> {
  data: T
}
@Injectable()
export class WrapperInterceptor<T = any> implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Data<T>> {
    const response = next.handle().pipe(
      map((data) => ({
        data,
        code: 200,
        success: true,
        message: 'success',
      })),
    )
    return response
  }
}
// main.ts
app.useGlobalInterceptors(new WrapperInterceptor())
```

查了下资料，这个全局成功的拦截器，之后相应的要配置`Filter`做异常拦截包裹，参数检查，`Guard`权限守卫等等，想了想，后面有时间再一起做吧，先完成主业务逻辑

# Database

数据库直接选型`mysql`, 模型映射的工具中，`sequelize`以前用过，`Prisma`看着挺麻烦的，但看评测最牛逼啊  
算了，老老实实用 nestjs 推荐的 `typeORM`~ 😢

## 安装依赖

```bash
$ pnpm install --save @nestjs/typeorm typeorm mysql2
```

机子上有 Docker，拉个 mysql 镜像起服务

## 编码

```typescript
//  typeorm.module.ts
import { TypeOrmModule } from '@nestjs/typeorm'
const entitiesPath = path.resolve(__dirname, '..', '/**/*.entity.{ts,.js}')
const TypeormModule = TypeOrmModule.forRoot({
  type: 'mysql',
  username: 'root',
  password: '123456',
  host: 'localhost',
  port: 3306,
  database: 'portal',
  entities: [entitiesPath],
  synchronize: true,
  retryDelay: 500,
  retryAttempts: 10,
  autoLoadEntities: true // 由 forFeature 注册，自动加载实体
})
export default TypeormModule

// app.module.ts
@Module({
  imports: [CategoryModule, TagModule, TypeormModule],
  controllers: [AppController, CatController],
  providers: [AppService]
})
```

编写 tag 文件夹下的 entity, 为什么不先写 category 呢， 因为其中有个字段关联没想好怎么搞 💔

```typescript
// blog/tag/entities/tag.entity.ts

import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from 'typeorm'
@Entity()
export class Tag {
  @PrimaryGeneratedColumn('uuid')
  id: number

  @Column({ type: 'varchar', length: 255 })
  text: string

  @Column({ type: 'char', length: 7 })
  color: string

  @CreateDateColumn({ type: 'timestamp' })
  createAt: Date

  @UpdateDateColumn({ type: 'timestamp' })
  updateAt: Date
}

// blog/tag/tag.module.ts

import { Tag } from './entities/tag.entity'
@Module({
  imports: [TypeOrmModule.forFeature([Tag])],
  controllers: [TagController],
  providers: [TagService]
})
```

# CURD

到这终于要开始 CURD 了 😤  
controller 不用改，先从`blog/tag/dto/create-tag.dto.ts`中的`CreateTagDto`开始  
是时候展示真真的技术了！
直接干掉类声明重写

```typescript
- export class CreateTagDto {}
+ export type CreateTagDto = Omit<Tag, 'id' | 'updateAt' | 'createAt'>
// 同样的
- export class UpdateTagDto extends PartialType(CreateTagDto) {}
+ export type UpdateTagDto = Partial<CreateTagDto>
```

```typescript
// blog/tag/tag.service.ts

import { Injectable } from '@nestjs/common'
import { InjectRepository } from '@nestjs/typeorm'
import { Repository } from 'typeorm'
import { CreateTagDto } from './dto/create-tag.dto'
import { UpdateTagDto } from './dto/update-tag.dto'
import { Tag } from './entities/tag.entity'

@Injectable()
export class TagService {
  constructor(
    @InjectRepository(Tag) private readonly tagRepo: Repository<Tag>,
  ) {}

  create(createTagDto: CreateTagDto) {
    return this.tagRepo.save(createTagDto)
  }

  findAll() {
    return this.tagRepo.find({ order: { updateAt: 'DESC' } })
  }

  findOne(id: number) {
    return this.tagRepo.findOne({ where: { id } })
  }

  update(id: number, updateTagDto: UpdateTagDto) {
    return this.tagRepo.update(id, updateTagDto)
  }

  remove(id: number) {
    return this.tagRepo.delete(id)
  }
}
```

前端 api 填充

```typescript
// Frontend  api/blog.ts
const BLOG_PREFIX = '/nest-api/blog/'
type Wrapping = { code: number; message: string; success: boolean }
type WithWrapping<T> = T & Wrapping

export const TagApi = {
  findAll() {
    return request
      .get<WithWrapping<Tag[]>>(`${BLOG_PREFIX}tag`)
      .then((res) => res.data)
  },
  add(data: Partial<Tag>) {
    return request.post<WithWrapping<Tag>>(`${BLOG_PREFIX}tag`, data)
  },
  edit(id: Tag['id'], data: Partial<Tag>) {
    return request.patch<WithWrapping<Tag>>(`${BLOG_PREFIX}tag/${id}`, data)
  },
  delete(id: Tag['id']) {
    return request.delete<WithWrapping<Tag>>(`${BLOG_PREFIX}tag/${id}`)
  },
}
```

代码写到这，权限，参数检查，分页这些罗布考虑的话，tag 的简单服务就算可以了。  
 在这个过程中，还有个小问题： dto 声明成 `Type`不会影响服务，但是 swagger 的参数说明，试用功能受限， 所以又老老实实改回来

```typescript
// blog/tag/dto/create-tag.dto.ts

import { ApiProperty } from '@nestjs/swagger'

export class CreateTagDto {
  @ApiProperty({ name: 'text', description: '文本', required: true })
  text: string

  @ApiProperty({ name: 'color', description: '颜色', required: true })
  color: string
}
```

![response](https://i.ibb.co/YDfJT71/tag-resonce.png)

简单搭建完 tag 相关的 CURD，接着开始扩充

# Category Module

按照之前 Tag Module 先抄一遍代码。  
这个过程中有个字段，之前比较犹豫的。在我设计中，`category.belongs`这个字段需要自关联

```typescript
// blog/category/entities/category.entity.ts
Entity()
export class Category {
  @ApiProperty({ name: 'id', description: 'ID', required: true })
  @PrimaryGeneratedColumn('uuid')
  id: string

  @ApiProperty({ name: 'text', description: '文本', required: true })
  @Column({ type: 'varchar', length: 255 })
  text: string

  @ApiProperty({ name: 'belongs', description: '属于' })
  @JoinColumn({ name: 'category', referencedColumnName: 'id' })
  belongs: string // ！ 注意这里， 用字符串成功了

  @CreateDateColumn({ type: 'timestamp' })
  createAt: Date

  @UpdateDateColumn({ type: 'timestamp' })
  updateAt: Date
}
```

用`@JoinColumn`装饰器关联自己，姑且一试，看能不能约束到删除操作，不能的话就要自己加逻辑了

这个过程中，Tag 出现了个小问题: 前端`add`的时候，传入了一个空字符值的`id`，竟然成功了。  
接着要删除该项又失败了，匹配不到这个 delete 接口`Cannot DELETE /nest-api/blog/tag/`，嗯，前端先过滤参数，脏数据先不管...🙄

经过长达 2 小时的试错，终于搞出来了，搞定了这个是鸡生蛋蛋生鸡的问题。 🙈  
![response image](https://i.ibb.co/2nSpP1w/belongs-snapshot.png)

并且删除有`belongs`的`category`，报错

---

Cannot delete or update a parent row: a foreign key constraint fails (`portal`.`category`, CONSTRAINT `FK_92f293204f627fe46208fa39a55` FOREIGN KEY (`belongs`) REFERENCES `category` (`id`))

---

不过前端还接不到，目前仍然是 500，Internal server error

# Article

文章表的创建过程中遇到一系列问题，主要还是和标签建立`ManyToMany`的过程中各种试错。  
贴一版成功后的代码, 主要是`entity`的的关联

```typescript
// blog/article/entities/article.entity.ts
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
  category: string

  @ApiProperty({ name: 'content', description: '内容', required: true })
  @Column({ type: 'varchar', length: 5000 })
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

`TagEntity`也需要加一行外键

```typescript
// blog/article/entities/article.entity
@Entity()
export class Tag {
  // ...
  @ApiProperty({ name: 'articles', description: '文章' })
  @ManyToMany(() => Article, (a: Article) => a.tags)
  articles: Article[]
  // ...
}
```

比较有意思的一点，这部分代码中，一开始，没有改动`TagEntity`, `ArticleEntity`中的`tags`定义为`string[]`, `@ManyToMany(() => Tag, t => t.id)`,这种方式创建了文章，但是 tags 并没有关联进去， 很是疑惑，明明`category`这个多对一关系就能成功。  
最后 article service 也要小改一下创建和更新, tags 必须转为和实体一样的 Object, 又不用完全查找出来，就是简单的 { id: id }

```typescript
// blog/article/article.service.ts
@Injectable()
export class ArticleService {
  constructor(
    @InjectRepository(Article)
    private readonly articleRepo: Repository<Article>,
  ) {}
  async create(createArticleDto: CreateArticleDto) {
    const tagsIds = createArticleDto.tags
    return this.articleRepo.save({
      ...createArticleDto,
      tags: tagsIds.map((t) => ({ id: t })),
    })
  }

  findAll() {
    return this.articleRepo.find({
      order: {
        updateAt: 'DESC',
      },
      relations: ['category', 'tags'],
    })
  }

  findOne(id: string) {
    return this.articleRepo.findOne({
      where: { id },
      relations: ['category', 'tags'],
    })
  }

  async update(id: string, updateArticleDto: UpdateArticleDto) {
    const tagsIds = updateArticleDto.tags
    return this.articleRepo.save({
      ...updateArticleDto,
      tags: tagsIds.map((t) => ({ id: t })),
    })
  }

  remove(id: string) {
    return this.articleRepo.delete(id)
  }
}
```

服务这里写的也比较简单，涉及到关联的两个表应该用事务来做，出错了可以回滚，这些容错先放一放，搞搞前端 UI 了  
完事，撒花

