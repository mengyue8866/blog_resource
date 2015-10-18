title: AppCache
date: 2015-09-10 14:01:04
tags: HTML5
---
AppCache：就是对app内存缓存的方案，具体表现为当请求某个文件时不是从网络获取该文件，而是从本地获取。
优点：
>-离线浏览 - 用户可在应用离线时使用它们
>-速度 - 已缓存资源加载得更快
>-减少服务器负载 - 浏览器将只从服务器下载更新过或更改过的资源

AppCache实现：
添加文件index.appcache,内容为：
{% codeblock %}
CACHE MANIFEST
// 要缓存的文件
index.html
index.css
index.js
logo.jpg
{% endcodeblock %}
修改index.html
{% codeblock %}
<html manifest="index.appcache">
{% endcodeblock %}

第一次请求会把所有文件都拉下来， 同时会请求index.appcache文件。
这时修改index.html文件再次请求并不会更新index.html文件， 说明index.html已经缓存，并不会从server端获取。

更新文件的条件：
>-用户清空浏览器缓存
>-manifest 文件被修改
>-由程序来更新应用缓存

>**注：** 更新index.appcache会导致所有文件都重新请求， 但当次并不生效而是要等到下一次请求才能生效， 当次显示还是采用之前缓存过得文件。

CACHE MANIFEST：下面列出的是首次下载进行缓存
NETWORK：下面列出的是不会被缓存的
FALLBACK：下面列出的是无法访问时的回退页面

>**注意：** 主页一定会被缓存起来的，因为AppCache主要是用来做离线应用的，如果主页不缓存就无法离线插件了，因此把index.html添加到NETWORK中是不起效果的。

AppCache的缺点及改进点：
1、对manifest文件更新，会重新请求所有文件，实际上可能只更新了很少量文件。（ 虽然重新请求资源会返回304， 但每个文件还会发起请求）；
针对此点可以只更新需要更新的文件， 比如可以建立一个文件版本或者MD5映射，对相同版本或者MD5不再请求
2、manifest文件每次都会请求，我们可以按照一定时间更新一次，或者启动时更新一次，以减少manifest文件更新次数
3、对hybrid方案还可以在打开app之前预缓存，提前下载文件或者更新manifest文件。