NestJS 项目中的鉴权（Authorization）, localUserGuard 以及 jwtGuard

# 前言

建设个人网站中的 CMS 项目，希望有个简单的用户登录模块，来控制页面的权限。  
最初的设想中，只需要两个用户，做到以下要求

- 无用户登录情况下，只能访问白名单页面，如 login, 404
- anonymous 用户拥有部分页面浏览权限
- admin 用户拥有所有权限  

于是开始参考[官网](https://docs.nestjs.com/security/authentication)推荐用法，本文做以记录

# 环境

|        Env | Vision   |
| ---------: | :------- |
|      macOS | v13.1    |
|       Node | v16.13.1 |
|       pnpm | v7.25.0  |
| @nestjs/\* | v9.0.0   |

# local passport

## login

登录接口用的是`local passport`，对比本地信息，判断是否通过。  
用到了几个第三方库  
安装相关库，生成 auth module/service, 和 auth module/controller/service

```bash
$ npm install --save @nestjs/passport passport passport-local
$ npm install --save-dev @types/passport-local
$ nest g module auth
$ nest g service auth
$ nest g resource user
```

user.service 提供本地信息作为对比依据，后面需要扩充注册功能的话，可以在这里注入表 CURD，目前需求简单，直接 static

```typescript
/*
 * @FilePath: src/user/user.service.ts
 */
import { Injectable } from '@nestjs/common'

// This should be a real class/interface representing a user entity
export interface User {
  id: string
  name: string
  password: string
}

@Injectable()
export class UserService {
  private readonly User: User[] = [
    {
      id: '1',
      name: 'admin',
      password: '123456',
    },
    {
      id: '2',
      name: 'anonymous',
      password: '123456',
    },
  ]
  constructor(private jwtService: JwtService) {}
  async findOne(name: string): Promise<User | undefined> {
    return this.User.find((user) => user.name === name)
  }
}

/*
 * @FilePath: /src/user/user.module.ts
 */
import { Module } from '@nestjs/common'
import { UsersService } from './users.service'

@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

auth.service 注入 user.service, 调用 findOne 查询对比

```typescript
/*
 * @FilePath: src/auth/auth.service.ts
 */
import { Injectable } from '@nestjs/common'
import { UserService } from '../user/user.service'

@Injectable()
export class AuthService {
  constructor(private usersService: UserService) {}

  async validateUser(username: string, pass: string): Promise<any> {
    const user = await this.usersService.findOne(username)
    if (user && user.password === pass) {
      const { password, ...result } = user
      return result
    }
    return null
  }
}
```

## local.strategy & local-auth.guard

`local.strategy`是这类鉴权的核心

```typescript
/*
 * @FilePath: src/auth/local.strategy.ts
 */
import { Strategy } from 'passport-local'
import { PassportStrategy } from '@nestjs/passport'
import { Injectable, UnauthorizedException } from '@nestjs/common'
import { AuthService } from './auth.service'

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({ usernameField: 'name' }) // ! import to convert field
  }

  async validate(username: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(username, password)
    if (!user) {
      throw new UnauthorizedException('name or password error')
    }
    return user
  }
}

/*
 * @FilePath: src/auth/local-auth.guard.ts
 */
import { Injectable } from '@nestjs/common'
import { AuthGuard } from '@nestjs/passport'

@Injectable()
export class LocalAuthGuard extends AuthGuard('local') {}

/*
 * @FilePath: src/auth/auth.module.ts
 */
import { Module } from '@nestjs/common'
import { AuthService } from './auth.service'
import { UserModule } from '../user/user.module'
import { PassportModule } from '@nestjs/passport'
import { LocalStrategy } from './local.strategy'

