```html
<!--HTMLl&CSS-->
Q1：浏览器解析渲染页面的过程
    - Q1.1：布局渲染过程发生的回流和重绘的次数
Q2：关于transform开启GPU加速渲染
    Q2.1：相比top&left，transform的优势在哪里
    Q2.2：transform如何开启GPU硬件加速
    Q2.3：开启GPU硬件加速可能会触发哪些问题，如何处理
Q3：Doctype是什么，三种模式有区别在什么地方
Q4：两种盒模型
Q5：关于IFC和BFC
Q6：CSS定位的方式有哪些分别相对于谁
<!--javascript-->
Q1：跨域的方式有哪些？jsonp的原理以及服务端如何处理
    - Q1.1：为什么需要跨域，同源策略拦截客户端请求还是服务器响应
    - Q1.2：跨域的方式有哪些
    - Q1.3：JSONP的原理是什么，缺点是什么，浏览器端如何做
    - Q1.4：JSONP服务器端如何处理
    - 1.5：（扩展）cors
Q2：介绍ES6熟悉的几个新特性
    - Q2.1：var，let，const的区别
Q3：高频率触发事件解决方案-防抖和节流

```

# HTML&CSS篇

## Q1：浏览器解析渲染页面过程

HTML解析构建DOM->CSS解析构建CSSOM树->根据DOM树和CSSOM树构建render树->根据render树进行布局渲染render layer->根据计算的布局信息进行绘制
### Q1.1：布局渲染的过程发生的回流和重绘区别&减少回流次数的方法
区别：
- 回流指当前窗口发生改变，发生滚动操作，或者元素的位置大小相关属性被更新时会触发布局过程，发生在render树
- 重绘指当前视觉样式属性被更新时触发的绘制过程，发生在渲染层render layer

减少回流次数的方法：
- 1）避免一条一条的修改DOM样式，而是修改className或者style.classText
- 2）对元素进行一个复杂的操作，可以先隐藏它，操作完成后在显示
- 3）在需要经常获取那些引起浏览器回流的属性值时，要缓存到变量中
- 4）不使用table布局，一个小的改动可能就会引起整个table重新布局
- 5）在内存中多次操作节点，完成后在添加到文档中

## Q2：关于transform开启GPU加速渲染

>分析：页面性能优化有一条，用transform代替top，left来实现动画。那么transform的优势在哪里？如何开启GPU加速渲染？开启GPU硬件加速可能会触发的问题，如何解决？

### Q2.1：相比top&left，transform的优势在哪里
首先相比定位的top&left来说，transform不会引起整个页面的回流和重绘。其次我们可以通过transform开启GPU硬件加速，提高渲染速度，但相应的transform也会占用更多的内存。
### Q2.2：transform如何开启GPU硬件加速
```css
.box{
    transform:translateZ(0);
    //或者
    transfor:translate3d(0,0,0);
}
```
### Q2.3：开启GPU硬件加速可能会触发哪些问题，如何处理
可能会导致浏览器频繁闪烁或者抖动，解决方案：
```css
.box{
    backface-visibility: hidden;
    perspective: 1000;
    -webkit-backface-visibility: hidden;
    -webkit-perspective: 1000;
}
```

### Q3：Doctype是什么，三种模式的区别在什么地方
Doctype是一种DTD文档定义类型，必须声明在HTML文档的第一行，用来规范文档使用哪种方式解析HTML，三种模式分别是怪异模式，标准模式，近乎模式(IE8的一种近乎于前两者之间的一种模式)；标准模式按照HTML和CSS定义渲染，怪异模式会模拟更旧的浏览器行为

### Q4：两种盒模型
标准盒模型和IE怪异盒模型，标准盒模型下：盒子总宽度/高度=width/height+padding+border+margin
![](http://www.chenqaq.com/assets/images/box-model1.png)

怪异盒模型，IE5.X 和 6 在怪异模式中使用自己的非标准模型，盒子的总宽度和高度是包含内边距padding和边框border宽度在内的：盒子总宽度/高度=width/height + margin = width/height + margin;
![](http://www.chenqaq.com/assets/images/box-model2.png)
```css
box-sizing : content-box || border-box || inherit;
```
boxsizing属性content-box使用标准盒模型的计算方式，border-box则使用怪异盒模型的计算方式

### Q5：关于IFC和BFC，哪些元素会触发BFC
BFC块级格式化上下文，IFC行级格式化上下文，

哪些元素会触发BFC：
- 根元素
- float的属性不为none
- position属性为absolute或fixed
- display为inline-block，table-cell，table-caption，flex
- overflow不为visible




## Q6：CSS定位的方式有哪些分别相对于谁
```
static(默认值)
absolute(绝对定位)
fixed(固定定位，相对于窗口)
relative(相对定位，相对于自身)
```
