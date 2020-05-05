---
title: *JS常用函数和方法
date: 19-03-03 20:19:47
tags:
categories: 前端
---

# [ ]/数组/列表/集合Set/字典Map



`Map`和`Set`是ES6标准新增的数据类型，请根据浏览器的支持情况决定是否要使用

Set和Map主要的应用场景在于**数组去重**和**数据存储**，

Set是一种叫做**集合**的数据结构，Map是一种叫做**字典**的数据结构

集合

- 集合是由一组无序且唯一(即不能重复)的项组成的，可以想象成集合是一个既没有重复元素，也没有顺序概念的数组
- ES6提供了新的数据结构Set。它类似于数组，但是成员的值都是唯一的，没有重复的值
- Set 本身是一个构造函数，用来生成 Set 数据结构
- 这里说的Set其实就是我们所要讲到的集合，先来看下基础用法

- Set的属性：
  - size：返回集合所包含元素的数量
- Set的方法：
  - 操作方法
    - add(value)：向集合添加一个新的项
    - delete(value)：从集合中移除一个值
    - has(value)：如果值在集合中存在，返回true,否则false
    - clear(): 移除集合里所有的项
  - 遍历方法
    - keys()：返回一个包含集合中所有键的数组
    - values()：返回一个包含集合中所有值的数组
    - entries：返回一个包含集合中所有键值对的数组(感觉没什么用就不实现了)
    - forEach()：用于对集合成员执行某种操作，没有返回值



### Array[index] 获取/修改数组的某一个值

```
var arr = new Array(6)
arr[0] = "George"
arr[1] = "John"
arr[2] = "Thomas"
arr[3] = "James"
arr[4] = "Adrew"<br><br>arr.slice(0,3)
```



### *Array.join() 数组转成字符串

```
let arr = [1, 2, 3, 4, 5];
   let str1 = arr.toString()
   let str2 = arr.toString(',')
   let str3 = arr.toString('##')
   console.log(str1)// 12345
   console.log(str2)// 1,2,3,4,5
   console.log(str3)// 1##2##3##4##5
```

### *new  Set(Array)去重

### set：

定义：新数据结构Set，类似于数组，但成员值不重复

使用： new Set()

> ```
> let set = new Set([1,2,3,4,3,2,1]);  […set] // [1,2,3,4]
> ```

ps:New Set() 接受一个数组或类数组对象,在Set内部， NAN相等，两个对象不等，可以用length检测，可以用for...of遍历



作者：静_summer
链接：https://www.jianshu.com/p/630800e6d2af
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### *Array.slice截取(切片)数组 得到截取的数组**

返回从原数组中指定开始索引(包含)到结束索引(不包含)之间的项组成的新数组,原数组木变 ，索引从0开始

```js
var a = ['a','b','c','d','e']; 
a.slice(1,3);//["b", "c"] a:['a','b','c','d','e']
a.slice(0,4);//["a", "b", "c", "d"]
a.slice(3,4);//["d"]
```



### Array.splice(开始位置， 删除的个数，元素)

**剪接数组 原数组变化 可以实现shift前删除，pop后删除,unshift前增加,同push后增加一样的效果**

```js
let arr = [1, 2, 3, 4, 5];
     let arr1 = arr.splice(2, 0,'haha')
     let arr2 = arr.splice(2, 3)
     let arr1 = arr.splice(2, 1,'haha')
     console.log(arr1) //[1, 2, 'haha', 3, 4, 5]新增一个元素
     console.log(arr2) //[1, 2] 删除三个元素
     console.log(arr3) //[1, 2, 'haha', 4, 5] 替换一个元素
```



### *Array.push()

 此方法是在数组的后面添加新加元素，此方法改变了数组的长度：

### Array.pop()

此方法在数组后面删除最后一个元素，并返回数组，此方法改变了数组的长度：

```
let arr = [1, 2, 3, 4, 5]
    arr.pop()
    console.log(arr) //[1, 2, 3, 4]
    console.log(arr.length) //4
```



