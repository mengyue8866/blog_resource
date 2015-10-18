title: javascript好用的API
date: 2015-09-08 11:09:28
tags:
---
Function.prototype.bind
bind() 最简单的用法是创建一个函数，使这个函数不论怎么调用都有同样的 this 值。
{% codeblock %}
this.x = 9; 
var module = {
  x: 81,
  getX: function() { return this.x; }
};

module.getX(); // 81

var getX = module.getX;
getX(); // 9, 因为在这个例子中，"this"指向全局对象

// 创建一个'this'绑定到module的函数
var boundGetX = getX.bind(module);
boundGetX(); // 81
{% endcodeblock %}
>IE8及以下版本中不支持

不支持的替代方案：
{% codeblock %}
if (!Function.prototype.bind) {
  Function.prototype.bind = function (oThis) {
    if (typeof this !== "function") {
      // closest thing possible to the ECMAScript 5 internal IsCallable function
      throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var aArgs = Array.prototype.slice.call(arguments, 1), 
        fToBind = this, 
        fNOP = function () {},
        fBound = function () {
          return fToBind.apply(this instanceof fNOP && oThis
                                 ? this
                                 : oThis || window,
                               aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
}
{% endcodeblock %}

>this指向是在运行函数时确定的，而不是定义函数时候确定的，最初的做法是缓存上下文对象，
  {% codeblock %}
  function Person(name){
    this.nickname = name;
    this.distractedGreeting = function() {
      var self = this; // <-- 注意这一行!
      setTimeout(function(){
        console.log("Hello, my name is " + self.nickname); // <-- 还有这一行!
      }, 500);
    }
  }

  var alice = new Person('Alice');
  alice.distractedGreeting();
  {% endcodeblock %}
解析字符串对象
我们都知道，JavaScript对象可以序列化为JSON，JSON也可以解析成对象，但是问题是如果出现了一个既不是JSON也不是对象的”东西”，转成哪一方都不方便，那么eval就可以派上用场
{% codeblock %}
var obj = "{a:1,b:2}"; // 看起来像对象的字符串
eval("("+ obj +")") // {a: 1, b: 2}
{% endcodeblock %}
>因为 eval 可以执行字符串表达式，我们希望将 obj 这个字符串对象 执行成真正的对象，那么就需要用eval。但是为了避免eval 将带 {} 的 obj 当语句来执行，我们就在obj的外面套了对 ()，让其被解析成表达式。

& (按位与)
判断一个数是否为2的n次幂，可以将其与自身减一相与
{% codeblock %}
var number = 4
(number & number -1) === 0 // true
{% endcodeblock %}

^ (按位异或)
不同第三个变量，就可以交换两个变量的值
{% codeblock %}
var a = 4,b = 3
a = a ^ b // 7
b = a ^ b // 4
a = b ^ a // 3
{% endcodeblock %}

想得到format后的时间？现在不用再get年月日时分秒了，三步搞定
{% codeblock %}
var temp = new Date();
var regex = /\//g;
(temp.toLocaleDateString() + ' ' + temp.toLocaleTimeString().slice(2)).replace(regex,'-');
{% endcodeblock %}

想将format后的时间转换为时间对象？直接用Date的构造函数
{% codeblock %}
new Date("2015-5-7 9:04:10");
{% endcodeblock %}


想将一个标准的时间对象转换为unix时间戳？valueOf搞定之
{% codeblock %}
(new Date).valueOf();
// 快速得到时间戳
+new Date
{% endcodeblock %}

快速转换为数字类型
{% codeblock %}
var number = "23"
typeof +number // number
"100" | 0
"100" >> 0
"100" << 0
"100" >> 0
"100" >>> 0 //仅适用于正数
/**
 * Firefox对位操作进行了优化，运行的代码比parseInt和+运算速度快约99％。
 * 而Chrome显然对位运算符没有偏爱，他们比parseInt函数还慢62％。
 * parseFloat比+运算符在两种浏览器（Firefox 28％，Chrome 39％）上都要快。
 */
{% endcodeblock %}

获取原型主的属性和方法：
Object.create(Shape.prototype)

重组对象：
删除操作比分配一个null属性慢很多,但delete可以改变对象结构，也可以防止对象的内存泄漏的情况。

后面添加属性：
最好一开始定义好对象的架构，后面再追加的属性，比原型定义要慢很多。

字符串处理：
+=运算比+快很多，他们比String.prototype.concat和Array.prototype.join都更快，Array.prototype.join是最慢的。

正则表达式方法：
test方法最快，indexOf方法返回索引位置
exec方法和match速度差不多，

限制声明/传递变量的范围：
你调用一个函数，浏览器必须做一些所谓的范围查找，它的昂贵程度取决于它要查找多少范围。尽量不要依辣全局/高范围的变量，尽量使局部范围变量，并将它们传递给函数。更少的范围查找，更少的牺牲速度。

jquery的使用：
对于原生比较方便的属性，尽量不要用jQuery来代替。原生的速度比jQuery快100%。

splice：返回被删除元素组成的数组，或者为空数组