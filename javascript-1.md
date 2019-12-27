---
description: >-
  JavaScript篇（第一个版本预计总结常见问题20个左右），JavaScript是应届求职过程的重中之重，既能体现对于基础的掌握，也会了解到应届生对于一门编程语言的熟练程度，以及是否能够揭开编程语言表层，对底层有一定的了解
---

# JavaScript

## 1：跨域的方式有哪些，为什么需要跨域，同源策略拦截客户端请求还是服务器响应

之所以需要跨域，是因为浏览器同源策略的约束，面对不同源的请求，我们无法完成，这时候就需要用到跨域。同源策略拦截的是跨源请求，原因：CORS缺少`Access-Control-Allow-Origin`头

跨域的方式主要有：`JSONP、proxy代理、CORS、XDR`

## 2：JSONP的原理是什么，缺点是什么，浏览器端如何做

JSONP主要是因为`script`标签的`src`属性不被同源策略所约束，同时在没有阻塞的情况下资源加载到页面后会立即执行

缺点：

* JSONP只支持get请求而不支持post请求，如果想传给后台一个json格式的数据,此时问题就来了,浏览器会报一个http状态码415错误，请求格式不正确
* JSONP本质是一种代码注入，存在安全问题

## 3：JSONP服务器端如何处理

实际项目中JSONP通常用来获取`json`格式的数据，这时候前后端通常约定一个参数`callback`，该参数的值，就是处理返回数据的函数名称

```javascript
var f = function(data){
    alert(data.name)
}

var _script = document.createElement('script');
_script.type = 'text/javascript'
_script.src = 'http://localhost:8888/jsonp?callback=f'
document.head.appendCild(_script);
```

```javascript
//node处理
var query = _scr.query;
var params = qs.parse(query);
var f = '';

f = params.callback;

res.writeHead(200,{'Content-Type','text/javascript'})
res.write(f + "({name:'hello world'})")
res.end
```

```javascript
//php处理(注意输出格式)
$data = array(
    'rand' => $_GET['random'],
    'msg' => 'Success'
);

echo $_GET['callback'].'('.json_encode($data).')';
```

补充：cors是一种现代浏览器支持跨域资源请求的一种方式，它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制

```javascript
//node处理
if(req.headers.origin){
    res.wirteHead(200,{
        'Content-type':'text/html;charset=UTF-8',
        'Access-Control-Allow-Origin':'*', // *通配，或者安全考虑替换为指定域名
    });
    res.write('cors');
    res.end();
}
```

## 4：列表无限滚动曾经有遇到过嘛

简单列表滚动加载是监听滚动条在满足条件的时候触发回调，然后通过把新的元素加入到页面页尾的方法完成，但是如果用户加载过多列表数据\(比如我这一个列表页有一万条数据需要展示\)，那么用户不断加载，页面不断增加新的元素，很容易就导致页面元素过多而造成卡顿，所以就提出的列表的无限滚动加载，主要是在删除原有元素并且维持高度的基础上，生成并加载新的数据

## 4：如果滚动过快怎么办，高频率触发事件解决方案-防抖和节流

节流：在一段时间内不管触发了多少次都只认为触发了一次，等计时结束进行响应（假设设置的时间为2000ms，再触发了事件的2000ms之内，你在多少触发该事件，都不会有任何作用，它只为第一个事件等待2000ms。时间一到，它就会执行了。 ）

```javascript
//时间戳方式
function throttle(fn,delay){
    let pre = Date.now();
    return function(){
        let context = this;
        let args = arguments;
        let now = Date.now();
        if(now - pre > delay){
            fn.apply(context,args);
            pre = Date.now();
        }
    }
}
//定时器方式
function throttle(fn,delay){
    let timer = null;
    return function(){
        let context = this;
        let args = arguments;
        if(!timer){
            timer = setTimeout(function(){
                fn.apply(context,args);
                timer = null;
            },delay)
        }
    }
}
```

防抖：在某段时间内，不管你触发了多少次事件，都只认最后一次（假设设置时间为2000ms，如果触发事件的2000ms之内，你再次触发该事件，就会给新的事件添加新的计时器，之前的事件统统作废。只执行最后一次触发的事件。）

