重学`设计模式`过程中做些记录, 因个人兴趣，尝试以`TypeScript`语法基于`Node`运行 - 行为型（Behavioral）篇

# 模板方法模式 (Template)

- 在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。
- 定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤

```typescript
class CanvasCard {
  ctx: CanvasRenderingContext2D
  constructor(ctx: CanvasRenderingContext2D) {
    this.ctx = ctx
  }
  drawCircleImage(path: string, x: number, y: number, r: number) {
    const img = new Image()
    img.src = path
    this.ctx.save()
    this.ctx.beginPath()
    this.ctx.arc(x, y, r, 0, 2 * Math.PI)
    this.ctx.stroke()
    this.ctx.clip()
    this.ctx.drawImage(img, x - r, y - r, r * 2, r * 2)
    this.ctx.restore()
  }
}

class BusinessCard extends CanvasCard {
  constructor(ctx: CanvasRenderingContext2D) {
    super(ctx)
    // ... other operations
    this.drewAvatar('http://avatar.jpg', 120, 20, 25)
    this.drewLogo('http://logo.jpg', 20, 20, 8)
  }
  drewAvatar(path: string, x: number, y: number, r: number) {
    this.drawCircleImage(path, x, y, r)
  }
  drewLogo(path: string, x: number, y: number, r: number) {
    this.drawCircleImage(path, x, y, r)
  }
}

const canvas: HTMLCanvasElement | null = document.querySelector('canvas')
const ctx = canvas?.getContext('2d')
if (ctx) {
  const businessCard = new BusinessCard(ctx)
}
```

# 迭代器模式 (Iterator)

- 指提供一种方法，顺序访问某哥聚合对象中的各个元素，而不需要暴露该对象的内部表示。
- 迭代器模式可以把迭代的过程从业务逻辑中分离出来，即使用迭代器模式之后，
- 既不关心对象内部构造，也可以按顺序访问其中的每个元素。
- ES6 内置了 Generator 函数，可以生成 Iterator，用 for on/in 访问

```typescript
// ES6 之前，通常的做法是用 for/ while 循环来构造
// 以遍历某种树形结构为例

interface IFiber {
  state: string
  children?: IFiber[]
  // 这三个属性是链表结构需要
  child?: IFiber
  sibling?: IFiber
  return?: IFiber
}

class Fiber implements IFiber {
  state: string = ''
  children: Fiber[]
  child?: Fiber
  sibling?: Fiber
  return?: Fiber
  constructor(state: string = '') {
    this.state = state
    this.children = []
  }
  getLastChild() {
    let lastChild = this.child
    while (lastChild?.sibling) {
      lastChild = lastChild.sibling
    }
    return lastChild
  }
  append(child: Fiber) {
    this.children.push(child)
    if (!this.child) {
      this.child = child
    } else {
      this.getLastChild()!.sibling = child
    }
    child.return = this
    return this
  }
}

const root = new Fiber('root')
const n_2 = new Fiber('2')
const n_1 = new Fiber('1')
root.append(n_1).append(n_2)

const n_1_1 = new Fiber('1_1')
n_1.append(n_1_1)

const n_2_1 = new Fiber('2_1')
const n_2_2 = new Fiber('2_2')
const n_2_3 = new Fiber('2_3')
n_2.append(n_2_1).append(n_2_2).append(n_2_3)

const n_2_2_1 = new Fiber('2_2_1')
const n_2_2_2 = new Fiber('2_2_2')
n_2_2.append(n_2_2_1).append(n_2_2_2)

// ES5

function each(root: Fiber, callback: Function) {
  let currentNode: Fiber | undefined = root
  currentNode && callback(currentNode)
  if (currentNode?.children.length) {
    // forEach
    for (let i = 0; i < currentNode.children.length; i++) {
      const node = currentNode.children[i]
      each(node, callback)
    }
  }
}
each(root, (node: Fiber) => console.log(node.state))

/* echo separator */
console.log(Array.from(new Array(30)).join('='))
/* echo separator */

// 先序遍历
function* generateFiber(root: Fiber) {
  let currentNode: Fiber | undefined = root
  while (true) {
    // forEach
    if (!currentNode) return
    yield currentNode
    if (currentNode.child) {
      currentNode = currentNode.child
      continue
    }
    if (currentNode === root) {
      break
    }
    while (!currentNode || !currentNode.sibling) {
      if (!currentNode.return || currentNode.return === root) {
        return
      }
      currentNode = currentNode.return
    }
    currentNode = currentNode!.sibling
  }
}

const iteratorFiber = generateFiber(root)
for (const iterator of iteratorFiber) {
  console.log(iterator.state)
}
```

