# Symbol

## 作用
`Symbol`表示独一无二的值。在ES5中，对象的属性名都是字符串，而在ES6中，对象的属性名除了可以是字符串，还可以是`Symbol`类型。  
`Symbol()`前不能使用`new`，用`Symbol()`生成的变量是一个原始类型的值，不是对象。
```javascript
let a = Symbol();
```
`Symbol()`函数可以接受一个字符串作为参数，主要是为了显示时易于区分，相当于给每个独一无二的值加个描述。如果传入的不是字符串，会转成字符串。
```javascript
let a = Symbol('str')
let b = Symbol({})
let c = Symbol([1,2,3])
console.log(a) // Symbol(str)
console.log(b) // Symbol([object Object])
console.log(c) // Symbol(1,2,3)
```

## Symbol.prototype.description
对于我们传进去的参数，也就是刚才说的描述，我们可以通过description属性来获取。
```javascript
let a = Symbol('hello world')
console.log(a.description) // hello world
```

## 属性名的遍历
现在我们知道`Symbol`类型的值能够作为对象的属性，在开发中，如果我们需要给对象添加一个唯一的属性名，或者我们需要这个属性在一些遍历时不暴露键名，那我们便可以用到`Symbol`。
```javascript
let key = Symbol('Symbol key');
let obj = {
    [key]: 'symbol key value', 
    [Symbol()]: 'symbol key value2',
    a: 'a'
}
console.log(Object.getOwnPropertyNames(obj)); // ['a']
```
像是`for...in`，`Object.getOwnPropertyNames()`，`Object.keys()`都是无法获取到`Symbol`类型的键名的。虽然以上方法无法获取，并不代表`Symbol`类型的属性名是私有属性，我们可以通过`Object.getOwnPropertySymbols()`或者`Reflect.ownKeys()`来获取。
```javascript
let key = Symbol('Symbol key')
let obj = {
    [key]: 'symbol key value', 
    [Symbol()]: 'symbol key value2',
    a: 'a'
}
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(Symbol key), Symbol()]
console.log(Reflect.ownKeys(obj)); // ['a', Symbol(Symbol key), Symbol()]
```

## Symbol.for(), Symbol.keyFor()
使用`Symbol.for()`同样能得到一个`Symbol`类型的值，它接受一个字符串参数，如果这个字符串参数相同，则是同一个值。
```javascript
let a = Symbol.for('hello');
let b = Symbol.for('hello');

a === b // true
```
所以如果我们用`Symbol.for()`的同一个`Symbol`值来做对象的键名，那仍符合后面覆盖前面的原则。
```javascript
let key = Symbol.for('hello');
console.log(typeof key);
let obj = {
    [key]: '1',
    [Symbol.for('hello')]: '2'
}
console.log(obj); // {Symbol(hello): '2'}
```

`Symbol.keyFor()`返回一个`Symbol`类型值的登记记录，即有没有用过`Symbol.for()`
```javascript
let s1 = Symbol.for('hello');
  let s2 = Symbol();
  console.log(Symbol.keyFor(s1)); // hello
  console.log(Symbol.keyFor(s2)); // undefined
```

## 内置的Symbol值
### Symbol.hasInstance
对象的`Symbol.hasInstance`属性，指向一个内部方法。当对象调用`instanceof`运算时，涉及到这个方法。比如我们可以自定义`instanceof`的判断：
```javascript
class Test {
    // 我自定义为如果一个对象有a属性，则是该类对象的实例
    [Symbol.hasInstance](val) {
        return Reflect.has(val, 'a');
    }
}

let obj1 = {a: '123'}
let obj2 = {b: '456'}
console.log(obj1 instanceof new Test()) // true
console.log(obj2 instanceof new Test()) // false
```

### Symbol.iterator
对象的`Symbol.iterator`属性，指向该对象的默认遍历器方法。运用该方法，我们可以实现自己的迭代器对象。在前面章节如变量的解构赋值、数组的扩展等章节中，已经详细实现过，不再赘述。
```javascript
let obj = {
    value: [1,2,3],
    [Symbol.iterator]() {
        let _this = this;
        let index = 0;
        return {
            next() {
                if (index < _this.value.length) {
                    return {
                        value: _this.value[index++],
                        done: false
                    }
                } else {
                    return {
                        value: undefined,
                        done: true
                    }
                }
            }
        }
    }
}

for(let item of obj) {
    console.log(item); // 1 2 3
}
```

### 其他一些属性
* `Symbol.isConcatSpreadable`，对象属性，可以使布尔值或undefined。用来设置数组对象或类数组对象在数组`concat`时的展开行为。
* `Symbol.species`，指向一个构造函数。
* `Symbol.match`，指向一个方法。
* `Symbol.replace`，指向一个方法
* `Symbol.search`，指向一个方法
* `Symbol.split`，指向一个方法
* `Symbol.toPrimitive`，指向一个方法，涉及到类型转换。
* `Symbol.toStringTag`
* `Symbol.unscopables`

## 如何实现一个Symbol
```javascript
(function () {
    var root = this;

    var generateName = (function () {
        // 保证在转成字符串时唯一
        var postfix = 0;
        return function (desc) {
            postfix++;
            return `$$${desc}_${postfix}`;
        };
    })();

    var MySymbol = function (description) {
        // 不能使用new命令
        if (this instanceof MySymbol) {
            throw new TypeError("Symbol is not a constructor");
        }
        // 接收一个参数
        var desc =
            description === undefined
                ? undefined
                : String(description);

        var symbol = Object.create({
            toString: function () {
                return this.__Name__;
            },
        });

        Object.defineProperties(symbol, {
            __Description__: {
                value: desc,
                writeable: false,
                enumerable: false,
                configurable: false,
            },
            __Name__: {
                value: generateName(desc), //接收的参数用来当描述符
                writeable: false,
                enumerable: false,
                configurable: false,
            },
        });

        return symbol;
    };

    var forMap = {};

    Object.defineProperties(MySymbol, {
        for: {
            // Symbol.for()
            value: function (description) {
                var desc =
                    description === undefined
                        ? undefined
                        : String(description);
                if (!Reflect.has(forMap, desc)) {
                    Reflect.set(forMap, desc, MySymbol(desc))
                }
                return Reflect.get(forMap, desc);
            },
            writable: true,
            enumerable: false,
            configurable: true,
        },
        keyFor: {
            // Symbol.keyFor()
            value: function (symbol) {
                for (var key in forMap) {
                    if (Reflect.get(forMap, key) === symbol)
                        return key;
                }
            },
            writable: true,
            enumerable: false,
            configurable: true,
        },
    });
    root.MySymbol = MySymbol;
})();

let symbol1 = MySymbol('1')
let symbol2 = MySymbol('1');
let symbol3 = MySymbol.for('hello');
let symbol4 = MySymbol.for('hello');

console.log(symbol1 === symbol2); // false
console.log(symbol2 === symbol3); // false
console.log(symbol3 === symbol4); // true


console.log(MySymbol.keyFor(symbol3)); // hello

let symbol5 = new MySymbol(); // Uncaught TypeError: Symbol is not a constructor
```

## 参考
* [Symbol - ECMAScript6入门](https://es6.ruanyifeng.com/#docs/symbol)
* [ES6 系列之模拟实现 Symbol 类型](https://github.com/mqyqingfeng/Blog/issues/87)
* [sec-symbol-description](https://262.ecma-international.org/6.0/#sec-symbol-description)