```javascript
//定时器方式
function debounce(fn,delay){
    let timer = null;
    return function(){
        let context = this;
        let args = arguments;
        clearTimeout(timer);
        timer = setTimeout(function(){
            fn.apply(context,args);
        },delay)
    }
}
```

## 5：介绍ES6你熟悉的几个新特性

* let，const声明变量
* 解构赋值
* 箭头函数
* 扩展运算符
* 数组的新方法 -- map，reduce，filter
* promise

## 6：说说var，let，const的区别

首先var相对`let/const`，后者可以使用块级作用域，var声明变量则存在函数作用域内（该域内存在变量提升）。`let/const`有一个暂时性死区的概念，即进入作用域创建变量到变量可以开始访问的一段时间。

其次let，const主要的区别在于：const一旦被赋值就不再"改变"

```javascript
const name = 'Jack';
name = 'Tom'  //TypeError:Assignment to constant
```

但需要理解的是：这并不意味着声明的变量本身不可变，只是说它不可再次被赋值了（const定义引用类型时，只要它的引用地址不发生改变，仍然可以改变它的属性）

```javascript
const person = {
    name : 'Jack‘
}
person.name = 'Tom'
```

## 7：数据类型有哪些，对symbol有了解嘛

基本数据类型：`string`、`number`、`Boolean`、`undefined`、`null`

复杂数据类型：`Object`

ES6新增数据类型：`symbol`、`Map`、`Set`

> * 其中symbol是基本数据类型，每一个symbol都是一个全局唯一的字符串
>
> ```javascript
> // 在一个块级作用域里使用symbol，创造一个隐藏属性
> {
>     let a = Symbol();
>     let object = {
>         name : 'okaychen',
>         age : 3,
>         [a] : '隐藏属性'
>     }
>     window.object = object
> }
> ```
>
> * set是object里面的一种，set里无论原始值还是引用类型的值，重复的都只会保留一个
> * Map可以允许任何类型作为对象的键，弥补了object只能使用字符串作为键（注意Map是ES6中的一种数据结构，区别于数组方法map）

## 8：分别说一下数组中常用的方法

数组也是一种数据类型，类比数据类型的学习我们可以从其特性，增删改查，其他方法，支持的运算符七个方面来学习，可以明显提高效率

数组的方法我们除了作用以外，我们还比较关心的就是该数组方法是否改变原数组，下面就按照这个规则来分类：

```javascript
// 改变原数组的方法：
pop()  // 删除数组中的最后一个元素，把数组长度减 1，并且返回它删除的元素的值
push() // 该方法可把它的参数顺序添加到数组的尾部。它直接修改数组，返回后修改数组的长度
reverse() // 将数组元素倒序，改变原数组
unshift() // 可以向数组开头增加一个或多个元素，并返回新的长度
shift() // 数组的第一个元素从其中删除，并返回第一个元素的值，减少数组的长度
sort() // 在原数组上进行排序，不生成副本
splice(start,删除的个数,插入的元素) //可删除从index处开始的零个或多个元素，并且用参数列表中声明的一个或多个值来替换那些被删除的元素

// 不改变原数组：
concat() // 用于连接两个或多个数组，不改变原数组，返回一个新的数组
join() // 将数组中的所有元素都转化为字符串并拼接在一起，默认使用逗号，返回最终生成的字符串
map() // 对数组的每一项运行给定函数，返回每次函数调用的结果组成的数组，不改变原数组
indexof // 返回指定位置的元素值或字符串，通过搜索值与下标寻找
every() // 如果每一项都为true，则返回true
some() // 某一项返回true，则返回true
forEach() // 对数组的每一项运行给定函数，没有返回值

```

## 8：对数组方法map和reduce方法的理解，区别在哪里

map方法的调用者一般是数组，参数是一个callback函数，返回值是一个由原数组中每个元素执行callback函数得到的返回值组成的新数组

举个栗子：

```javascript
// 接口数据映射
let arr = res.map(item=>{
    return {
        name:item.name,
        age:item.age,
        sex:item.sex === 1? '男':item.sex === 0?'女':'保密',
        avatar: item.img
    }
})
```

