---
title: 懒加载和预加载
date: 19-03-03 20:19:47
tags:
categories: 前端
---

### 什么是懒加载？

懒加载也就是延迟加载。 当访问一个页面的时候，先把img元素或是其他元素的背景图片路径替换成一张大小为1*1px图片的路径（这样就只需请求一次，俗称占位图），只有当图片出现在浏览器的可视区域内时，才设置图片正真的路径，让图片显示出来。这就是图片懒加载。

延迟加载有多种实现，这里只说其中一种，其实原理都差不多的.

### 为什么要使用懒加载？

很多页面，内容很丰富，页面很长，图片较多。比如说各种商城页面。这些页面图片数量多，而且比较大，少说百来K，多则上兆。要是页面载入就一次性加载完毕。 影响页面打开速度，用户体验也不好。

### 懒加载的优点是什么？

页面加载速度快、可以减轻服务器的压力、节约了流量,用户体验好

### 懒加载的原理是什么？

页面中的img元素，如果没有src属性，浏览器就不会发出请求去下载图片，只有通过javascript设置了图片路径，浏览器才会发送请求。 懒加载的原理就是先在页面中把所有的图片统一使用一张占位图进行占位，把正真的路径存在元素的“data-url”（这个名字起个自己认识好记的就行）属性里，要用的时候就取出来，再设置为图片的真实src.



### 懒加载的实现步骤？

1. 首先，不要将图片地址放到src属性中，而是放到其它属性(data-original)中。
2. 页面加载完成后，根据scrollTop判断图片是否在用户的视野内，如果在，则将data-original属性中的值取出存放到src属性中。
3. 在滚动事件中重复判断图片是否进入视野，如果进入，则将data-original属性中的值取出存放到src属性中。

### 实现方式

第一种是纯粹的延迟加载，使用setTimeOut或setInterval进行加载延迟.

第二种是条件加载，符合某些条件，或触发了某些事件才开始异步下载。

第三种是可视区加载，即仅加载用户可以看到的区域，这个主要由监控滚动条来实现，一般会在距用户看到某图片前一定距离遍开始加载，这样能保证用户拉下时正好能看到图片。

### 实现代码（可视区加载为例）

JS的实现

```
var imgs = document.getElementsByTagName('img');// 获取视口高度与滚动条的偏移量function lazyload(){    var scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop;    var viewportSize = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;    for(var i=0; i<imgs.length; i++) {	var x =scrollTop+viewportSize-imgs[i].offsetTop;	if(x>0){	    imgs[i].src = imgs[i].getAttribute('loadpic');   	}    }}setInterval(lazyload,1000);复制代码
```

JQ 的实现：

```
/*** 图片的src实现原理*/$(document).ready(function(){    // 获取页面视口高度    var viewportHeight = $(window).height();    var lazyload = function() {       // 获取窗口滚动条距离       var scrollTop = $(window).scrollTop();       $('img').each(function(){           // 判断 视口高度+滚动条距离 与 图片元素距离文档原点的高度		            var x = scrollTop + viewportHeight - $(this).position().top;           // 如果大于0 即该元素能被浏览者看到，则将暂存于自定义属性loadpic的值赋值给真正的src			           if (x > 0) {               $(this).attr('src',$(this).attr('loadpic'));            }       })    }    // 创建定时器 “实时”计算每个元素的src是否应该被赋值    setInterval(lazyload,100);});复制代码
```

### Vue实现

### 图片懒加载

#### 使用vue-lazyload插件：

1. 下载

   `$ npm install vue-lazyload -D`

2. 注册插件

   

   ```js
   // main.js:
   
   import Vue from 'vue'
   import App from './App.vue'
   import VueLazyload from 'vue-lazyload'
   // 使用方法1:
   Vue.use(VueLazyload)
   
   // 使用方法2: 自定义参数选项配置
   Vue.use(VueLazyload, {
     preLoad: 1.3, // 提前加载高度（数字 1 表示 1 屏的高度） 默认值:1.3
     error: 'dist/error.png', // 当加载图片失败的时候
     loading: 'dist/loading.gif', // 图片加载状态下显示的图片
     attempt: 3 //  加载错误后最大尝试次数 默认值:3
   })
   ```

