---
title: Angular面试准备
date: 19-03-03 20:19:47
tags:
categories: 前端
---

## 1. 请解释Angular 2应用程序的生命周期hooks是什么？

Angular 2组件/指令具有生命周期事件。是由@angular/core管理的。

@angular/core会创建组件。渲染它。创建并呈现它的后代。当@angular/core的数据绑定属性更改时，处理就会更改，在从DOM中删除其模板之前，就会销毁掉它。Angular提供了一组生命周期hooks（特殊事件）。能够被分接到生命周期中，并在须要时执行操作。

构造函数会在全部生命周期事件之前执行。

每一个接口都有一个前缀为ng的hook方法。比如，ngOnint界面的OnInit方法，这种方法必须在组件中实现。 

一部分事件适用于组件/指令，而少数事件仅仅适用于组件。

- ngOnChanges：当Angular设置其接收当前和上一个**对象值的数据绑定属性时响应**。

- ngOnInit：在第一个ngOnChange触发器之后，初始化组件/指令。

  这是最经常使用的方法。用于从后端服务检索模板的数据。

- ngDoCheck：检測并在Angular上下文发生变化时执行。

  每次更改检測执行时，会被调用。

- ngOnDestroy：在Angular销毁指令/组件之前清除。取消订阅可观察的对象并脱离事件处理程序，以避免内存泄漏。

  

  

组件特定hooks：

- ngAfterContentInit：组件内容已初始化完毕
- ngAfterContentChecked：在Angular检查投影到其视图中的绑定的外部内容之后。
- ngAfterViewInit：Angular创建组件的视图后。
- ngAfterViewChecked：在Angular检查组件视图的绑定之后。

### 2. 使用Angular 2，和使用Angular 1相比。有什么优势？

1. Angular 2是一个平台，不仅是一种语言
2. 更好的速度和性能
3. 更简单的依赖注入
4. 模块化。跨平台
5. 具备ES6和Typescript的优点。
6. 灵活的路由，具备延迟载入功能
7. 更easy学习


### 3. Angular 2中的路由工作原理是什么？

路由是能够让用户在视图/组件之间导航的机制。Angular 2简化了路由，并提供了在模块级（延迟载入）下配置和定义的灵活性。 

Angular应用程序具有路由器服务的单个实例。而且每当URL改变时。对应的路由就与路由配置数组进行匹配。在成功匹配时，它会应用重定向，**此时路由器会构建ActivatedRoute对象的树**。同一时候包括路由器的当前状态。在重定向之前，路由器将通过执行保护（[CanActivate](https://blog.thoughtram.io/angular/2016/07/18/guards-in-angular-2.html)）来检查是否同意新的状态。

Route Guard仅仅是路由器执行来检查路由授权的接口方法。

保护执行后，它将解析路由数据并通过将所需的组件实例化到<router-outlet> </ router-outlet>中来激活路由器状态。

### 3. Angular7的核心依赖是什么？

Angular 7的核心依赖性 核心依赖关系有两种类型：RxJS和TypeScript。 RxJS 6.3Angular 7使用RxJS版本6.3.Angular 6的版本没有变化 TypeScript 3.1Angular 7使用TypeScript 3.1版。它是Angular 6版本2.9的升级版。