reduce方法调用者也一般为数组，参数是callback和一个可选的initialValue，为数组中每个元素执行callback函数，返回一个具体的结果，如果给定initialValue可以作为第一次调用callback的第一个参数，可以控制返回值的格式

举个栗子：

```javascript
// 求一个字符串中每个字母出现的次数
let str = 'abcabdaac'
str.split('').reduce((res,cur)=>{
    res[cur] ? res[cur]++ : res[cur] = 1
    return res
},{})  
// {a: 4, b: 2, c: 2, d: 1}
```

## 9：对map和reduce实现的理解，能手写一下嘛

数组方法map模拟实现：

```javascript
Array.prototype._map = fn => {
    let newArr = [];
    let me = this;
    for(let i = 0;i<me.length;i++){
        newArr.push(fn(me[i]));
    }
    return newArr;
}
```

数组方法reduce模拟实现：方法类似核心是数组的遍历，因为reduce有第二个可选参数initialValue做起始值，所以要判断是否有可选参数作为遍历的起始值，并将得到的参数传入回调函数中执行

```javascript
Array.prototype._reduce = (callback, initialValue) => {
    let arr = this;
    let base = typeof initialValue == 'undefined' ? arr[0] : initialValue;
    let startPoint = typeof initialValue === 'undefined' ? 1 : 0;
    arr.slice(startPoint).forEach((val, index) => {
        base = callback(base, val, index + startPoint, arr)
    })
    return base
}
```

## 10：Ajax的概念，手写一下原生实现的思路

首先需要知道的是Ajax主要是通过`XMLHttpRequest`对象向服务器提出请求并处理响应，进行页面的局部更新，XMLHttpRequest对象常用的三大属性：`onreadystatechange`，`readyState`，`status`

<table>
  <thead>
    <tr>
      <th style="text-align:center">&#x5C5E;&#x6027;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center"><code>onreadystatechange</code>
      </td>
      <td style="text-align:left"><code>readyState</code>&#x5C5E;&#x6027;&#x7684;&#x503C;&#x53D1;&#x751F;&#x6539;&#x53D8;&#xFF0C;&#x5C31;&#x4F1A;&#x89E6;&#x53D1;readystatechange&#x4E8B;&#x4EF6;</td>
    </tr>
    <tr>
      <td style="text-align:center"><code>readyState</code>
      </td>
      <td style="text-align:left">
        <p>&#x5B58;&#x6709;XMLHttpRequest&#x7684;&#x72B6;&#x6001;</p>
        <ul>
          <li>0&#xFF1A;&#x8BF7;&#x6C42;&#x672A;&#x521D;&#x59CB;&#x5316;</li>
          <li>1&#xFF1A;&#x670D;&#x52A1;&#x5668;&#x8FDE;&#x63A5;&#x5DF2;&#x5EFA;&#x7ACB;</li>
          <li>2&#xFF1A;&#x8BF7;&#x6C42;&#x5DF2;&#x63A5;&#x6536;</li>
          <li>3&#xFF1A;&#x8BF7;&#x6C42;&#x5904;&#x7406;&#x4E2D;</li>
          <li>4&#xFF1A;&#x8BF7;&#x6C42;&#x5DF2;&#x5B8C;&#x6210;&#xFF0C;&#x4E14;&#x54CD;&#x5E94;&#x5DF2;&#x5C31;&#x7EEA;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:center"><code>status</code>
      </td>
      <td style="text-align:left">
        <p><code>status</code>&#x5C5E;&#x6027;&#x4E3A;&#x53EA;&#x8BFB;&#x5C5E;&#x6027;&#xFF0C;&#x8868;&#x793A;&#x672C;&#x6B21;&#x8BF7;&#x6C42;&#x6240;&#x5F97;&#x5230;&#x7684;HTTP&#x72B6;&#x6001;&#x7801;</p>
        <ul>
          <li>200, OK&#xFF0C;&#x8BBF;&#x95EE;&#x6B63;&#x5E38;</li>
          <li>301, Moved Permanently&#xFF0C;&#x6C38;&#x4E45;&#x79FB;&#x52A8;</li>
          <li>302, Move temporarily&#xFF0C;&#x6682;&#x65F6;&#x79FB;&#x52A8;</li>
          <li>304, Not Modified&#xFF0C;&#x672A;&#x4FEE;&#x6539;</li>
          <li>307, Temporary Redirect&#xFF0C;&#x6682;&#x65F6;&#x91CD;&#x5B9A;&#x5411;</li>
          <li>401, Unauthorized&#xFF0C;&#x672A;&#x6388;&#x6743;</li>
          <li>403, Forbidden&#xFF0C;&#x7981;&#x6B62;&#x8BBF;&#x95EE;</li>
          <li>404, Not Found&#xFF0C;&#x672A;&#x53D1;&#x73B0;&#x6307;&#x5B9A;&#x7F51;&#x5740;</li>
          <li>500, Internal Server Error&#xFF0C;&#x670D;&#x52A1;&#x5668;&#x53D1;&#x751F;&#x9519;&#x8BEF;</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>```javascript
