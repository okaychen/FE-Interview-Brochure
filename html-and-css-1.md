---
description: >-
  HTML&CSS篇（第一个版本该篇预计总结常见问题50个左右），HTML&CSS篇很多问题都能体现应届生对于前端基础的掌握程度，是应届生求职时不可忽视的重要一环
---

# HTML&CSS

## Q1：浏览器解析渲染页面过程

HTML解析构建DOM-&gt;CSS解析构建CSSOM树-&gt;根据DOM树和CSSOM树构建render树-&gt;根据render树进行布局渲染render layer-&gt;根据计算的布局信息进行绘制

## Q2：布局渲染的过程发生的回流和重绘区别&减少回流次数的方法

区别：

* 回流指当前窗口发生改变，发生滚动操作，或者元素的位置大小相关属性被更新时会触发布局过程，发生在render树
* 重绘指当前视觉样式属性被更新时触发的绘制过程，发生在渲染层render layer

减少回流次数的方法：

* 1）避免一条一条的修改DOM样式，而是修改className或者style.classText
* 2）对元素进行一个复杂的操作，可以先隐藏它，操作完成后在显示
* 3）在需要经常获取那些引起浏览器回流的属性值时，要缓存到变量中
* 4）不使用table布局，一个小的改动可能就会引起整个table重新布局
* 5）在内存中多次操作节点，完成后在添加到文档中

## Q3：关于transform开启GPU加速渲染

> 分析：页面性能优化有一条，用transform代替top，left来实现动画。那么transform的优势在哪里？如何开启GPU加速渲染？开启GPU硬件加速可能会触发的问题，如何解决？

## Q4：相比top&left，transform的优势在哪里

首先相比定位的top&left来说，transform不会引起整个页面的回流和重绘。其次我们可以通过transform开启GPU硬件加速，提高渲染速度，但相应的transform也会占用更多的内存。

## Q5：transform如何开启GPU硬件加速

```css
.box{
    transform:translateZ(0);
    //或者
    transfor:translate3d(0,0,0);
}
```

## Q6：开启GPU硬件加速可能会触发哪些问题，如何处理

可能会导致浏览器频繁闪烁或者抖动，解决方案：

```css
.box{
    backface-visibility: hidden;
    perspective: 1000;
    -webkit-backface-visibility: hidden;
    -webkit-perspective: 1000;
}
```

## Q7：Doctype是什么，三种模式的区别在什么地方

Doctype是一种DTD文档定义类型，必须声明在HTML文档的第一行，用来规范文档使用哪种方式解析HTML，三种模式分别是怪异模式，标准模式，近乎模式\(IE8的一种近乎于前两者之间的一种模式\)；标准模式按照HTML和CSS定义渲染，怪异模式会模拟更旧的浏览器行为

## Q8：对于两种盒模型的理解

标准盒模型和IE怪异盒模型，标准盒模型下：盒子总宽度/高度=width/height+padding+border+margin [![](https://camo.githubusercontent.com/11f03b0d6118eea403a33fe2d64205ad66256e3f/687474703a2f2f7777772e6368656e7161712e636f6d2f6173736574732f696d616765732f626f782d6d6f64656c312e706e67)](https://camo.githubusercontent.com/11f03b0d6118eea403a33fe2d64205ad66256e3f/687474703a2f2f7777772e6368656e7161712e636f6d2f6173736574732f696d616765732f626f782d6d6f64656c312e706e67)

怪异盒模型，IE5.X 和 6 在怪异模式中使用自己的非标准模型，盒子的总宽度和高度是包含内边距padding和边框border宽度在内的：盒子总宽度/高度=width/height + margin = width/height + margin; [![](https://camo.githubusercontent.com/fe676305e5f72b3e682664a683697eac68e0eb37/687474703a2f2f7777772e6368656e7161712e636f6d2f6173736574732f696d616765732f626f782d6d6f64656c322e706e67)](https://camo.githubusercontent.com/fe676305e5f72b3e682664a683697eac68e0eb37/687474703a2f2f7777772e6368656e7161712e636f6d2f6173736574732f696d616765732f626f782d6d6f64656c322e706e67)

```css
box-sizing : content-box || border-box || inherit;
```

boxsizing属性content-box使用标准盒模型的计算方式，border-box则使用怪异盒模型的计算方式

## Q9：关于IFC和BFC，哪些元素会触发BFC

BFC块级格式化上下文，IFC行级格式化上下文，

哪些元素会触发BFC：

* 根元素
* float的属性不为none
* position属性为absolute或fixed
* display为inline-block，table-cell，table-caption，flex
* overflow不为visible

## Q10：CSS定位的方式有哪些分别相对于谁

```css
static(默认值)
absolute(绝对定位)
fixed(固定定位，相对于窗口)
relative(相对定位，相对于自身)
sticky(2017年浏览器开始支持，粘性定位)
```

## Q11：移动端布局的解决方案

* 使用Flexbox
* 百分比布局结合媒体查询
* 使用rem

rem转换像素大小（根元素的大小乘以rem值），取决与页面根元素的字体大小，即HTML元素的字体大小

em转换像素大小（em值乘以使用em单位的元素的字体大小），比如一个div的字体大小为16px，那么10em就是180px（或者接近它）

{% hint style="info" %}
rem平时怎么做的转换：为了方便计算，时常将html的字体大小设置为62.5%，那么12px就会是1.2rem
{% endhint %}

## Q12：垂直水平居中的多种解决方案

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

## Q13：圣杯布局和双飞翼布局



