# ES5/6开发规范
ES6语法的支持一般需要ponyfill或[polifill](https://en.wikipedia.org/wiki/Polyfill)。Electron/RN/小程序/高版本的node可直接使用ES6规范编程
其他的环境使用Babel做ponyfill，一些特性如Reflect，Generator, Proxy等并不完全可以ponyfill
(polifill做patch, ponyfill导出正常的模块，不会影响工作环境)

(1) var/let/const关键字
```shell
var: 
    函数作用域(function scoped)
    声明变量前访问该变量返回undefined
let: 
    块级作用域(block scoped)
    声明变量前访问该变量直接报错: ReferenceError
const:
    块级作用域(block scoped)
    声明变量前访问该变量直接报错: ReferenceError
    变量申明初始化后，不能重新赋值
```

No: for语句中使用var
```js
let a=[]; 
for (var i = 0; i<10; i++) { 
	var q = i;
	a[i]=function(){console.log(q)}
}
a[0](); // 9
```

Yes: 使用let关键字来限定作用域在{}
```js
let a=[];
for (let i = 0; i<10; i++) {
    let q = i;
    a[i]=function(){console.log(q)}
} 
a[6](); // 6
```

(2) 函数声明使用箭头表达式

No: 采用传统的方式
```js
function foo() {
    // code
}
```

Yes: 使用箭头函数
```js
let foo = () => {
    // code
}
```

(3)使用ES6模块(modules)

Yes: 命名导出，default导出，Re-exporting
```js
// lib.js
export const sqrt = Math.sqrt;
export function square(x) {
    return x * x;
}
export default function () {}

// main.js
import myfunc, { square, diag } from 'lib';
console.log(square(11)); // 121
// re-exporting
export { default } from 'lib';
export { square } from 'lib';
```

No: 使用条件语句导入导出模块
```js
if (Math.random()) {
    import 'foo'; // SyntaxError
}
```

(4) 字符串连接

No: 采用传统的`+`来连接
```js
let message = 'Hello ' + name + ", it's " + time + '  now'
```
Yes: 采用模板字符串(或tagged模板)
```js
let message = `Hello ${name}, it's ${time} now`
```

(5)解构赋值

No: 直接赋值
```js
let data = { name: 'dys', age: 1 };
let name = data.name;
let age = data.age;
```

Yes: 采用解构语法
```js
// 对象
const data = {name: 'dys', age: 1};
const { name, age } = data;

// 数组
const fullName = ['Erik', 'Peng'];
const [firstName, lastName] = fullName;
const [first, last, ...others] = [1, 2, 3, 6, 8];
```

(6)使用类class

No: 采用函数原型链实现继承

Yes: 采用ES6中的类(代码可读性高)
```js
class Animal {
    constructor(age) {
        this.age = age;
    }

    move() {
        /* ... */
    }
}

class Mammal extends Animal {
    constructor(age, furColor) {
        super(age);
        this.furColor = furColor;
    }

    liveBirth () {
        /* ... */
    }
}
```

Yes: 属性的键值是Symbol可以实现方法的真正私有(private)
```js
const _counter = Symbol('counter');
const _action = Symbol('action');

class Countdown {
    constructor(counter, action) {
        this[_counter] = counter;
        this[_action] = action;
    }
    dec() {
        // ...
    }
}
// const c = new Countdown(2, _ => _);
// hidden from outside world and can not clash
console.log(Object.keys(c)); // []
```

(7) 优先使用函数式编程

No: 使用for循环编程
```js
for(i = 1; i <= 10; i++) {
    a[i] = a[i] + 1;
}
```

Yes: 使用函数式编程
```js
let b = a.map(item => ++item)
```

(8) 避免过多的if else

No: if else过多
```js
if (a === 1) {
    ...
}
else if (a === 2) {
    ...
}
else if (a === 3) {
    ...
}
```

Yes: 使用switch
```js
switch(a) {
    case 1:
        ...
    case 2:
        ...
    case 3:
        ...
    default:
        ...
}
Or
let handler = {
    1: () => { ... },
    2: () => { ... },
    3: () => { ... }
    default: () => {...}
}
```

(9) 优先使用Map与Set(WeakMap, WeakSet)

Yes: Dom中使用WeakMap存储数据
```js
// Map与Set集成了丰富的操作集合的方法，has, get, remove, entries..
// WeakMap可以自动处理垃圾回收
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```

(10)对象赋值与扩展

Yes: 使用spread语法与Object.assign来实现赋值
```js
let obj3 = Object.assign(obj1, obj2)
let obj4 = {...ob1, ...obj2 };

const set = new Set(['a', 'b']);
const array = [...set]; // 或者Array.from(set)
```

(11) 异步编程，尽量使用Promise

No: 使用回调的方式
```js
function isGreater (a, b, cb) {
  
  var greater = false
  if(a > b) {
    greater = true
  }
  cb(greater)
}
isGreater(1, 2, function (result) {
  if(result) {
    console.log('greater');
  } else {
    console.log('smaller')
  }
})
```

Yes: 使用Promise
```js
// 可以防止回调地狱与处理success/error逻辑
const isGreater = (a, b) => {
    return new Promise ((resolve, reject) => {
        if(a > b) {
            resolve(true)
        } else {
            reject(false)
        }
    })
}

isGreater(1, 2)
 .then(result => {
    console.log('greater')
 })
 .catch(result => {
    console.log('smaller')
 })
```

(12)避免for..in中的继承属性

No: 使用for..in不处理继承属性
```js
for(let props in obj) {
    // do something
}
```

Yes: 使用hasOwnproperty或者Object.keys+forEach来做遍历
```js
1.
for (let props in obj) {
    // if (!Reflect.has(props))
    if (!obj.hasOwnProperty(props)) {
       // do something
    }
}

2.
// forEach如果要break，需要在外层使用try catch
Object.keys(obj).forEach(props =>
  // do something
});
```

(13)适当使用bind/apply/call指定函数调用过程中的this指向

Yes: 使用bind来动态改变this指向
```js
function f(y, z) {
    return this.x + y + z;
}
let m = f.bind({x : 1}, 2);
console.log(m(3)); //6
```

Babel、Eslint、Prettier这三个可以保证代码的格式一致，
如果想让JS支持静态类型，可使用flow或者Typescript

(14)What's more?
- 迭代器
- 生成器
- Reflect/Proxy
- Object中方法, keys, is, entries

#### 相关文献
- [var vs let vs const](https://medium.freecodecamp.org/var-vs-let-vs-const-in-javascript-2954ae48c037)
- [JS代码风格](https://mp.weixin.qq.com/s/4wjt0iEqRv-blnPeSesY9A)
- [Private data via symbols](http://exploringjs.com/es6/ch_classes.html#sec_private-data-for-classes)


(1) var/let/const关键字
```shell
var: 
    函数作用域(function scoped)
    声明变量前访问该变量返回undefined
