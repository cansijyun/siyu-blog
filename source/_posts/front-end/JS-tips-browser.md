---
title: JS知识点
date: 19-03-03 20:19:47
tags:
categories: 前端
---

# 函数声明提升和变量声明提升

1. 函数声明提升

```
func()
function func () {
}
```


上例不会报错，正是因为 ‘函数声明提升’，即将函数声明提升（整体）到作用域顶部（注意是函数声明，不包括函数表达式），实际提升后结果同下：

```
// 函数声明提升
function func () {
}
func()
```


2. ### 变量声明提升（只有var声明的变量才有变量提升，let、const无；变量赋值无提升）

   ```
   console.log(num)
   var num = 10
   
   console.log(func)
   var func = function () {
   }
   ```


   上两例均会打印出 'undefined'，变量声明提升会将变量声明提升到作用域顶部，但只会提升声明部分，不会提升赋值部分，实际提升后结果如下：

   ```
   var num 
   console.log(num)
   num = 10
   
   var func
   console.log(func)
   func = function () {
   }
   ```




## *var和let区别

ES6 新增了`let`命令，用来声明局部变量。它的用法类似于`var`，但是所声明的变量，只在`let`命令所在的代码块内有效，而且有暂时性死区的约束。

先看个`var`的常见变量提升的面试题目：

```
题目1：
var a = 99;            // 全局变量a
f();                   // f是函数，虽然定义在调用的后面，但是函数声明会提升到作用域的顶部。 
console.log(a);        // a=>99,  此时是全局变量的a
function f() {
  console.log(a);      // 当前的a变量是下面变量a声明提升后，默认值undefined
  var a = 10;
  console.log(a);      // a => 10
}

// 输出结果：
undefined
10
99
```

实际顺序注意变量声明前移

```
var a          // 全局变量a
a = 99;  
f();                   // f是函数，虽然定义在调用的后面，但是函数声明会提升到作用域的顶部。 
console.log(a);        // a=>99,  此时是全局变量的a

function f() {
  var a
  console.log(a);      // 当前的a变量是下面变量a声明提升后，默认值undefined
   a = 10;
  console.log(a);      // a => 10
}


```

### ES6可以用let定义块级作用域变量

在ES6之前，我们都是用var来声明变量，而且JS只有函数作用域和全局作用域，没有块级作用域，所以`{}`限定不了var声明变量的访问范围。
例如：

```
{ 
  var i = 9;
} 
console.log(i);  // 9
```

ES6新增的`let`，可以声明块级作用域的变量。

```
{ 
  let i = 9;     // i变量只在 花括号内有效！！！
} 
console.log(i);  // Uncaught ReferenceError: i is not defined
```

## *let 配合for循环的独特应用

`let`非常适合用于 `for`循环内部的块级作用域。JS中的for循环体比较特殊，每次执行都是一个全新的独立的块作用域，用let声明的变量传入到 for循环体的作用域后，不会发生改变，不受外界的影响。看一个常见的面试题目：

```
for (var i = 0; i <10; i++) {  
  setTimeout(function() {  // 同步注册回调函数到 异步的 宏任务队列。
    console.log(i);        // 执行此代码时，同步代码for循环已经执行完成
  }, 0);
}
// 输出结果
10   共10个
// 这里面的知识点： JS的事件循环机制，setTimeout的机制等
```

如果把 `var`改成 `let`声明：

```
// i虽然在全局作用域声明，但是在for循环体局部作用域中使用的时候，变量会被固定，不受外界干扰。
for (let i = 0; i < 10; i++) { 
  setTimeout(function() {
    console.log(i);    //  i 是循环体内局部作用域，不受外界影响。
  }, 0);
}
// 输出结果：
0  1  2  3  4  5  6  7  8 9
```

# *浏览器事件流向

　　这里我们使用最简单的，最原始的事件绑定方式。2个div嵌套并且绑定有弹窗事件，那么当我们点击里面的div的时候，两个div的点击事件都会被触发这个是没有疑问的，那么它们的处理函数谁先被执行?