# 观察者模式 (Observer)

- 观察者模式包含观察目标（主体）和 观察者两类对象
- 观察目标（主体）和 观察者是一对多的关系
- 观察目标对象更改状态，其所有依赖项都会自动更新。

```typescript
type ChangeParams = Record<string, string>

class Observer {
  update(params: ChangeParams) {
    console.log(`update: ${JSON.stringify(params)}`)
  }
}

class Subject {
  dependencies: Observer[] = []
  add(dependency: Observer) {
    if (this.dependencies.indexOf(dependency) === -1) {
      this.dependencies.push(dependency)
    }
  }
  remove(dependency: Observer) {
    this.dependencies = this.dependencies.filter((d) => d !== dependency)
  }
  notify(params: ChangeParams) {
    this.dependencies.forEach((d) => d.update(params))
  }
}

// 举例: 户主寄售房子，中介接收通知

// 更严格的写法应该是 class Host/Agent extends Subject, 再 new Host/Agent
const host = new Subject()
const agent1 = new Observer()
const agent2 = new Observer()
const agent3 = new Observer()
host.add(agent1)
host.add(agent2)
host.add(agent3)

host.notify({
  type: 'decrease',
  number: '1000',
})

host.remove(agent1)

host.notify({
  type: 'increase',
  number: '500',
})
```

# 发布/订阅模式 （Publish/Subscribe）

- 这个模式其实是观察者模式的变种，在其基础上加入调度中心，达到订阅主题和观察者的完全解耦
- Vue 中的 EventBus 和 Nodejs 中的 EventEmitter
- 这两个是典型应用，其作用就是调度

```typescript
class EventEmitter<T> {
  callbacks: Record<string, Function[]> = {}
  on(type: string, cb: Function) {
    !this.callbacks[type] && (this.callbacks[type] = [])
    this.callbacks[type].push(cb)
  }
  off(type: string, cb?: Function) {
    if (!cb) {
      this.callbacks[type] = []
    } else {
      this.callbacks[type] = this.callbacks[type].filter((c) => c !== cb)
    }
  }
  emit(type: string, ...arr: T[]) {
    ;(this.callbacks[type] || []).forEach((cb) => cb(...arr))
  }
}

// 示例
class Server extends EventEmitter<string> {
  port: number = 3000
  requestHandler: Function
  constructor(requestHandler: Function) {
    super()
    this.requestHandler = requestHandler
  }
  listen(port: number) {
    this.port = port
    this.on('request', this.requestHandler)
    return this
  }
}

const server = new Server((data: string) => {
  console.log(`catch a request with ${data}`)
}).listen(3000)

server.emit('request', 'jsonData')

// 示例2，不使用继承

class Store {
  data: string = ''
  updateHandler?: Function
  update(data: string) {
    this.data = data
    this.updateHandler && this.updateHandler(data)
  }
  onUpdate(updateHandler: Function) {
    this.updateHandler = updateHandler
  }
}
class Component {
  constructor(data: string = '') {
    this.render(data)
  }
  render(data: string) {
    console.log('render data: ', data)
  }
}

const storeEvent = new EventEmitter()
const store = new Store()
store.onUpdate((data: string) => {
  storeEvent.emit('change', data)
})

const headerComponent = new Component()
const footerComponent = new Component()

storeEvent.on('change', (data: string) => {
  headerComponent.render(data)
  footerComponent.render(data)
})

store.update('Hello')
store.update('World')
```

# 命令模式（Command Pattern）

