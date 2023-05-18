# 继承与原型链

对于使用过基于类的语言 (如 Java 或 C++) 的开发者们来说，JavaScript 实在是有些令人困惑 —— JavaScript 是动态的，本身不提供一个 `class` 的实现。即便是在 ES2015/ES6 中引入了 `class` 关键字，但那也只是语法糖，JavaScript 仍然是基于原型的。

当谈到继承时，JavaScript 只有一种结构：**对象**。每个实例对象（object）都有一个私有属性（称之为 \_\_proto\_\_）指向它的构造函数的原型对象（**prototype**）。该原型对象也有一个自己的原型对象（\_\_proto\_\_），层层向上直到一个对象的原型对象为 null。根据定义，null 没有原型，并作为这个 [原型链](https://mrw1108.github.io/2021/05/27/yuan-xing-yuan-xing-lian/) 中的最后一个环节。

几乎所有 JavaScript 中的对象都是位于原型链顶端的 Object 的实例。

尽管这种原型继承通常被认为是 JavaScript 的弱点之一，但是原型继承模型本身实际上比经典模型更强大。例如，在原型模型的基础上构建经典模型相当简单。

> 思路： 继承是什么 >> JS 继承依赖原型 >> 原型 >> 原型式继承 >> 原型链继承 >> 组合继承 >> 寄生式组合继承

**继承**，就是 A 的属性方法可以继承给 B，B 可以继续往下传。 同时，我们还希望

- **B 能够新增自己的属性和方法。**
- **B 能够修改继承来的属性和方法，但不影响 A 也不干扰同代。**

要实现继承，可以通过“继承”（Inheritance）和“组合”（Composition）来实现

## 原型式继承

利用了原型上的数据能被共享这个特性，**从外部看来，我们只是输入对象，然后就会产生一个新的拥有同样数据的对象**

**引用值问题：** 如果 B 要修改 A 的属性，对于基本类型会在 B 上创建同名属性，而修改引用类型就会修改其原型

```javascript
function create (o) {
  let F = function () { }
  F.prototype = o
  return new F()
)
}A
const A = {
  name: 'li',
  sayName: function () {
    console.log(this.name)
  }
}
let B = create(A)
B.age = '18'
let C = create(A)
C.age = '20'
```

## 原形链继承

原型链继承的优势之一是可以更好地模拟**类**，从而更接近**面向对象编程**。

核心：将父类实例作为子类原型，将需要复用、共享的方法定义在父类原型上

**原型链继承的两个问题：** 引用值问题、调用子类时无法对父类进行传参

```javascript
function Parent() {
  this.name = "父亲"; // 实例基本属性 (该属性，强调私有，不共享)
  this.arr = [1]; // (该属性，强调私有)
}
Parent.prototype.say = function () {
  // -- 将需要复用、共享的方法定义在父类原型上
  console.log("hello");
};
function Child(like) {
  this.like = like;
}
Child.prototype = new Parent(); // 核心

let boy1 = new Child();
let boy2 = new Child();

// 优点：共享了父类构造函数的say方法
console.log(boy1.say(), boy2.say(), boy1.say === boy2.say); // hello , hello , true

// 缺点1：不能传参数
console.log(boy1.name, boy2.name, boy1.name === boy2.name); // 父亲，父亲，true

// 缺点2：共享引用值
boy1.arr.push(2);
console.log(boy2.arr); // [1,2]
```

## 构造函数继承

- 核心：借用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类
- 优点：实例之间独立。
- 缺点：父类的方法不能复用（由于方法定义在构造函数中，每次创建实例都要创建一遍方法，方法应该要复用、共享）

```javascript
function Parent(name) {
  this.name = name; // 实例基本属性 (该属性，强调私有，不共享)
  this.arr = [1]; // (该属性，强调私有)
  this.say = function () {
    // 实例引用属性 (该属性，强调复用，需要共享)
    console.log("hello");
  };
}
function Child(name, like) {
  Parent.call(this, name); // 核心
  this.like = like;
}
let boy1 = new Child("小红", "apple");
let boy2 = new Child("小明", "orange ");

// 优点1：可传参
console.log(boy1.name, boy2.name); // 小红， 小明

// 优点2：不共享父类构造函数的引用属性
boy1.arr.push(2);
console.log(boy1.arr, boy2.arr); // [1,2] [1]

// 缺点1：方法不能复用
console.log(boy1.say === boy2.say); // false (说明，boy1和boy2 的say方法是独立，不是共享的)

// 缺点2：不能继承父类原型上的方法
Parent.prototype.walk = function () {
  // 在父类的原型对象上定义一个walk方法。
  console.log("我会走路");
};
boy1.walk; // undefined (说明实例，不能获得父类原型上的方法)
```

## 组合继承

组合是指**原型链+借用构造函数**。组合继承弥补了很多问题（如引用值问题），**是 JavaScript 中使用最多的继承模式。**

缺点：会调用 2 次父类的构造方法。由于子类调用了父类的构造函数复制了父类的实例属性，又创建父类的实例作为子类的原型，会存在一份多余的父类实例属性。

**注意**：`组合继承` 这种方式，要记得修复 `Child.prototype.constructor` 指向

```javascript
function Parent(name) {
  this.name = name; // 实例基本属性 (该属性，强调私有，不共享)
  this.arr = [1]; // (该属性，强调私有)
}
Parent.prototype.say = function () {
  // --- 将需要复用、共享的方法定义在父类原型上
  console.log("hello");
};
function Child(name, like) {
  Parent.call(this, name, like); // 核心   第二次
  this.like = like;
}
Child.prototype = new Parent(); // 核心   第一次
// 注意：为啥要修复构造函数的指向？
console.log(boy1.constructor); // Parent 你会发现实例的构造函数居然是Parent。
Child.prototype.constructor = Child;

let boy1 = new Child("小红", "apple");
let boy2 = new Child("小明", "orange");

// 优点1：可以传参数
console.log(boy1.name, boy1.like); // 小红，apple

// 优点2：可复用父类原型上的方法
console.log(boy1.say === boy2.say); // true

// 优点3：不共享父类的引用属性，如arr属性
boy1.arr.push(2);
console.log(boy1.arr, boy2.arr); // [1,2] [1] 可以看出没有共享arr属性。
```

优化：使用 `Child.prototype = Parent.prototype` 代替 `Child.prototype = new Parent()` ，只调用 1 次父类构造函数。子类原型和父类原型，实质上是同一个。

**上述优化的缺点：** 修正构造函数的指向之后，父类实例的构造函数指向，同时也发生变化

因为通过原型来实现继承的，`Child.prototype` 的上面是没有 `constructor` 属性的, 所以会顺着原型链向上，修改的是 `Parent.prototype` 上面的 `constructor` 属性

## 寄生组合继承

使用 `Object.create` 优化了组合继承的缺点

```javascript
function Parent(name) {
  this.name = name; // 实例基本属性 (该属性，强调私有，不共享)
  this.arr = [1]; // (该属性，强调私有)
}
Parent.prototype.say = function () {
  // --- 将需要复用、共享的方法定义在父类原型上
  console.log("hello");
};
function Child(name, like) {
  Parent.call(this, name, like); // 核心
  this.like = like;
}
Child.prototype = Object.create(Parent.prototype); // 核心  通过创建中间对象，子类原型和父类原型，就会隔离开。不是同一个啦，有效避免了组合继承的缺点。

// 修复构造函数指向
Child.prototype.constructor = Child;

let boy1 = new Child("小红", "apple");
let boy2 = new Child("小明", "orange");
let p1 = new Parent("小爸爸");
```
