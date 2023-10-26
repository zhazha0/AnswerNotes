# HTML

## Node和Element的区别
1.  `Node`  类型是**所有节点**类型的基础类型，包括元素节点、文本节点、注释节点等。
		包括标签，也包括标签内的文本和注释
2. 而  `Element`  类型则是**元素节点**的类型（不包括文本和注释节点），它继承自  `Node`  类型。
		- 也就是可以写成标签的<div><p><img>...
		- 元素节点可以拥有标签名tagName和属性attributes，文本和注释节点不能。
3. `nodeName`、`nodeValue`、`parentNode`、`childNodes`、`appendChild()`等属于Node，而`tagName`、`classList`、`getAttribute()`这些仅元素才有的属性和方法属于Element

```html
<div  class="cont">
	beforeText
	<div  class="child1"></div>
	<div  class="child2"></div>
	afterText
</div>
```

```js
let  cont = document.querySelector('.cont')
console.log(cont  instanceof  Element, cont  instanceof  Node) // true true
console.log('childNodes', cont.childNodes) // 子node,5个
console.log('children', cont.children) // 子元素，2个
```

----------------------------------

# CSS

## css权重
  - important > 内联 > id > class | 属性 > 标签 > 相邻兄弟h1+p，子ul>li, 后代li a, 通配符*
  - important最重，权重相同后出现的生效

## 响应式布局方案
1. rem + 设置根字体大小
2. flex弹性盒子
3. 百分比 或 vw,vh(vw未来趋势，目前兼容性不太好)
4. 媒体查询设置宽度
5. 三方库tailwinds bootstrap

## css三角形
长宽为0盒子一侧border，三侧透明
## 左右定宽中不定
 - float+margin留出左右，上下不行因为没高度。
 - flex1：左右给定宽，中间flex：1