​     这里用IE8/9/10和Chrome浏览器同时实验，结果都是先弹出bt2，然后弹出bt1，也就是里面小div的事件先被处理了。我们来思考一下这是什么样的一个顺序，从DOM的结构上看，应该是这样的body > bt1 > bt2。我们把这个结构竖过来，bt2在整个结构的最下面，body在最上面。想象一下，当点击发生时产生一个泡泡(也就是事件对象)，然后这个泡泡慢慢，向上浮，首先路过bt2，然后路过bt1，在路过它们时依次执行事件函数，这就是冒泡型事件。



### 捕获阶段 概念：

> 事件从根节点流向目标节点，途中流经各个DOM节点，在各个节点上触发捕获事件，直到达到目标节点。

### 目标阶段 概念:

> 事件到达目标节点时，就到了目标阶段，事件在目标节点上被触发

### 冒泡阶段 概念:

> 事件在目标节点上触发后，不会终止，一层层向上冒，回溯到根节点

### addEventListener

#### attachEvent (IE 特有)

的第三个参数的作用，通过查文档可以知，addEventListener的第三个参数是一个布尔类型
1、第三个参数是false：事件从里到外执行，这种效果叫事件冒泡
2、第三个参数是true：事件从里到外执行，执行顺序颠倒过来了，这种效果叫做事件捕获。

#### a. 事件从里到外执行，这种效果叫事件冒泡

#### b. 事件从里到外执行，执行顺序颠倒过来了，这种效果叫做事件捕获。

注意：事件发生的时候，要经过事件的三个阶段，我们常常使用的是事件冒泡阶段，而其他两个阶段不能人为干预。

小结：事件的三个阶段

```
第一阶段：捕获阶段
第二阶段：目标阶段（执行当前点击的元素）
第三阶段：冒泡阶段
```

事件三个阶段如下如

#### 第一阶段：捕获阶段