@Module({
  imports: [UserModule, PassportModule],
  providers: [AuthService, LocalStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

`PassportModule` 属于 `@nestjs/passport` 内置模块，我推测是提供`Guards`套接 `Strategy`  
调用 `Passport` 策略并启动并且检索凭据、运行验证功能、创建用户属性等

`super({ usernameField: 'name' })`是遇到的一个坑，库中的默认字段是 username，需要在这里转换

`AuthGuard('local')` 中的 `local`默认取自文件命名，可以在定义策略，继承的时候配置， `extends PassportStrategy(Strategy， 'foo')`

`local-auth.guard` 这里其实没必要，暂时不需要扩展`Guards`规则，抄就抄了

回到 user.controller 增加 login 接口, 增加`Guards`

```typescript
/*
 * @FilePath: src/user/user.controller.ts
 */
import { Controller, Post, Request, UseGuards } from '@nestjs/common'
import { LocalAuthGuard } from 'src/auth/local-auth.guard'
import { UserService } from './user.service'

@Controller('user')
export class UserController {
  constructor(private userService: UserService) {}
  @UseGuards(LocalAuthGuard)
  @Post('login')
  async login(@Request() req) {
    return req.user
  }
}
```

`UseGuards`从名字就能看出这个装饰器 ,链式调用参数项`Guard`.

`local passport`相关的代码到这里就没了，回过头去看，其实一个 user.service 方法就完事了，官方绕了我一大圈，又是策略又是守卫，是不是想折腾我？

# JWT passport

关于`jwt`，其实就是无状态，把登录 token 保存在客户端，回来的时候解密  
同样借助第三方库

```bash
$ npm install --save @nestjs/jwt passport-jwt
$ npm install --save-dev @types/passport-jwt
```

这部分代码和官网示例有一定不同了，做了适配性的改造

由 user.service 增加 login 方法，调用 JwtService

```typescript
/*
 * @FilePath: src/user/user.service.ts
 */
import { Injectable } from '@nestjs/common'
import { JwtService } from '@nestjs/jwt'

// This should be a real class/interface representing a user entity
export interface User {
  id: string
  name: string
  password: string
}

@Injectable()
export class UserService {
  private readonly User: User[] = [
    {
      id: '1',
      name: 'carlos',
      password: '123456',
    },
    {
      id: '2',
      name: 'anonymous',
      password: '123456',
    },
  ]
  constructor(private jwtService: JwtService) {}
  async findOne(name: string): Promise<User | undefined> {
    return this.User.find((user) => user.name === name)
  }
  async login(user: User) {
    const payload = { username: user.name, sub: user.id }
    return {
      token: this.jwtService.sign(payload),
    }
  }
}
```

user.module 导入 `jwtModule`

```typescript
/*
 * @FilePath: src/user/user.module.ts
 */
import { Module } from '@nestjs/common'
import { UserService } from './user.service'
import { UserController } from './user.controller'
import { jwtModule } from 'src/jwt.module'

@Module({
  imports: [jwtModule],
  providers: [UserService],
  exports: [UserService],
  controllers: [UserController],
})
export class UserModule {}
```

## JwtModule

JwtModule 独立成文件，按需引入

```typescript
/*
 * @FilePath: /nest-portal/src/jwt.module.ts
 */
import { jwtConstants } from 'src/auth/constants'
import { JwtModule } from '@nestjs/jwt'

export const jwtModule = JwtModule.register({
  secret: jwtConstants.secret,
  signOptions: { expiresIn: '60s' },
})

/*
 * @FilePath: src/constants.ts
 */
export const jwtConstants = {
  secret: 'carlos',
}
```

值得说道的一点，secret 不应该写死在代码里，生产环境的服务应该以其方式注入，比如环境变量，运行参数...

改造 user.controller user.service

- user.service.login 接管 user.controller.login
- user.service.login 调用 `jwtService`

```typescript
/*
 * @FilePath: /nest-portal/src/user/user.controller.ts
 * @Description: null
 */
import { Controller, Post, Req, UseGuards } from '@nestjs/common'
import { LocalAuthGuard } from 'src/auth/local-auth.guard'
import { UserService } from './user.service'

@Controller('user')
export class UserController {
  constructor(private userService: UserService) {}
  @UseGuards(LocalAuthGuard)
  @Post('login')
  async login(@Req() req) {
    return this.userService.login(req.user) // user is auto injected from the guard
  }
}

/*
 * @FilePath: src/user/user.service.ts
 */
import { Injectable } from '@nestjs/common'
import { JwtService } from '@nestjs/jwt'

// This should be a real class/interface representing a user entity
export interface User {
  id: string
  name: string
  password: string
}

@Injectable()
export class UserService {
  private readonly User: User[] = [
    {
      id: '1',
      name: 'carlos',
      password: '123456',
    },
    {
      id: '2',
      name: 'anonymous',
      password: '123456',
    },
  ]
  constructor(private jwtService: JwtService) {}
  async findOne(name: string): Promise<User | undefined> {
    return this.User.find((user) => user.name === name)
  }
  async login(user: User) {
    const payload = { username: user.name, sub: user.id }
    return {
      token: this.jwtService.sign(payload),
    }
  }
}
```

## passport-jwt strategy

```typescript
/*
 * @FilePath: src/auth/jwt.strategy.ts
 */
import { ExtractJwt, Strategy } from 'passport-jwt'
import { PassportStrategy } from '@nestjs/passport'
import { Injectable } from '@nestjs/common'
import { jwtConstants } from '../constants'

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: jwtConstants.secret,
    })
  }

  async validate(payload: any) {
    return { id: payload.sub, name: payload.name }
  }
}

/*
 * @FilePath: src/auth/auth.module.ts
 */
import { Module } from '@nestjs/common'
import { AuthService } from './auth.service'
import { UserModule } from '../user/user.module'
import { PassportModule } from '@nestjs/passport'
import { LocalStrategy } from './local.strategy'
import { jwtModule } from 'src/jwt.module'
import { JwtStrategy } from './jwt.strategy'

@Module({
  imports: [UserModule, PassportModule, jwtModule],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

## jwt-auth.guard

代码到这儿， jwt 信息注入完成，但是如果像`localUserGuard`， 每个模块，接口都去引入 `jwtGuard` 是很麻烦的，要做成全局 guard  
`jwt-auth.guard` 自定义 `canActivate`规则，提供装饰器注册到全局

```typescript
/*
 * @FilePath: src/auth/jwt-auth.guard.ts
 */
import { ExecutionContext, Injectable, UnauthorizedException } from '@nestjs/common'
import { AuthGuard } from '@nestjs/passport'

import { SetMetadata } from '@nestjs/common'
import { Reflector } from '@nestjs/core'

export const IS_PUBLIC_KEY = 'isPublic'
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true)

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super()
  }
  canActivate(context: ExecutionContext) {
    // Add your custom authentication logic here
    // for example, call super.logIn(request) to establish a session.
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass()
    ])
    if (isPublic) {
      return true
    }
    return super.canActivate(context)
  }

  handleRequest(err, user, info, context: ExecutionContext, status: any) {
    // You can throw an exception based on either "info" or "err" arguments
    if (err || !user) {
      throw err || new UnauthorizedException()
    }
    return user
  }
}