## 图片下方有空隙
img行内，底部对齐即可
## 清除浮动
  为什么要：浮动元素抽离文档流，会导致父元素高度坍缩。
  1.容器中额外加一个空的子元素加属性clear:left/right/both
  2.或用伪元素加属性clear:left/right/both，和1相同，好处是不用加一个冗余元素。
  3.使父元素变成bfc(block formatting context块级格式上下文），一个独立渲染区域，内部样式不会影响外部。
      - 加属性overflow:hidden 最常用
      - 有浮动，absolute/fixed定位，flex的元素都是bfc元素
行布局用flex就没有清除浮动的问题了。
  

## 1px问题
  原因：似乎是移动端dpr(device pixcel ratio)不同，一逻辑像素等于多少个物理像素
    也就是移动端设备对于小于1px兼容不同造成的
  scale0.5

## sass和scss
sass无大括号，靠缩进
scss有大括号，类似css
    @extend,继承
    @mixin，混入
    都是复用手段，一个复制代码段到本类，一个把本类名加到继承的代码段逗号分割

## ios元素点击有半透明灰色框：
  a, button, input, textarea, div, span, i {
    -webkit-tap-highlight-color: transparent;
  }

## 怎么解决垂直margin合并
父子：父bfc; 
兄弟，其中一个设置float定位、绝对定位、inline-block(兄弟用bfc不行，因此overflow: hidden不行)
都可：中间插入clear:both元素；设置flex, grid布局

-----------------------------------

# JS

## 原型&原型链
 - 基本原理记住：
.prototype只有函数有，作为构造函数时挂公共函数用
__proto__只有对象有，指向构造函数的.prototype属性。

- 其他可以推导：
Array.prototype === [].__proto__ // true

## 如何让a == 1 && a == 2 && a == 3成立
==在比较时，类型不同会做隐式类型转换
===不会进行隐式类型转换，**先比较类型，后比较值**，只有类型相同，才会继续比较值，**如果类型不同, 直接返回false**
解法：
1. 重写toString方法，自增1
2. 使用defineProperty在getter返回a++，每次取值自增1

### 类似题目：如何让a == a && a == !a成立
a为[]时，因为
```js
[] == ![] // true
let a = []
a == a && a == !a // true
```

---------------
## apply, call作用及自己实现
1. apply,call 都是临时改变函数中this指向后，并以此this指向执行一次函数。
    - apply的入参是`非展开`的数组形式，如：fn.apply(this, arguments)
    - call的入参是将数组`展开`形式，如:fn.call(this, ...arguments)
2. bind是改变this指向，并返回一个永久改变了this指向的新函数。和apply,call的区别：
   - 在调用bind时，这个函数没有执行，
   - 这个返回的函数可以反复执行，this指向被永久改变。

**这三个函数都是改变函数中this指向，如果函数中没有使用this关键字，则没有任何作用**

### 实现
1. apply
思路：利用在对象中的成员方法，其this指向为该对象，因此将函数放入newThis对象中，执行完毕，再将其删掉。
```js
export function newApply (fn, newThis, arguments) {
	newThis.fn = fn
	let res = fn()
	delete newThis.fn
	return res
}
```
创建在函数原型上，使每个函数实例都可以调用，像原apply函数一样
```js
Function.prototype._apply = function (argThis, argArray = []) {
    if (!argThis) {
        argThis = window
    }

    argThis.fn = this
    let res = argThis.fn(...argArray)

    delete argThis.fn

    return res
}
```


--------------------------------
## 判断对象是否存在某个属性的最佳实践
用in
```js
function hasProperty(obj, key) {  
	return key in obj;  
}
```
其他方法各有缺陷
```js
return obj[key] // 如果obj.key值为null或undefined，会认为key不存在

return obj[key] !== undefined; // 同理如果obj.key值正好为undefined,则会认为key不存在

// 通过Object.keys()方法得到对象里所有属性形成的数组，然后使用includes方法判断该属性存不存在
return  Object.keys(obj).includes(key); // 如果obj.key是不可枚举属性，则会认为key不存在

return obj.hasOwnProperty(key); // 只能判断自身属性，不会沿着原型链查找
```

## 字符串、数组方法
slice数组字、符串 
带str的只字符串：substring头尾 substr头长度 
slice 与 substring 功能一致 
splice只数组，可删，可插
 
index参数都是：含头不含尾，忽略到末尾  
-----------------------

## ajax实现
关键字：XMLHttpRequest, open, send, onreadystatechange 4, 200
```js
let xhr = new XMLHttpRequest()
xhr.onreadystatechange = function () {
  if (xhr.readystate === 4) {
    if (xhr.status === 200) {
      // 数据获取
    }
  }
}
xhr.open(method, url, async) // 默认true异步
xhr.send(postData) // get时为空,参数为qs拼接在url上
```
-------
## promise和await、async
promise链式调用解决回调地狱
  - promise.all全部resolve才一起resolve,结果形成数组；用于需获得两个结果综合处理。
  - promise.race第一个resolve就resolve,结果只是一个；可用于超时计时。
await/async利用同步方式调用异步操作，利用generator实现

----------
## 基本数据类型个数：5 + 1 + 1
null, undefined
Number, String, Boolean
Symbol(es6)
BigInt(es10)
引用数据类型reference type
Object Array Function...

### Symbol
每次调用Symbol()生成一个唯一值
### BigInt 
Number的扩充，大于2^53-1的数，大数字后面加个n形成一个BigInt
  let pageView = 9007199254740991n;
  console.log(typeof(pageView)); // 'bigint'
---------------
## instanceof与typeof区别及原理
typeof只能判断出基本数据类型和Object, 额外一个function, 
  Array及自定义对象类型判断不出
  原理是数据低三位是类型，000是对象，null因为是全零所以也返回Object。
instanceOf可判断具体对象类型，
  原理是沿着原型链obj.__proto__, obj.__proto__.__proto__ 是否有等于className.prototype的
  所以可判断子类型也可判断父类型。
或者Object.prototype.toString.call([]) // '[object Array]'
------------------
## 0.1 + 0.2 !== 0.3
0.1和0.2转化成二进制是无限循环的小数，超出位数会进行截断，精度丢失，所以最终不等于0.3
解决：先转化成整数计算后转回小数

--------------------------------------------

## 闭包:
 - 函数里返回函数形成闭包，
 - 作用：形成私有变量，如传统类写法共有挂在this上，私有var,let通过成员函数访问。
 - 缺点：容易占用大量内存。

----------------------------------------
## 科里化
通过函数科里化可以将一个具有多个参数的函数转化为一系列只接受一个参数的函数，并返回新函数的链式调用形式。

可以使函数更加灵活和模块化。

实现思路：创建新函数，利用闭包和递归。

一般有两种形式的题目，一是实现一个科里化的函数，二是实现一个将其他函数科里化的函数。

 ### 实现一个科里化的函数
例如，实现add函数，使
add(1)(2)(3)()为6
add(1, 2, 3)(4)()为10
思路：创建新函数，利用闭包和递归。有参数则累加参数返回函数；无参数则返回累加结果。
```js
function  add (...arg) {
	let  res = 0
	function  loop (...args) {
		if (args.length > 0) {
			res += args.reduce((pre, cur) =>  pre + cur)
			return  loop
		} else {
			return  res
		}
	}
	return  loop(...arg)
}
console.log('add(1)(2)(3)()', add(1)(2)(3)())
console.log('add(1)(2)(3)()', add(1,2,3)(4)())
```

> 函数形参`...args`，表示可以接受任意数量的参数，并将这些参数转换为一个数组。
JavaScript中也可以使用`arguments`对象来实现类似功能。`arguments`对象是一个类数组对象，它包含了函数调用时传递的所有参数。
但是，使用`...args`语法相对更加直观和易读，而且它返回的是一个真正的数组，提供了更多的数组方法和功能。因此，如果使用ES6及以上版本的JavaScript，推荐使用`...args`语法。

### 实现一个将普通函数科里化的函数：
```js
/**
* 接收一个函数，并返回一个新的科里化后的函数
*/
function  currying (func) {
	let  args = [] // 利用闭包，存储每次带参调用的参数
	return  function  loop (...args1) { // 需要命名，因为其他地方也需要返回这个函数
		if (args1.length) { // 有参数，则为中间步骤，存储参数
			args.push(...args1)
			return  loop  // 返回该函数，供链式调用
		} else {
			return  func(...args) // 无参数，则为最终执行，返回原函数执行结果
		}
	}
}
// 被科里化的加和函数
function  sum (...args) {
	return  args.reduce((pre, cur) =>  pre + cur)
}

console.log('sum(1, 3, 4, 5, 6)', sum(1, 3, 4, 5, 6))
console.log('curring(sum)(1)(3)(4, 5)(6)()', currying(sum)(1)(3)(4, 5)(6)())
```


## 深拷贝解决循环引用问题
利用cache, 用weakMap
-----------------------------------------

# 模块化
1. 匿名立即执行函数iife
2. `commonJs`，同步，服务器端node使用规范，require(), module.exports =, exports.xxx =, 浏览器需browserify/webpack转化后使用
3. amd，异步，requireJs ，暴露xxx: define(function () {return xxx}),有依赖第一个参数传入一个依赖数组
```js
define([depen1, depen2], function () {

})
```
* 4. cmd, 国产，不重要，语法结合commonJs，amd:  define(function(export, require, factory) {})

5. `es6 module`, import, export, import()动态加载，分离单独chunk, 使用内联注释进行设置，webpack可原生支持

    * 多个文件import同一个文件，只执行一次，缓存结果，每次导入的是同一执行结果，典型单例模式
      所以导出的vuex对象，observable对象都是同一个实例
    * 想要不同实例，导出类，引入后各依赖方自己new出实例

    存在`符号绑定`:效果时导出的基本类型数据，在模块外和模块内的修改会共享。
      这与js的值类型赋值修改不会互相影响相反。

6. umd:universal module通用，可识别commonJs、amd两种，都可引入

## import引入
import有路径，按路径查找，例如`import './xxx'; import '/xxx';`
import模块，无路径，在node_modules中查找该文件夹，
  之后根据目录下package.json配置，寻找引入哪一个文件
   - node环境，会引入main字段指向的文件，如果没有配置main，则寻找index.js，无此文件则报错
   - 浏览器环境，先寻找browser字段，再寻找module字段，再没有则使用main字段
      * browser为浏览器环境下入口
      * modules为esm规范入口，node环境下无效
      * node和浏览器环境均生效
  如果没有package.json，或里面没有配置控制字段，则引入index.js
--------------------------------------------

## 重排(reflow)和重绘(repaint)

改变元素位置大小，即会影响到其他元素位置时发生重排
改变元素颜色外观，会发生重绘

    增删可见元素
 - 会改变位置大小的css属性
    1. 盒模型相关:width, height, padding, margin; box-sizing
    2. 文字相关：font-size, letter-space, word-space, 
    3. 位置相关：position, float, display

- 优化
    1. 动画和改变位置的用transform --- 避免重排
    2. 2D transform改为3D transform即使只想做2D改变, translate3D() --- 强制开启GPU加速
    3. 分层 ---浏览器将元素分layers绘制，最后进行composition合成，
        这样只需绘制改变的层，浏览器会猜测可能改变的元素，将他们单独放置一层
        will-change告知浏览器可能改变，浏览器会将它们分层。
    4. 一段同步代码中多次改变位置大小，并不会每次改变都立刻回流
        但如果调用El.getBoundingClientRect(), getComputedStyle(),会立刻触发一次重排，以获取真实大小和位置，避免反复改大小再获取，例如while循环中
    5. 图片位置给图片合适的尺寸避免图片加载完毕后页面跳动、发生重排




-------
## 其他性能优化
 - dns-prefetch: head 添加link元素写上rel="dns-prefetch" href="www.xxx.com"
 - link标签preload js脚本，提前加载但不执行
 - 静态资源放cdn: 无cookie，在dns服务器添加一条规则，把用户域名解析请求导向cdn服务，会在服务器网络中选择最近的物理结点或者不繁忙的节点。
    - CDN也是一种`缓存`，在各个CDN节点存有资源副本，命中直接返回给客户端，未命中向源服务器请求并缓存在当前节点。
 - 资源压缩，小资源合并（小图片合并成雪碧图，background-position）
 - 利用缓存机制：
   - 强制缓存,返回头中`expires`或`cache-control: max-age`控制
    expires绝对时间点，如Mon,11 Nov 2019 08:36:00，但服务器时间可能和客户端不一致，故已被max-age替代
    max-age存活时间，是相对时间，如1000，指浏览器收到后1000秒内有效，优先级比expires高
    优先级高于协商缓存，没过期就用缓存，不用和服务器协商。`强制缓存过期才进行协商缓存`。
   - 协商缓存，返回头中`lastmodify`与`etag`这两个值，浏览器要拿去给服务器判断才能决定是不是用缓存，故称协商。
    last-modify文件最后修改的时间，但一次发布，可能有的文件有改动，有的文件没有改动
    etag文件内容唯一标识，有修改则etag改变，比last-modify更精确的标识出是否需要给浏览器，故优先级高于last-modify
   > - 是否要缓存是服务器在响应头字段告知的，前端不能决定。
   > - 前端让打包js文件名变化使浏览器的缓存强制失效，如使用contenthash值作为文件名。
   > - 缓存内容同时`存放于内存和磁盘`，关掉浏览器，内存缓存失效，因为文件存放在进程空间占用的内存里面，以后取磁盘。

 - 长列表优化：虚拟列表，进入可视区域再渲染

## 首屏性能优化：
 - loading图标、界面或骨架屏提升`感知性能`

 - 路由懒加载/按需加载

 - 组件模块懒加载, codespliting：
  es6动态引入语句，import()替代原require.ensure，webpack会打包成单独chunk

 - 图片懒加载：判断图片进入可视区域，监听滚动（可节流优化），scrollTop + clientHeight >= 图片距离页面顶部距离, 再给src赋真实值
  * 判断元素是否在/是否进入可视区域方法：
    1. (监听scroll)offsetTop - clientHeight <elementheight
    2. (监听scroll)el.getBoundingClientRect()获取的都是相对于viewport的top, left
    3. IntersectionObserver, 无需监听scroll,性能更佳

 - webpack配置预渲染：
    预渲染插件prerender-spa-plugin，利用无头浏览器获取并执行js，把数据插入html
    - 适用于首屏数据不常变化，不依赖登录态

 - ssr渲染：首屏ssr，之后调spa

 - 单页改多页：配置webpack entry，同时js、css分开打包
-------
## 跨域
 - 浏览器限制`跨域资源可编程访问`
xhr不能访问跨域资源
img标签跨域获取的内容不能在canvas绘制(src显示图片可以)

 - 解决方案：
三个标签没有跨域限制，link引用css，script引用js, img引用图片，
  jsonp利用script无跨域的特点，仅限get请求，指定一个回调，执行返回的脚本

跨域限制是浏览器行为，服务器返回了内容，但是浏览器不接受不符合同源策略的数据。
跨域限制是浏览器安全限制，故可通过服务器端转发；本地开发localhost跨域---可开启忽略浏览器安全警告配置项
服务器端把当前请求域名添加到allow-origin

 - 防盗链：和跨域不同，是服务器的限制行为，一般为读取referer, 看请求来源是否
  * 一般为图片，音视频，软件包的等二进制文件做防盗链
  * 请求的request header中有referer字段：为请求资源的页面的地址（是从哪个服务器下载的就是什么地址），包含了来源域名
  * referer可伪造，所以可采用加密签名，请求图片或其他资源，必须带一个签名，验证通过才给下载。
  * iframe没有referer, 所以服务器限制了referer为空也不允许，则iframe也不能访问。

------
## 页面，脚本，资源加载

### 文档加载过程
文档加载和解析包括：
1. 浏览器下载文档
2. 浏览器解析文档, 解析步骤
    1. HTML -> DOM, CSS -> CSSOM
    2. DOM + CSSOM => Render Tree(渲染树)
    3. Layout, 根据渲染树计算出节点在页面大小和位置
    4. Paint, 将节点绘制到浏览器上

    - 其中3,4页面首次渲染肯定会进行一次，
      之后页面布局改变会再次触发relayout/reflow, repaint即重排和重绘


### DOMContentLoaded与Load

1. DOMContentLoaded发生在document对象上，文档加载并解析完成后触发（大概渲染树生成后触发），
会等待同步脚本(及defer脚本)加载并执行完后触发，不会等待css,img,video等资源加载。
  - 但script脚本会等待css加载完成后执行，所以结果上还是在css之后触发。
  - async脚本不阻塞DOMContentLoaded，defer会。
2. Load发生在window对象上，页面所有图片，音视频加载完毕后触发，晚于DOMContentLoaded。
  - 此时可获得图片正确尺寸

### async与defer:
  无属性script标签：暂停文档解析，加载完毕后立即执行，执行完毕再继续解析文档
    请求和执行脚本都阻塞。
  async：异步请求，请求回来后停止文档解析立即执行，执行完毕恢复文档解析，
    若返回时文档已解析完毕则不阻塞，否则执行时阻塞文档解析。
    因请求返回时间不确定，多个async script标签间执行顺序不确定。先到先执行。
  defer：异步请求，推迟到文档解析完成后执行；
    不阻塞文档解析。
    多个defer script间执行顺序与书写顺序一致，不破坏依赖关系。
  async, defer同时存在视为只有async

### sendBeacon方法
  页面关闭仍执行(xhr不行)，故不必延迟页面的跳转，用于离开页面时发送统计数据。
  - 数据通过http post发送


---------------------------------------

# 浏览器进程、线程与事件循环（有待商榷）

### 浏览器的进程与线程
chrome浏览器至少会开启四个进程：
1. 浏览器主进程，
2. GPU进程，
3. 网络进程，
4. 插件进程
5. 渲染进程：每打开标签页都会开启一个新的渲染进程，负责页面显示、用户交互等我我们能看到的绝大多数工作在这里完成。V8引擎运行在该进程中。

### 渲染进程
 - GUI渲染线程，负责解析html，css，构建dom树，CSSOM树，渲染树和绘制页面
 - JS引擎线程，一个tab页面只有一个该线程（单线程），负责解析与执行JS，
		 它与GUI渲染进程不能同时运行，因此JS任务执行时间过长会阻塞渲染(看起来卡顿)。
 - 计时器线程
 - 异步http请求线程
 - 事件触发线程
-------------------------------------

## url输入后经历过程：
1.查找url里域名对应的ip: 向dns服务器发送域名，dns服务器返回ip
2.向该ip发送请求：三次握手建立tcp连接，发送http请求(method, url, httpvesion, request header, body)
3.服务器返回http response(状态行，response header, body), body里是html正文
4.浏览器加载并解析html（不需要加载结束，可同时加载同时渲染）, 遇到资源文件，则再发送http请求进行获取，如遇到图片，css，字体，异步请求，js资源会暂停解析再发送请求（async,defer除外，故js放在文档body标签底部）
5.直到解析完毕。

## 事件循环/Event Loop

1. 事件触发，回调才会放入异步任务队列，不是注册了就放
2. 不是放了就执行，要等当前宏任务完成
3. script就是第一个宏任务，如果宏任务长时间完成不了，则其他宏/微任务都不会得到执行
4. 每一个宏任务完成，会去清空一次微任务队列；然后再执行其他宏任务或渲染

### 宏微任务和渲染的关系
微任务渲染之前，宏任务渲染之后？？？

### 宏任务
1. click, scroll等事件回调；
2. setTimeout,
3. requestAnimationFrame

### 微任务
1. Promise
    * new Promise时传入的代码是同步，then/catch/finally才是异步




------------------------------------

# vue2：

## 响应式原理
template 会被编译成render函数，函数执行的时候，访问什么变量，就出触发相应变量的get，然后才会添加watcher。

keepalive标签，vue内置组件，动态组件里缓存组件实例和组件状态例如滚动条位置，有activated, deactivated生命周期。

安装plugin: Vue.use(obj/func), obj需包含一个install方法，或本身就是一个方法(视为install)，调用use时执行install方法，执行一些操作，例如挂到Vue.prototype
## v-model与.sync

1. v-model: input, select类元素双向绑定语法糖
    相当于两部分(val1是用户定义变量):
      @input="val1 = e.target.value" :value="val1"

  自定义父子组件也可用: 不过绑定的值永远叫value；且子组件必须发送input事件
  <comp v-model="msg"> 等同扩展为 <comp :value="msg" @input="val => msg = val" />
  子组件：
  props: ['value']
  this.$emit('input', 'newVal')

2. .sync 父子组件需共同修改一个变量语法糖， props的双向绑定
  <comp :foo.sync="bar"></comp> 父可修改bar,子也可通过发事件修改bar
  相当于
  <comp :foo="bar" @update:foo="val => bar = val"></comp>
  child: this.$emit('update:foo', newValue)

  场景：如弹窗开关，父可控制弹窗显隐，子也有个开关可控制。

  它和v-model用在自定义组件时功能几乎相同，
  .sync可多个：<comp :value.sync="username" :age.sync="userAge" />
  
组件样式隔离scoped: 子组件每个元素加了一个`data-v-hash属性`，元素选择器加`属性选择器`设置样式
  对三方库要改其组件样式；deep深度选择器

## 父子传值：
0.props，emit
1.父provide,任意级子inject(只能父传子孙，反之不行)，深层父子嵌套；要用data中属性，要使用返回对象的函数；要响应式，父provide传入对象
  - 依赖注入(provide/inject)可看作`long range props`, 除了:
     1. 父不知道哪些子孙组件使用它provide的property
     2. 子孙组件不知道inject的property来自哪里
2.$attrs, $listeners:解决深层嵌套层层传递的繁琐
  child用$attrs获取parent所有:xxx传入的属性，<grandchild v-bind="$attrs">全部传入孙组件
  $listener获取grandp所有@xxx监听事件，<grandchild v-on="$listener">监听grandchild抛出的事件，上抛给grandp
3.vuex
4.eventBus，创建vue实例，都引入它emit, on
5.observable对象

## nextTick原理
dom异步渲染，更改data数据后，视图不是立刻改变
nextTick回调中可获取到数据更新后的视图

原因：
data改变后，其watch中update视图的回调没有立即执行，而是内部也调用nextTick放入一个异步任务中，
内部通过promise创建一个微任务，待当前同步代码执行完毕就会执行全部微任务
此时我们自己调用nextTick创建的微任务也会插入到队列尾部，先进先出，我们获取dom的操作执行时，前面的update dom肯定已经执行完了。

## Vue.extend与创建可函数调用组件

1. Vue.extend接收组件配置对象，生成一个组件构造函数Ctor,
2. new Ctor()得到组件实例componentInstance
3. componentInstance.$mount(el)可挂载到el元素，
    * 不传le，生成组件dom但在文档之外，需自己appendChild(componentInstance.$el)插入文档
    * $mount()编译template生成真实dom放在$el,不执行$mount()则$el为空

## render函数
组件模板编译后会得到一个render函数，这个函数执行后返回组件vnode树。
执行时机：render函数首次渲染会执行；在每次有数据改变时也都会执行。
  （Vue3对此有优化，静态提升；blockTree等）

### 作用域插槽是什么
插槽的作用是可以让父组件调用子组件时，`向子组件传递`一块自定义内容，显示在子组件内部。
但是这块内容虽然最终属于子组件，但是调用不了子组件中数据。
    > 因为组件的编译存在作用域，父组件的内容是在父组件作用域中编译，使用不了子组件属性。
作用域插槽就是可以把`子组件中数据传递给父组件`，父组件用一个变量接收，就可以在插槽模板中使用该变量。

 - 总结：用于插槽中实现**父组件使用子组件值**，或**子组件向父组件传值**

---------------------------------------

# vue3:
composition APIs入口是setup函数(或带setup属性的script标签)
两种定义响应式数据方式:
1. ref() accept `primitive` and `reference` type
  返回一个包含.value的响应式对象。
    基本类型包裹在一个对象中{value: xxx}, setup中需用.value访问，模板中不需要
        使用defineProperty get() set()创建响应式
    改值要用.value = newVal
    重新赋值.value = newObj, newObj同样具有响应式
    引用类型则还是调用reactive()
2. reactive()只接受引用类型
  只能对对象创建响应式，解构或读取其中某个属性，会失去响应式
    如果需要子属性拆解后(...{})也是响应式，用`toRefs()`
defineProperty一次只能定义一个属性，故需遍历对象属性值；还需额外变量存储，可以用函数闭包存，vue2即如此

3. <setup>是构建工具提供的语法糖，省去setup()函数声明和return大量对象
    
### ref(), reactive()解耦式创建响应式对象：
  不同于vue2响应式对象需声明在data options中，
  ref(), reactive()等api可解耦式创建响应式对象，可替代vue2 Vue.observable() api
  - 结合es6 import同一实例，可创建小型状态管理及共享机制  

cheetsheet:
 - 生命周期beforeCreate, created 被setup()函数取代
    * composition api生命周期前加on, onBeforeMount, onMounted
    * beforeDestroy被onBeforeUnmount
 - setup()函数需要return模板使用的响应式变量，`<script setup>`语法糖省去return

## Vue3相对于Vue2的优化
编译上：
 - 静态提升
 - 
 - blockTree
----------------
# webpack相关：
可以处理es6,commonjs,amd等所有模块引入方式
loader作用：加载及处理某种文件
css-loader，使webpack能够处理引入样式文件如import css from './style.css'，将其转换为commonjs模块，返回样式字符串
style-loader,将样式插入到html文件中：创建style节点append到引入该打包后js 的HTML header中

webpack原理: 
  从入口文件分析依赖，使用各种loader解析各种模块
  根据split chunk配置分块
  生成文件
  写入文件系统

  每一步都可以通过plugin改写生成物。

## vue-cli
vue-cli基于webpack:
 可以用
 `chainWebpack`函数方式配置
 `configureWebpack`对象方式配置
# Vite相关
vite是基于esbuild和rollup开发的，esbuild用于开发环境，rollup用于打包发布包。

-------------------

# 安全：

### sql注入：文本变指令，用户输入文本没有做过滤或转义，被作为sql指令一部分执行
防范：过滤，转义；将sql语句预编译

------

# 自动化测试
单元测试：Jest

--------
# 业务
防抖：输入停止时才真正出发

------------

# 后端/全栈
Node.js: Node.js提供了非浏览器端的javascript语言运行环境，和操作文件、数据库的api
  用途：前端工程化工具，包括编译、打包、自动化等----用的最多的
        服务器端：像其他后端语言一样，编写服务器端程序
### 代理和反向代理
代理：代理客户端，客户端通过一个跳板和服务器联络，服务器不知道客户端真实身份
    用途: 隐藏身份, 绕墙，有一定cache作用
反向代理：代理服务器端，客户端访问的是一个代理服务器，代理服务器转给真实的业务服务器，再转发其返回。
  服务器端知道客户端身份，但客户端不知道自己真正访问的是哪台服务器
    用途：load balance; 服务器端防火墙；给服务器端提供加密及安全服务
      也具有一定cache作用

-------

-------
# 兼容性问题
 - 输入类：
ios键盘弹起页面可视高度window.innerHeight不变，但页面发生滚动，可滚动最大区域为键盘弹起高度。
 - 滚动类：
 ios8以前滚动时scroll事件不派发，导致滚动时联动比如索引列表失效，吸顶使用position: sticky; 其他iscroll或betterscroll
 ios8以后已经解决
-------
# 项目搭建与结构
 - 权限：
  页面级，路由守卫
  导航按钮级，自定义指令/v-if + 权限对象存放vuex
    私以为v-if更好因为权限通常是白名单，一开始为false不会渲染，不用像指令一样已创建了dom元素再移除

 - 组件抽取：
  高频使用全局注册
  项目通用插件化上传npm

--------
dom,css结构分割方式大致三种
行、--flex, grid
列、--flex, grid
重叠：--父relative/absolute, 子absolute+pos
  有遮盖
  浮标角标

状态切换，悬浮气泡弹窗等

弹窗,toast,loading 独立aside结构， body直接子元素，fixed/absolute定位，translate水平垂直居中