1. 在页面中使用

   

   ```html
   <!-- mobile.vue -->
   
   <!-- 使用方法1: 可能图片url是直接从后台拿到的，把':src'替换成'v-lazy'就行 --> 
   <template>
     <ul>
       <li v-for="img in list">
         <img v-lazy="img.src" >
       </li>
     </ul>
   </template>
   
   <!-- 使用方法2: 使用懒加载容器v-lazy-container,和v-lazy差不多,通过自定义指令去定义的，不过v-lazy-container扫描的是内部的子元素 --> 
   <template>
     <div v-lazy-container="{ selector: 'img'}">
       <img data-src="/static/mobile/bohai/p2/bg.jpg">
       <img data-src="/static/mobile/bohai/p3/bg.jpg">
       ...
       <img data-src="/static/mobile/bohai/p13/bg.jpg">
     </div>
   </template>
   ```

   > 注意：v-lazy='src'中的src一定要使用data里面的变量，不能写真实的图片路径，这样会报错导致没有效果，因为vue的自定义指令必须对应data中的变量 只能是变量；v-lazy-container内部指定元素设置的data-src是图片的真实路径，不能是data变量，这个和v-lazy完全相反。

2. 给每一个状态添加样式

   

   ```css
   <style>
     img[lazy=loading] { }
     img[lazy=error] { }
     img[lazy=loaded] { }
   </style>
   ```

### 组件懒加载

> 主要分以下几步：
>
> 1.兼容低版本浏览器 => 2.新建懒加载组件 => 3.新建公共骨架屏组件 => 4.异步加载子组件 => 5.页面中使用

#### 1.兼容低版本浏览器

1. 该项目依赖 [IntersectionObserver API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)，如需在较低版本浏览器运行，需要首先处理兼容低版本浏览器，需要引入插件 **IntersectionObserver API polyfill**

2. 使用

   

   ```js
   // 1.下载
   $ npm install intersection-observer -D
   
   // 2.在mian.js引入
   import 'intersection-observer';
   ```

#### 2.新建一个VueLazyComponent.vue 文件

> 因为使用的懒加载组件插件部分不满足我们官网项目，所以把vue-lazy-component插件的核心代码取出来，新建一个VueLazyComponent.vue文件存放，在项目中需要使用到懒加载组件的页面引入即可。

1. 在需要使用懒加载的页面引入 VueLazyComponent.vue 文件

   - 引入

   

   ```vue
   <script>
     // 引入存放在mobile公共文件夹里的VueLazyComponent.vue 来包裹需要加载的子组件
     import VueLazyComponent from 'components/mobile/common/VueLazyComponent';
     // 引入骨架屏组件 (详细见-骨架屏组件) 在子组件未加载时,为待加载的子组件先占据一块空间
     import MobileSkeleton from 'components/mobile/common/MobileSkeleton';
   
     export default {
       components: {
         'vue-lazy-component': VueLazyComponent,
         'mobile-skeleton': MobileSkeleton,
       },
     };
   </script>
   ```

