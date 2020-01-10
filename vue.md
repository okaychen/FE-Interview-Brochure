# Vue框架篇

框架对于应届生也是较重点，除了基础的知识和部分深度问题，还要看对于技术的广度，是否有一个自己完整的技术栈，以及对于自身技术栈的认知程度

## 对MVVM的理解

MVC的弊端：MVC即"Model-View-Controller"，当视图上发生变化，通过Controller（控件）将响应传入到Model（数据源），由数据源改变View上面的数据，允许view和model直接通信，随着业务不断庞大，会出现向蜘蛛网一样难以处理的依赖关系，违背了开发应该遵循的"开放封闭原则"

MVVM，萌芽于2005年微软推出的基于 Windows 的⽤户界⾯框架WPF ，前端最早的 MVVM 框架 knockout 在2010年发布

![&#x56FE;&#x7247;&#x6765;&#x6E90;&#x4E8E;&#x7F51;&#x7EDC;](https://cdn.jsdelivr.net/gh/okaychen/CDN@2.2/brochure/image/mvvm.png)

即"Model-View-ViewModel"，View通过View-Model的DOM Listeners将事件绑定到Model上，而Model则通过Data Bindings来管理View中的数据，View-Model从中起到一个连接桥的作用

## vue双向数据绑定的实现

vue内部利用Object.defineProperty监听数据变化，使数据具有可观测性，结合发布订阅模式，在数据发生变化时更新视图

* 利用Proxy或Object.defineProperty生成的Observer针对对象/对象的属性进行"劫持",在属性发生变化后通知订阅者
* 解析器Compile解析模板中的Directive\(指令\)，收集指令所依赖的方法和数据,等待数据变化然后进行渲染
* Watcher属于Observer和Compile桥梁,它将接收到的Observer产生的数据变化,并根据Compile提供的指令进行视图渲染,使得数据变化促使视图变化

```javascript
// 简单的双向数据绑定
const data = {};
Object.keys(data).forEach(function(key) {
    Object.defineProperty(data, key, {
        enumerable: true,
        configurable: true,
        get: function() {
            console.log('get');
        },
        set: function(newVal) {
            // 当属性值发⽣变化时我们可以进⾏额外操作
        },
    });
});
```

## Object.defineProperty和proxy的优劣区别

Object.defineProperty兼容性较好，但不能直接监听数组的变化，只能监听对象的属性\(有时需要深层遍历\)

与之相比proxy的优点：

* 可以直接监听数组的变化
* 可以直接监听对象而非属性
* proxy有多达13种的拦截方法，不限于apply、ownKeys、deleteProperty、has等等
* proxy受到各大浏览器厂商的重视

## 除了数据劫持，vue为什么还需要虚拟DOM进行diff检测差异

现代前端框架主要有两种监听数据的方式：一种是pull的方式，一种是push的方式

pull，其代表为react，react和vue基于双向数据绑定的依赖收集的订阅式机制不同，react是通过显式的触发函数调用来更新视图，比如setState，然后React会一层层的进行Virtual Dom Diff操作找出差异，通过较暴力diff的方式查找哪里发生变化。另一个代表是angular的脏值检测

push，其代表为vue，当Vue程序初始化的时候就会对数据data进行依赖的收集，一但数据发生变化，响应式系统就会立刻得知；我们知道绑定一个数据通常就需要一个watcher，那么一旦细粒度过高会产生大量的watcher，会给增加内存以及依赖追踪的开销，而细粒度过低会无法精准检测变化，因此vue选择中细粒度方案，在组件级进行push检测的方式\(即依赖响应式系统\)，在组件内部进行Virtual Dom Diff获取更加具体的差异，所以vue采用了push+pull结合的方式

## 对vue响应式系统的理解

![vue&#x54CD;&#x5E94;&#x7CFB;&#x7EDF;](https://cdn.jsdelivr.net/gh/okaychen/CDN@2.2/brochure/image/data.png)

响应式系统简述：

* 任何⼀个 Vue Component 都有⼀个与之对应的 Watcher 实例
* Vue 的 data 上的属性会被添加 getter 和 setter 属性
* 当Vue Component render函数被执⾏的时候，data上会被触碰\(touch\)， 即被读，getter ⽅法会被调⽤， 此时 Vue 会去记录此 Vue component所依赖的所有data\(这⼀过程被称为依赖收集\)
* data 被改动时\(主要是⽤户操作\)， setter ⽅法会被调⽤， 此时 Vue 会去通知所有依赖于此 data 的组件去调⽤他们的 render 函数进⾏更新

## vue的生命周期讲一下

vue 实例有⼀个完整的⽣命周期，也就是从开始创建、初始化数据、编译模版、挂载Dom -&gt; 渲染、更新 -&gt; 渲染、卸载等⼀系列过程，即vue实例从创建到销毁的过程，我们称这是vue的⽣命周期

![img](https://cdn.jsdelivr.net/gh/okaychen/CDN@2.2/brochure/image/lifecycle.png)

1.beforeCreate：完成实例初始化，初始化非响应式变量

2.created：实例初始化完成\(未挂载DOM\)

3.berofeMount：找到对应的template，并编译成render函数

4.mounted：完成创建vm.$el和双向绑定，完成DOM挂载

5.beforeUpdate：数据更新之前\(可在更新前访问现有的DOM\)

6.updated：完成虚拟DOM的重新渲染和打补丁

7.activated：子组件需要在每次加载时候进行某些操作，可以使用activated钩子触发

8.deactivated：keep-alive 组件被移除时使用

9.beforeDestroy：可做一些删除提示，销毁定时器，解绑全局时间 销毁插件对象

10.destroyed：当前组件已被销毁

## computed和watch有什么区别

当我们要进⾏数值计算,⽽且依赖于其他数据，我们需要使用computed

如果你需要在某个数据变化时做⼀些事情，使⽤watch来观察这个数据

computed：

* 1.是计算值，
* 2.应用：就是简化tempalte里面计算和处理props或$emit的传值
* 3.具有缓存性，页面重新渲染值不变化,计算属性会立即返回之前的计算结果，而不必再次执行函数

watch：

* 1.是观察的动作，
* 2.应用：监听props，$emit或本组件的值执行异步操作
* 3.无缓存性，页面重新渲染时值不变化也会执行

## vue指令有哪些，v-if和v-for能不能一起使用

v-html，v-text，v-show，v-for，v-if v-else-if v-else，

v-bind（用来动态的绑定一个或者多个特性）

v-model（创建双向数据绑定）

v-cloak（保持在元素上直到关联实例结束时进行编译）

v-pre（用来跳过这个元素和它的子元素编译过程）

> v-if和v-for能不能一起使用\(或者问v-for和v-if谁的优先级更高\)：
>
> v-for指令的优先级要高于v-if，当处于同一节点时候，意味着v-if将分别重复运行于每个 v-for 循环中，所以应该尽量避免v-for和v-if在同一结点

## vue中的key值有什么用\(diff算法的过程\)

vue采用“就地复用”策略，如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序， 而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素，key 的作用主要是为了高效的更新虚拟DOM。

> 虚拟dom的diff算法的过程中，先会进⾏新旧节点的⾸尾交叉对⽐，当⽆法匹配的时候会⽤新节点的 key 与旧节点进⾏⽐对，然后超出差异

![img](https://cdn.jsdelivr.net/gh/okaychen/CDN@2.2/brochure/image/webp)

vue的diff位于patch.js中，这里简单总结一下patchVnode比较的的过程，首先要判断vnode和oldVnode是否都存在，都存在并且vnode和oldVnode是同一节点时，才会进入patchVnode进行比较，结点比较五种情况：

① 引用一致，可以认为没有变化

② 文本节点的比较，如果需要修改：则会调用Node.textContent = vnode.text

③ 两个节点都有子节点，而且它们不一样：则调用updateChildren函数比较子节点

④ 只有新的节点有子节点：则调用addVnodes创建子节点

⑤ 只有老节点有子节点，则调用removeVnodes把这些子节点都删除

updateChildren的过程：updateChildren用指针的方式把新旧节点的子节点的首尾节点标记，即oldStartIndex\(1\)，oldEndIndex\(2\)，newStartIndex\(3\), oldEndIndex\(4\)（这里简单用1 2 3 4顺序标记）即依次比较13，14，23，24，有10种左右情况分别做出对应的处理

## vue的路由实现

更新视图但不重新请求页面，是前端路由原理的核心，目前在浏览器环境主要有两种方式：

* Hash 模式

hash\("\#"\)符号的本来作用是加在URL指示网页中的位置：

```markup
http://www.example.com/index.html#print
```

`#`本身以及它后面的字符称之为hash，可通过window.location.hash属性读取，hash虽然在url中，但是却不会被包含在http请求中，也不会重新加载页面，它用来指导浏览器动作

* History 模式

History interface是浏览器历史记录栈提供的接口，从HTML5开始，History interface提供了2个新的方法：`pushState()`，`replaceState()`使得我们可以对浏览器历史记录栈进行修改；这两个方法有有一个特点，当调用他们修改浏览器历史栈后，虽然当前url改变了，但浏览器不会立即发送请求该url，这就为单页应用前端路由，更新视图但不重新请求页面提供了基础

## vue组件间通信的方式

* props/$emit+v-on

  父组件通过props的方式向子组件传递数据，而通过$emit 子组件可以向父组件通信

* eventBus

  通过eventBus向中心事件发送或者接收事件，所有事件都可以共用事件中心

* vuex

  状态管理模式，采用集中式存储管理应用的所有组件的状态，可以通过vuex管理全局的数据

> 参考：[vue中8种组件通信方式](https://juejin.im/post/5d267dcdf265da1b957081a3) 只需要熟悉上面三种即可，其他基本不会用到，可以做为了解

## vue路由懒加载的方式有哪些

懒加载简单来说就是延迟加载或按需加载，即在需要的时候的时候进行加载，常用的懒加载方式有三种：即使用vue异步组件 和 ES6中的import，以及webpack的require.ensure\(\)

* vue异步组件 

  ```javascript
  // 路由配置，使用vue异步组件
  {
  	path: '/home',
      name: 'home',
      component: resolve => require(['@/components/home'],resolve)
  }
  ```

* ES6中的import

  ```javascript
  // 指定了相同的webpackChunkName，合并打包成一个js文件
  // 如果不指定，则分开打包
  const Home = () => import(/*webpackChunkName:'ImportFuncDemo'*/ '@/components/home')
  const Index = () => import(/*webpackChunkName:'ImportFuncDemo'*/ '@/components/index')
  ```

* webpack推出的require.ensure\(\)

  ```javascript
  {
  	path: '/home',
   	name: 'home',
      component: r => require.ensure([], () => r(require('@/components/home')), 'demo')
  }
  ```

