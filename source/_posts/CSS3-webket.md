title: CSS3 webkit
date: 2015-09-08 13:40:31
tags: css
---
-webkit-appearance是用来改变按钮和其他控件的外观

>因为Safari下默认按钮样式
{% codeblock %}
input[type="button"], input[type="submit"], input[type="reset"] {
  -webkit-appearance: push-button; //通常改为none,有很多原生样式提供。
  white-space: pre;
}
{% endcodeblock %}

-webkit-touch-callout：当你触摸并按住触摸目标时候，禁止或显示系统默认菜单，通常设置为none

-webkit-tap-highlight-color：
当用户点击iOS的Safari浏览器中的链接或JavaScript的可点击的元素时，覆盖显示的高亮颜色，该属性可以只设置透明度。