2. VueLazyComponent.vue 源码解读

   - 使用到的参数和事件

     **Props**

     | 参数      | 说明                                                         | 类型        | 可选值 | 默认值           |
     | --------- | ------------------------------------------------------------ | ----------- | ------ | ---------------- |
     | viewport  | 组件所在的视口，如果组件是在页面容器内滚动，视口就是该容器   | HTMLElement | true   | `null`，代表视窗 |
     | direction | 视口的滚动方向, `vertical`代表垂直方向，`horizontal`代表水平方向 | String      | true   | `vertical`       |
     | threshold | 预加载阈值, css单位                                          | String      | true   | `0px`            |
     | tagName   | 包裹组件的外层容器的标签名                                   | String      | true   | `div`            |
     | timeout   | 等待时间，如果指定了时间，不论可见与否，在指定时间之后自动加载 | Number      | true   | -                |

     **Events**

     | 事件名       | 说明                                         | 事件参数 |
     | ------------ | -------------------------------------------- | -------- |
     | before-init  | 模块可见或延时截止导致准备开始加载懒加载模块 | -        |
     | init         | 开始加载懒加载模块，此时骨架组件开始消失     | -        |
     | before-enter | 懒加载模块开始进入                           | el       |
     | before-leave | 骨架组件开始离开                             | el       |
     | after-leave  | 骨架组件已经离开                             | el       |
     | after-enter  | 懒加载模快已经进入                           | el       |
     | after-init   | 初始化完成                                   |          |

   

   ```vue
   <!-- VueLazyComponent.vue -->
   
   <template>
     <transition-group :tag="tagName" name="lazy-component" style="position: relative;"
                       @before-enter="(el) => $emit('before-enter', el)"
                       @before-leave="(el) => $emit('before-leave', el)"
                       @after-enter="(el) => $emit('after-enter', el)"
                       @after-leave="(el) => $emit('after-leave', el)">
       <div v-if="isInit" key="component">
         <slot :loading="loading"></slot>
       </div>
       <div v-else-if="$slots.skeleton" key="skeleton">
         <slot name="skeleton"></slot>
       </div>
       <div v-else key="loading"></div>
     </transition-group>
   </template>
   
   <script>
   export default {
     name: 'VueLazyComponent',
   
     props: {
       timeout: {
         type: Number,
         default: 0
       },
       tagName: {
         type: String,
         default: 'div'
       },
       viewport: {
         type: typeof window !== 'undefined' ? window.HTMLElement : Object,
         default: () => null
       },
       threshold: {
         type: String,
         default: '0px'
       },
       direction: {
         type: String,
         default: 'vertical'
       },
       maxWaitingTime: {
         type: Number,
         default: 50
       }
     },
   
     data() {
       return {
         isInit: false,
         timer: null,
         io: null,
         loading: false
       };
     },
   
     created() {
       // 如果指定timeout则无论可见与否都是在timeout之后初始化
       if (this.timeout) {
         this.timer = setTimeout(() => {
           this.init();
         }, this.timeout);
       }
     },
   
     mounted() {
       if (!this.timeout) {
         // 根据滚动方向来构造视口外边距，用于提前加载
         let rootMargin;
         switch (this.direction) {
           case 'vertical':
             rootMargin = `${this.threshold} 0px`;
             break;
           case 'horizontal':
             rootMargin = `0px ${this.threshold}`;
             break;
           default:
           // do nothing
         }
   
         // 观察视口与组件容器的交叉情况
         this.io = new window.IntersectionObserver(this.intersectionHandler, {
           rootMargin,
           root: this.viewport,
           threshold: [0, Number.MIN_VALUE, 0.01]
         });
         this.io.observe(this.$el);
       }
     },
   
     beforeDestroy() {
       // 在组件销毁前取消观察
       if (this.io) {
         this.io.unobserve(this.$el);
       }
     },
   
     methods: {
       // 交叉情况变化处理函数
       intersectionHandler(entries) {
         if (
           // 正在交叉
           entries[0].isIntersecting ||
           // 交叉率大于0
           entries[0].intersectionRatio
         ) {
           this.init();
           this.io.unobserve(this.$el);
         }
       },
   
       // 处理组件和骨架组件的切换
       init() {
         // 此时说明骨架组件即将被切换
         this.$emit('beforeInit');
         this.$emit('before-init');
   
         // 此时可以准备加载懒加载组件的资源
         this.loading = true;
   
         // 由于函数会在主线程中执行，加载懒加载组件非常耗时，容易卡顿
         // 所以在requestAnimationFrame回调中延后执行
         this.requestAnimationFrame(() => {
           this.isInit = true;
           this.$emit('init');
         });
       },
   
       requestAnimationFrame(callback) {
         // 防止等待太久没有执行回调
         // 设置最大等待时间
         // setTimeout(() => {
         //   if (this.isInit) return
         //   callback()
         // }, this.maxWaitingTime)
   
         // 兼容不支持requestAnimationFrame 的浏览器
         return (callbackto => setTimeout(callbackto, 300))(callback);
       }
     }
   };
   </script>
   ```