- 请求以命令的形式包裹在对象中，并传给调用对象。
- 调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。
- 命令模式一般由三种角色构成：
- 1.  发布者(invoker): 发出命令，调用命令对象，不与实际执行对象关联
- 2.  接受者(receiver): 提供对应接口处理请求，不知道谁发起请求
- 3.  命令对象(command): 接收命令，调用接收者处理发布者的请求 （middle）

```typescript
// 随便以空调举例 (空调 - 命令对象 - 遥控)
type CommandType = 'on'
// type CommandType = 'on' | 'off' | 'up' | 'down'

class AirCondition {
  on() {
    console.log('AirCondition turn on.')
  }
}

class ServiceCommand {
  airCondition: AirCondition
  constructor(airC: AirCondition) {
    this.airCondition = airC
  }
  execute(type: CommandType) {
    if (typeof this.airCondition[type] === 'function') {
      this.airCondition[type]()
    }
  }
}

class Remote {
  command: ServiceCommand
  constructor(command: ServiceCommand) {
    this.command = command
  }
  on() {
    this.command.execute('on')
  }
}

const airCondition = new AirCondition()
const command = new ServiceCommand(airCondition)
const remote = new Remote(command)

remote.on()
```

# 策略模式 (Strategy)

- 策略模式 (Strategy)
- 将多个算法分组到一个模块中以提供可替换方案。也称为 Policy Pattern
- 通过对算法进行封装，分离算法的责任和逻辑，委派给不同的对象进行管理
- 通常用于解决多重 if...else... 具有良好的扩展性
- 举例：路由

```typescript
enum performanceLevel {
  A,
  B,
  C,
  D,
}
const strategies: Record<performanceLevel, (base: number) => number> = {
  [performanceLevel.A]: (base: number) => base * 1.4,
  [performanceLevel.B]: (base: number) => base * 1.2,
  [performanceLevel.C]: (base: number) => base,
  [performanceLevel.D]: (base: number) => base * 0.8,
}

function getPerformanceBonuses(level: performanceLevel, base: number) {
  return strategies[level](base)
}

console.log(
  getPerformanceBonuses(performanceLevel.A, 2000),
  getPerformanceBonuses(performanceLevel.D, 2000),
)
```

# 职责链模式 (Chain of responsibility)

- 为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。
- 让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。

```typescript
type Params = Record<string, string>
type Rule = (p: Params) => string | void

class Task {
  rule: Rule
  constructor(rule: Rule) {
    this.rule = rule
  }
  validate(value: Params) {
    return this.rule(value)
  }
}
class Validator {
  errors: string[] | null = null
  tasks: Task[] = []
  addTask(task: Task) {
    this.tasks.push(task)
    return this
  }
  validate(p: Params) {
    this.errors = null
    this.tasks.forEach((task) => {
      const err = task.validate(p)
      if (err) {
        !this.errors && (this.errors = [])
        this.errors.push(err)
      }
    })
  }
}

const requireField = (field: string, errorTip: string) => (p: Params) => {
  if (!p[field]) return errorTip
}
const lengthMaxLimit = (field: string, errorTip: string, max: number) => (
  p: Params,
) => (p[field].length > max ? errorTip : undefined)
const requireName = requireField('name', '昵称不能为空')
const requirePwd = requireField('password', '密码不能为空')
const requireDoublePwd = requireField('doublePassword', '确认密码不能为空')
const maxName = lengthMaxLimit('name', '昵称最多10个字符', 10)
const maxPwd = lengthMaxLimit('password', '密码最多12位', 12)
const eqPwd = (p: Params) =>
  p['password'] !== p['doublePassword'] ? '两次密码不一致' : undefined

const registerValidator = new Validator()
registerValidator
  .addTask(new Task(requireName))
  .addTask(new Task(requirePwd))
  .addTask(new Task(requireDoublePwd))
  .addTask(new Task(maxName))
  .addTask(new Task(maxPwd))
  .addTask(new Task(eqPwd))

registerValidator.validate({
  name: 'abcdefghijklmnopqrstuvwxyz',
  password: '1234567890abc',
  doublePassword: '1234567890abc',
})
console.log(registerValidator.errors)

registerValidator.validate({
  name: '',
  password: '123',
  doublePassword: 'xxxxx',
})
console.log(registerValidator.errors)
```
