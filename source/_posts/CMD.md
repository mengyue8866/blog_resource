title: AMD与CMD及其他模块化工具
date: 2015-09-06 06:40:45
tags: js模块化
---
实现浏览器的模块化加载，解析模块依赖关系，自动加载相关模块。
Node支持CommonJs
AMD是CommonJs的分支：AMD采用异步加载模块，模块加载不影响后面语句运行。所有依赖某些模块的语句均放置在回调函数中。

AMD 是提前执行，CMD 是延迟执行。
CMD 推崇依赖就近，AMD 推崇依赖前置

styleCombine：
---------
将 HTML 页面上的多个 js/css 请求自动地合并成一个请求，发送给combo服务器。                                                                      

对于入口的 AMD/CMD 模块，能够自动解析出模块的深层依赖关系，并将所依赖文件及页面上的其它 js 文件合并为一个请求发送。         

对 HTML 页面中每个 js/css 链接都会根据文件内容自动地添加版本号后缀，js/css 内容更新将触发版本号的实时更新，使得浏览器端缓存或 CDN 缓存能够强制失效。

Browserify：
-----

让浏览器支持CommonJs：
>浏览器不兼容CommonJS的根本原因，在于缺少四个Node.js环境的变量module exports require global，只要能够提供这四个变量，浏览器就能加载 CommonJS 模块。

{% codeblock %}
var module = {
  exports: {}
};

(function(module, exports) {
  exports.multiply = function (n) { return n * 1000 };
}(module, module.exports))

var f = module.exports.multiply;
f(5) // 5000 
{% endcodeblock %}

>一个立即执行函数提供 module 和 exports 两个外部变量，模块就放在这个立即执行函数里面。模块的输出值放在 module.exports 之中，这样就实现了模块的加载

Browserify 的实现：
---------
例子：
{% codeblock %}
// foo.js
module.exports = function(x) {
  console.log(x);
};

// main.js
var foo = require("./foo");
foo("Hi");
{% endcodeblock %}
命令：browserify main.js > compiled.js *将main.js转为浏览器可用的格式*

安装一下browser-unpack：npm install browser-unpack -g

将前面生成的compile.js解包：browser-unpack < compiled.js

{% codeblock %}
[
  {
    "id":1,
    "source":"module.exports = function(x) {\n  console.log(x);\n};",
    "deps":{}
  },
  {
    "id":2,
    "source":"var foo = require(\"./foo\");\nfoo(\"Hi\");",
    "deps":{"./foo":1},
    "entry":true
  }
]
{% endcodeblock %}

>browerify 将所有模块放入一个数组，id 属性是模块的编号，source 属性是模块的源码，deps 属性是模块的依赖。
main.js 里面加载了 foo.js，所以 deps 属性就指定 ./foo 对应1号模块。执行的时候，浏览器遇到 require('./foo') 语句，就自动执行1号模块的 source 属性，并将执行后的 module.exports 属性值输出。