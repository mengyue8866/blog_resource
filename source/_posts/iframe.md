title: iframe
date: 2015-09-10 10:56:51
tags:
---
一、浏览器窗口与iframe
在子窗口判断顶层窗口，跳转：
{% codeblock %}
(function () {
    if (window != window.top) {
        // 不起效时，可以尝试var location = document.location;
        window.top.location.replace(window.location); //或者干别的事情
    }
})();
{% endcodeblock %}

使用HTTP 响应头信息中的 X-Frame-Options属性(服务器也需要进行相应的配置)：
>-DENY：浏览器拒绝当前页面加载任何Frame页面
>-SAMEORIGIN：frame页面的地址只能为同源域名下的页面
>-ALLOW-FROM：origin为允许frame加载的页面地址

原理：
>-在window内部的子window(如iframe)的history发生变化时，点回退接钮时会对子window回退。
>-一个隐藏的iframe,通过改变这个iframe的history实现点回退时界面的回退。
>-window有个onhashchange事件，即window.location.hash发生变化时会触发，因此，可以将要加载的页面放置到location.hash上面，这样在点回退时根据此hash回退到过去的页面

当然也可以用一个栈来记录详细的参数，页hash对应一个key值，在onhashchange时根据key取出对应的参数来加载历史页面。

二、iframe的应用：
{% codeblock %}
// 作URL跳转
var loadURL = function(url) {
  if (url.length == 0) {
      return;
  }

  var iframe = document.createElement("iframe");
  var cont = document.body || document.documentElement;

  iframe.style.display = "none";
  iframe.setAttribute('src', url);
  cont.appendChild(iframe);

  setTimeout(function(){
      iframe.parentNode.removeChild(iframe);
      iframe = null;
  }, 200);
}
{% endcodeblock %}


主页面和其中的 iframe 是共享这些连接的。这意味着 iframe 在加载资源时可能用光了所有的可用连接，从而阻塞了主页面资源的加载。
在主页面上重要的元素加载完毕后，再动态设置 iframe 的 SRC。