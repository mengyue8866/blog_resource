title: javascript常用方法实现讲解
date: 2015-09-10 17:51:08
tags: javascript
---
一、setTimeout和setInterval
----
>**注意：**定时处理不是ECMAScript 的标准，它们在DOM (文档对象模型) 被实现。
{% codeblock %}
function foo() {}
var id = setTimeout(foo, 500); // 返回一个大于零的数字
{% endcodeblock %}
当 setTimeout 被调用时，它会返回一个 ID 标识并且计划在将来大约 1000 毫秒后调用 foo 函数。

基于 JavaScript 引擎的计时策略，以及本质上的单线程运行方式，所以其它代码的运行可能会阻塞此线程。 因此没法确保函数会在 setTimeout 指定的时刻被调用。


setInterval的堆调用：
不鼓励使用：当回调函数的执行被阻塞时，setInterval 仍然会发布更多的回调指令。在很小的定时间隔情况下，这会导致回调函数被堆积起来。

可以使用setTimeout自调用来替代上面的方法。

清除定时器：
通过将定时产生的ID标识，传递给clearTimeout和clearInterval来清除。

setTimeout和setInterval第一个参数也接受字符串，内部使用了eval。

{% codeblock %}
function foo() {
    // 将会被调用
}
 
function bar() {
    function foo() {
        // 不会被调用
    }
    setTimeout('foo()', 1000);
}
bar();
{% endcodeblock %}
>**注：**由于 eval 在这种情况下不是被直接调用，因此传递到 setTimeout 的字符串会自全局作用域中执行； 因此，上面的回调函数使用的不是定义在 bar 作用域中的局部变量 foo。

eval：在当前作用域执行一段javascript代码字符串。
{% codeblock %}
var foo = 1;
function test() {
    var foo = 2;
    eval('foo = 3');
    return foo;
}
test(); // 3
foo; // 1
{% endcodeblock %}

但是 eval 只在被直接调用并且调用函数就是 eval 本身时，才在当前作用域中执行。
{% codeblock %}
var foo = 1;
function test() {
    var foo = 2;
    var bar = eval;
    bar('foo = 3');
    return foo;
}
test(); // 2
foo; // 3
{% endcodeblock %}

{% codeblock %}
// 写法一：直接调用全局作用域下的 foo 变量
var foo = 1;
function test() {
    var foo = 2;
    window.foo = 3;
    return foo;
}
test(); // 2
foo; // 3
 
// 写法二：使用 call 函数修改 eval 执行的上下文为全局作用域
var foo = 1;
function test() {
    var foo = 2;
    eval.call(window, 'foo = 3');
    return foo;
}
test(); // 2
foo; // 3
{% endcodeblock %}

自动插入分号，是javascript语言最大的设计缺陷之一，因为它能改变代码的行为。

前置括号：
{% codeblock %}
// 不会自动加分号
log('testing!')
(options.list || []).forEach(function(i) {})

// 会被解析器为一行
log('testing!')(options.list || []).forEach(function(i) {})
{% endcodeblock %}

undefined：
可定义一个变量：
var k = undefined //不是常量，也不是关键字

但在ES5严格模式下，undefined不再是可写的，但名称仍可以被隐藏，比如定义一个函数名为undefind
以下情况返回的为undefined：
>-访问未修改的全局变量 undefined。
>-由于没有定义 return 表达式的函数隐式返回。
>-return 表达式没有显式的返回任何内容。
>-访问不存在的属性。
>-函数参数没有被显式的传递值。
>-任何被设置为 undefined 值的变量。


