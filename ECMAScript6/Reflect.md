# Reflect

Reflect有两个用处，一是规范原来操作对象的一些写法，将操作对象数据和操作对象底层分开。比如Object.defineProperty

另一个作用是辅助Proxy内部操作。

Reflect有13个静态方法，和Proxy的`handler`参数中的方法一一对应，不再细说。
* `Reflect.apply(target, thisArg, args)`
* `Reflect.construct(target, args)`
* `Reflect.get(target, name, receiver)`
* `Reflect.set(target, name, value, receiver)`
* `Reflect.defineProperty(target, name, desc)`
* `Reflect.deleteProperty(target, name)`
* `Reflect.has(target, name)`
* `Reflect.ownKeys(target)`
* `Reflect.isExtensible(target)`
* `Reflect.preventExtensions(target)`
* `Reflect.getOwnPropertyDescriptor(target, name)`
* `Reflect.getPrototypeOf(target)`
* `Reflect.setPrototypeOf(target, prototype)`

主要来说下刚才说到的Reflect的用处，我们可以怎么用。

## 规范语法

## 辅助Proxy
receiver存在的意义就是为了正确的get/set方法中传递上下文

如果没有传receiver，setter/getter中的this是调用对象，传了便是receiver
```javascript
// let obj = {
//   a: 1,
//   set b(value) {
//     this.a = value;
//   }
// }
// let receiver = {};

// // Reflect.set(obj, 'b', 2);
// // console.log(obj, receiver); // {a: 2} {}
// Reflect.set(obj, 'b', 3, receiver);
// console.log(obj, receiver); // {a: 1} {a: 3}

// let obj = {
//   a: 1,
//   get b() {
//     return this;
//   }
// }
// let receiver = {};
// console.log(Reflect.get(obj, 'b') === obj); // true
// console.log(Reflect.get(obj, 'b', receiver) === receiver); // true
```

```javascript
let father = {
  _name: '老明',
  get name() {
    return this._name;
  }
}
let proxy = new Proxy(father, {
  get(target, key) {
    return target[key];
  }
});
console.log(proxy.name); // 老明

let son = {
  _name: '小明'
}
Object.setPrototypeOf(son, proxy);
console.log(son.name); // 老明
```

## 参考
* [Reflect - ECMAScript6入门](https://es6.ruanyifeng.com/#docs/reflect)
* [Proxy 和 Reflect](https://juejin.cn/post/6844904090116292616#heading-13)