# Proxy

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
* [// TODO - ECMAScript6入门](// TODO)
* [ES6 Features - 10 Use Cases for Proxy](https://dealwithjs.io/es6-features-10-use-cases-for-proxy/)
* [微前端01 : 乾坤的Js隔离机制原理剖析（快照沙箱、两种代理沙箱）](https://juejin.cn/post/7070032850237521956)
* [immer.js:也许更适合你的immutable js库](https://juejin.cn/post/6844904111402385422)