![_](https://yqfile.alicdn.com/31a60070e9785fdab36dcaa12b41c28e0867a823.png)

#### 第二阶段：目标阶段（执行当前点击的元素）

#### 第三阶段：冒泡阶段

![_](https://yqfile.alicdn.com/853a3d173a3dede773e33c8fc8fd0c69192fd830.png)

注意，注册事件有三种，其中onclick、attachEvent没有第三个参数，**实际上我们无法通过onclick、attachEvent来干预事件的第一阶段和第二阶段，并且很多时候我们只关心事件的第三阶段，即冒泡阶段**。

### 事件传播

[事件传播的三个阶段：捕获，目标对象，冒泡。](https://www.w3.org/TR/DOM-Level-3-Events/#event-flow)




![img](https:////upload-images.jianshu.io/upload_images/3748553-8a2a00abefbffd10.png?imageMogr2/auto-orient/strip|imageView2/2/w/707/format/webp)

image.png



1.其中捕获（Capture）是 事件对象([event object](https://dom.spec.whatwg.org/#event)) 从 window 派发到 目标对象父级的过程。
 2.目标（Target）阶段是 事件对象派发到目标元素时的阶段，如果事件类型指示其不冒泡，那事件传播将在此阶段终止。
 3.冒泡（Bubbling）阶段 和捕获相反，是以目标对象父级到 window 的过程。
 在任一阶段调用 [stopPropagation](https://dom.spec.whatwg.org/#dom-event-stoppropagation) 都将终止本次事件的传播。



```javascript
  window.addEventListener('click',function (event) {
    console.log(event)
  },true)
```

> note: [`addEventListener`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener)形式注册的监听事件接受参数以指定是否在捕获阶段触发本次事件，默认值为否(既冒泡阶段)。以事件处理器注册的事件在非捕获阶段触发。

点击视窗的内任一元素，输出 [MouseEvent](https://w3c.github.io/uievents/#events-mouseevents) 对象。查看对象的 path 属性，既该对象的传播路径。


![img](https:////upload-images.jianshu.io/upload_images/3748553-213ab8eab7baad0f.png?imageMogr2/auto-orient/strip|imageView2/2/w/615/format/webp)



### 阻止冒泡和默认事件

- 调用  `stopPropagation` 严格来说不是阻止冒泡，是阻止事件传播，捕获阶段也可以直接阻止。
  

  

  ![img](https:////upload-images.jianshu.io/upload_images/3748553-0ed6a96650f45f13.png?imageMogr2/auto-orient/strip|imageView2/2/w/909/format/webp)

  image.png

  

- 事件接口还有一个  `cancelBubble` 因历史原因的 stopPropagation 的别名，给其赋值 true 可以达到调用 stopPropagation 同样的效果。

- 或者你在事件处理器（通过 `on...`  属性注册）的回调返回 true，这个返回值决定了事件是否被取消，这里的取消的作用要根据事件的类型而定。



```jsx
element.onclick = function () { return true}
```

- 调用 `preventDefault` 则是阻止默认事件。

> 注： jQuery 注册事件时返回 false 是一种用于同时调用 e.preventDefault 和 e.stopPropagation 的快捷方式。

### 事件回调

- [`addEventListener()`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener)
- on-event 事件处理器

这里主要提我自己平常没重视的地方~ 例如：
 1.回调函数的`this`被设置为注册该事件的 DOM 元素。
 2.混用时的调用顺序。
 首次设定 事件处理器 为非 null 值时的顺序，就是该事件处理器的值被调用的顺序。



```javascript
<button id="test" >Start Demo</button>
<script>
var button = document.getElementById('test');
button.addEventListener('click', function () { alert('ONE') }, false);
button.setAttribute('onclick', "alert('NOT CALLED')"); // event handler listener is registered here
button.addEventListener('click', function () { alert('THREE') }, false);
button.onclick = function () { alert('TWO'); };
button.addEventListener('click', function () { alert('FOUR') }, false);
</script>
//alert 顺序 one 、two 、three、 four
```

### *介绍事件委托及其优点，并说明如何使用？

事件委托是利用事件冒泡原理，让节点的父级代为执行事件。而不需要循环遍历元素的子节点，大大减少dom操作；

举个例子：100个li都有相同的点击事件，那么常见的方法是for循环100个节点，都执行同一个事件，我们使用事件委托的话，让它的父级ul做事件处理，当点击li时，事件会冒泡到ul上，这是我们定义的事件就会执行啦。
作者：清汤饺子链接：https://www.jianshu.com/p/d5e8dfbe1c41来源：简书著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

DOM 事件提供了关于元素的有用信息：通过 `Event.target` 可以获取触发事件的元素。这允许父元素像目标元素监听事件一样去处理事件，而不是去监听处理父元素的所有子元素或单独处理父元素本身。

事件委托有如下优点：

- 不仅能提高性能，还可以减少内存消耗：他只需要注册一个单独的事件监听器就可以处理所有需要该类事件处理的元素。
- 如果元素是动态的添加到父元素上，则不需要为他们注册新的事件侦听器。

事件委托可优化如下逐一绑定事件的案例：

```
  document.querySelectorAll("button").forEach(button => {
     button.addEventListener("click", handleButtonClick)
}) 
```

事件委托主要使用条件来确保目标子元素与我们所期待的元素是否匹配：

```
  document.addEventListener("click", e => {
  if (e.target.closest("button")) {
    handleButtonClick()
  }
}) 
```

### 加分回答

- 需要对每种类型的事件进行逐一绑定
- 通过 `event.currentTarget` 来获取绑定事件而非触发事件的元素，从而减少元素匹配
- `focus`、`blur` 这类事件没有冒泡机制，因此无法进行委托
- 当 `mousemove`、`mouseout` 这类事件需要计算定位时只能通过逐层计算子元素的位置来获取，这对性能消耗较高。因此此类事件如需要计算定位的话，也不太适合用于事件委托
- 在执行所绑定的函数时需注意 `this` 和传入的参数
- `Element.matches()` 可以帮助判断元素是否匹配
- 事件捕获与事件冒泡相反，事件会从最外层开始发生，直到最具体的元素。如下：

# JS对象

## JavaScript Window - 浏览器对象模型

#### 属性

两个属性可用用于确定浏览器窗口的尺寸。

这两个属性都以像素返回尺寸：

- window.innerHeight - 浏览器窗口的内高度（以像素计）
- window.innerWidth - 浏览器窗口的内宽度（以像素计）

浏览器窗口（浏览器视口）不包括工具栏和滚动条。

对于 Internet Explorer 8, 7, 6, 5：

- document.documentElement.clientHeight
- document.documentElement.clientWidth

或

- document.body.clientHeight
- document.body.clientWidth

### 方法

- window.open() - 打开新窗口
- window.close() - 关闭当前窗口
- window.moveTo() -移动当前窗口
- window.resizeTo() -重新调整当前窗口

# 浅拷贝和深拷贝

1，对于字符串类型，浅复制是对值的复制，对于对象来说，**浅复制是对对象地址的复制**，并没   有开辟新的栈，也就是复制的结果是两个对象指向同一个地址，修改其中一个对象的属性，则另一个对象的属性也会改变，***而深复制则是开辟新的栈\***，两个对象对应两个不同的地址，修改一个对象的属性，不会改变另一个对象的属性



- **浅拷贝**

```
复制// 第一层为深拷贝
Object.assign()
Array.prototype.slice()
扩展运算符 ...
```

- **深拷贝**

```
复制JSON.parse(JSON.stringify())
```

递归函数

```
复制function cloneObject(obj) {
  var newObj = {} //如果不是引用类型，直接返回
  if (typeof obj !== 'object') {
    return obj
  }
  //如果是引用类型，遍历属性
  else {
    for (var attr in obj) {
      //如果某个属性还是引用类型，递归调用
      newObj[attr] = cloneObject(obj[attr])
    }
  }
  return newObj
}
```

#### **1.javascript变量包含两种不同数据类型的值：基本类型和引用类型。**

基本类型值指的是简单的数据段，包括es6里面新增的一共是有6种，具体如下：

number、string、boolean、null、undefined、symbol

引用类型值指那些可能由多个值构成的对象，只有一种如下：

object

在将一个值赋给变量时，解析器必须确定这个值是基本类型值还是引用类型值。基本数据类型是按值访问的，因为可以操作保存在变量中的实际的值。

引用类型的值是保存在内存中的对象。与其他语言不同，JavaScript 不允许直接访问内存中的位置，也就是说不能直接操作对象的内存空间。 在操作对象时， 实际上是在操作对象的引用而不是实际的对象。



#### **2.javascript的变量的存储方式--栈（stack）和堆（heap）**

栈：自动分配内存空间，系统自动释放，里面存放的是基本类型的值和引用类型的地址

堆：动态分配的内存，大小不定，也不会自动释放。里面存放引用类型的值。

![img](https://pic2.zhimg.com/80/v2-e79fc1f234a9b78d4241e3187dffa11b_720w.jpg)



#### **3.javascript值传递与址传递**

基本类型与引用类型最大的区别实际就是传值与传址的区别

值传递：基本类型采用的是值传递。

```javascript
var a = 100;
var b = a;
b++;
console.log(a);//100
console.log(b);//101
```

址传递：引用类型则是地址传递，将存放在栈内存中的地址赋值给接收的变量。

```javascript
var a =[1,2,3]
var b = a;
b.push(4);
console.log(a);//[1,2,3,4]
console.log(b);//[1,2,3,4]
```

分析：由于a和b都是引用类型，采用的是址传递，即a将地址传递给b，那么a和b必然指向同一个地址(引用类型的地址存放在栈内存中)，而这个地址都指向了堆内存中引用类型的值。当b改变了这个值的同时，因为a的地址也指向了这个值，故a的值也跟着变化。

就好比是a租了一间房，将房间的地址给了b，b通过地址找到了房间，那么b对房间做的任何改变（添加了一些绿色植物）对a来说肯定同样是可见的。

那么如何解决上面出现的问题，就是使用浅拷贝或者深拷贝了。



下面我将以最常见的数组和对象的深浅拷贝为例。

浅拷贝解决就是先设置一个新的对象obj2,通过遍历的方式将obj1对象的值一一赋值给obj2对象。

代码实现如下：

```
jQuery.extend([deep], target, object1, [objectN])；
```



#### 三、深拷贝的实现

1. 首先看一下乞丐版的深拷贝吧！**JSON.stringify()以及JSON.parse()**



```javascript
 var obj1 = {
    a: 1,
    b: 2,
    c: 3
}
var objString = JSON.stringify(obj1);
var obj2 = JSON.parse(objString);
obj2.a = 5;
console.log(obj1.a);  // 1
console.log(obj2.a); // 5
```

**可以看到没有发生引用问题，修改obj2的数据，并不会对obj1造成任何影响**
 **但是为什么说它是乞丐版的呢？**
 **那是因为 使用JSON.stringify()以及JSON.parse()它是不可以拷贝 undefined ， function， RegExp 等等类型的**

1. 接着来看第二种方式 **Object.assign(target, source)**



```javascript
 var obj1 = {
    a: 1,
    b: 2,
    c: 3
}
var obj2 = Object.assign({}, obj1);
obj2.b = 5;
console.log(obj1.b); // 2
console.log(obj2.b); // 5
```

第二种方式实现的看起来也没有任何的问题，但是这是一层对象，如果是有多层嵌套呢



```javascript
 var obj1 = {
    a: 1,
    b: 2,
    c: ['a','b','c']
}
var obj2 = Object.assign({}, obj1);
obj2.c[1] = 5;
console.log(obj1.c); // ["a", 5, "c"]
console.log(obj2.c); // ["a", 5, "c"]
```

可以看到对于一层对象来说是没有任何问题的，但是如果对象的属性对应的是其它的引用类型的话，还是只拷贝了引用，修改的话还是会有问题

1. 第三种方式  **递归拷贝**



```javascript
// 定义一个深拷贝函数  接收目标target参数
function deepClone(target) {
    // 定义一个变量
    let result;
    // 如果当前需要深拷贝的是一个对象的话
    if (typeof target === 'object') {
    // 如果是一个数组的话
        if (Array.isArray(target)) {
            result = []; // 将result赋值为一个数组，并且执行遍历
            for (let i in target) {
                // 递归克隆数组中的每一项
                result.push(deepClone(target[i]))
            }
         // 判断如果当前的值是null的话；直接赋值为null
        } else if(target===null) {
            result = null;
         // 判断如果当前的值是一个RegExp对象的话，直接赋值    
        } else if(target.constructor===RegExp){
            result = target;
        }else {
         // 否则是普通对象，直接for in循环，递归赋值对象的所有值
            result = {};
            for (let i in target) {
                result[i] = deepClone(target[i]);
            }
        }
     // 如果不是对象的话，就是基本数据类型，那么直接赋值
    } else {
        result = target;
    }
     // 返回最终结果
    return result;
}
```

##### 可以看一下效果



```javascript
    let obj1 = {
        a: {
            c: /a/,
            d: undefined,
            b: null
        },
        b: function () {
            console.log(this.a)
        },
        c: [
            {
                a: 'c',
                b: /b/,
                c: undefined
            },
            'a',
            3
        ]
    }
    let obj2 = deepClone(obj1);
        console.log(obj2);
```



![img](https:////upload-images.jianshu.io/upload_images/18087456-4df102c214490d01.png?imageMogr2/auto-orient/strip|imageView2/2/w/664/format/webp)

深拷贝


**可以看到最终拷贝的结果是null、undefinde、function、RegExp等特殊的值也全部拷贝成功了，而且我们修改里边的值也不会有任何问题的**
**到这里我们就实现了一个简单的深拷贝，当然，我的这个也只是简单实现一下，还有很多问题没有解决，只是给您提供一个思路**



作者：郝晨光
链接：https://www.jianshu.com/p/f4329eb1bace
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。