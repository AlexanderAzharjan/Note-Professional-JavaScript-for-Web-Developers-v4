---
title: JavaScript中创建对象的那些事儿
author: 云影sky
issuse: https://github.com/lxfriday/Note-Professional-JavaScript-for-Web-Developers-v4/issues/1
---

[discuss](https://github.com/lxfriday/Note-Professional-JavaScript-for-Web-Developers-v4/issues/1)

![](https://qiniu1.lxfriday.xyz/blog/e1f4e75480c9533dc8c6a3e2074ff2c_20191231200037.jpg)

本文原载自 [http://js-professional.lxfriday.xyz/blog/2019/12/31/%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA](http://js-professional.lxfriday.xyz/blog/2019/12/31/%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA)，作为学习笔记总结呈现。

## 创建对象的几种基本方式

- `{}` 对象字面量
- `Object()` 或者 `new Object()`
- `new Constructor()`
- `Object.create()`
- `Object.assign()`

关于 `new Constructor()` `Object.create()` 和 `Object.assign()` 创建对象的过程和模拟实现可以参考这篇文章 [前端面试必备 | 5000 字长文解释千万不能错过的原型操作方法及其模拟实现（原型篇：下）](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483946&idx=1&sn=acbf08e208b23ddd813c2f3266f375c0&chksm=fd3c3e82ca4bb79458205dde161449b483317ae45eb214b0225af467bc448f398c4f09cd9999&token=479601613&lang=zh_CN#rd)。

## 工厂模式

```js
function createPerson(name, age, job) {
  const o = new Object()
  o.name = name
  o.age = age
  o.job = job
  o.sayName = function() {
    console.log(this.name)
  }
  return o
}
const person1 = createPerson('Nicholas', 29, 'Software Engineer')
const person2 = createPerson('Greg', 27, 'Doctor')
```

每一次调用上面的 `createPerson` 工厂函数都可以创建一个对象，这个对象有 `name` `age` `job` 三个属性和一个 `sayName` 方法，依据传入的参数的不同，返回对象的值也会不同。

缺点：没有解决这个对象是一个什么类型的对象（没有更精确的对象标识，即没有精确的构造函数）。

## 构造函数模式

将工厂改造成构造函数之后，如下

```js
function Person(name, age, job) {
  this.name = name
  this.age = age
  this.job = job
  this.sayName = function() {
    console.log(this.name)
  }
}
const person1 = new Person('Nicholas', 29, 'Software Engineer')
const person2 = new Person('Greg', 27, 'Doctor')
person1.sayName() // Nicholas
person2.sayName() // Greg
```

构造函数和工厂的区别：

1. 没有显式创建对象；
2. 直接把属性和方法赋值给 `this`
3. 没有 `return`

使用构造函数创建对象将会有以下几个步骤：

1. 在内存中创建一个新对象
2. 新对象内部的 `[[Prototype]]` 指针指向构造函数的 `prototype` 属性指向的对象；
3. 将构造函数的上下文 `this` 指向新创建的对象；
4. 执行构造函数内部的代码（给新对象添加属性）；
5. 如果构造函数 `return` 非 `null` 的对象，那返回的就是这个对象，否则返回新创建的这个对象。没有 `return` 时，隐式返回新创建的对象，`return null` 会返回新创建的对象；

缺点：每次实例化一个新对象，都会在内部创建一个 `sayName` 对应的匿名函数，而这个函数对所有实例来讲是没有必要每次都创建的，他们只需要指向同一个函数即可。

所以上面的代码经过改造之后，变成下面这样：

```js
function Person(name, age, job) {
  this.name = name
  this.age = age
  this.job = job
  this.sayName = sayName
}
function sayName() {
  console.log(this.name)
}
const person1 = new Person('Nicholas', 29, 'Software Engineer')
const person2 = new Person('Greg', 27, 'Doctor')
person1.sayName() // Nicholas
person2.sayName() // Greg
```

上述的做法虽然解决了重复创建匿名函数的问题，但是又引入了新的问题。

外面的 `sayName` 函数仅仅在构造函数中用到，如果对象需要很多个这样的函数，那么就需要在外部定义很多个这种函数，这无疑会导致代码很难组织。

## 原型模式

函数创建之后都会有一个 `prototype` 属性，每个使用该构造函数创建的对象都有一个 `[[prototype]]` 内部属性指向它。

使用原型的好处在于它所有的属性和方法会在实例间共享，并且这个共享的属性和方法是直接在原型上设置的。

```js
function Person() {}

Person.prototype.name = 'Nicholas'
Person.prototype.age = 29
Person.prototype.job = 'Software Engineer'
Person.prototype.sayName = function() {
  console.log(this.name)
}
const person1 = new Person()
person1.sayName() // "Nicholas"
const person2 = new Person()
person2.sayName() // "Nicholas"
console.log(person1.sayName == person2.sayName) // true
```

关于原型的工作原理，可以查看下面三篇文章，看完之后相信你对原型的认识比大多数人都要深刻！

1. [前端面试必备 | 5000 字长文解释千万不能错过的原型操作方法及其模拟实现（原型篇：下）](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483946&idx=1&sn=acbf08e208b23ddd813c2f3266f375c0&chksm=fd3c3e82ca4bb79458205dde161449b483317ae45eb214b0225af467bc448f398c4f09cd9999&token=479601613&lang=zh_CN#rd)
1. [前端面试必备 | 古怪的原型（鸡生蛋还是蛋生鸡）（原型篇：中）](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483942&idx=1&sn=20d88f820800bcb6b6374096cea25a93&chksm=fd3c3e8eca4bb7984588fbfe10609e12df6087bfd32939a1a99ce95682c6233acc42de362139&token=92313189&lang=zh_CN#rd)
1. [前端面试必备 | 使用原型和构造函数创建对象（原型篇：上）](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483938&idx=1&sn=60586911f43a69a905801e5407c2b8e7&chksm=fd3c3e8aca4bb79cd5c7bb64e6c6e5df0c7da3129d7481c8d51e1a9a0590fe4359bbaef575d2&token=212038910&lang=zh_CN#rd)

### 理解原型的层级

对象中属性的查找机制：

当从对象中访问一个属性的时候，JS 引擎将会按属性名进行查找。JS 引擎会先查找对象自身。如果找到了这个属性，就会停止查找并返回属性对应的值，如果在对象自身没有找到，则会通过原型链到原型对象中继续查找这个属性，如果找到了这个属性，就会停止查找并返回属性对应的值，否则会继续到上层原型链中查找，直到碰到 `null` 。

当一个属性添加到实例中时，这个属性会覆盖原型上的同名属性，这个覆盖指的是查找的时候不会到原型中查找同名属性。即使属性的值被赋值为 `null` 或 `undefined`，它依然会阻止到原型链上访问。所以如果想要访问，就需要删除这个属性，使用 `delete obj.xx` 。

可以使用 `hasOwnProperty` 判断实例是否拥有某个属性，返回 `true` 则表示实例本身拥有该属性，否则表示它没有这个属性。当一个属性存在于原型链上时，可以访问到这个属性，但是使用 `hasOwnProperty` 将返回 `false`。

### `in` 操作符

`in` 操作符用在两个地方，一个是用在 `for ... in` 循环中，另一个是单独使用。单独使用时，返回 `true` 表示属性可以在对象或者其原型链上找到。

```js
function Person() {}
Person.prototype.name = 'Nicholas'
Person.prototype.age = 29
Person.prototype.job = 'Software Engineer'
Person.prototype.sayName = function() {
  console.log(this.name)
}
const person1 = new Person()
const person2 = new Person()
console.log(person1.hasOwnProperty('name')) // false
console.log('name' in person1) // true
person1.name = 'Greg'
console.log(person1.name) // "Greg" - from instance
console.log(person1.hasOwnProperty('name')) // true
console.log('name' in person1) // true
console.log(person2.name) // "Nicholas" - from prototype
console.log(person2.hasOwnProperty('name')) // false
console.log('name' in person2) // true
delete person1.name
console.log(person1.name) // "Nicholas" - from the prototype
console.log(person1.hasOwnProperty('name')) // false
console.log('name' in person1) // true
```

可以通过组合使用 `hasOwnProperty` 和 `in` 来实现判断一个属性是否存在于原型链上：

```js
function hasPrototypeProperty(object, name) {
  return !object.hasOwnProperty(name) && name in object
}
const obj = Object.create({ name: 'lxfriday' })
console.log(obj)
console.log(hasPrototypeProperty(obj, 'name'))
```

### 关于对象属性的枚举顺序

`for ... in` `Object.keys()` `Object.getOwnPropertyNames/Symbols()` 和 `Object.assign()` 在处理属性枚举顺序的时候会有很大差别。

**`for ... in` `Object.keys()` 没有确定的枚举顺序，它们的顺序取决于浏览器实现。**

而 `Object.getOwnPropertyNames()` `Object.getOwnPropertySymbols()` 和 `Object.assign()` 是有确定的枚举顺序的。

1. 数字键会按照升序先枚举出来；
2. 字符串和 symbol 键按照插入的顺序枚举出来；
3. 对象字面量中定义的键会按照代码中的逗号分割顺序枚举出来；

```js
const k2 = Symbol('k2')
const k1 = Symbol('k1')
const o = { 1: 1, [k2]: 'sym2', second: 'second', 0: 0, first: 'first' }
o[k1] = 'sym1'
o[3] = 3
o.third = 'third'
o[2] = 2
// [ '0', '1', '2', '3', 'second', 'first', 'third' ]
console.log(Object.getOwnPropertyNames(o))
// [ Symbol(k2), Symbol(k1) ]
console.log(Object.getOwnPropertySymbols(o))
```

### 对象的迭代

ES 2017 引入了两个静态方法来将对象的内容转换为可迭代的格式。

`Object.values()` 返回对象值构成的数组； `Object.entries()` 返回一个二维数组，数组中的每个小数组由对象的属性和值构成，类似于 `[[key, value], ...]`。

```js
const o = { foo: 'bar', baz: 1, qux: {} }
console.log(Object.values(o)) // ["bar", 1, {}]
console.log(Object.entries(o)) // [["foo", "bar"], ["baz", 1], ["qux", {}]]
```

在输出的数组中，非字符串的属性会转换成字符串，上述的两个方法对引用类型是采取的浅拷贝。

```js
const o = { qux: {} }
console.log(Object.values(o)[0] === o.qux) // true
console.log(Object.entries(o)[0][1] === o.qux) // true
```

symbol 键名会被忽略掉。

```js
const sym = Symbol()
const o = { [sym]: 'foo' }
console.log(Object.values(o)) // []
console.log(Object.entries(o)) // []
```

## 原型的另一种写法

上面的例子中，给原型赋值都是一个个赋，比较繁琐，看看下面的赋值方式：

```js
function Person() {}
Person.prototype = {
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  sayName() {
    console.log(this.name)
  }
}
```

上面的例子中，`Person` 的原型直接指向一个对象字面量，这种方式最终的结果和前面的单个赋值是一样的，除了原型的 `constructor` 属性，`constructor` 不再指向 `Person` 构造函数。默认情况下，当一个函数创建的时候，会创建一个 `prototype` 对象，并且这个对象上的 `constructor` 属性也会自动指向这个函数。所以这种做法覆盖了默认的 `prototype` 对象，意味着 `constructor` 属性指向新对象的对应属性。虽然 `instanceof` 操作符依然会正常工作，但是已经无法用 `constructor` 来判断实例的类型。看下面的例子：

```js
const friend = new Person()
console.log(friend instanceof Object) // true
console.log(friend instanceof Person) // true
console.log(friend.constructor == Person) // false
console.log(friend.constructor == Object) // true
```

如果 `constructor` 属性很重要，那么你可以手动的给它修复这个问题：

```js
function Person() {}
Person.prototype = {
  constructor: Person,
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  sayName() {
    console.log(this.name)
  }
}
```

不过上面的设置方法有一个问题，`constructor` 的属性描述如下

```js
{
  value: [Function: Person],
  writable: true,
  enumerable: true,
  configurable: true
}
```

我们再看看 `Object.prototype.constructor`：

```js
{
  value: [Function: Object],
  writable: true,
  enumerable: false,
  configurable: true
}
```

我们自己赋值时枚举属性会被默认设置为 `true`，所以需要通过 `Object.defineProperty` 来设置不可枚举：

```js
Object.defineProperty(Person.prototype, 'constructor', {
  value: Person,
  enumerable: false,
  configurable: true,
  writable: true
})
```

### 原型存在的问题

我们知道原型属性对所有实例是共享的，当原型属性是原始值时没有问题，当原型属性是引用类型时将会出现问题。看看下面的例子：

```js
function Person() {}
Person.prototype = {
  constructor: Person,
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  friends: ['Shelby', 'Court'],
  sayName() {
    console.log(this.name)
  }
}
const person1 = new Person()
const person2 = new Person()
person1.friends.push('Van')
console.log(person1.friends) // "Shelby,Court,Van"
console.log(person2.friends) // "Shelby,Court,Van"
console.log(person1.friends === person2.friends) // true
```

上述例子中，原型属性 `friends` 原本是一个包含两个字符串的数组，但是由于 `person1` 修改了它的内容，导致了原型上的这个属性被更改了，所以 `person2` 访问的时候也会打印三个字符串。

由于这个问题，原型模式并不会单独使用，我们经常会结合构造函数和原型来创建对象。

## 总结

我们知道，使用构造函数或者原型创建对象都会存在问题，接下来我们组合使用这两者来解决上面的问题。

1. 构造函数的问题：每个对象都会声明对应的函数，浪费内存；
2. 原型的问题：更改引用类型的原型属性的值会影响到其他实例访问该属性；

为了解决上面的问题，**我们可以把所有对象相关的属性定义在构造函数内，把所有共享属性和方法定义在原型上**。

```js
// 把对象相关的属性定义在构造函数中
function Human(name, age) {
  ;(this.name = name), (this.age = age), (this.friends = ['Jadeja', 'Vijay'])
}
// 把共享属性和方法定义在原型上
Human.prototype.sayName = function() {
  console.log(this.name)
}
// 使用 Human 构造函数创建两个对象
var person1 = new Human('Virat', 31)
var person2 = new Human('Sachin', 40)

// 检查 person1 和 person2 的 sayName 是否指向了相同的函数
console.log(person1.sayName === person2.sayName) // true

// 更改 person1 的 friends 属性
person1.friends.push('Amit')

// 输出: "Jadeja, Vijay, Amit"
console.log(person1.friends)
// 输出: "Jadeja, Vijay"
console.log(person2.friends)
```

我们想要每个实例对象都拥有 `name` `age` 和 `friends` 属性，所以我们使用 `this` 把这些属性定义在构造函数内。另外，由于 `sayName` 是定义在原型对象上的，所以这个函数会在所有实例间共享。

在上面的例子中，`person1` 对象更改 `friends` 属性时， `person2` 对象的 `friends` 属性没有更改。这是因为 `person1` 对象更改的是自己的 `friends` 属性，不会影响到 `person2` 内的。

![](https://qiniu1.lxfriday.xyz/blog/image_20191220134010.png)

## 最后

往期精彩：

- [前端面试必备 | 5000 字长文解释千万不能错过的原型操作方法及其模拟实现（原型篇：下）](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483946&idx=1&sn=acbf08e208b23ddd813c2f3266f375c0&chksm=fd3c3e82ca4bb79458205dde161449b483317ae45eb214b0225af467bc448f398c4f09cd9999&token=479601613&lang=zh_CN#rd)
- [前端面试必备 | 古怪的原型（鸡生蛋还是蛋生鸡）（原型篇：中）](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483942&idx=1&sn=20d88f820800bcb6b6374096cea25a93&chksm=fd3c3e8eca4bb7984588fbfe10609e12df6087bfd32939a1a99ce95682c6233acc42de362139&token=870932706&lang=zh_CN#rd)
- [前端面试必备 | 使用原型和构造函数创建对象（原型篇：上）](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483938&idx=1&sn=60586911f43a69a905801e5407c2b8e7&chksm=fd3c3e8aca4bb79cd5c7bb64e6c6e5df0c7da3129d7481c8d51e1a9a0590fe4359bbaef575d2&token=92313189&lang=zh_CN#rd)
- [前端面试必会 | 一文读懂 JavaScript 中的 this 关键字](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483933&idx=1&sn=8aa346f4ee64f6da46a38413710b8267&chksm=fd3c3eb5ca4bb7a3bdd94b6005720b774523e601b626eb23cb4914169677a181fb9574cfa186&token=212038910&lang=zh_CN#rd)
- [前端面试必会 | 一文读懂现代 JavaScript 中的变量提升 - let、const 和 var](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483928&idx=1&sn=8a9900ad8fbf5a8861166e721840806c&chksm=fd3c3eb0ca4bb7a6b1e92ae4e4e4bd71f8e7c837ece1b29b0e89ed09bdd278f46a25d6aa4341&token=1211815934&lang=zh_CN#rd)
- [前端面试必会 | 一文读懂 JavaScript 中的闭包](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483924&idx=1&sn=fa69401f5b562dd81dcf1c4162852908&chksm=fd3c3ebcca4bb7aafca6548e862af76c49daf5282b7c5b9d2b4c46400782a78fe8bcf49a377a&token=1491323947&lang=zh_CN#rd)
- [前端面试必会 | 一文读懂 JavaScript 中的作用域和作用域链](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483918&idx=1&sn=49cb36e7fcab1b99662c94a4e2bca8ab&chksm=fd3c3ea6ca4bb7b072178251a0984aca870b580a7b48376848a755d4635f4a68b9b7f1ac263e&token=558341442&lang=zh_CN#rd)
- [前端面试必会 | 一文读懂 JavaScript 中的执行上下文](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483911&idx=1&sn=2922e6dc26a8aed4ec733c5ec0a24696&chksm=fd3c3eafca4bb7b9cf15b43f3abedc270f6e18a0942364448a42862766c1bde7c42eb0cc61b9&token=1392282839&lang=zh_CN#rd)
- [IntersectionObserver 和懒加载](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483904&idx=1&sn=856723984d94268ad80acc1b187c2de3&chksm=fd3c3ea8ca4bb7be622635bb9ab75141174a1c9acee2c965a9e9abbed9cbb698a1697aae0bcf&token=1392282839&lang=zh_CN#rd)
- [初探浏览器渲染原理](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483899&idx=1&sn=7c30a7f988b849dcf78c53d31047b53c&chksm=fd3c3d53ca4bb445fb0f2daef575e692ad02de4b15d605577d05261ba626283cb6ca2b6acdcd&token=1392282839&lang=zh_CN#rd)
- [CSS 盒模型、布局和包含块](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483894&idx=1&sn=7464988cafe35295f7b078c257b3156e&chksm=fd3c3d5eca4bb4481fd41a719a8b2f05ac2882a73a8c71fd2cfe294f9b7495d76133de3a1c25&token=1392282839&lang=zh_CN#rd)
- [详细解读 CSS 选择器优先级](https://mp.weixin.qq.com/s?__biz=MzU3MzcxMzg2Mw==&mid=2247483890&idx=1&sn=3542cdb5682055766ca5d13cf87a231f&chksm=fd3c3d5aca4bb44ca661d2fe59734d789b6d1f1750041113edce94dc544df0ce321809b0a70c&token=1392282839&lang=zh_CN#rd)

关注公众号可以看更多哦。

感谢阅读，欢迎关注我的公众号 **云影 sky**，带你解读前端技术，掌握最本质的技能。关注公众号可以拉你进讨论群，有任何问题都会回复。

![公众号](https://qiniu1.lxfriday.xyz/blog/image_20191209232941.png)

![交流群](https://qiniu1.lxfriday.xyz/blog/image_20200101220142.png)

您的在看和分享将是我继续前进的动力~
