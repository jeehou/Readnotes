## Chapter 1. MVC和类
### 什么是MVC
MVC是一种设计模式,它将应用分为3个部分：数据（模型）、展现（视图）和用户交互（控制器）。换句话说，一个事件的发生是这样的过程 ：
1. 用户和应用产生交互。
2. 控制器的事件处理器被触发。
3. 控制器从模型中请求数据，并将其交给视图。
4. 视图将数据呈现给用户。
### 模型
模型只需包含数据及直接和这些数据相关的逻辑。任何事件处理代码、视图模板，以及那些和模型无关的逻辑都应当隔离在模型之外。
### 视图
视图是呈现给用户的，用户与之产生交互。除了模板中简单的条件语句之外，不应该包含任何其他逻辑。对于视觉呈现相关的逻辑可以放在helper中，而不应该放在视图中。
### 控制器
控制器从视图获得事件和输入，对它们进行处理（很可能包含模型），并相应地更新视图。
### 向模块化进军，创建类
1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
2. Object.create
传入一个对象，返回一个继承了这个对象的新对象。
```javascript
Object.create = function(o) {
  function F(){}
  F.prototype = o;
  return new F();
}
```
3. JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型（someObject.[[Prototype]] or someObject.__proto__ or Object.getPrototypeOf()和Object.setPrototypeOf()），以及该对象的原型的原型，依此层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾（null）。
4. prototype & this, 所有实例对象需要共享的属性和方法，都放在prototype；那些不需要共享的属性和方法，就放在构造函数里面，通过this引用。
5. 当继承的函数被调用时，this 指向的是当前继承的对象，而不是继承的函数所在的原型对象。
6. new & prototype
```javascript
var Foo = function(name) {
  this.name = name;
};
var o = Person('bob'); //=> undefined
var o = new Foo('bob');
// 实际执行
var o = new Object();
o.[[Prototype]] = Foo.prototype;
Foo.call(o, name);
```
如果省略new，会创建一个全局变量name。
### 添加函数
```javascript
//给类添加静态
Persion.find = function(id) {/*...*/};
Persion.find(1);
//给实例对象添加函数
Persion.prototype.breath = function() {/*...*/};
var p = new Persion;
p.breath();
```
### 基于原型的类继承
```javascript
var Animal = function(){};
var Dog = function(){};
// Dog继承了Animal
Dog.prototype = new Animal;
```
### 函数调用
Apply()第一个参数是上下文（默认全局对象），第二个参数是参数组成的数组。
```javascript
function.apply(this, [1,2,3])
```
call()第一个参数是上下文，第二个参数是参数组成的序列。
```javascript
function.call(this,1,2,3)
```
可以通过apply，call，bind来解决event中this指向错误问题
```javascript
// ES 中的bind，thisArg表示希望的this指向，arg1...表示函数运行时的参数
Func.bind(thisArg, [, arg1, [, arg2 [,...]]])
// jQuery 中的bind，使用bind之前确保DOM元素存在
$('...').bind(eventType, [,eventData], handler)
```
arguments是解释器内置的当前作用域内用来保存参数的数组，不可变，需通过jQuery.makeArray()将其转换为可用的数组。
### 添加私有函数
仅仅添加下划线前缀是作者认为不可取的，可以利用匿名函数创建私有作用域。
## Chapter 2. 事件和监听

