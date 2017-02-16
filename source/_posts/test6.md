---
title: test
date: 2016-06-12 13:25:57
category: 随笔
---

###  1. 块级作用域

* **使用let代替var**

```JavaScript
if (true) {
  let x = 'hello';
}
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```
上面代码如果用var替代let，实际上就声明了一个全局变量，这显然不是本意。

```JavaScript
if(true) {
  console.log(x); 
  let x = 'hello';
}
```
上面代码如果使用var替代let，console.log那一行就不会报错，而是会输出undefined，因为变量声明提升到代码块的头部。这违反了变量先声明后使用的原则。

所以，建议不再使用var命令，而是使用let命令取代。

* **全局常量和线程安全**

在let和const中优先使用const，尤其是全局环境中不应使用变量，应该使用常量。
所有的函数都应该设置为常量。

### 2. 字符串

```JavaScript
const a = 'foobar';
const b = `foo${a}bar`;
const c = 'foobar';
```

静态字符串使用单引号，动态字符串使用反引号。

### 3. 解构赋值
```JavaScript## 

> ***标题***

 ##
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```
使用数组成员对变量进行赋值的时候，优先使用解构赋值。

```JavaScript
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;
}

// good
function getFullName(obj) {
  const { firstName, lastName } = obj;
}

// best
function getFullName({ firstName, lastName }) {
}
```
函数的参数如果是对象的成员，优先使用解构赋值。


``` JavaScript
// bad
function processInput(input) {
  return [left, right, top, bottom];
}

// good
function processInput(input) {
  return { left, right, top, bottom };
}

const { left, right } = processInput(input);
```
如果函数返回多个值，优先使用对象的解构赋值，而不是数组的解构赋值。这样便于以后添加返回值，以及更改返回值的顺序。

### 4. 对象
```JavaScript
// bad
const obj = {};
obj.x = 3;

// if reshape unavoidable
const obj = {};
Object.assign(obj, { x: 3 });

// good
const obj = { x: null };
a.x = 3;
```

对象尽量静态化，一旦定义之后，最好不要随意添加新的属性，如果需要的话，使用Object.assgin方法。

waiting......
属性表达式来动态定义属性。



### 5. 数组

```JavaScript
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);
```
使用Array.from方法，将类似数组的对象转为数组。
粗体文本
> *Tip.*  （...）为扩展运算符

```JavaScript
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i++) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```
使用扩展运算符（...）拷贝数组。