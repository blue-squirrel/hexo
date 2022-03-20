---
title: JS简化技巧
---

一些JS的简写技巧，可以有效提高工作效率和代码可读性。
<!-- more -->

## 简化技巧

#### 同时声明多个变量时，可简写成一行

```
//Longhand
let x;
let y = 20;

//Shorthand
let x, y = 20;
```

#### 过滤空值，不含0

```
//Longhand
if(!!a || a === 0 ){
	b = a;
}

//Shorthand
b = a ?? 666;
```

#### ~ 按位非判断是否存在于数组

```
// JavaScript代码
if ( !~items.indexOf( item ) ) {  
    items.push(item);  
}  

// 如果在集合中查找到了item，则该函数返回对应下标,是一个大于0的整数，该整数按位非的结果一定不为0，取逻辑非后，表达式结果为假。
// 如果在集合中没找到item，则该函数返回-1这个值。而恰好，-1这个值按位非的结果刚好是0,再取逻辑非后，表达式结果为真。
```

#### 使用双波浪线运算符（~~）代替Math.floor()

```
//Longhand
const floor = Math.floor(6.8); // 6

// Shorthand
const floor = ~~6.8; // 6

// ~~两次取反码，将会获取到int值
```

#### 利用解构，可为多个变量同时赋值

```
//Longhand
let a, b, c;

a = 5;
b = 8;
c = 12;

//Shorthand
let [a, b, c] = [5, 8, 12];
```

#### 使用解构交换两个变量的值

```
let x = 'Hello', y = 55;

//Longhand
const temp = x;
x = y;
y = temp;

//Shorthand
[x, y] = [y, x];
```

#### 使用&&运算符简化if语句

```
//Longhand
if (isLoggedin) {
    goToHomepage();
 }

//Shorthand
isLoggedin && goToHomepage();
```

#### 对于多值匹配，可将所有值放在数组中

```
//Longhand
if (value === 1 || value === 'one' || value === 2 || value === 'two') {
  // Execute some code
}

// Shorthand 2
if ([1, 'one', 2, 'two'].includes(value)) { 
    // Execute some code 
}
```

#### 使用repeat()方法简化重复一个字符串

```
//Longhand
let str = '';
for(let i = 0; i < 5; i ++) {
  str += 'Hello ';
}
console.log(str); // Hello Hello Hello Hello Hello

// Shorthand
'Hello '.repeat(5);
```

#### 简化数组合并

```
let arr1 = [20, 30];

//Longhand
let arr2 = arr1.concat([60, 80]); // [20, 30, 60, 80]

//Shorthand
let arr2 = [...arr1, 60, 80]; // [20, 30, 60, 80]
```

#### 单层对象的拷贝

```
let obj = {x: 20, y: {z: 30}};

//Shorthand
const cloneObj = JSON.parse(JSON.stringify(obj));

//Shorthand for single level object
let obj = {x: 20, y: 'hello'};
const cloneObj = {...obj};
```

#### 寻找数组中的最大和最小值

```
// Shorthand
const arr = [2, 8, 15, 4];
Math.max(...arr); // 15
Math.min(...arr); // 2
```

#### 简化获取字符串中的某个字符

```
let str = 'jscurious.com';

//Longhand
str.charAt(2); // c

//Shorthand
str[2]; // c
```

#### 移除对象属性

```
let obj = {x: 45, y: 72, z: 68, p: 98};

// Longhand
delete obj.x;
delete obj.p;
console.log(obj); // {y: 72, z: 68}

// Shorthand
let {x, p, ...newObj} = obj;
console.log(newObj); // {y: 72, z: 68}
```

