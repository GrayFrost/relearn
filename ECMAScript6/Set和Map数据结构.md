# Set和Map数据结构

## Set
`Set`可以用于去重，`Set`是一个构造函数，接收一个参数，这个参数是具有iterable接口的数据，比如数组，字符串，返回一个`Set`类型对象。
```javascript
const set = new Set([1,2,3,4,4]);
const strSet = new Set('abcccd');
console.log([...set]); // [1, 2, 3, 4]
console.log([...strSet]); // ['a', 'b', 'c', 'd']
```

经典的数组去重方式：
```javascript
Array.from(new Set(array))
```

### 属性
`Set.prorotype.size`返回`Set`实例的成员总数。
```javascript
const set = new Set([1,2,3,4,4]);
console.log(set.size); // 4 去重后的数量
```

### 方法
* `add(value)`：添加某个值，返回`Set`结构本身
* `delete(value)`：删除某个值，返回一个布尔值，表示删除是否成功
* `has(value)`：返回一个布尔值，表示是否有某个值
* `clear()`：清除所有成员，没有返回值
* `keys()`：返回键名的迭代器，因为`Set`只有值，所以就是返回键值的迭代器
* `values()`：返回键值的迭代器
* `entries()`：返回键值对的迭代器
* `forEach()`

```javascript
const set = new Set([1,2,3,4,4]);
console.log(set.keys()); // SetIterator {1, 2, 3, 4}
```

## WeakSet
`WeakSet`也是不重复值的集合，但它的成员值只能是对象。同时`WeakSet`不可遍历，因为`WeakSet`中的对象都是弱引用，其内部有多少个成员取决于垃圾回收机制有没有运行，运行前后成员个数不确定。

```javascript
const set = new WeakSet([1,2,3,4,4]); // Uncaught TypeError: Invalid value used in weak set

const set = new WeakSet([[1],[1],[2]]);
console.log(set); // WeakSet {[1],[1],[2]}
```

### 方法
`WeakSet`没有`size`属性，也没有`clear()`方法，也没有用于遍历的方法。
* `add(value)`
* `delete(value)`
* `has(value)`

## Map
`Map`是比对象更完善的键值对结构，它的键接收各种类型的值，而且添加的key有顺序。
```javascript
let num = 1;
let str = 'string';
let bool = true;
let symbol = Symbol('symbol');
let obj = {};
let value = 'hello world';

const map = new Map();
map.set(num, value);
map.set(str, value);
map.set(bool, value);
map.set(symbol, value);
map.set(obj, value);

console.log(map); // Map(5) {1 => 'hello world', 'string' => 'hello world', true => 'hello world', Symbol(symbol) => 'hello world', {…} => 'hello world'}
console.log(map.keys()); // MapIterator {1, 'string', true, Symbol(symbol), {…}}
```

我们知道，`Set`和`WeakSet`接收的是一个数组或者说是一个具有迭代器属性的对象，而`Map`构造函数则接收一个键值对的数组。
```javascript
let map = new Map([
    ['key1', 'value1'],
    ['key2', 'value2']
]);
console.log(map); // Map(2) {'key1' => 'value1', 'key2' => 'value2'}

let set = new Set([1,2]);
let map2 = new Map(set.entries());
console.log(map2); // Map(2) {1 => 1, 2 => 2}
```

对象和`Map`对象如何互转？
```javascript
// 对所有键都是字符串的Map对象比较有用
function map2Obj(map) {
    const entries = map.entries();
    const obj = Object.create(null);
    for(let [key, value] of entries) {
        Reflect.set(obj, key, value);
    }
    return obj;
}

// 或者 return new Map(Object.entries(obj))
function obj2map(obj) {
    const map = new Map();
    Object.keys(obj).forEach((key) => {
        map.set(key, obj[key]);
    });
    return map;
}

let map = new Map([
    ['key1', 'value1'],
    ['key2', 'value2']
]);
let obj = map2Obj(map);
console.log(obj); // {key1: 'value1', key2: 'value2'}
let newMap = obj2map(obj);
console.log(newMap); // Map(2) {'key1' => 'value1', 'key2' => 'value2'}
```

### 属性
`size`属性返回`Map`解构的成员总数。

### 方法
和`Set`对象的方法相似，触类旁通。
* `set(key, value)`
* `get(key)`
* `has(key)`
* `delete(key)`
* `clear()`
* `keys()`
* `values()`
* `entries()`
* `forEach()`

## WeakMap
同`WeakSet`只能接收对象作为成员类似，`WeakMap`只能接收对象作为键名（`null`除外），同样`WeakMap`的键名所引用的对象都是弱引用。即一旦这个对象不在被引用，不再需要了，`WeakMap`里的这个键名对象和所对应键值自动消失，无需手动删除引用。

### 方法
同样`WeakMap`没有`size`属性，没有`clear()`方法和遍历的方法。
* `get(key)`
* `set(key, value)`
* `has(key)`
* `delete(key)`

## 参考
* [Set和Map数据结构 - ECMAScript6入门](https://es6.ruanyifeng.com/#docs/set-map)