## Chapter 3. 模型和数据
### MVC和命名空间
与数据操作和行为相关的逻辑都应当放入Model中，通过命名空间进行管理。
### 构建对象关系映射(ORM)
对象关系映射(Ojbect-relational mapper, ORM)将Model的改变和远程服务（发起一个Ajax请求）捆绑在一起。ORM是一个包装了一些数据的对象。我们可以给它添加自定义的函数和属性来增强基础数据的功能。比如数据的验证、监听、数据持久化及服务器端回调处理等。
### 原型继承
现在创建Model，并创建新的类和实例对象，create是创建类，init是创建实例对象。
```javascript
// var a=Object.create(o)中最重要的是a.prototype=o
var Model = {
  create: function(){
    var obj = Object.create(this);
    obj.prototype = Object.create(this.prototype);
    return obj;
  }
  init: function(){
    var instance = Object.create(this.prototype);
    return instance;
  }
}
// 使用
var User = Model.create();
var user = User.init();
```
### 通过Ajax载入数据
本章主要介绍的是jQuery的API，基于jQuery.ajax()函数。ajax函数接受存放配置信息的对象作为参数。
1. url
请求的url
2. success
成功的回调函数
3. contentType
默认值为application/x-www-form-urlencoded
4. type
GET，POST或DELETE
5. data
6. datatype
Ajax的一个限制是同源策略，它限制所有的请求都必须来自同一个域名、子域名和端口。
CORS打破同源限制，即跨域Ajax。只需要在HTTP协议的响应头中加入如下两行：
```javascript
Access-Control-Allow-Origin: some.com
Access-Control-Reques-Method: GET, POST
```
### JSONP
Web页面调用js时不受跨域的影响。
原理是通过创建一个script标签，所辖的外部文件包含一段JSON数据，数据由服务器返回，作为参数包含在一个函数调用中。
http://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html
### 本地存储数据
已经能做到为每个域名提供5MB的存储空间。HTML5定义了两类：local storage和session storage。浏览器关闭后local storage数据也能保持，而session storage数据则不存在了。浏览器端所存储的数据是以域名分开的。数据存储均为字符串。
## Chapter 4. 控制器和状态
### 模块模式
模块模式是用来封装逻辑并避免全局命名空间污染的好方法。使用匿名函数可以做到这一点。
```javascript
(function(){
  /*...*/
})();
```
### 全局导入
隐式的全局变量会让程序变得更慢，因为JavaScript解释器不得不遍历作用域。局部变量的读取会更快、更高效。
```javascript
(function(jQuery){
  /*...*/
})(jQuery);
```
### 全局导出
可以将页面的window导入我们的模块，直接给它定义属性，通过这种方式可以暴露全局变量。
```javascript
(function(jQuery, export){
  /*...*/
  export.Foo = 'a';
})(jQuery, window);
assertEqual(Foo, 'a');
```
### 文档加载完成后载入控制器
根据目前情况，控制器的一部分在生成DOM之前就载入了，另一部分则在页面文档载入完成后触发的回调里。作者推荐在DOM生成之后统一载入控制器，这样不用考虑加载时考虑DOM的渲染处于什么阶段。
可以用jQuery(function(){/*...*/})，这句话的意思就是在页面DOM节点构建完成后才进行初始化的动作加载控制器。
### 访问视图
一种常见的模式是一个视图对应一个控制器。视图包含一个ID，因此可以很容易地传入控制器。然后在视图之中的元素则使用class而不是ID，所以和其他视图中的元素不会产生冲突。
为了不用重复地查找DOM，可以创建一个空间专门存放选择器到变量的映射表，并在控制器实例化时创建DOM。
```javascript
elements : {
  "form.searchForm":"searchForm"
}
```
### 委托事件
跟elements类似，通过events存放事件类型和选择器到回调函数的映射。
```javascript
events : {
  "submit form":"submit"
}
```
### 状态机
状态机由两部分组成：状态和转换器。它只有一个活动状态，但也包含很多非活动状态（passive state）。当状态之间互相切换时会调用转换器。
通过状态机来处理互斥视图。
### 路由选择
将应用的状态和URL绑定，建立状态和URL的某种对应关系。
### 使用URL中的hash
定位本页面所用的URL（基于URL）是不能更改的，如果改变则会引起页面的刷新，通过操作hash来改变页面（例如：http://twitter.com/#!/maccman).
通过window.location.hash来读取或者修改页面hash。
### 检测hash的变化
可以通过监听hashchange事件。
### 抓取Ajax
Google对它的搜索引擎做了改进，提出了“Ajax抓取规则”。
### 使用HTML5 History API
history.pushState()函数，包含三个参数，数据对象（可以是任意自定义对象，当触发popstate事件的时候随之传入回调）、标题（浏览器历史记录中的新页面标题）和URL（相对地址）。
history.replaceState()函数的作用几乎和history.pushState()一模一样，但它不会给历史记录堆栈添加新东西。可以通过history.back()和history.forward()函数在浏览器历史记录中做跳转。
## Chapter 5. 视图和模板
### 动态渲染视图
渲染内容不多，推荐使用document.createElement()或者jQuery API完成。
作者更推荐将静态HTML包含在页面中，在必要的时候显示或隐藏，使得视图和控制器彼此尽可能地分离开来。
### 模板存储
在JS中以行内形式存储（违背MVC原则）
在自定义script标签里以行内形式存储（推荐）
远程加载（慢）
在HTML中以行内形式存储（增加首包提及）
### 绑定
绑定将视图和模型挂接到一起，一旦数据变化时，视图会根据新修改后的对象做适时更新。
## Chapter 6. 依赖管理
### CommonJS
CommonJs是服务器端模块的规范，Node.js采用了这个规范。加载模块是同步的，所以只有加载完成才能执行后面的操作。
### AMD
异步加载，requireJS
https://github.com/amdjs/amdjs-api/wiki/AMD
### CMD
异步加载，seaJS
https://github.com/seajs/seajs/issues/242
### 无交互行为内容的闪烁（FUBC）
依赖JS操作样式时，需要将样式提取出来放入初始化CSS之中，比如隐藏一些元素或展示一个加载指示器，提示页面正在加载中。
## Chapter 7. 使用文件
### 浏览器支持
HTML5提供了文件API，渐进增强处理文件操作（对于不支持的提供传统的上传功能，对于支持的提供文件拖拽功能）
```javascript
if (window.File && window.FileReader && window.FileList) {
    //支持文件操作API
}
```
### 获取文件信息
HTML5使用File对象表示文件
name：文件名，size：文件大小，type：文件的MIME类型
### 文件输入
多文件上传
```javascript
<input type="file" multiple>
```
利用files的属性过滤上传文件：
```javascript
var input = $("input[type=file]");
input.change(function(){
    var files = this.files;

    for (var i = 0; i < files.length; i++) {
        if (files[i].type.match(/image.*/)) {}
    }
});
```
### 拖拽
事件：dragstart、drag、dragover、dragenter、dragleave、drop 和 dragend。
### 开始拖拽
将元素的draggable属性设置为true
```javascript
<div id="test" draggable="true">Drag</div>
```
监听dragstart事件并调用事件的setData
```javascript
var element = $("#test");
element.bind("dragstart", function (event) {
    event = event.originalEvent;

    event.dataTransfer.effectAllowd = "move";

    //设置拖拽内容
    event.dataTransfer.setData("text/plain", $(this).text());
    event.dataTransfer.setData("text/html", $(this).html());
    event.dataTransfer.setData("text/uri-list", "http://www.weibo.com");
    event.dataTransfer.setDragImage("/images/drag.png", -10, -10);
});
```
可以让用户拖拽文件到浏览器之外，只需设置DownloadURL即可，目前只有Chrome支持。
### 释放拖拽
撤销了 dragenter 和 dragover 事件，才能开始监听 drop 事件。
```javascript
//撤销事件
e.stopPropagation();
e.preventDefault();
```
### 撤销默认的Drag／Drop
默认情况下，将一个文件拖拽到web页面中会让浏览器重定向到这个文件，撤销body的 dragover 事件即可。
### 复制和粘贴
事件：beforecopy、copy、beforecut、cut、beforepaste和paste。
### 复制
和拖拽的dataTransfer对象类似，可以使用clipboardData对象设置自定义的剪切板数据。
IE将clipboardData对象设置在window上，而不是事件上。
Firefox不允许访问clipboardData对象，Chrome会忽略在对象上设置的任何数据。
### 读文件
当你获得File的引用时，可以用它来实例化一个FileReader对象,将文件内容读入内存。文件的读取是异步的,你需要给FileReader实例提供一个回调函数，当读文件时触发这个回调。
FileReader对象包含四个方法读取文件数据。
readAsBinaryString（以二进制形式读取），readAsDataURL，readAsText（以字符串形式读取），readAsArrayBuffer。
需要关注的事件包括：onerror，onprogress，onload。
### 二进制大文件和文件切割
有时最好将文件的一个片段读入内存，而不是整个文件。HTML5提供file对象的slice函数，第一个参数是稳健的其实字节位置，第二个参数是以字节为单位的偏移量。
### 自定义浏览器按钮
面对default按钮不支持某些事件时，在相同位置放置一个透明的文件输入框，获取任何事件。
### 上传文件
XMLHttpRequest赋予了Ajax上传文件的能力。
### Ajax进度条
XHR2规范加入了对progress事件的支持，下载和上传都支持这个事件，使用它即可实现实时文件上传进度条。
```javascript
var req = new XMLHttpRequest();
req.upload.addEventListener("progress", updateProgress, false);
```
## Chapter 8. 实时Web
### 实时Web的发展历史
1. 轮询（Polling，pull）：每隔一段时间向服务器请求新数据。（可跨域，服务器压力大，延时，应用场景：对实时性要求不高的应用，如微博提示回复等）。
2. Comet（push）：直到有数据push给客户端（或time out）才发送response，客户端收到response才发起新的request。（减少轮询，低延迟，不可跨域，可利用jsonp实现跨域，服务器需要hold住大量connection，应用场景：实时性要求较高的应用，如微博私信等）。
http://kenny7.com/2013/05/technical-guide-for-website-realtime-communication.html
### WebSocket
WebSocket提供了基于TCP的双向的、全双工的socket连接。对于那些不支持WebSocket的浏览器，可以降级使用笨方法来实现，比如Comet或轮询。
使用WebSocket时，一旦服务器和客户端之间完成握手，信息就可以畅通无阻地随意往来于两端，而不用附加那些无用的HTTP头信息。
默认情况下WebSocket使用80端口建立非加密的连接(ws)，使用443端口建立加密的连接(wss)。
为了更好更成功的使用WebSocket，这里给出一些建议：
1. 使用安全的WebSocket连接（wss）。
2. 在WebSocket服务器前面使用TCP负载均衡器，而不要使用HTTP负载均衡器。
3. 不要假设浏览器支持WebSocket，需要判断并适当降级处理。
### Node.js和Socket.IO
Socket.IO是一个Node.js库，实现了WebSocket。
若环境支持WebSocket，那么Socket.IO就会尝试使用WebSocket，若有必要也会降级使用其他的传输方式。这里列出了所支持的传输方式，基本支持所有浏览器。
1. WebSocket
2. Adobe Flash Socket
3. ActiveX HTMLFile (IE)
4. 基于multipart编码发送XHR
5. 基于长轮询的XHR
6. JSONP轮询
### 实时架构
要想为你的应用构建实时架构，则需要考虑两件事：
1. 哪个模型需要是实时的
2. 当模型实例发生改变时，需要通知哪些用户
最佳方法是使用发布／订阅模式：客户端订阅某个特定的信道，服务器向这个信道发布消息。每个用户订阅唯一的信道，信道包含一个ID，可能是用户在数据库中存放的ID。
