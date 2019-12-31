---
description: >-
  JavaScript篇（第一个版本预计总结常见问题25个左右），JavaScript是应届求职过程的重中之重，既能体现对于基础的掌握，也会了解到应届生对于一门编程语言的熟练程度，以及是否能够揭开编程语言表层，对底层有一定的了解。面试官一般都会根据回答进行追问，所以小册总结上下几个问题，一般具有连贯性
---

# JavaScript

## 跨域的方式有哪些，为什么需要跨域，同源策略拦截客户端请求还是服务器响应

之所以需要跨域，是因为浏览器同源策略的约束，面对不同源的请求，我们无法完成，这时候就需要用到跨域。同源策略拦截的是跨源请求，原因：CORS缺少`Access-Control-Allow-Origin`头

跨域的方式主要有：`JSONP、proxy代理、CORS、XDR`

## JSONP的原理是什么，缺点是什么，浏览器端如何做

JSONP主要是因为`script`标签的`src`属性不被同源策略所约束，同时在没有阻塞的情况下资源加载到页面后会立即执行

缺点：

* JSONP只支持get请求而不支持post请求，如果想传给后台一个json格式的数据,此时问题就来了,浏览器会报一个http状态码415错误，请求格式不正确
* JSONP本质是一种代码注入，存在安全问题

## JSONP服务器端如何处理

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

## 列表无限滚动曾经有遇到过嘛

简单列表滚动加载是监听滚动条在满足条件的时候触发回调，然后通过把新的元素加入到页面页尾的方法完成，但是如果用户加载过多列表数据\(比如我这一个列表页有一万条数据需要展示\)，那么用户不断加载，页面不断增加新的元素，很容易就导致页面元素过多而造成卡顿，所以就提出的列表的无限滚动加载，主要是在删除原有元素并且维持高度的基础上，生成并加载新的数据

## 如果滚动过快怎么办，高频率触发事件解决方案-防抖和节流

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

## 介绍ES6你熟悉的几个新特性

* let，const声明变量
* 解构赋值
* 箭头函数
* 扩展运算符
* 数组的新方法 -- map，reduce，filter
* promise

## 说说var，let，const的区别

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

## 解释一下变量提升

JavaScript引擎会先进行预解析，获取所有变量的声明复制undefined，然后逐行执行，这导致所有被声明的变量，都会被提升到代码的头部（被提升的只有变量，值不会被提升），也就是变量提升（hoisting）

```javascript
console.log(a) // undefined

var a = 1
function b(){
    console.log(a)
}
b() // 1
```

预解析阶段会先获的变量a赋值undefined，并将`var a = undefined放在最顶端`，此时a = 1还在原来的位置，实际就是：

```javascript
var a = undefined
console.log(a) // undefined

a = 1
function b(){
    console.log(a)
}

b() // 1
```

然后会是执行阶段，逐行执行造成了打印出a是undefined



## 数据类型有哪些，对symbol有了解嘛

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

## 值类型和引用类型有哪些区别

1. 值类型： 字符串`string`，数值`number`，布尔值`boolean`，`null`, `undefined`
2. 引用类型： 对象 `Object`，数组`Array`，函数`Function`

值类型：

* 占用空间固定，保存在`栈`中：当一个方法执行时，每个方法都会建立自己的**内存栈**，也就是所谓的函数作用域，**基础变量的值是存储在栈中的，而引用类型变量存储在栈中的是指向堆中的数组或者对象的"地址"**
* 保存与复制的是值本身
* 可以用**`typeof`**检测值类型
* 基本数据类型是值类型

引用类型：

* 占用空间不固定，保存在`堆`中：由于对象的创建成本比较大，在程序中创建一个对象时，这个对象将被保存到**运行时数据区**中，以便反复利用，这个运行时数据区就是**堆内存**
* 保存与复制的是指向对象的一个指针
* 使用**`instanceof（）`**检测数据类型
* 使用 `new()` 方法构造出的对象是引用型

## 分别说一下数组中常用的方法

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

## 对数组方法map和reduce方法的理解，区别在哪里

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

## 对map和reduce实现的理解，能手写一下嘛

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

## Ajax的概念，手写一下原生实现的思路

首先需要知道的是Ajax主要是通过`XMLHttpRequest`对象向服务器提出请求并处理响应，进行页面的局部更新，XMLHttpRequest对象常用的三大属性：`onreadystatechange`，`readyState`，`status`

