title: jsonP
date: 2015-09-06 20:51:22
tags:
---
jsonP的实现原理：
------
利用script标签，通过特定的src地址的调用，来执行一个客户端的js函数，在服务器端生成相对的数据，以参数形式传给客户端的js函数，并执行此函数。

url?callback=?
1、客户端注册一个callback，将callback的名字传给服务器，服务器生成json数据，以js语法生成一个function，function名字是传递上来的参数callback
2、json数据直接以入参的方式，放到function中，生成一段js语法的文档，返回给客户端，客户端解析script标签 ，并执行返回的javascript文档，数据作为参数，传入客户端预先定义好的callback函数里。