#### 3.新建骨架屏组件

   骨架屏组件并没有去获取获取不同子组件的dom节点生成，只是为了和参考的懒加载插件的逻辑保持一致，子组件是异步加载，所以在子组件加载前，父组件上有`slot="skeleton"`的组件会先执行，为子组件在加载前在页面上占据位置。

#### 4.异步引入组件

1. 两种语法形式

   

   ```js
   // 两种语法形式：
       1 component: () => import('/component_url/');
       2 component: (resolve) => {
           require(['/component_url/'],resolve)
         }
   ```

2. 页面中路由配置

   

   ```js
   // xxx.vue
   
   export default {
       name: 'xxx',
       metaInfo: {
         title: 'xxx',
       },
       components: {
         'mobile-header-container': MobileHeaderContainer,
         'vue-lazy-component': VueLazyComponent,
         'mobile-skeleton': MobileSkeleton,
        xxx: () => import('components/xxx'),
         xxxP2: () => import('components/mobile/xxx'),
         ...
         MobileFooterContainer: () => import('components/mobile/common/FooterContainer'),
       },
     };
   </script>
   ```

#### 5.页面中使用

1. 项目中使用

   

   ```vue
   <!-- xxx.vue -->
   
   <vue-lazy-component>
      <template slot-scope="scope">
            <!-- 真实组件-->
         <mobile-xxx v-if="scope.loading"/>
      </template>
        <!-- 骨架组件,在真实组件渲染完毕后消失 -->
      <mobile-skeleton slot="skeleton"/>
   </vue-lazy-component>
   ```

   - 通过 `vue-lazy-component`标签的包裹，先预加载`mobile-skeleton`页面预留空间，待滑到当前页面时，显示当前子组件。
   - 通过 `slot-scope` 特性从子组件获取数据，`scope.loading=true`时，`<mobile-paceOs-p13/>`组件渲染成功。

2. 使用这个懒加载组件遇到的问题

   - 用懒加载组件包裹的子组件`<mobile-xxx/>`在页面上滑出、进入时,动画未执行.

     

     ```vue
     <!--PaceOs.vue-->
     
     <vue-lazy-component>
       <template slot-scope="scope">
        <mobile-xxx v-if="scope.loading"/>
       </template>
       <mobile-skeleton slot="skeleton"/>
     </vue-lazy-component>
     ```

     

     ```vue
     <!--xxx.vue-->
     
     <template>
       <div class="xxx">
            ...
         <on-scroll-view :config="onScrollViewConfig">
           <div class="p3-dial zIndex1" ref="img1">
             <img src="xxx.png">
           </div>
           ...
         </on-scroll-view>
       </div>
     </template>
     ```

   - 通过打印,发现是xxx.vue文件里,`on-scroll-view`包裹的元素获取的父节点不是最外层父节点,所以在OnScrollView.vue修改:

     

     ```js
     // OnScrollView.vue
     
     // 当前元素头部距离整个页面顶部的距离
     // this.offsetTop = component.offsetTop + component.offsetParent.offsetTop;
     this.offsetTop = component.offsetTop + this.getParentsNode().offsetTop;
     
     // 获取元素包裹的父节点
     getParentsNode() {
       let node = this.$el.offsetParent;
       if (this.targetClass !== '') { 
         // 如果当前元素的父节点的class不存在'lazyload',则一直向上找
         while (node.getAttribute('class').indexOf(this.targetClass) === -1) {
           node = node.offsetParent;
         }
       }
       return node;
     }
     ```

   - 在`on-scroll-view`标签里添加`targetClass="lazyLoad"`

     

     ```vue
     <!--xxx.vue-->
     
         <on-scroll-view :config="onScrollViewConfig" targetClass="lazyLoad">
           ...
         </on-scroll-view>
     </template>
     ```

   - 在`transition-group`标签添加`class="lazyLoad"`

     

     ```vue
     <!-- VueLazyComponent.vue -->
     
     <template>
       <transition-group :tag="tagName" name="lazy-component" style="position: relative;" class="lazyLoad"...>
       </transition-group>
     </template>
     ```

   - 通过添加的`getParentsNode()`方法,能解决动画未执行的问题.