//Ajax原生简单实现
let xhr = XMLHttpRequest;
    
xhr.onreadystatechange = () => {
    if(xhr.readyState === 4){
        if(xhr.status === 200){
            console.log(xhr.responseText);
        }else{
            console.error(xhr.statusText);
        }
    }
}

xhr.onerror = e => {
    console.error(xhr.statusText);
}

xhr.open('GET','/EndPonint',true);

xhr.send(null);
```

## 11：有了解过fetch嘛，和XMLHttpRequest的区别在哪

XMLHttpRequest历史悠久，因为其API设计其实并不是很好，输入，输出，状态都在同一个接口管理，容易写出非常非常混乱的代码，Fetch API采取了一种新规范，用来取代XMLHttpReques，Fetch更现代化，更接近于未来，内部使用了Promise，用起来也更加简洁

```javascript
fetch('./api/demo.json')
    .then((response) => {
        response.json().then((data) => {
            ...
        });
    });
    .catch((err) => {...});
```

## 12：Promise的了解，手撕Promise\(Promise.all或者Promise.race\)

Promise是一种异步编程的解决方法，相比容易陷入回调地狱的回调函数，采用链式调用的方式更合理也更方便，Promise有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败），接受一个作为函数作为参数，该函数有两个参数，分别是`resolve`和`reject`两个函数

```javascript
// Promise的模拟实现
class _Promise {
    constructor(fn) {
        let _this = this;
        this._queue = [];
        this._success = null;
        this._error = null;
        this.status = '';
        fn((...arg) => {
            // resolve
            if (_this.status != 'error') {
                _this.status = 'success';
                _this._success = arg;
                _this._queue.forEach(json => {
                    json.fn1(...arg)
                })
            }
        }, (...arg) => {
            // reject
            if (_this.status != 'success') {
                _this.status = 'error';
                _this._error = arg;
                _this._queue.forEach(json => {
                    json.fn2(...arg)
                })
            }
        })
    }

