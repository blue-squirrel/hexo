---
title: Commonjs和Es Module
date: 2022-04-10
---

总是对exports和module.exports的用法分不清？export和export default混用？快来看看他们的用法吧！
<!-- more -->

# Commonjs和Es Module

## 模块化

早期 JavaScript 开发很容易存在**全局污染**和**依赖管理**混乱问题。这些问题在多人开发前端应用的情况下变得更加棘手。

```
<body>

  <script src="./index.js"></script>
  <script src="./home.js"></script>
  <script src="./list.js"></script>

</body>
```

如上在没有模块化的前提下,不同的js文件中的变量可能产生全局污染。

所以就需要模块化来解决上述的问题，今天我们就重点讲解一下前端模块化的两个重要方案：**Commonjs** 和 **Es Module**

## Commonjs

目前 `commonjs` 广泛应用于以下几个场景：

- `Node` 是 CommonJS 在服务器端一个具有代表性的实现；
- `Browserify` 是 CommonJS 在浏览器中的一种实现；
- `webpack` 打包工具对 CommonJS 的支持和转换；也就是前端应用也可以在编译之前，尽情使用 CommonJS 进行开发。

#### commonjs 实现原理

每个模块文件上存在 `module`，`exports`，`require`三个变量，然而这三个变量是没有被定义的，但是我们可以在 Commonjs 规范下每一个 js 模块上直接使用它们。在 nodejs 中还存在 `__filename` 和 `__dirname` 变量。

- `module` 记录当前模块信息。
- require 引入模块的方法。
- `exports` 当前模块导出的属性
- __filename 当前文件的绝对路径
- __dirname 当前文件夹的绝对路径

在 Commonjs 规范下模块中，会形成一个**包装函数**，我们写的代码将作为包装函数的执行上下文，使用的 `require` ，`exports` ，`module` 本质上是通过形参的方式传递到包装函数中的。

这就是为什么可以直接使用以上变量。

```javascript
function wrapper (script) {
    return '(function (exports, require, module, __filename, __dirname) {' + 
        script +
     '\n})'
}
```

#### require 文件加载流程

```
const fs =      require('fs')      // ①核心模块
const sayName = require('./hello.js')  //② 文件模块
const crypto =  require('crypto-js')   // ③第三方自定义模块
```

首先我们看一下 ` nodejs` 中对标识符的处理原则。

- 首先像 fs ，http ，path 等标识符，会被作为 nodejs 的**核心模块**。
- ` ./` 和 `../` 作为相对路径的**文件模块**， `/` 作为绝对路径的**文件模块**。
- 非路径形式也非核心模块的模块，将作为**自定义模块**。

**核心模块的处理：**

核心模块的优先级仅次于缓存加载，在 `Node` 源码编译中，已被编译成二进制代码，所以加载核心模块，加载过程中速度最快。

**路径形式的文件模块处理：**

以 `./` ，`../` 和 `/` 开始的标识符，会被当作文件模块处理。`require()` 方法会将路径转换成真实路径，并以真实路径作为索引，将编译后的结果缓存起来，第二次加载的时候会更快。

**自定义模块处理：** 自定义模块，一般指的是非核心的模块，它可能是一个文件或者一个包，它的查找会遵循以下原则：

- 在当前目录下的 `node_modules` 目录查找。
- 如果没有，在父级目录的 `node_modules` 查找，如果没有在父级目录的父级目录的 `node_modules` 中查找。
- 沿着路径向上递归，直到根目录下的 `node_modules` 目录。
- 在查找过程中，会找 `package.json` 下 main 属性指向的文件，如果没有  `package.json` ，在 node 环境下会以此查找 `index.js` ，`index.json` ，`index.node`。

#### require 模块引入与处理

a.js

```
const getMes = require('./b')
console.log('我是 a 文件')
exports.say = function(){
    const message = getMes()
    console.log(message)
}
```

b.js

