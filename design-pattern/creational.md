
重学`设计模式`过程中做些记录, 因个人兴趣，尝试以`TypeScript`语法基于`Node`运行 - 创建型（Creational）篇

# 单例模式 (Singleton)
保证一个类仅有一个实例，并提供一个全局访问入口  
主要解决一个全局或多处使用的类，频繁的创建或销毁实例，占用内存和消耗性能问题，或者是为了保存统一配置或数据  
早期的 JQuery, Vuex 都是其典型应用  
```typescript
// JavaScript 早期常用自执行函数来做单例
var JQuery = (function() {
  var $: _JQuery
  // 用 class 是为了配合 ts, es5 是用 function + prototype 的方式
  class _JQuery {
    text() {
      console.log('render innerText of HTMLElement')
    }
    html() {
      console.log('render innerHTML of HTMLElement')
    }
  }
  return function () {
    if ($) return $
    return ($ = new _JQuery())
  }
})()
console.log('JQuery: ', JQuery() === JQuery())

// ES6 的单例一般直接用 class 实现

class Vuex {
  static store: Vuex
  constructor() {
    if (!Vuex.store) {
      Vuex.store = this  // ! important
    }
    return Vuex.store
  }
}
new Vuex()
console.log('Vuex: ', new Vuex() === new Vuex())
console.log('Vuex store: ', new Vuex() === Vuex.store)
console.log('test')
```

# Factory Method (工厂模式)
由一个工厂对象决定创建某种产品对象的类实例，主要创建`同一种`对象  
场景：  
根据角色类型，创建具有不同页面访问权限的用户  
```typescript

type Role = 'superAdmin' | 'admin' | 'user' | 'visitor'
type Auth = 'register' | 'login' | 'setting' | 'home' | 'system'

// type Factory = <T>(c: { new (): T }) => T

interface IUser {
  readonly role: Role
  readonly name: string
  auths: Auth[]
}
const superAdminAuths: Auth[] = [
  'register',
  'login',
  'setting',
  'home',
  'system'
]
const adminAuths: Auth[] = ['register', 'login', 'setting', 'home']
const userAuths: Auth[] = ['register', 'login', 'home']
const visitorAuths: Auth[] = ['register', 'login']
// const emptyAuths = []

class User implements IUser {
  readonly role: Role
  readonly name: string
  readonly auths: Auth[]
  constructor(role: Role, name: string, auths?: Auth[]) {
    this.role = role
    this.name = name
    this.auths = auths || []
  }
  static factory(role: Role, name: string): User {
    switch (role) {
      case 'superAdmin':
        return new User(role, name, superAdminAuths)
      case 'admin':
        return new User(role, name, adminAuths)
      case 'user':
        return new User(role, name, userAuths)
      case 'visitor':
        return new User(role, name, visitorAuths)
      default:
        return new User(role, name)
    }
  }
}
const superAdmin = User.factory('superAdmin', 'Mary')
const admin = User.factory('admin', 'Bob')
const user = User.factory('user', 'David')
const visitor = User.factory('visitor', 'Vik')

console.log(superAdmin, admin, user, visitor)

```
# 原型模式 (Prototype)
原型模式 (Prototype)  
JavaScript 中的数据类型都有其原型  
比如 Object.prototype, String.prototype  
```typescript
interface Student {
  name: string
  age: number
  grade: number
  study: () => void
}

// ES6 之前，原型继承是典型用法，作用是在 __proto__ 共享属性和方法
function studentCreator(
  this: Student,
  name: string,
  age: number,
  grade: number
): Student {
  this.name = name
  this.age = age
  this.grade = grade
  return this
}
studentCreator.prototype.study = function () {
  console.log(`${this.name} is learning~~1`)
}

// ES6 class 中的 constructor 就是构造函数
class StudentClass implements Student {
  name: string
  age: number
  grade: number
  constructor(name: string, age: number, grade: number) {
    this.name = name
    this.age = age
    this.grade = grade
  }
  study() {
    console.log(`${this.name} is learning~~2`)
  }
}

const student_1: Student = new (studentCreator as any)('Lily', 13, 6)

const student_2: Student = new StudentClass('David', 16, 10)

console.log(student_1, student_2)
student_1.study()
student_2.study()
```
# 构造器模式 (Constructor)
构造器模式并未在常规的23模式命名中，但 JS 中常用这种说法，列出参考
```typescript
// ES6 之前，构造器继承是典型用法
interface Student {
  name: string
  age: number
  grade: number
}
// const student_1: Student = {
//   name: 'David',
//   age: 16,
//   grade: 10
// }

// const student_2: Student = {
//   name: 'Lily',
//   age: 13,
//   grade: 6
// }

export function createStudent(
  name: string,
  age: number,
  grade: number
): Student {
  return {
    name,
    age,
    grade
  }
}
 

// ES6 class 中的 constructor 就是构造函数
export class StudentClass implements Student {
  name: string
  age: number
  grade: number
  constructor(name: string, age: number, grade: number) {
    this.name = name
    this.age = age
    this.grade = grade
  }
}

const student_1: Student = createStudent('Lily', 13, 6)

const student_2: Student = new StudentClass('David', 16, 10)

console.log(student_1, student_2)
```
# 抽象工厂模式 (Abstract Factory)
顾名思义，抽象工厂模式并不直接产生实例，而是用于对产品类簇的创建  
TypeScript 中有所谓的抽象类，是其一定的应用  
JavaScript 中会狭义一些，更多代表的生成工厂的方法  

