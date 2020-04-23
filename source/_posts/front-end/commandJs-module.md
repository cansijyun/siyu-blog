---
title: 模块化CommonJS 与 ES6 Modules的区别
date: 19-03-03 20:19:47
tags:
categories: 前端
---

> 在 ES6 之前，社区制定了一些模块加载方案，最主要的有 **CommonJS** 和 **AMD** 两种。前者用于**服务器**，后者用于**浏览器**。**ES6** 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，**成为浏览器和服务器通用的模块解决方案**。
> **ES6** 模块的设计思想是尽量的**静态化**，使得**编译时**就能**确定模块的依赖关系**，以及输入和输出的变量。**CommonJS** 和 **AMD** 模块，都只能在**运行时确定**这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

```
// CommonJS模块let { stat, exists, readFile } = require('fs');// 等同于let _fs = require('fs');let stat = _fs.stat;let exists = _fs.exists;let readfile = _fs.readfile;
```

- 上面代码的实质是**整体加载**fs模块（即加载fs的**所有方法**），生成一个对象（_fs），然后再从这个对象上面读取 3 个方法。这种加载称为“运行时加载”，因为只有**运行时**才能得到这个对象，导致完全没办法在编译时做“静态优化”。

#### ES6模块化

- ES6 模块不是对象，而是通过**export命令**显式指定输出的代码，再通过**import命令**输入。

  ```
  // ES6模块import { stat, exists, readFile } from 'fs';
  ```

- 上面代码的实质是从fs模块加载 3 个方法，其他方法不加载。这种加载称为**“编译时加载”**或者**静态加载**，即 ES6 可以**在编译时就完成模块加载**，效率要比 CommonJS 模块的加载方式高。当然，这也导致了**没法引用 ES6 模块本身**，因为它不是对象。

- 由于 ES6 模块是**编译时加载**，使得静态分析成为可能。有了它，就能进一步拓宽 JavaScript 的语法，比如引入宏（macro）和类型检验（type system）这些只能靠静态分析实现的功能。

- 除了静态加载带来的各种好处，ES6 模块还有以下好处：

  - 不再需要UMD模块格式了，将来服务器和浏览器都会支持 ES6 模块格式。目前，通过各种工具库，其实已经做到了这一点。
  - 将来浏览器的新 API 就能用模块格式提供，不再必须做成全局变量或者navigator对象的属性。
    不再需要对象作为命名空间（比如Math对象），未来这些功能可以通过模块提供。

- ES6 模块的特点：

  - 静态化，必须在顶部，不能使用条件语句，自动采用严格模式
  - treeshaking和编译优化，以及webpack3中的作用域提升
  - 外部可以拿到实时值，而非缓存值(是引用而不是copy)

- 简单来说

  - CommonJS是在内存中的对象，运行时才加载。
  - ES6 Modules是编译时就加载的代码。



#### ES6 模块与 CommonJS 模块的差异

- 讨论 Node 加载 ES6 模块之前，必须了解 ES6 模块与 CommonJS 模块完全不同。
- 它们有两个重大差异：
  - CommonJS 模块输出的是一个**值的拷贝**，ES6 模块输出的是**值的引用**。
  - CommonJS 模块是**运行时**加载，ES6 模块是**编译时**输出接口。

CommonJs模块化

```javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
// main.js
var mod = require('./lib');
console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```


ES6模块化

```javascript
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```


从上面我们看出，CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。而ES6 模块是动态地去被加载的模块取值，并且变量总是绑定其所在的模块。


#### 代码举例

- 现在有三个模块：

  > m1.mjs

  ```
  export let a1 = 1;
  ```

  

  > m2.mjs

  ```
  export let a2 = 2;
  ```

  

  ```
  export let a3 = 3;
  ```

- 1、基本使用

  ```
  //index.mjsimport { a1 } from './m1';import { a2 } from './m2';import { a3 } from './m3';console.log(a1, a2, a3);
  ```

- 结果:

  ```
  (node:18435) ExperimentalWarning: The ESM module loader is experimental.1 2 3
  ```

- 2、验证ES6 module的静态特性和接口特性：

  - 将m1改成

  ```
  export let a1= 1;setTimeout(()=>a1=2,500);
  ```

  ```
  //index.mjsimport { a1 } from './m1';import { a2 } from './m2';import { a3 } from './m3';console.log(a1, a2, a3);setTimeout(() => {    console.log(a1, a2, a3);},1000)
  ```

- 结果：

  ```
  (node:18994) ExperimentalWarning: The ESM module loader is experimental.1 2 32 2 3
  ```

- 体现了接口性。

#### 重点

##### ES6 Module和CommonJS模块的区别：

- CommonJS是对模块的浅拷贝，ES6 Module是对模块的引用。

  - 即ES6 Module只存只读，不能改变其值，具体点就是指针指向不能变，类似const

- import的接口是read-only（只读状态），不能修改其变量值。

  - 即不能修改其变量的指针指向，但可以改变变量内部指针指向。

- 可以对commonJS对重新赋值（改变指针指向），但是对ES6 Module赋值会编译报错。

  ##### ES6 Module和CommonJS模块的共同点：

- CommonJS和ES6 Module都可以对引入的对象进行赋值，即对对象内部属性的值进行改变。



### [字节跳动] common.js 和 es6 中模块引入的区别？(霍小叶)

CommonJS 是一种模块规范，最初被应用于 Nodejs，成为 Nodejs 的模块规范。运行在浏览器端的 JavaScript 由于也缺少类似的规范，在 ES6 出来之前，前端也实现了一套相同的模块规范 (例如: AMD)，用来对前端模块进行管理。自 ES6 起，引入了一套新的 ES6 Module 规范，在语言标准的层面上实现了模块功能，而且实现得相当简单，有望成为浏览器和服务器通用的模块解决方案。但目前浏览器对 ES6 Module 兼容还不太好，我们平时在 Webpack 中使用的 export 和 import，会经过 Babel 转换为 CommonJS 规范。在使用上的差别主要有：



1. CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。

1. CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

1. CommonJs 是单个值导出，ES6 Module可以导出多个

1. CommonJs 是动态语法可以写在判断里，ES6 Module 静态语法只能写在顶层

1. CommonJs 的 this 是当前模块，ES6 Module的 this 是 undefined