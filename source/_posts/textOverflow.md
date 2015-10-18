title: textOverflow
date: 2015-09-09 14:57:42
tags: css
---
-webkit-line-clamp：是一个不规范的属性，限制在一个块元素显示的文本行数。
{% codeblock %}
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box/-webkit-flexbox;
-webkit-line-clamp: 2;
-webkit-box-orient: vertical;
// 兼容方案
p::after {
  content:"...";
  font-weight:bold;
  position:absolute;
  bottom:0;
  right:0;
  padding:0 20px 1px 45px;
}
{% endcodeblock %}