let: 
    块级作用域(block scoped)
    声明变量前访问该变量直接报错: ReferenceError
const:
    块级作用域(block scoped)
    声明变量前访问该变量直接报错: ReferenceError
    变量申明初始化后，不能重新赋值
```

No: for语句中使用var
```js
let a=[]; 
for (var i = 0; i<10; i++) { 
	var q = i;
	a[i]=function(){console.log(q)}
}
a[0](); // 9
```

Yes: 使用let关键字来限定作用域在{}
```js
let a=[];
for (let i = 0; i<10; i++) {
    let q = i;
    a[i]=function(){console.log(q)}
} 
a[6](); // 6
```

(2) 函数声明使用箭头表达式

No: 采用传统的方式
```js
function foo() {
    // code
}
```

Yes: 使用箭头函数
```js
let foo = () => {
    // code
}
```

(3)使用ES6模块(modules)

Yes: 命名导出，default导出，Re-exporting
```js
// lib.js
export const sqrt = Math.sqrt;
export function square(x) {
    return x * x;
}
export default function () {}

// main.js
import myfunc, { square, diag } from 'lib';
console.log(square(11)); // 121
// re-exporting
export { default } from 'lib';
export { square } from 'lib';
```

No: 使用条件语句导入导出模块
```js
if (Math.random()) {
    import 'foo'; // SyntaxError
}
```

(4) 字符串连接

No: 采用传统的`+`来连接
```js
let message = 'Hello ' + name + ", it's " + time + '  now'
```
Yes: 采用模板字符串(或tagged模板)
```js
let message = `Hello ${name}, it's ${time} now`
```

(5)解构赋值

No: 直接赋值
```js
let data = { name: 'dys', age: 1 };
let name = data.name;
let age = data.age;
```

Yes: 采用解构语法
```js
// 对象
const data = {name: 'dys', age: 1};
const { name, age } = data;