```
const say = require('./a')
const  object = {
   name:'《React进阶实践指南》',
   author:'我不是外星人'
}
console.log('我是 b 文件')
module.exports = function(){
    return object
}
```

main.js

```
const a = require('./a')
const b = require('./b')

console.log('node 入口文件')
```

运行结果如下

![5.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b728ce249df740ce8b0232a889283f22~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp)

从上面的运行结果可以得出以下结论：

- `main.js` 和 `a.js` 模块都引用了 `b.js` 模块，但是 `b.js` 模块只执行了一次。
- `a.js` 模块 和 `b.js` 模块互相引用，但是没有造成循环引用的情况。
- 执行顺序是父 -> 子 -> 父；

##### require 加载原理

require 的源码大致长如下的样子：

```
 // id 为路径标识符
function require(id) {
   /* 查找  Module 上有没有已经加载的 js  对象*/
   const  cachedModule = Module._cache[id]
   
   /* 如果已经加载了那么直接取走缓存的 exports 对象  */
  if(cachedModule){
    return cachedModule.exports
  }
 
  /* 创建当前模块的 module  */
  const module = { exports: {} ,loaded: false , ...}

  /* 将 module 缓存到  Module 的缓存属性中，路径标识符作为 id */  
  Module._cache[id] = module
  /* 加载文件 */
  runInThisContext(wrapper('module.exports = "123"'))(module.exports, require, module, __filename, __dirname)
  /* 加载完成 *//
  module.loaded = true 
  /* 返回值 */
  return module.exports
}
```

从上面我们总结出一次 `require` 大致流程是这样的；

- require 会接收一个参数——文件标识符，然后分析定位文件，分析过程我们上述已经讲到了，加下来会从 Module 上查找有没有缓存，**如果有缓存，那么直接返回缓存的内容。**
- 如果没有缓存，会创建一个 module 对象，**缓存到** Module 上，**然后执行文件**，加载完文件，将 loaded 属性设置为 true ，然后返回 module.exports 对象。借此完成模块加载流程。
- **模块导出**就是 return 这个变量的其实跟 a = b 赋值一样， 基本类型导出的是值， 引用类型导出的是引用地址。
- exports 和 module.exports 持有相同引用，因为最后导出的是 module.exports， 所以对 exports 进行赋值会导致 exports 操作的不再是 module.exports 的引用。

##### require 避免重复加载

从上面我们可以直接得出，require 如何避免重复加载的，首先加载之后的文件的 `module` 会被缓存到 `Module` 上，比如一个模块已经 require 引入了 a 模块，如果另外一个模块再次引用 a ，那么会直接读取缓存值 module ，所以无需再次执行模块。

##### require 避免循环引用

那么接下来这个循环引用问题，也就很容易解决了。为了让大家更清晰明白，那么我们接下来一起分析整个流程。

- ① 首先执行 `node main.js` ，那么开始执行第一行 `require(a.js)`；
- ② 那么首先判断 `a.js` 有没有缓存，因为没有缓存，先加入缓存，然后执行文件 a.js （**需要注意 是先加入缓存， 后执行模块内容**）;
- ③ a.js 中执行第一行，引用 b.js。
- ④ 那么判断 `b.js` 有没有缓存，因为没有缓存，所以加入缓存，然后执行 b.js 文件。
- ⑤ b.js 执行第一行，再一次循环引用 `require(a.js)` 此时的 a.js 已经加入缓存，直接读取值。接下来打印 `console.log('我是 b 文件')`，导出方法。
- ⑥ b.js 执行完毕，回到 a.js 文件，打印 `console.log('我是 a 文件')`，导出方法。
- ⑦ 最后回到 `main.js`，打印 `console.log('node 入口文件')` 完成这个流程。

不过这里我们要注意问题：

- 如上第 ⑤ 的时候，当执行 b.js 模块的时候，因为 **a.js 还没有导出 `say` 方法**，**所以 b.js 同步上下文中，获取不到 say。**

那么如何获取到 say 呢，有两种办法：