| 属性 | 描述 |
| :--- | :--- |
| `onreadystatechange` | `readyState`属性的值发生改变，就会触发readystatechange事件 |
| `readyState` | 存有XMLHttpRequest的状态 0：请求未初始化 1：服务器连接已建立 2：请求已接收 3：请求处理中 4：请求已完成，且响应已就绪 |
| `status` | `status`属性为只读属性，表示本次请求所得到的HTTP状态码 200, OK，访问正常 301, Moved Permanently，永久移动 302, Move temporarily，暂时移动 304, Not Modified，未修改 307, Temporary Redirect，暂时重定向 401, Unauthorized，未授权 403, Forbidden，禁止访问 404, Not Found，未发现指定网址 500, Internal Server Error，服务器发生错误 |

```javascript
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

## 有了解过fetch嘛，和XMLHttpRequest的区别在哪

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

## Promise的了解，手撕Promise\(Promise.all或者Promise.race\)

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

## js为什么0.1+0.2不等于0.3

主要是因为JavaScript同样采用IEEE754标准，在64位中存储一个数字的有效数字形式

![](https://upload-images.jianshu.io/upload_images/16299591-8bfb445d26370aa4.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

第0位表示符号位，0表示整数1表示负数，第1~11位存储指数部分，第12~63位存小数部分；

由于二进制的有效数字总是表示为`1.xxx...`这样的形式，尾数部分在规约形式下的第一位默认为1，故存储时第一位省略不写，尾数部分存储有效数字小数点后的xxx...，最长52位，因此，JavaScript提供的有效数字最长为53个二进制位（尾数部分52位+被省略的1位）

由于需要对求和结果规格化\(用有效数字表示\)，右规导致低位丢失，此时需对丢失的低位进行舍入操作，遵循IEEE754舍入规则，会有精度损失

## 如何正确判断与使用this，箭头函数有没有自己的this指针

> this有四种绑定规则，默认绑定、隐式绑定、显示绑定、new 绑定，优先级由低到高
>
> 在ECMA内，this 会调用原生方法 `ResolveThisBinding()` 原生方法，该方法使用正在运行的执行上下文的`LexicalEnvironment`确定关键字this的绑定
>
> 可以简单总结为：谁**直接调用产生这个this指针的函数**，this就指向谁

* this在一般模式下指向全局对象；严格模式下 this 默认为undefined 
* 箭头函数没有自己的this指针，它的this绑定取决于外层（函数或全局）作用域）
* call，apply，bind在非箭头函数下修改this值（箭头函数下只传递参数），不管call , bind, apply多少次，函数的this永远由第一次的决定

## 对闭包的了解及其应用场景

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

// 利用闭包来取正确值
for (var i=1; i<=5; i++) { 
    setTimeout( (function(i) {
        return function() {
            console.log(i);
        }
    })(i), i*1000 );
}

// 方案二：或者使用ES6的let，这里let本质也是形成了一个闭包
for (let i=1; i<=5; i++) { 
    setTimeout( function timer() {
        console.log(i);
    }, i*1000 );
}
```

## 对eventloop事件循环机制的了解

首先，JavaScript一大特点就是单线程，这样的设计让它在同一时间只做一件事；作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM，避免了复杂性，比如假设JavaScript有两个线程，那么在同一时间进行添加删除节点操作，为浏览器分辨以哪个线程为准带来困难，所以单线程是它作为浏览器脚本语言的优势，也是它的核心特征。

> 注：虽然为了利用多核CPU的计算能力，HTML5提出了web worker标准，允许JavaScript创建多个线程，但是子线程完全受主线程控制，且不得操作DOM，所以也并没有改变JavaScript单线程的本质

那么，单线程就意味着，所有任务需要排队，前一个任务结束才会执行后一个任务，所以为了提高CPU的利用效率，就把所有任务分成了**同步任务**（synchronous）和**异步任务**（asynchronous），同步任务在主线程顺序执行，异步任务则不进入主线程而是进入到**任务队列**（task queue）中。在主线程上会形成一个**执行栈**，等执行栈中所有任务执行完毕之后，会去任务队列中查看有哪些事件，此时异步任务结束等待状态，进入执行栈中，开始执行。

主线程从任务队列中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）

