title: 浏览器输入URL
date: 2015-09-06 20:55:35
tags: 浏览器
---

输入URL之后，浏览器都做了什么？
-------
1、浏览器向DNS服务器查找输入URL对应的IP地址（DNS是分布式的）
2、DNS服务器返回网站的IP
3、根据IP地址与目标web服务器在80商品上建立TCP连接
4、向web服务器发送一个GET请求
5、服务器返回请求页面的html代码
6、显示窗口内渲染HTML
7、请求CSS、图片、js
8、关闭时终止与服务器的连接

DNS域名解析：
------
1、客户端提出域名解析请求，发送给本地的域名服务器
2、当地域名服务器收到请求，查询本地缓存，有记录就直接返回结果
3、否则，本地域名服务器把请求发给根域名服务器，根域名服务器再返回本地域名服务器一个所查询域（根的子域）的主域名服务器的地址
4、本地域名服务器向返回的主域名服务器发送请求，查询到返回，否则返回下级的域名服务器地址