### Array.shift()

 此方法在数组后面删除第一个元素，并返回数组，此方法改变了数组的长度：

```
let arr = [1, 2, 3, 4, 5]
    arr.shift()
    console.log(arr) //[2, 3, 4, 5]
    console.log(arr.length) //4 
```

### Array.unshift()

 此方法是将一个或多个元素添加到数组的开头，并返回新数组的长度：

```
let arr = [1, 2, 3, 4, 5]
    arr.unshift(6, 7)
    console.log(arr) //[6, 7, 1, 2, 3, 4, 5]
    console.log(arr.length) //7 
```



### Array.concat()**数组合并**

 此方法是一个可以将多个数组拼接成一个数组：

```
let arr1 = [1, 2, 3]
      arr2 = [4, 5]
  let arr = arr1.concat(arr2)
  console.log(arr)//[1, 2, 3, 4, 5]
```

### Array.isArray()

```
 判断一个对象是不是数组，返回的是布尔值
```

***Array.indexOf   数组元素索引/是否存在某元素**

并返回元素索引，不存在返回-1,索引从0开始

```
var a = ['a','b','c','d','e']; 
a.indexOf('a');//0
a.indexOf(a);//-1
a.indexOf('f');//-1
a.indexOf('e');//4
```

### **Array.reverse 数组翻转**

并返回翻转后的原数组，原数组翻转了

```
var a = [1,2,3,4,5]; 
a.reverse()//a：[5, 4, 3, 2, 1] 返回[5, 4, 3, 2, 1]
```



## **ES6

### **Array.forEach(element,index,array)**

　　遍历数组，参数为一个回调函数，回调函数有三个参数：当前元素，元素索引，整个数组

```js
var a=new Array(1,2,3,4,5,6);
a.forEach(function(e,i,array){
    array[i]=e+1;//array：当前数组，i：当前索引，e：当前元素
});
console.log(a); //[2, 3, 4, 5, 6, 7] 
```

### Array.map():

    map和forEach等遍历方法不同，在forEach中return语句是没有任何效果的，而map则可以改变当前循环的值，返回一个新的被改变过值之后的数组（map需return），一般用来处理需要修改某一个数组的值。

```
let arr1 = [1,2,3];
let arr2 = arr1.map((value,key,arr) => {
    console.log(value)    // 1，2，3
    console.log(key)      // 0，1，2
    console.log(arr)      //[1,2,3] [1,2,3] [1,2,3]
    return value * value;
})
console.log(arr1); // [ 1, 2, 3 ]
console.log(arr2); // [ 1, 4, 9 ]
```



```
//map()方法主要用于遍历数组,操作数据不会改变原数组
        let arr4 = [
            {name: 'zhangsan', age: 18, sex: 'male'},
            {name: 'lisi', age: 30, sex: 'male'},
            {name: 'xiaohong', age: 20, sex: 'female'}
        ];
 
        //map示例1：循环遍历数组，返回每个人的年龄+1之后的数据
        let arr5 = arr4.map(item => item.age + 1);
        console.log(arr5);  //[19, 31, 21]
        console.log(arr4);  //[{name: 'zhangsan', age: 18, sex: 'male'},{name: 'lisi', age: 30, sex: 'male'},{name: 'xiaohong', age: 20, sex: 'female'}]
 
        //map示例2：循环遍历数组,根据某个条件筛选过滤数组
        let arr6 = arr4.map((item) => {
            if (item.age > 20) {
                return item;
            }
        });
        console.log(arr6);  //[undefined,{name: 'lisi', age: 30, sex: 'male'},undefined]
```



### Array.filter()



