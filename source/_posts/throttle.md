title: throttle与debounce
date: 2015-09-10 16:32:06
tags:
---

throttle(函数节流)：函数调用的频度控制器，是连续执行时间间隔控制。
--------
应用场景：
>-鼠标移动，mousemove 事件。
>-DOM 元素动态定位，window对象的resize和scroll 事件。

优化window的resize和scroll事件：
{% codeblock %}
// 一个简单的throttle
var resizeTimer=null;
$(window).on('resize',function(){
 if(resizeTimer){
   clearTimeout(resizeTimer)
 }
  resizeTimer=setTimeout(function(){
   console.log("window resize");
  },400);
});
{% endcodeblock %}

debounce：空闲时间必须大于或等于 一定值的时候，才会执行调用方法。
--------
应用场景：
文本输入keydown 事件，keyup 事件，例如做autocomplete，这时需要我们很好的控制输入文字时调用方法时间间隔。一般时第一个输入的字符马上开始调用，根据一定的时间间隔重复调用执行的方法。对于变态的输入，比如按住某一个建不放的时候特别有用。

{% codeblock %}
/*
* 频率控制 返回函数连续调用时，fn 执行频率限定为每多少时间执行一次
* @param fn {function}  需要调用的函数
* @param delay  {number}    延迟时间，单位毫秒
* @param immediate  {bool} 给 immediate参数传递false 绑定的函数先执行，而不是delay后后执行。
* @return {function}实际调用函数
*/
var throttle = function (fn,delay, immediate, debounce) {
 var curr = +new Date(),//当前事件
     last_call = 0,
     last_exec = 0,
     timer = null,
     diff, //时间差
     context,//上下文
     args,
     exec = function () {
         last_exec = curr;
         fn.apply(context, args);
     };
 return function () {
   curr= +new Date();
   context = this,
   args = arguments,
   diff = curr - (debounce ? last_call : last_exec) - delay;
   clearTimeout(timer);
   if (debounce) {
     if (immediate) {
         timer = setTimeout(exec, delay);
     } else if (diff >= 0) {
         exec();
     }
   } else {
     if (diff >= 0) {
         exec();
     } else if (immediate) {
         timer = setTimeout(exec, -diff);
     }
   }
   last_call = curr;
 }
};
 
/*
* 空闲控制 返回函数连续调用时，空闲时间必须大于或等于 delay，fn 才会执行
* @param fn {function}  要调用的函数
* @param delay   {number}    空闲时间
* @param immediate  {bool} 给 immediate参数传递false 绑定的函数先执行，而不是delay后后执行。
* @return {function}实际调用函数
*/
 
var debounce = function (fn, delay, immediate) {
   return throttle(fn, delay, immediate, true);
};
{% endcodeblock %}
underscore也有这两个方法的实现

