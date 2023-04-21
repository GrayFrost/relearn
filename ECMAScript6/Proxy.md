# Proxy

## 用法
```javascript
var proxy = new Proxy(target, handler);
```
`target`和`handler`都是对象，然后我们操作构造出来的`proxy`对象，操作的一些步骤便会被我们在`handler`中定义的方法所拦截。
在`handler`中可以设置的操作有以下13种：
* `get(target, propKey, receiver)`
* `set(target, propKey, value, receiver)`
* `has(target, propKey)`
* `deleteProperty(target, propKey)`
* `ownKeys(target)`
* `getWonPropertyDescriptor(target, propKey)`
* `defineProperty(target, propKey, propDesc)`
* `preventExtensions(target)`
* `getPrototypeOf()`
* `isExtensible(target)`
* `setPrototypeOf()`
* `apply(target, object, args)`
* `construct(target, args)`

着重讲几种平时用到比较多的。

### get()
`get`方法用于拦截某个属性的读取操作，可以接受三个参数，依次为目标对象、属性名和Proxy实例本身，第三个参数可选。关于第三个参数`receiver`的作用，可以看看[这个](https://www.bilibili.com/read/cv17138537/)，我目前理解是没事别乱用这个参数。

```javascript
let person = {
    name: '小明'
}
let proxy = new Proxy(person, {
    get(target, key) {
        if (key === 'name') {
            console.log('劫持get');
            return '小红';
        }
    }
});
console.log(proxy.name);
// 控制台逐行输出'劫持get' '小红'
```

### set()
`set`方法用来拦截某个属性的赋值操作，可以接受四个参数，依次为目标对象、属性名、属性值和Proxy实例本身，其中最后一个参数可选。
```javascript
let person = {
    name: '小明'
}
let proxy = new Proxy(person, {
    set(target, key, value, receiver) {
        if (key === 'name') {
            target[key] = '小红';
        } else {
            target[key] = value;
        }
    }
});
proxy.name = '小刚';
proxy.hello = 'world';
console.log(proxy); // Proxy(Object) {name: '小红', hello: 'world'}
```

### has()
判断对象是否具有某个属性时，这个方法会生效。返回布尔值来判断一个对象是否有某个属性。
```javascript
const person = {
    name: '小明',
    age: 18,
    '@@job': '保密工作',
    [Symbol('hello')]: 'world',
}
let proxy = new Proxy(person, {
    has(targe, key) {
        if (key.startsWith('@@')) {
            return false;
        }
        return key in target;
    }
});

console.log('@@job' in proxy); // false
console.log(Reflect.has(proxy, '@@job')); // false
console.log(Object.keys(proxy)); // ['name', 'age', '@@job']
console.log(Object.getOwnPropertyNames(proxy)); // ['name', 'age', '@@job']
console.log(Reflect.ownKeys(proxy)); // ['name', 'age', '@@job', Symbol(hello)]
```
在前面学习`Symbol`的章节中，我们知道使用`Symbol`值作为键名时，一些遍历操作比如`Object.getOwnPropertyNames`是无法读取到键名的，可以起到一定的私有作用，在这里，我们使用`has`拦截，也可以做一些类似实现私有变量的操作。

### apply()
`apply`方法拦截函数的调用、`call`和`apply`操作。`apply`方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组。  
比如我们可以劫持对应的函数，在函数调用之前或之后，我们做一些其他操作，比如说统计调用次数，上报一些数据等。
```javascript
function fn() {
    console.log('hello');
}
let callCount = 0;
let proxy = new Proxy(fn, {
    apply(target, ctx, args) {
        callCount++;
        return target(...args);
    }
});
proxy();
proxy();
console.log('被调用：' + callCount + '次'); // 被调用：2次
```

### construct()
`construct()`方法用于拦截new命令，接收三个参数：目标对象（原来的构造函数）、构造函数的参数数组、创造实例对象时，new命令作用的构造函数（代理后的构造函数）。
```javascript
function deer(){
    return {
        name: '鹿'
    }
}
let horse = new Proxy(deer, {
    construct(target, args, newTarget) {
        console.log(target, args, newTarget);
        console.log('其实我是:' + target().name);
        return {
            name: '马'
        }
    }
});

console.log(new horse().name);
// 分别输出以下内容：
/**
 ƒ deer(){
      return {
          name: '鹿'
      }
  } [] Proxy(Function) {length: 0, name: 'deer', arguments: null, caller: null, prototype: {…}}
 **/

// 其实我是:鹿
// 马
```

### deleteProperty()
用来拦截删除属性的操作，接收两个参数，目标对象和准备删除的属性。
```javascript
let person = {
    name: '小明',
    age: 18
}
let proxy = new Proxy(person, {
    deleteProperty(target, propKey) {
        if (propKey === 'name') {
            throw new Error('你不能删除name');
        } else {
            delete target[propKey];
        }
    }
});
delete proxy.age;
console.log(proxy); // Proxy(Object) {name: '小明'}
delete proxy.name; // caught Error: 你不能删除name
```

## 用处
### 沙箱隔离
比如在qiankun微前端中，可以进行全局环境的隔离，当我们在子应用中操作window全局对象时，实际操作的是经过代理的一个假的全局对象。
```javascript
class ProxySandBox {
    proxyWindow;
    isRunning = false;
    active() {
        this.isRunning = true;
    }
    inactive() {
        this.isRunning = false;
    }
    constructor() {
        const fakeWindow = Object.create(null);
        this.proxyWindow = new Proxy(fakeWindow, {
            set: (target, prop, value, receiver) => {
                if (this.isRunning) {
                    target[prop] = value;
                }
            },
            get: (target, prop, receiver) => {
                return prop in target ? target[prop] : window[prop];
            },
        });
    }
}

const sandbox = new ProxySandBox();
sandbox.active();

sandbox.proxyWindow.hello = "world";

console.log("active:sandbox:window.hello: ", sandbox.proxyWindow.hello); // 'world'

console.log("window:window.hello: ", window.hello); // undefined
sandbox.inactive();

console.log("inactive:sandbox:window.hello: ", sandbox.proxyWindow.hello); // 'world'

console.log("inactive:browser:window.hello: ", window.hello); // undefined
```

### 双向绑定
Vue3的数据绑定相信大家再熟悉不过。
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="app">
      <p id="paragraph"></p>
      <input type="text" id="input" />
    </div>
    <script>
      const paragraph = document.getElementById("paragraph");
      const input = document.getElementById("input");

      const data = {
        value: "hello world",
      };

      const handler = {
        set: function (target, prop, value) {
          if (prop === "value") {
            target[prop] = value;
            paragraph.innerHTML = value;
            input.value = value;
            return true;
          } else {
            return false;
          }
        },
      };
      const proxy = new Proxy(data, handler);
      proxy.value = data.value;

      input.addEventListener(
        "input",
        function (e) {
          proxy.value = e.target.value;
        },
        false
      );
      
    </script>
  </body>
</html>

```

### 不可变结构
使用`Immutable.js`或`Immer.js`可以帮我们创建出不可变结构的对象。在`Immer.js`中，便使用到了Proxy来实现。
```javascript
class Store {
    constructor(state) {
        this.modified = false;
        this.source = state;
        this.copy = null;
    }
    get(key) {
        if (!this.modified) return this.source[key];
        return this.copy[key];
    }
    set(key, value) {
        if (!this.modified) this.modifing();
        return (this.copy[key] = value);
    }
    modifing() {
        if (this.modified) return;
        this.modified = true;
        this.copy = Array.isArray(this.source)
        ? this.source.slice()
        : { ...this.source };
    }
}

const PROXY_FLAG = Symbol();
const handler = {
    get(target, key) {
        if (key === PROXY_FLAG) return target;
        return target.get(key);
    },
    set(target, key, value) {
        return target.set(key, value);
    },
};

function produce(state, producer) {
  const store = new Store(state);
  const proxy = new Proxy(store, handler);

  producer(proxy);

  const newState = proxy[PROXY_FLAG];
  if (newState.modified) {
      return newState.copy;
  }
  return newState.source;
}

const state = { done: false };
const newState = produce(state, (draft) => { draft.done = true });
console.log(state.done); // false
console.log(newState.done); // true
```

## 参考
* [Proxy - ECMAScript6入门](https://es6.ruanyifeng.com/#docs/proxy)
* [ES6 Features - 10 Use Cases for Proxy](https://dealwithjs.io/es6-features-10-use-cases-for-proxy/)
* [微前端01 : 乾坤的Js隔离机制原理剖析（快照沙箱、两种代理沙箱）](https://juejin.cn/post/7070032850237521956)
* [immer.js:也许更适合你的immutable js库](https://juejin.cn/post/6844904111402385422)
* [ES6 Proxy中get和set的receiver参数详解](https://www.bilibili.com/read/cv17138537/)