```js
let arr1 = [
            {name: 'zhangsan', age: 18, sex: 'male'},
            {name: 'lisi', age: 30, sex: 'male'},
            {name: 'xiaohong', age: 20, sex: 'female'}
        ];
 
        //filter()方法：主要用于过滤筛选数组，数组filter后，返回的结果为新的数组,不会改变原数组的值
        //filter示例1:返回年龄大于20岁的数据
        let arr2 = arr1.filter(item => item.age > 20);
        console.log(arr2);   // [{name: "lisi", age: 30, sex: "male"}]
        console.log(arr1);   // [{name: 'zhangsan', age: 18, sex: 'male'},{name: 'lisi', age: 30, sex: 'male'},{name: 'xiaohong', age: 20, sex: 'female'}]
 
        //filter示例2：返回不是男生的数据
        let arr3 = arr1.filter((item) => {
            return item.sex !== 'male';
        });
        console.log(arr3);  // [{name: "xiaohong", age: 20, sex: "female"}]

```

### Array.find()方法

  //find示例1：查找name='zhangsan'的数据
        let arr8 = arr7.find(item => item.name === 'zhangsan');
        console.log(arr8);  //只返回了第一条数据 {name: "zhangsan", age: 18, sex: "male"}

        let arr9 = arr7.find(item => item.name === 'wangwu');
        console.log(arr9);  //没有找到符合条件的数据，返回undefined
     
        //注意： 与filter()方法区别 filter会一直查找下去，返回符合name='zhangsan'的全部数据
        let arr10 = arr7.filter(item => item.name === 'zhangsan');
        console.log(arr10); //返回两条数据 [{name: "zhangsan", age: 18, sex: "male"}1: {name: "zhangsan", age: 20, sex: "female"}]


**Array..reduce(function(v1,v2),value) / .reduceRight(function(v1,v2),value)**
　　遍历数组，调用回调函数，将数组元素累加成一个值，reduce从索引最小值开始，reduceRight反向，
　　方法有两个参数：
　　1、回调函数：把两个值合为一个，返回结果
　　2、value，一个初始值，可选

( acc, cur ,index) 

//acc就是累加值，return的值就是下一个循环的acc的值  

 //cur就是当前循环的源数组的值     //index就是当前循环的index值

```js
var array = [[1, 2], [3, 4], [5, 6]].reduce(( acc, cur ) => {
    return acc.concat(cur)
}, []);

console.log(array)  // [ 0, 1, 3, 4, 5, 6 ]
```

### Array.every()

```
此方法是将所有元素进行判断返回一个布尔值，如果所有元素都满足判断条件，则返回true，否则为false：
```

```
let arr = [1, 2, 3, 4, 5]
    const isLessThan4 = value => value < 4
    const isLessThan6 => value => value < 6
    arr.every(isLessThan4 ) //false
    arr.every(isLessThan6 ) //true
```

### Array.reduce()

```
 此方法是所有元素调用返回函数，返回值为最后结果,传入的值必须是函数类型：
```

```js
let arr = [1, 2, 3, 4, 5]
   const add = (a, b) => a + b
  let sum = arr.reduce(add)
   //sum = 15 相当于累加的效果
  与之相对应的还有一个 Array.reduceRight() 方法，区别是这个是从右向左操作的
```

### Array.from

定义：把泪数组对象和有iterator接口的对象(Set Map Array)转化为数组

使用：Array.from(arrayLike[, mapFn[, thisArg]]) 参数：类数组，处理函数map，map中的this指向的对象

Array.from([1, 2, 3, 4, 5], (n) => n  + 1) // 每个值都加一

- 转换map：

```
const map = new Map()

map.set(‘k1’, 1)

map.set(‘k2’, 2)

Const a = Array.from(map);  // [[‘k1’,1], [‘k2’, 2]]
```



- 转换set

```
const set1 = new Set()

Set1.add(1).add(2).add(3)

Var a = Array.from(set1) // [1,2,3]
```



- 转换string及unicode字符

```
console.log('%s', Array.from('hello world’)) //["h", "e", "l", "l", "o", " ", "w", "o", "r", "l", "d"]

console.log('%s', Array.from('\u767d\u8272\u7684\u6d77’)) //["白", "色", "的", "海"]
```