```typescript

/**
 *  TypeScript 示例场景
 * 几何体不能直接实例化，并包含计算面积的抽象方法
 * 圆形，正方形，正三角形继承自抽象类几何体，自行实现抽象方法
 */

abstract class Geometry {
  abstract getArea(): number
  echoArea() {
    console.log(
      Object.getPrototypeOf.call(this, this).constructor.name,
      this.getArea()
    )
  }
}

class Circle extends Geometry {
  r: number
  constructor(r: number) {
    super()
    this.r = r
  }
  getArea() {
    return Math.PI * Math.pow(this.r, 2)
  }
}

class Square extends Geometry {
  w: number
  constructor(w: number) {
    super()
    this.w = w
  }
  getArea() {
    return Math.pow(this.w, 2)
  }
}

class EquilateralTriangle extends Geometry {
  h: number
  constructor(w: number) {
    super()
    this.h = w
  }
  getArea() {
    return Math.pow(this.h, 2) / 2
  }
}

const circle = new Circle(2)
const square = new Square(4)
const equilateralTriangle = new EquilateralTriangle(4)

circle.echoArea()
square.echoArea()
equilateralTriangle.echoArea()

// echo separator
console.log(Array.from(new Array(30)).join('='))


/**
 * JavaScript 的应用大致是这样, 比较冗余，并不常用
 */

type Type = 'circle' | 'square' | 'equilateralTriangle'

function getGeometryFactory(type: Type) {
  switch (type) {
    case 'circle':
      return Circle
      case 'square':
        return Square
        case 'equilateralTriangle':
      return EquilateralTriangle
    default:
      throw new Error('not collect type')
  }
}

const Geom1 = getGeometryFactory('circle')
const Geom2 = getGeometryFactory('square')
const Geom3 = getGeometryFactory('equilateralTriangle')
const circle2 = new Geom1(1)
const square2 = new Geom2(1)
const equilateralTriangle2 = new Geom2(1)
circle2.echoArea()
square2.echoArea()
equilateralTriangle2.echoArea()
```

# 创建者模式 (Builder Pattern)
将复杂对象的创建和表示分离，使得同样的构建过程可以创建不同的表示  
创建者模式是一步一步的创建一个复杂对象，它允许用户只通过指定复杂的对象的类型和内容，就可以构建他们，用户不需要指定其内部的具体构造细节。  

```typescript
class NavBar {
  constructor() {}
  init() {
    console.log('NavBar init')
  }
  getData() {
    return new Promise<string>(resolve => {
      setTimeout(() => {
        console.log('NavBar getData')
        resolve('data')
      }, 200)
    })
  }
  render() {
    console.log('NavBar render')
  }
}

class ContentList {
  constructor() {}
  init() {
    console.log('ContentList init')
  }
  getData() {
    return new Promise<string>(resolve => {
      setTimeout(() => {
        console.log('ContentList getData')
        resolve('data')
      }, 200)
    })
  }
  render() {
    console.log('ContentList render')
  }
}
type PageComponent = InstanceType<typeof NavBar> | InstanceType<typeof ContentList>

async function buildPage(comp: PageComponent) {
  comp.init()
  await comp.getData()
  comp.render()
}

buildPage(new NavBar())
buildPage(new ContentList())
```
