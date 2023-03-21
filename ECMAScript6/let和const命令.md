# let和const命令

直接列出let和const公共特性：
* 不存在变量提升
* 会形成暂时性死区
* 不允许重复声明
* 声明的全局变量不是挂在顶层对象中

然后再列出const的单独特性：
* const声明一个只读常量。声明后，该常量值不能改变。

## 不存在变量提升
首先什么是变量提升？  
> **变量提升**通常发生在var声明的变量里，使用var声明一个变量时，该变量会被提升到作用域的顶端。或者使用function定义一个函数时，会发生函数变量提升。

```javascript
var a = '123';

function f(){
  console.log(a); // undefined
  if (false) {
    var a = '456'; // var声明的变量并不识别块级作用域
  }
}

f(); // 此时a变量被提升到f函数作用域顶端，变成var a = undefined
```

在对相同变量分别使用函数声明和变量式声明，函数提升优先级高于变量
```javascript
console.log(func); // ƒ func(){}

function func(){}

var func = '123';
```

```javascript
console.log(func); // ƒ func(){}

var func = '123';

function func(){}
```

```javascript
func(); // 2

var func = function(){
  console.log(1)
};

function func(){
  console.log(2)
}
```

使用let和const声明的变量不存在变量提升。或者我们直接理解为使用let和const声明的变量在声明之前使用都会报Reference错误，从而避免写出不合适的代码，而不必过于纠结是否也发生了变量提升。

```javascript
var a = 123;

if (true) {
  a = 'a'; // ReferenceError
  let a; //用let声明了变量a，在该块级作用域内，a不再受外部的a变量影响
}
```

最常见的变量提升的例子：
```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[5](); // 10
```
使用let之后：
```javascript
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[5](); // 5
```


## 会形成暂时性死区
什么是暂时性死区？
> 在一个代码块中，只要用let或const声明了变量，在这之前，声明的该变量都是不可用的。这部分代码区域被称为”暂时性死区“（Temporal Dead Zone，简称TDZ）

```javascript
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```
从上可以看出暂时性死区的范围。  
在暂时性死区中，类似使用`typeof`，`instanceof`等都会出错。

## 不允许重复声明

不允许重复声明是指在**相同作用域**内，不允许重复声明同一变量。 
```javascript
function func(arg) {
  {
    let arg; // 不报错
  }
}
```

然而for循环变量的部分是一个父作用域，循环体内部是一个单独子作用域，可以重复声明。同理的有if、else、while等的判断里也是另一个作用域。
```javascript
for (let i = 0; i < 10; i++) {
  let i = 'a'; // 不会报错
}
```
```javascript
let status = true;
if (status) {
  let status = 'a';
  console.log(status);
}
while(status) {
  let status = 'b';
  console.log(status);
}
```

## 声明的全局变量不是挂在顶层对象中
``` javascript
var a = 1;
console.log(window.a); // 1

let b = 2;
console.log(window.b); // undefined
```

## const
`const`声明后必须赋值，且之后不能更改。  

其底层的本质是保证变量指向的内存地址所保存的数据不变。所以当我们使用const声明一个引用类型的值时，我们是可以改变变量内部的属性值的。

## 参考
[let和const命令 - ECMAScript6入门](https://es6.ruanyifeng.com/#docs/let)  
[let - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let?retiredLocale=he#specifications)



