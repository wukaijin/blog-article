重学`设计模式`过程中做些记录, 因个人兴趣，尝试以`TypeScript`语法基于`Node`运行 - 结构型（Structural）篇

# 装饰器模式 (Decorator)

对已有功能进行扩展，不改动原有的代码，对其他业务造成的影响。  
TypeScript 中有装饰器， 但是必须额外添加配置  
比较经典的应用场景，是页面点击事件跳转页面前的统计 PV（page view）

```typescript
import fs from 'fs'
import path from 'path'

const gitignore = path.resolve(__dirname, '../../', '.gitignore')

// interface Function {
//   before(): void
// }
// @ts-ignore
Function.prototype.before = function (beforeFn) {
  const originalFn = this
  return function (this: Function) {
    beforeFn.apply(this, arguments)
    return originalFn.apply(this, arguments)
  }
}

const appendFile = (filePath: string, data: string) => {
  fs.appendFileSync(filePath, data)
}

function logChange(content: string) {
  console.log('change: ', content)
}

// @ts-ignore
const appendFileAndLog = appendFile.before(logChange)
appendFile(gitignore, '\r\nsomethings')
appendFileAndLog(gitignore, '\r\nsomethings with log')

// echo separator
console.log(Array.from(new Array(30)).join('='))

// TypeScript 中的装饰器， 展示部分功能。
// 简单使用， 直接装饰 class， log会提前执行 (声明D ump 的时候)
function log(str: string) {
  return function (target: Function) {
    console.log(str + ' log')
  }
}
@log('@')
class Dump {
  constructor() {}
}
// new Dump()

// 复杂应用，事件附加 PV
function PV(str: string) {
  return function (
    target: ClickEvent,
    methodName: string,
    descriptor: PropertyDescriptor,
  ) {
    const originalFn = descriptor.value
    // 拦截重写方法
    descriptor.value = function () {
      console.log(`${str} PV data send`)
      originalFn.apply(this, arguments)
    }
  }
}

class ClickEvent {
  @PV('home')
  linkToPage() {
    console.log('linkToPage')
  }
}

const event = new ClickEvent()
event.linkToPage()
```

# 适配器模式 (Adapter)

- 抹平接口差异
- 将现有接口转换为新接口，以实现不相关类的兼容性和可重用性
- 在一个应用程序中。也称为包装模式。

```typescript
class TencentSDK {
  receiver: string
  constructor(receiver: string) {
    this.receiver = receiver
  }
  pay(money: number) {
    console.log(`pay ${this.receiver} ￥${money}`)
  }
}

class AliSDK {
  recipient: string
  constructor(recipient: string) {
    this.recipient = recipient
  }
  give(money: number) {
    console.log(`pay ${this.recipient} ￥${money}`)
  }
}
// 实现一个阿里支付的适配器
class AliPayAdapter extends AliSDK {
  constructor(recipient: string) {
    super(recipient)
  }
  pay(money: number) {
    this.give(money)
  }
}

type PaySDK = TencentSDK | AliPayAdapter

function pay(sdk: PaySDK, money: number) {
  sdk.pay(money)
}

pay(new TencentSDK('David'), 13)
pay(new AliPayAdapter('Smith'), 20)
```

# 代理模式 (Proxy)

- 使用简单对象表示复杂对象，或为另一个对象提供占位符以控制对它的访问
- ES6 Proxy 类是典型应用