![&#xFF08;&#x4E0A;&#x56FE;&#x8F6C;&#x5F15;&#x81EA;Philip Roberts&#x7684;&#x6F14;&#x8BB2;&#x300A;Help, I&apos;m stuck in an event-loop&#x300B;&#xFF09;&#xFF09;](http://www.chenqaq.com/assets/images/event.png)

## 对宏任务和微任务的理解，微任务有哪些

异步任务又分为宏任务（macrotask）和微任务（microtask），那么任务队列就有了宏任务队列和微任务队列，微任务总是在宏任务之前执行，也就是说：同步任务&gt;微任务&gt;宏任务，

宏任务包括：

| \# | 浏览器 | Node |
| :--- | :--- | :--- |
| I/O | ✅ | ✅ |
| setTimeout | ✅ | ✅ |
| setInterval | ✅ | ✅ |
| setImmediate | ❌ | ✅ |
| requestAnimationFrame | ✅ | ❌ |

微任务包括：

| \# | 浏览器 | Node |
| :--- | :--- | :--- |
| process.nextTick | ❌ | ✅ |
| MutationObserver | ✅ | ❌ |
| Promise.then catch finally | ✅ | ✅ |

## setTimeout用作倒计时为什么会产生误差

考察的其实是同一个概念，正因为setTimeout属于宏任务，那么如果当前 **执行栈** 所花费的时间大于 **定时器** 时间，那么定时器的回调在 **宏任务** 里，来不及去调用，所有这个时间会有误差

## 对new的过程和原理的理解

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

## 有了解过js的垃圾回收机制嘛

JavaScript内存管理有一个主要概念是可达性，“可达性” 值是那些以某种方式可访问或可用的值，它们被保证存储在内存中。有一组基本的固有可达值，由于显而易见的原因无法删除，比如：本地函数的局部变量和参数，全局变量，当前嵌套调用链上的其他函数的变量和参数，这些值被称为"根"，如果引用或引用链可以从根访问任何其他值，则认为该值是可访问的

JavaScript在创建变量时自动进行了内存分配，并且在它们不使用时**"自动"**释放，释放的过程就被称为垃圾回收。

现在各大浏览器通常采用的垃圾回收有两种方法：标记清除、引用计数

* 引用计数垃圾收集：把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。但是有一个限制是"无法处理循环引用问题"

```javascript
let element = document.getElementById("some_element");
let myObj = new Object();
myObj.element = element;
element.someObject = myObj;
// 变量myObj有一个element属性执行element，而变量element有一个名为someObject属性指回myObj，因为循环引用，引用计数法将没办法回收该内存

// 我们需要手动切断它们的循环引用，防止内存泄露
myObj.element = null;
element.someObject =null;
```

* 标记清除：是js中最常用的垃圾回收方式，把“对象是否不再需要”简化定义为“对象是否可以获得”，定期会执行以下的"垃圾回收"步骤（这正是标记清除算法垃圾收集的工作原理）：
  * 首先垃圾回收器获取根并**“标记”**它们
  * 然后它访问并“标记”所有来自它们的引用
  * 接着它访问标记的对象并标记它们的引用\(子孙代的引用\)
  * 以此类推，直至有未访问的引用，此时进程中不能访问的对象将被认为是不可访问的
  * 除标记的对象外，其余对象将被删除

## 对函数柯里化有了解嘛

柯里化（Currying）是函数式编程的一个很重要的概念，将使用多个参数的一个函数转换成一系列使用一个参数的函数

主要有三个作用：1. 参数复用**；**2. 提前返回；3. 延迟计算/运行

* 参数复用

```javascript
// 举个栗子：正则验证字符串

// 函数封装后
function check(reg, txt) {
    return reg.test(txt)
}

check(/\d+/g, 'test')       //false
check(/[a-z]+/g, 'test')    //true

// 需要复用第一个reg参数，Currying后，将两个参数分开，可以直接调用hasNumber，hasLetter等函数
function curryingCheck(reg) {
    return function(txt) {
        return reg.test(txt)
    }
}

let hasNumber = curryingCheck(/\d+/g)
let hasLetter = curryingCheck(/[a-z]+/g)

hasNumber('test1')      // true
hasLetter('21212')      // false
```

* 提前返回

```javascript
// 比如：解决原生方法在现代浏览器和IE之间的兼容问题

// 提前返回: 使用函数立即调用进行了一次兼容判断（部分求值），返回兼容的事件绑定方法
// 延迟执行：返回新函数，在新函数调用兼容的事件方法。等待addEvent新函数调用，延迟执行
const addEvent = (function() {
    if(window.addEventListener) {
        return function(ele, type, fn, isCapture) {
            ele.addEventListener(type, fn, isCapture)
        }
    } else if(window.attachEvent) {
        return function(ele, type, fn) {
             ele.attachEvent("on" + type, fn)
        }
    }
})()
```

* 延迟执行

```javascript
// js中bind实现机制正是Currying
Function.prototype.bind = function (context) {
    var _this = this
    var args = Array.prototype.slice.call(arguments, 1)

    return function() {
        return _this.apply(context, args)
    }
}
```