/*
 * @FilePath: src/app.module.ts
 */
import { Module } from '@nestjs/common'
import { APP_GUARD } from '@nestjs/core'
import { JwtAuthGuard } from './auth/jwt-auth.guard'

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard
    }
    // ...
  ]，
  // ...
})
export class AppModule {}

```

这个时候，所有的接口地址都要鉴权了，不需要鉴权的接口，比如`user/login`怎么办呢...用刚才的 Public 装饰器啊

```typescript
/*
 * @FilePath: src/user/user.controller.ts
 */
import { Controller, Post, Req, UseGuards } from '@nestjs/common'
import { LocalAuthGuard } from 'src/auth/local-auth.guard'
import { UserService } from './user.service'
import { Public } from '../auth/jwt-auth.guard'

@Controller('user')
export class UserController {
  constructor(private userService: UserService) {}
  @Public()
  @UseGuards(LocalAuthGuard)
  @Post('login')
  async login(@Req() req) {
    return this.userService.login(req.user) // user is auto injected from the guard
  }
}
```

经过 `@Public()` 修饰，全局的 `JwtAuthGuard` 就 unActivated

# Front-End

最后看下前端接口

```typescript
/*
 * @FilePath:src/api/user.ts
 */
const USER_PREFIX = '/user/'

export const userApi = {
  login(data: Partial<User>) {
    return request<{ token: string }>({
      url: `${USER_PREFIX}/login`,
      data,
      method: 'POST',
    })
  },
}

//  login-form
login({ ...user }).then((res) => {
  if (res.token) {
    console.log('token', res.token)
    localStorage.setItem('token', res.token)
    injectToken(res.token)
    router.push('/')
  }
})

// utils/request.ts
// instance: AxiosInstance
export function injectToken(token: string) {
  instance.defaults.headers.common['Authorization'] = `Bearer ${token}`
}
```