- 一是用动态加载 a.js 的方法，等使用的时候再使用a.js。
- 二个就是如上放在异步中加载。

#### exports 和 module.exports

**exports的使用**

```
exports.name = `《React进阶实践指南》`
exports.author = `我不是外星人`
exports.say = function (){
    console.log(666)
}
```

**exports = {} 直接赋值一个对象是不可以的**， 等于重新赋值了形参，那么会重新赋值一份，但是不会在引用原来的形参。

**module.exports 使用**

module.exports 本质上就是 exports

```
module.exports ={
    name:'《React进阶实践指南》',
    author:'我不是外星人',
    say(){
        console.log(666)
    }
}
```

**exports 和 module.exports的区别**

**exports会被初始化成一个对象**，所以不能用 =赋值 去修改内容，但是module.exports可以。

如果我们不想在 commonjs 中导出对象，而是只导出一个**类或者一个函数再或者其他属性**的情况，那么 `module.exports` 就更方便了，如上我们知道 **`exports` 会被初始化成一个对象**，也就是我们只能在对象上绑定属性，但是我们可以通过 `module.exports` 自定义导出出对象外的其他类型元素。

## Es Module

**export 正常导出，import 导入**

导出模块 a.js

```
const name = '《React进阶实践指南》' 
const author = '我不是外星人'
export { name, author }
export const say = function (){
    console.log('hello , world')
}
```

导入模块 main.js

```
// name , author , say 对应 a.js 中的  name , author , say
import { name , author , say } from './a.js'
```

##### **混合导入｜导出**

导出模块：`a.js`

```
export const name = '《React进阶实践指南》'
export const author = '我不是外星人'
// 默认导出
export default  function say (){
    console.log('hello , world')
}
```

导入模块：main.js 中有几种导入方式：

第一种：

```
import theSay , { name, author as  bookAuthor } from './a.js'
console.log(
    theSay,     // ƒ say() {console.log('hello , world') }
    name,       // "《React进阶实践指南》"
    bookAuthor  // "我不是外星人"
)
```

第二种：

```
import theSay, * as mes from './a'
console.log(
    theSay, // ƒ say() { console.log('hello , world') }
    mes // { name:'《React进阶实践指南》' , author: "我不是外星人" ，default:  ƒ say() { console.log('hello , world') } }
)
```

导出的属性被合并到 `mes` 属性上， `export` 被导入到对应的属性上，`export default` 导出内容被绑定到 `default` 属性上。

##### **模块导出方式**

```
export * from 'module' // 第一种方式
export { name, author, ..., say } from 'module' // 第二种方式
export {   name as bookName ,  author as bookAuthor , ..., say } from 'module' //第三种方式
```

##### **无需导入模块，只运行模块**

```
import 'module' 
```

##### **import() 动态引入**

`import()` 返回一个 `Promise` 对象， 返回的 `Promise` 的 **then 成功回调**中，可以获取模块的加载成功信息。

```
setTimeout(() => {
    const result  = import('./b')
    result.then(res=>{
        console.log(res)
    })
}, 0);
```

## Commonjs 和 Es Module 总结

### Commonjs 总结

`Commonjs` 的特性如下：

- CommonJS 模块由 **JS 运行**时实现。
- CommonJs 是**单个值**导出，本质上导出的就是 exports 属性。
- CommonJS 是可以**动态加载**的，对每一个加载都存在缓存，可以有效的解决循环引用问题。
- CommonJS 模块**同步加载并执行**模块文件。

### es module 总结

`Es module` 的特性如下：

- ES6 Module **静态**的，不能放在块级作用域内，代码发生在编译时。
- ES6 Module 的值是**动态绑定**的，可以通过导出方法修改，可以直接访问修改结果。
- ES6 Module 可以**导出多个属性和方法**，可以单个导入导出，混合导入导出。
- ES6 模块**提前加载并执行**模块文件，
- ES6 Module 导入模块在**严格模式**下。
- ES6 Module 的特性可以很**容易实现 Tree Shaking 和 Code Splitting。**