- 转换类数组对象：属性名必须为数字，或者可以转换为数字，必须有length 不然为空数组

```
var a = {0:1, 2:3, 4:5, length: 5};var b = {0:1, 2:3, 4:5, length: 3}

Array.from(a) // [1,undefined,3,undefined,4]

Array.from(b) // [1,undefined,3]
```



## 属性

### length 返回长度

| 属性        | 描述                           |
| ----------- | ------------------------------ |
| constructor | 返回创建此对象的数组函数的引用 |
| length      | 设置或者返回数组中元素的数目   |
| prototype   | 此属性可以向对象添加属性和方法 |



## **交集/并集/差集

有两个数组arr1,arr2
实现arr2中去除arr1相同的元素
e.g arr1=[1,2,3] arr2=[2,3,4] ===> result = [4]
实现

获取两个数组(arr1,arr2)的交集arr3
获取交集arr3与arr2中arr2的差集就是我们要的result

### 交集

```js
var arr3 = arr2.filter(function(v){
            return arr1.indexOf(v)!==-1 // 利用filter方法来遍历是否有相同的元素
        })
```

### 差集

```js
var result = arr2.concat(arr3 ).filter(function (v) {
                return arr2.indexOf(v)===-1 || arr3 .indexOf(v)===-1
            })
```

### 并集

```json
let union = a.concat(b.filter(v => !a.includes(v))) // [1,2,3,4,5]
```

# { }/对象

## 属性和方法

属性是与对象相关的值。

方法是能够在对象上执行的动作。

举例：汽车就是现实生活中的对象。

汽车的属性：

```
car['name']=Fiat


car.name=Fiat

car.model=500

car.weight=850kg

car.color=white 



```

汽车的方法：

```
car.start()

car.drive()

car.brake()
```



声明对象

```
var obj= {

  name:"Arvin",

  lastName:"Huang" ,

  whatsName:function(){

     alert(this.name+" "+this.lastName);   

  },
  
}
```



## 使用内建方法

此例使用 String 对象的 toUpperCase() 方法，把文本转换为大写：

```
var message = "Hello world!";
var x = message.toUpperCase();
```

x 的值，在以上代码执行后将是：

```
HELLO WORLD!
```

### *This访问对象的属性和方法

向对象添加方法是在构造器函数内部完成的：

### 实例

```js
var person = {
  firstName: "Bill",
  lastName : "Gates",
  id     : 678,
};
person.name = function() {
  return this.firstName + " " + this.lastName;
};  //Bill Gates
```





# 字符串

### **split(): 把字符串分割成字符串数组。

```js
var str="AA BB CC DD";
var string1="1:2:3:4:5";
var str1=str.split("");//如果把空字符串 ("")用作分割符，那么字符串的每个字符之间都会被分割
var str2=str.split(" "); //以空格为分隔符
var str3=str.split("",4); //4指定返回数组的最大长度
var str4=string1.split(":");
console.log(str1); // ["A", "A", " ", "B", "B", " ", "C", "C", " ", "D", "D"]
console.log(str2); //["AA" "BB" "CC" "DD"]
console.log(str3); //["A", "A", " ", "B"]
console.log(str4); // ["1", "2", "3", "4", "5"]
```

### String.indexOf

1、JavaScript 只有`indexOf`方法，可以用来确定一个字符串是否包含在另一个字符串中。如果不存在返回-1，如果存在返回字符串的位置。ES6 又提供了三种新方法。

- **includes()**：返回布尔值，表示是否找到了参数字符串。
- **startsWith()**：返回布尔值，表示参数字符串是否在原字符串的头部。
- **endsWith()**：返回布尔值，表示参数字符串是否在原字符串的尾部。

6、lastIndexOf(): 返回某个指定的子字符串在字符串中最后出现的位置。