```typescript
class Star {
  aliasName: string
  constructor(aliasName: string) {
    this.aliasName = aliasName
  }
  show() {
    console.log(`${this.aliasName} is showing`)
  }
}

class StarProxy {
  star: Star
  constructor(star: Star) {
    this.star = star
  }
  show(currency: number) {
    if (currency < 100) {
      console.log(`${this.star.aliasName} deny to show`)
    } else {
      this.star.show()
    }
  }
}
const star = new Star('David')
const agent = new StarProxy(star)
agent.show(200)
agent.show(88)

// ES6 Proxy 应用示例

const vueData = {
  foo: 'foo',
  arr: [1, 2, 3],
}

const proxy = new Proxy(vueData, {
  get: (target, p) => {
    console.log(`collect dependencies about ${p.toString()}`)
    // @ts-ignore
    return target[p]
  },
  set: (target, p, newValue) => {
    // @ts-ignore
    target[p] = newValue
    console.log(`notify dependencies to rerender about ${p.toString()}`)
    return true
  },
})
proxy.foo
proxy.foo = 'coo'

proxy.arr[0] = 0 // 无法劫持对象中的数组的直接赋值变化, vue3 中会再添加数组代理来劫持
console.log(vueData)
```

# 组合模式（Composite）

- 把一组对象相似当做单一的对象，依据树形结构来组合对象。
- 部分-整体的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

```typescript
import { promises as fs } from 'fs'
import path from 'path'

type CocStr = {
  name: string
  isDirectory: boolean
  children: CocStr[]
}
/**
 * @description: 文件夹
 * @param {string} path
 * @return {*}
 */
class Doc {
  path: string
  isDirectory: boolean
  children: Doc[]
  constructor(path: string) {
    this.path = path
    this.isDirectory = false
    this.children = []
  }
  scan() {
    fs.readdir(this.path).then((files) => {
      if (files.length) {
        this.isDirectory = true
        files.forEach((file) => {
          const filePath = path.resolve(this.path, file)
          if (file.indexOf('node_modules') !== -1) return
          fs.stat(filePath).then((fileStat) => {
            const f = new Doc(filePath)
            this.children.push(f)
            if (fileStat.isDirectory()) {
              f.scan()
            }
          })
        })
      }
    })
  }
  toString(): CocStr {
    // easy to console
    return {
      name: this.path.split('/').pop() || '',
      // name: this.path,
      isDirectory: this.isDirectory,
      children: this.children.map((c) => c.toString()),
    }
  }
}

const repository = path.resolve(__dirname, '../..')
const repositoryDoc = new Doc(repository)
repositoryDoc.scan()
setTimeout(() => console.log(repositoryDoc.toString()), 1000)
```

# 桥接模式 （Bridge）

- 这个模式在前端应用还是比较生硬的，有点难理解。
- 关键词是`多维度，增加复杂度，解藕`
- 桥接模式（Bridge Pattern）是用于把抽象化与实现化解耦，使得二者可以独立变化。
- 它通过提供抽象化和实现化之间的桥接结构，来实现二者的解耦。
- 这种模式涉及到一个作为桥接的接口，使得实体类的功能独立于接口实现类。
- 这两种类型的类可被结构化改变而互不影响。将抽象部分与实现部分分离，使它们都可以独立的变化。

```typescript
// 比较难举例唉
interface IAnimationLib {
  hide: (str: string) => void
  show: (str: string) => void
}

class DialogComponent {
  animation: IAnimationLib
  name: string = ''
  constructor(animation: IAnimationLib) {
    this.animation = animation
  }
  show() {
    this.animation.show(this.name)
  }
  hide() {
    this.animation.hide(this.name)
  }
}
class Modal extends DialogComponent {
  name: string = 'Modal'
}
class Message extends DialogComponent {
  name: string = 'Message'
}
class Toast extends DialogComponent {
  name: string = 'Toast'
}

const logWithName = (str: string) => (str2: string) => console.log(str2, str)

const Animations: Record<string, IAnimationLib> = {
  bounce: {
    show: logWithName('bounce.show'),
    hide: logWithName('bounce.hide')
  },
  slide: {
    show: logWithName('slide.show'),
    hide: logWithName('slide.hide')
  },
  rotate: {
    show: logWithName('rotate.show'),
    hide: logWithName('rotate.hide')
  }
}
const modal = new Modal(Animations.slide)
const message = new Message(Animations.bounce)

modal.show()
message.hide()
```