// 数组
const fullName = ['Erik', 'Peng'];
const [firstName, lastName] = fullName;
const [first, last, ...others] = [1, 2, 3, 6, 8];
```

(6)使用类class

No: 采用函数原型链实现继承

Yes: 采用ES6中的类(代码可读性高)
```js
class Animal {
    constructor(age) {
        this.age = age;
    }

    move() {
        /* ... */
    }
}

class Mammal extends Animal {
    constructor(age, furColor) {
        super(age);
        this.furColor = furColor;
    }

    liveBirth () {
        /* ... */
    }
}
```

Yes: 属性的键值是Symbol可以实现方法的真正私有(private)
```js
const _counter = Symbol('counter');
const _action = Symbol('action');

class Countdown {
    constructor(counter, action) {
        this[_counter] = counter;
        this[_action] = action;
    }
    dec() {
        // ...
    }
}
// const c = new Countdown(2, _ => _);
// hidden from outside world and can not clash
console.log(Object.keys(c)); // []
```

(7) 优先使用函数式编程

No: 使用for循环编程
```js
for(i = 1; i <= 10; i++) {
    a[i] = a[i] + 1;
}
```

Yes: 使用函数式编程
```js
let b = a.map(item => ++item)
```

(8) 避免过多的if else

No: if else过多
```js
if (a === 1) {
    ...
}
else if (a === 2) {
    ...
}
else if (a === 3) {
    ...
}
```

Yes: 使用switch
```js
switch(a) {
    case 1:
        ...
    case 2:
        ...
    case 3:
        ...
    default:
        ...
}
Or
let handler = {
    1: () => { ... },
    2: () => { ... },
    3: () => { ... }
    default: () => {...}
}
```

(9) 优先使用Map与Set(WeakMap, WeakSet)

Yes: Dom中使用WeakMap存储数据
```js
// Map与Set集成了丰富的操作集合的方法，has, get, remove, entries..
// WeakMap可以自动处理垃圾回收
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```

(10)对象赋值与扩展

Yes: 使用spread语法与Object.assign来实现赋值
```js
let obj3 = Object.assign(obj1, obj2)
let obj4 = {...ob1, ...obj2 };

const set = new Set(['a', 'b']);
const array = [...set]; // 或者Array.from(set)
```

(11) 异步编程，尽量使用Promise

No: 使用回调的方式
```js
function isGreater (a, b, cb) {
  
  var greater = false
  if(a > b) {
    greater = true
  }
  cb(greater)
}
isGreater(1, 2, function (result) {
  if(result) {
    console.log('greater');
  } else {
    console.log('smaller')
  }
})
```

Yes: 使用Promise
```js
// 可以防止回调地狱与处理success/error逻辑
const isGreater = (a, b) => {
    return new Promise ((resolve, reject) => {
        if(a > b) {
            resolve(true)
        } else {
            reject(false)
        }
    })
}

isGreater(1, 2)
 .then(result => {
    console.log('greater')
 })
 .catch(result => {
    console.log('smaller')
 })
```

(12)避免for..in中的继承属性

No: 使用for..in不处理继承属性
```js
for(let props in obj) {
    // do something
}
```

Yes: 使用hasOwnproperty或者Object.keys+forEach来做遍历
```js
1.
for (let props in obj) {
    // if (!Reflect.has(props))
    if (!obj.hasOwnProperty(props)) {
       // do something
    }
}

2.
// forEach如果要break，需要在外层使用try catch
Object.keys(obj).forEach(props =>
  // do something
});
```

(13)适当使用bind/apply/call指定函数调用过程中的this指向

Yes: 使用bind来动态改变this指向
```js
function f(y, z) {
    return this.x + y + z;
}
let m = f.bind({x : 1}, 2);
console.log(m(3)); //6
```

Babel、Eslint、Prettier这三个可以保证代码的格式一致，
如果想让JS支持静态类型，可使用flow或者Typescript

(14)What's more?
- 迭代器
- 生成器
- Reflect/Proxy
- Object中方法, keys, is, entries

#### 相关文献
- [var vs let vs const](https://medium.freecodecamp.org/var-vs-let-vs-const-in-javascript-2954ae48c037)
- [JS代码风格](https://mp.weixin.qq.com/s/4wjt0iEqRv-blnPeSesY9A)
- [Private data via symbols](http://exploringjs.com/es6/ch_classes.html#sec_private-data-for-classes)

