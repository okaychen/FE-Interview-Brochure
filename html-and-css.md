---
description: >-
  HTML&CSS篇（第一个版本该篇预计总结常见问题20个左右），HTML&CSS不少问题都能体现应届生对于前端基础的掌握程度，是应届生求职时不可忽视的重要一环
---

# HTML/CSS

> HTML&CSS篇（第一个版本该篇预计总结常见问题20个左右），应届生前端面试较重基础，HTML&CSS不少问题都能体现应届生对于前端基础的掌握程度，是应届生求职时不可忽视的重要一环

## 浏览器解析渲染页面过程

大致过程：

HTML解析构建DOM-&gt;CSS解析构建CSSOM树-&gt;根据DOM树和CSSOM树构建render树-&gt;根据render树进行布局渲染render layer-&gt;根据计算的布局信息进行绘制

不同浏览器的内核不同，所以渲染过程其中有部分细节有不一样，以webkit主流程为例：

![](https://cdn.jsdelivr.net/gh/okaychen/CDN@2.0/brochure/image/browser.jpg)

一篇很棒的文章\(需科学上网\)：[How Browser Work](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)

> 编者有话说：浏览器解析渲染页面过程是一个复杂的过程，其中有不少的细节和规则，如果把上面分享的文章翻译成译文，至少有3～5页PDF左右，所以这里只能总结大致过程（作为面试回答【很可能让回答的尽可能详细】了解来说已经足够，更深入的了解可以好好读下上面那篇文章）

较详细过程：

HTML解析构建DOM树：其中HTML Parser就起到了将HTML标记解析成DOM Tree的作用，HTML Parser将文本的HTML文档，提炼出关键信息，嵌套层级的树形结构，便于计算拓展；这其中也有很多的规则和操作，比如容错机制，识别特殊标签`<br></br>`等

CSS解析构建CSSOM树：CSS Parser将很多个CSS文件中的样式合并解析出具有树形结构Style Rules，也叫做CSSOM。

> ※其中还有一个细节是浏览器解析文档：当遇到`<script>`标签的时候会停止解析文档，立即解析脚本，将脚本中改变DOM和CSS的地方分别解析出来，追加到DOM Tree和CSSOM上

根据DOM树和CSSOM树构建Render树：Render Tree的构建其实就是DOM Tree和CSSOM Attach的过程，在webkit中，解析样式和创建呈现器的过程称为"附加"，每个DOM节点都有一个"attach"方法，Render Tree其实就相当于一个计算好样式，与HTML对应的Tree

根据Render树进行布局渲染render layer：创建渲染树后，Layout根据根据渲染树中渲染对象的信息，计算好每一个渲染对象的位置和尺寸，将其放在浏览器窗口的正确位置，某些时候会在文档布局完成之后进行DOM修改，重新布局的过程就称为回流

> ※其中计算（样式计算）一个复杂的过程，因为DOM中的一个元素可以对应样式表中的多个元素，Firefox采用了规则树和样式上下文树来简化样式计算，规则树包含了所有已知规则的匹配路径，样式上下文包含端值，webkit也有样式对象，但它们不保存在类似上下文树这样的结构中，只是由DOM节点指向此类对象的相关样式

根据计算的布局信息进行绘制：绘制阶段则会遍历呈现树，并调用呈现器的paint方法，将呈现器的内容显示在屏幕上，绘制的顺序其实就是元素进入堆栈样式上下文的顺序，例如，块呈现器的堆栈顺序如下：1.背景颜色，2.背景图片，3.边框，4.子代，5.轮廓

## 回流和重绘区别&减少回流重绘次数的方法

区别：

* 回流指当前窗口发生改变，发生滚动操作，或者元素的位置大小相关属性被更新时会触发布局过程，发生在render树，比如元素的几何尺寸变化，就需要重新验证并计算Render Tree
* 重绘指当前视觉样式属性被更新时触发的绘制过程，发生在渲染层render layer
* 所以相比之下，回流的成本要比重绘高得多

减少回流重绘次数的方法：

* 1）避免一条一条的修改DOM样式，而是修改className或者style.classText
* 2）对元素进行一个复杂的操作，可以先隐藏它，操作完成后在显示
* 3）在需要经常获取那些引起浏览器回流的属性值时，要缓存到变量中
* 4）不使用table布局，一个小的改动可能就会引起整个table重新布局
* 5）在内存中多次操作节点，完成后在添加到文档中

## 关于transform开启GPU加速渲染，相比top&left，优势在哪里

> 页面性能优化有一条，用transform代替top，left来实现动画。那么transform的优势在哪里？如何开启GPU加速渲染？开启GPU硬件加速可能会触发的问题，如何解决？

首先相比定位的top&left来说，transform不会引起整个页面的回流和重绘。其次我们可以通过transform开启GPU硬件加速，提高渲染速度，但相应的transform也会占用更多的内存。

## transform如何开启GPU硬件加速

```css
.box{
    transform:translateZ(0);
    //或者
    transfor:translate3d(0,0,0);
}
```

## 开启GPU硬件加速可能会触发哪些问题，如何处理

可能会导致浏览器频繁闪烁或者抖动，解决方案：

```css
.box{
    backface-visibility: hidden;
    perspective: 1000;
    -webkit-backface-visibility: hidden;
    -webkit-perspective: 1000;
}
```

## 移动端点透现象有遇到过嘛

首先需要了解的是，移动端在touch上一共有4个事件，

执行顺序为`touchstart -> touchmove -> touchend -> touchcancel`

当用户点击屏幕时，会触发touch和click事件，touch事件会优先处理，touch事件经过捕获，目标，冒泡一系列流程处理完成之后，才会触发click，所有我们经常会谈到移动端点击事件300ms延迟的问题

移动端点击事件300ms问题，常见的解决方案：

* 阻止用户双击缩放，并限制视口大小

```css
<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no"/>
```

* 设置css`touch-action`用于指定某个给定的区域是否允许用户操作，以及如何相应用户操作

```css
* {
  touch-action: none;
}
```

* fastclick.js来解决，其原理是在检测到touchend事件的时候，会通过自定义事件立即触发模拟一个click事件，并在300ms之后把真正的click事件阻止掉

点透现象：

发生条件：①按钮A和按钮B不是后代继承关系，②A发生touch，A touch后立即消失，B绑定click，③A z-index大于B，即 A 显示在 B 浮层之上

发生原因：当点击屏幕时，系统生成touch和click两个事件，touch先执行，touch执行完之后A消失，然后要执行click的时候，就会发现用户点击的是B，所以就执行了B的click

解决方法：①阻止默认事件，在touch的某个时间段执行event.preventDefault，去取消系统生成的click事件，一半在 touchend 中执行。②要消失的元素延迟300ms后在消失

## Doctype是什么，三种模式的区别在什么地方

Doctype是一种DTD文档定义类型，必须声明在HTML文档的第一行，用来规范文档使用哪种方式解析HTML，三种模式分别是怪异模式，标准模式，近乎模式\(IE8的一种近乎于前两者之间的一种模式\)；标准模式按照HTML和CSS定义渲染，怪异模式会模拟更旧的浏览器行为

## 说说对两种盒模型的理解

标准盒模型和IE怪异盒模型，标准盒模型下：盒子总宽度/高度=width/height+padding+border+margin 

![](https://cdn.jsdelivr.net/gh/okaychen/CDN@2.0/brochure/image/box_model01.png)

怪异盒模型，IE5.X 和 6 在怪异模式中使用自己的非标准模型，盒子的总宽度和高度是包含内边距padding和边框border宽度在内的：盒子总宽度/高度=width/height + margin = width/height + margin;

![](https://cdn.jsdelivr.net/gh/okaychen/CDN@2.0/brochure/image/box_model02.png)

```css
box-sizing : content-box || border-box || inherit;
```

boxsizing属性content-box使用标准盒模型的计算方式，border-box则使用怪异盒模型的计算方式

## margin塌陷和合并问题

首先，margin塌陷是相对于父子级关系的两个元素，而margin合并是相对两个兄弟级关系的两个元素

两个兄弟级关系的元素，垂直方向上的margin，其外边距会发生重叠现象，两者两个的外边距取的是两个所设置margin的最大值，就是所说的margin合并问题

两个父子级关系的元素，垂直方向上的margin会粘合在一起，外层和模型的margin-top取两个元素中margin-top的最大值，发生margin塌陷的内层元素相对于整个文档移动

解决方案：两者都可以通过触发BFC来解决

## 关于IFC和BFC，哪些元素会触发BFC

首先需要知道文档流分为定位流，浮动流和普通流三种，而普通流则是BFC中的FC（格式化上下文），它是页面的一块渲染区域，有一套渲染规则，决定了子元素如何布局，以及和其他元素之间的关系和作用，常见的FC有BFC，IFC，还有GFC和FFC

BFC块级格式化上下文，IFC行级格式化上下文，

哪些元素会触发BFC：

* 根元素
* float的属性不为none
* position属性为absolute或fixed
* display为inline-block，table-cell，table-caption，flex
* overflow不为visible

## CSS定位的方式有哪些分别相对于谁

```css
static(默认值)
absolute(绝对定位，相对于最近已定位的父元素，如果没有则相对于<html>)
fixed(固定定位，相对于窗口)
relative(相对定位，相对于自身)
sticky(2017年浏览器开始支持，粘性定位)
```

absolute会使元素位置与文档流无关，不占据空间，absolute 定位的元素和其他元素重叠

relative相对定位时，无论元素是否移动，仍然占据原来的空间

sticky是2017年浏览器才开始支持，会产生动态效果，类似relative和fixed的结合，一个实例是"[动态固定](http://www.ruanyifeng.com/blog/2019/11/css-position.html)"，生效前提是必须搭配`top,left,bottom,right`一起使用，不能省略，否则等同于`relative`定位，不产生"动态固定"的效果

## 移动端布局的解决方案，平时怎么做的处理

* 使用Flexbox
* 百分比布局结合媒体查询
* 使用rem

rem转换像素大小（根元素的大小乘以rem值），取决与页面根元素的字体大小，即HTML元素的字体大小

em转换像素大小（em值乘以使用em单位的元素的字体大小），比如一个div的字体大小为16px，那么10em就是180px（或者接近它）

{% hint style="info" %}
rem平时怎么做的转换：为了方便计算，时常将html的字体大小设置为62.5%，那么12px就会是1.2rem
{% endhint %}

## 垂直水平居中的多种解决方案

### 未知宽高元素实现垂直水平居中

① flex实现水平垂直居中

```css
.parent {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 600px;
    height: 600px;
    margin: auto;
    border: 1px solid yellow;
}

.child {
    width: 100px;
    height: 100px;
    border: 1px solid blue;
}
```

② 进行垂直水平居中（利用transform中translate偏移的百分比值是相对于自身大小的特性）

```css
.parent {
      position: relative;
      width: 600px;
      height: 600px;
      margin: auto;
      border: 1px solid yellow;
    }

.child {
      position: absolute;
      width: 100px;
      height: 100px;
      border: 1px solid blue;
    }

.method3 {
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
```

### 已知宽高元素的垂直水平居中

① 绝对定位+`margin:auto`

```css
.parent {
  position: relative;
  width: 600px;
  height: 600px;
  margin: auto;
  border: 1px solid red;
}

.child {
  position: absolute;
  margin: auto;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100px;
  height: 100px;
  border: 1px solid blue;
}
```

② 使用绝对定位与负边距

```css
.parent {
      position: relative;
      width: 600px;
      height: 600px;
      margin: auto;
      border: 1px solid red;
    }

.child {
      position: absolute;
      top: 50%;
      left: 50%;
      margin: -50px 0 0 -50px;
      width: 100px;
      height: 100px;
      border: 1px solid blue;
}
```

## 经典的三栏布局的实现方式

三栏布局：两侧定宽中间自适应，在开发中很常见的布局方式，主要有下面六种实现方式：

* 最简单的方式就是通过float实现
* 利用绝对定位
* 利用BFC规则
* 圣杯布局（通过margin负值调整）
* 双飞翼布局（和双飞翼布局类似，只是处理中间栏被遮挡的方式不同：双飞翼布局在中间栏内部加一个div，设置其margin来避开遮挡）
* 利用flex布局 👇 _Q14_ 

## 对flex布局的了解，flex如何实现经典的三栏布局

flex意为弹性布局，有两大概念，一是容器\(container\)，二是项目\(item\)，两者都有各自的六个常用属性

用在容器上的六个属性：

* `flex-direction`主轴的方向，
* `flex-wrap`一条轴线装满，如何换行，
* `flex-flow`是前两者的简写，
* `justify-content`项目在主轴上的对齐方式
* `align-items`在交叉轴上如何对齐
* `align-content`多跟轴线的对齐方式

用在项目上的六个属性:

* `order`排序顺序，默认为0，越小越靠前
* `flex-grow`项目的放大比例，默认为0，不放大
* `flex-shrink`项目的缩小比例，默认为1，空间不足该项目会缩小
* `flex-basis`项目占据主轴空间大小
* `flex`是上面放大，缩小，大小三者的简写，默认为0 1 auto
* `align-self`允许单个项目有不同于其他的对齐方式

```css
<!--flex实现经典的三栏布局-->
<style>
.flex-container{
        display: flex;
        height: 200px;
}
.middle {
        height: 200px;
        background-color: yellowgreen;
    flex-grow: 1;
}
.left,.right {
    height: 200px;
    flex: 0 1 200px;
    background-color: green;
}

</style>

<div class="flex-container">
        <div class="left">左侧内容</div>
        <div class="middle">中间内容</div>
        <div class="right">右侧内容</div>
</div>
```

![](https://cdn.jsdelivr.net/gh/okaychen/CDN@2.0/brochure/image/layout01.png)

## 说说你知道的清除浮动的方法，知道其中的原理嘛

清除浮动主要是为了解决父元素因为子元素浮动内部高度为0的问题

* 额外标签，即在最后一个浮动标签后加一个空div，给其设置`clear:both`，缺点是增加无意义标签，语义化差
* 给父级元素添加`overflow:不为visible`通过触发BFC的方式清除浮动
* 使用after伪元素，缺点是IE6-7不支持伪元素，zoom:1触发hasLayout

```css
.clearfix:after{
    content:"";
    display:block;
    height:0;
    clear:both;
    visibility:hidden;
}
.clearfix{
    *zoom:1; /*ie6清除浮动的方式 *只有IE6-IE7执行*/
}
```

* 使用before和after双伪元素清除浮动

```css
.clearfix:after,.clearfix:before{
    content:'';
    display:table;
}
.clearfix:after{
    clear:both;
}
.clear{
    *zoom:1;
}
```

## 对css sprits的理解，好处是什么

css sprits又名雪碧图，也叫css精灵，开发人员会将很多小图标合并在一起之后的图片成为雪碧图，使用时通过background-image、background-position和background-size属性，将对应的小图标添加元素中

优点是：

* 减少加载多张图片的HTTP请求数
* 可以提前加载资源

缺点是：

* 维护成本较高，需要改动这张合并的图片
* 加载优势在http2之后不复存在，http2采用多路复用，多张图片也可以重复使用一个连接

## 说一下前端的几种存储方式

cookie，localstorage，sessionstorage，IndexedDB

（追问）这些方式的区别是什么，分别介绍一下：

* cookie：HTML5标准前本地存储主要方式，请求头带着数据，缺点是大小只有4k左右，并且自动请求头加入cookie浪费流量，每个domain限制20个cookie，使用起来也麻烦需要自行封装
* localstorage：HTML5标准后加入，以键值对（key-value）方式存储，永久存储，永不失效，除非手动删除，IE8+支持，每个域名限制5M
* sessionstorage：和localstorage操作方法一样，区别在于sessionstorage在关闭页面后即被清空，刷新不会清空，但是不能在所有同源窗口中共享，是会话级别的存储方式
* IndexedDB：被纳入HTML5标准的数据库存储方案，它是一个NoSql数据库，用键值对进行存储，可以快速读取操作，适合web场景和JavaScript操作

