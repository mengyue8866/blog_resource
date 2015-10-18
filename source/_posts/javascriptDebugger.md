title: javascriptDebugger
date: 2015-09-08 14:40:34
tags: debugger
---
1、debugger;
{% codeblock %}
if (somethingHappens) {
    debugger;
}
// 在代码中加入强制断点
{% endcodeblock%}

2、chrome支持调试节点时断开，右击节点Break on。节点删除或者当节点的属性更改或者其子树中的节点变化时。

3、Ajax断点(XHR Breakpoints)：允许当一个预期Ajax请求创建时断开
Event Lintener Breakpoints与 DOM Breakpoints同样好用

4、chrome下有Audits工具审核网站，类似于YSlow。