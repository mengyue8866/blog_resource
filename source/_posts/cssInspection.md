title: 检测css属性支持
date: 2015-09-09 07:21:25
tags: css
---
@supports：跟@media查询语句的语法相似，使用如下：

{% codeblock %}
@supports (display: flex) {
	div { display: flex; }
}

// @supports标记可以和‘not’关键字组合，用来应对不支持的情况
@supports not (display: flex) {
	div { float: left; } /* 替换样式 */
}
{% endcodeblock%}

多条件组合检测：
{% codeblock %}
/* or */
@supports (display: -webkit-flex) or
          (display: -moz-flex) or
          (display: flex) {
 
    /* use styles here */
}
 
/* and */
@supports (display: flex) and (-webkit-appearance: caret) {
 
	/* something crazy here */
}
{% endcodeblock %}

用括号将多个条件进行嵌套关联：
{% codeblock %}
/* and and or */
@supports ((display: -webkit-flex) or
          (display: -moz-flex) or
          (display: flex)) and (-webkit-appearance: caret) {
 
    /* use styles here */
}
{% endcodeblock %}

JavaScript CSS.supports，使用如下：
以两个参数：属性名，值
var supportsFlex = CSS.supports("display", "flex");
或者一个参数
var supportsFlexAndAppearance = CSS.supports("(display: flex) and (-webkit-appearance: caret)");

是否支持此方法：
var supportsCSS = !!((window.CSS && window.CSS.supports) || window.supportsCSS || false);