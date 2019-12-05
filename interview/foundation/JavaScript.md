# JavaScript篇

## Q1：跨域的方式有哪些？jsonp的原理以及服务端如何处理

> 分析:谈到跨域，首先想到的是我们为什么需要跨域？要实现跨域，就要知道跨域的方式有哪些？你最常用那种方式跨域？原理是什么？浏览器端如何做？服务端又该如何处理？
### Q1.1：为什么需要跨域，同源策略拦截客户端请求还是服务器响应
之所以需要跨域，是因为浏览器同源策略的约束，面对不同源的请求，我们无法完成，这时候就需要用到跨域。同源策略拦截的是跨源请求，原因：CORS缺少`Access-Control-Allow-Origin`头
### Q1.2：跨域的方式有哪些
JSONP、proxy代理、CORS、XDR
### Q1.3：JSONP的原理是什么，缺点是什么，浏览器端如何做
JSONP主要是因为`script`标签的`src`属性不被同源策略所约束，同时在没有阻塞的情况下资源加载到页面后会立即执行

缺点：
- JSONP只支持get请求而不支持post请求，如果想传给后台一个json格式的数据,此时问题就来了,浏览器会报一个http状态码415错误，请求格式不正确
- JSONP本质是一种代码注入，存在安全问题
### Q1.4：JSONP服务器端如何处理
实际项目中JSONP通常用来获取`json`格式的数据，这时候前后端通常约定一个参数`callback`，该参数的值，就是处理返回数据的函数名称
```js
var f = function(data){
    alert(data.name)
}

var _script = document.createElement('script');
_script.type = 'text/javascript'
_script.src = 'http://localhost:8888/jsonp?callback=f'
document.head.appendCild(_script);
```

```js
//node处理
var query = _scr.query;
var params = qs.parse(query);
var f = '';

f = params.callback;

res.writeHead(200,{'Content-Type','text/javascript'})
res.write(f + "({name:'hello world'})")
res.end
```

```php
//php处理(注意输出格式)
$data = array(
    'rand' => $_GET['random'],
    'msg' => 'Success'
);

echo $_GET['callback'].'('.json_encode($data).')';
```
### 1.5：（扩展）cors
cors是一种现代浏览器支持跨域资源请求的一种方式，它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制
```js
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
## Q2：介绍ES6熟悉的几个新特性
- let，const声明变量
- 解构赋值
- 箭头函数
- 扩展运算符
- 数组的新方法 -- map，reduce，filter
- promise
### Q2.1：var，let，const的区别
首先var相对let/const，后者可以使用块级作用域，var声明变量则存在函数作用域内（该域内存在变量提升）。let/const有一个暂时性死区的概念，即进入作用域创建变量到变量可以开始访问的一段时间。

其次let，const主要的区别在于：const一旦被赋值就不在改变了
```js
const name = 'Jack';
name = 'Tom'  //TypeError:Assignment to constant
```
但值得注意的是：这并不意味着声明的变量本身不可变，只是说它不可再次被赋值了（const定义引用类型时，只要它的引用地址不发生改变，仍然可以改变它的属性）
```js
const person = {
    name : 'Jack‘
}
person.name = 'Tom'
```
## Q3：高频率触发事件解决方案-防抖和节流

节流：在一段时间内不管触发了多少次都只认为触发了一次，等计时结束进行响应（假设设置的时间为2000ms，再触发了事件的2000ms之内，你在多少触发该事件，都不会有任何作用，它只为第一个事件等待2000ms。时间一到，它就会执行了。
）
```js
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
```js
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