    then(fn1, fn2) {
        let _this = this;
        return new _Promise((resolve, reject) => {
            if (_this.status == 'success') {
                resolve(fn1(..._this._success))
            } else if (_this.status == 'error') {
                fn2(..._this._error)
            } else {
                _this._queue.push({fn1,fn2});
            }
        })
    }
}
```

Promise.all和Promise.race在实际应用中的比较，比如从接口中获取数据，等待所有数据到达后执行某些操作可以用前者，如果从几个接口中获取相同的数据哪个接口先到就用哪个可以使用后者

```javascript
//Promise.all的模拟实现(race的实现类似)
Promise.prototype._all = interable => {
    let results = [];
    let promiseCount = 0;
    return new Promise(function (resolve, reject) {
        for (let val of iterable) {
            Promise.resolve(val).then(res => {
                promiseCount++;
                results[i] = res;
                if (promiseCount === interable.length) {
                    return resolve(results);
                }
            }, err => {
                return reject(err);
            })
        }
    })
}
```

## 13：js为什么0.1+0.2不等于0.3

主要是因为JavaScript同样采用IEEE754标准，在64位中存储一个数字的有效数字形式

![](https://upload-images.jianshu.io/upload_images/16299591-8bfb445d26370aa4.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

第0位表示符号位，0表示整数1表示负数，第1~11位存储指数部分，第12~63位存小数部分；

由于二进制的有效数字总是表示为`1.xxx...`这样的形式，尾数部分在规约形式下的第一位默认为1，故存储时第一位省略不写，尾数部分存储有效数字小数点后的xxx...，最长52位，因此，JavaScript提供的有效数字最长为53个二进制位（尾数部分52位+被省略的1位）

由于需要对求和结果规格化\(用有效数字表示\)，右规导致低位丢失，此时需对丢失的低位进行舍入操作，遵循IEEE754舍入规则，会有精度损失

## 14：如何正确判断与使用this，箭头函数有没有自己的this指针

> this有四种绑定规则，默认绑定、隐式绑定、显示绑定、new 绑定，优先级由低到高
>
> 在ECMA内，this 会调用原生方法 `ResolveThisBinding()` 原生方法，该方法使用正在运行的执行上下文的`LexicalEnvironment`确定关键字this的绑定
>
> 可以简单总结为：谁**直接调用产生这个this指针的函数**，this就指向谁

* this在一般模式下指向全局对象；严格模式下 this 默认为undefined 
* 箭头函数没有自己的this指针，它的this绑定取决于外层（函数或全局）作用域）
* call，apply，bind在非箭头函数下修改this值（箭头函数下只传递参数），不管call , bind, apply多少次，函数的this永远由第一次的决定

## 15：对闭包的了解及其应用场景

闭包是指有权访问另外一个函数作用域中的变量的函数.可以理解为\(**能够读取其他函数内部变量的函数**\)

**闭包的作用:** 正常函数执行完毕后,里面声明的变量被垃圾回收处理掉,但是闭包可以让作用域里的变量,在函数执行完之后依旧保持没有被垃圾回收处理掉

具体有以下几个应用场景：

```javascript
//经典案例：斐波那契数列 ：1, 1, 2, 3, 5, 8, 13, …
let count = 0;
const fib = (()=>{
    let arr = [1,1]
    return function(n){
        count++;
        let res = arr[n];
        if(res){
            return res;
        }else{
            arr[n] = fib(n-1) + fib(n-2);
            return arr[n]
        }
    }
})();
```

```javascript
//通过闭包特性，模拟私有变量
const book = (function () {
    var page = 100;
    return function () {
        this.auther = 'okaychen';
        this._page = function () {
            console.log(page);
        }
    }
})();

var a = new book();
a.auther  // "okaychen"
a._page() // 100
a.page    // undefined
```

```javascript
for (var i=1; i<=5; i++) { 
    setTimeout( function timer() {
        console.log(i);
    }, i*1000 );
}
// 经典老问题，输出结果：6 6 6 6 6
// js执行的时候首先会先执行主线程,异步相关的(setTimeout)会存到异步队列里,
// 当主线程执行完毕开始执行异步队列,主线程执行完毕后,此时 i 的值为 6,
// 所以在执行异步队列的时候,打印出来的都是 6
// ---需要对eventloop有一定的认识(见19)---

// 利用闭包来取正确值
for (var i=1; i<=5; i++) { 
    setTimeout( (function(i) {
        return function() {
            console.log(i);
        }
    })(i), i*1000 );
}
```

## 16：对new的过程和原理的理解

JavaScript中new一个对象，我们需要了解主要发生了以下四步：

1.创建一个空的对象 

`let obj = { }`

2.设置新对象的constructor属性为构造函数的名称，设置新对象的**proto**属性指向构造函数的prototype对象

`obj.proto = ClassA.prototype`

3.使用新对象调用函数，函数中的this被指向新实例对象

`ClassA.call(obj); //{}.构造函数()`

4.将初始化完毕的新对象地址，保存到等号左边的变量中

```javascript
// new构造函数的模拟实现
const _new = function(){
    let obj = new Object();
    let _constructor = [].shift.call(arguments);
    
    // 使用中间函数来维护原型关系
    const F = function(){};
    F.prototype = _constructor.prototype;
    obj = new F();
    
    let res = _constructor.apply(obj,arguments);
    return typeof res === 'object' ? res || obj : obj;
}
```







* 问题准备：
* 对new的过程原理的理解
* 值类型和引用类型，在内存中的存放位置
* 对eventloop事件循环机制的理解
* 说一下js的垃圾回机制
* 对函数柯里化有认识嘛
* setTimeout用作倒计时为什么会产生误差