```
var str="Hello World";
var str1=str.lastIndexOf("o");
var str2=str.lastIndexOf("world");
var str3=str.lastIndexOf("o",str1-1);
console.log(str1); //7
console.log(str2); //-1
console.log(str3); //4
```



slice(): 返回字符串中提取的子字符串。

```
var str="Hello World";
var str1=str.slice(2); //如果只有一个参数，则提取开始下标到结尾处的所有字符串
var str2=str.slice(2,7); //两个参数，提取下标为2，到下标为7但不包含下标为7的字符串
var str3=str.slice(-7,-2); //如果是负数，-1为字符串的最后一个字符。提取从下标-7开始到下标-2但不包含下标-2的字符串。前一个数要小于后一个数，否则返回空字符串

console.log(str1); //llo World
console.log(str2); //llo W
console.log(str3); //o Wor
```



substring(): 提取字符串中介于两个指定下标之间的字符。

```
var str="Hello World";
var str1=str.substring(2)
var str2=str.substring(2,2);
var str3=str.substring(2,7);
console.log(str1); //llo World
console.log(str2); //如果两个参数相等，返回长度为0的空串
console.log(str3); //llo W
```

注意：substring()用法与slice()一样，但不接受负值的参数。



substr(): 返回从指定下标开始指定长度的的子字符串

```
var str="Hello World";
var str1=str.substr(1)
var str2=str.substr(1,3);
var str3=str.substr(-3,2);
console.log(str1); //ello World 
console.log(str2); //ell
console.log(str3); //rl
```

1、toLowerCase(): 把字符串转为小写，返回新的字符串。

```
var str="Hello World";
var str1=str.toLowerCase();
console.log(str); //Hello World
console.log(str1); //hello world
```

2、toUpperCase(): 把字符串转为大写，返回新的字符串。

```
var str="hello world";
var str1=str.toUpperCase();
console.log(str); //hello world
console.log(str1); //HELLO WORLD
```

3、charAt(): 返回指定下标位置的字符。如果index不在0-str.length(不包含str.length)之间，返回空字符串。

```
var str="hello world";
var str1=str.charAt(6);
console.log(str1); 
```

4、charCodeAt(): 返回指定下标位置的字符的unicode编码,这个返回值是 0 - 65535 之间的整数。

```
var str="hello world";
var str1=str.charCodeAt(1);
var str2=str.charCodeAt(-2); //NaN
console.log(str1); //101
```

注意：如果index不在0-str.length(不包含str.length)之间，返回NaN。



## 正则

12、match(): 返回所有查找的关键字内容的数组。

```
var str="To be or not to be";
var reg=/to/ig;
var str1=str.match(reg);
console.log(str1); //["To", "to"]
console.log(str.match("Hello")); //null
```

5、matchAll()方法



## ES6



### String.repeat

2、`repeat`方法返回一个新字符串，表示将原字符串重复`n`次。

如下：

```
let str="hello"
str.repeat(2);//输出结果：hellohello
```

注意：如果是小数，那么取整，如果是0至-1之间的小数或者是0至1之间的小数，取整数。

3、字符串补全长度的功能。

`　　(1)padStart()`用于头部补全

`　　(2)padEnd()`用于尾部补全。

`　　padStart()`和`padEnd()`一共接受两个参数，第一个参数是字符串补全生效的最大长度，第二个参数是用来补全的字符串。

 4、`trimStart()`和`trimEnd()方法，消除字符串头部的空格、消除尾部的空格。`

　　ES2019对字符串实例新增了`trimStart()`和`trimEnd()`这两个方法。它们的行为与`trim()`一致，`trimStart()`消除字符串头部的空格，`trimEnd()`消除尾部的空格。它们返回的都是新字符串，不会修改原始字符串。

　　浏览器还部署了额外的两个方法，`trimLeft()`是`trimStart()`的别名，`trimRight()`是`trimEnd()`的别名。

5、matchAll()方法