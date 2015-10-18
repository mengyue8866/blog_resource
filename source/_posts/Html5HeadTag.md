title: Html5HeadTag
date: 2015-09-08 13:52:11
tags:
---
1、<!DOCTYPE html>  html5不再区分大小写
2、<meta charset="utf-8">  声明编码
>之前的声明方式：<meta http-equiv="Content-Type" content="text/html; charset=utf-8">

3、声明语言：<html lang="zh-cmn-Hans">简体 <html lang="zh-cmn-Hant">繁体，单一的 zh 和 zh-CN 均属于废弃用法。
4、<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" /> 优先使用chrome与IE
5、360读到后会切致极速模式
<meta name="renderer" content="webkit"> 
<meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1">
5、禁止百度转码：
<meta http-equiv="Cache-Control" content="no-siteapp" />

6、SEO部分：
head中的title标签
定义关键词：<meta name="keywords" content="your keywords">
页面描述：<meta name="description" content="your description">
作者：<meta name="author" content="author,email address">
网页搜索引擎索引方式：<meta name="robots" content="index,follow">

7、移动端viewport
<meta name ="viewport" content ="initial-scale=1, maximum-scale=3, minimum-scale=1, user-scalable=no"> 
>width=device-width` 会导致 iPhone 5 添加到主屏后以 WebApp 全屏模式打开页面时出现黑边

可以在页面加载时最小化上下状态栏。这是一个布尔值，可以直接这样写：
<meta name="viewport" content="width=device-width, initial-scale=1, minimal-ui">

content参数：
width viewport 宽度(数值/device-width)
height viewport 高度(数值/device-height)
initial-scale 初始缩放比例
maximum-scale 最大缩放比例
minimum-scale 最小缩放比例
user-scalable 是否允许用户缩放(yes/no)

8、ios设备：
>-添加到主屏后的标题：
<meta name="apple-mobile-web-app-title" content="标题">

>-是否启用 WebApp 全屏模式：
<meta name="apple-mobile-web-app-capable" content="yes" /> 

>-设置状态栏的背景颜色：
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
在值为"apple-mobile-web-app-capable" content="yes"时生效
content 参数：
default 默认值。
black 状态栏背景是黑色。
black-translucent 状态栏背景是黑色半透明。 如果设置为 default 或 black ,网页内容从状态栏底部开始。 如果设置为 black-translucent ,网页内容充满整个屏幕，顶部会被状态栏遮挡。

>-禁止数字识自动别为电话号码
<meta name="format-detection" content="telephone=no" />

iOS 图标：
rel 参数： apple-touch-icon 图片自动处理成圆角和高光等效果。 apple-touch-icon-precomposed 禁止系统自动添加效果，直接显示设计原图。
<link rel="apple-touch-icon-precomposed" href="/apple-touch-icon-57x57-precomposed.png" />

设置像素大小：
<link rel="apple-touch-icon-precomposed" sizes="144x144" href="/apple-touch-icon-57x57-precomposed.png" />

iOS 启动画面：
iPad 的启动画面是不包括状态栏区域的。
iPad 竖屏 768 x 1004（标准分辨率）
<link rel="apple-touch-startup-image" sizes="768x1004" href="/splash-screen-768x1004.png" />

iPad竖屏(Retina)：
<link rel="apple-touch-startup-image" sizes="1536x2008" href="/splash-screen-1536x2008.png" />

iPad横屏：
<link rel="apple-touch-startup-image" sizes="1024x748" href="/Default-Portrait-1024x748.png" />

iPad 横屏 2048×1496（Retina）
<link rel="apple-touch-startup-image" sizes="2048x1496" href="/splash-screen-2048x1496.png" />

iPhone 和 iPod touch 的启动画面是包含状态栏区域的：

iPhone/iPod Touch 竖屏 320×480 (标准分辨率)
<link rel="apple-touch-startup-image" href="/splash-screen-320x480.png" /> <!-- iPhone/iPod Touch 竖屏 320x480 (标准分辨率) -->

iPhone/iPod Touch 竖屏 640×960 (Retina)
<link rel="apple-touch-startup-image" sizes="640x960" href="/splash-screen-640x960.png" /> <!-- iPhone/iPod Touch 竖屏 640x960 (Retina) -->

iPhone 5/iPod Touch 5 竖屏 640×1136 (Retina)
<link rel="apple-touch-startup-image" sizes="640x1136" href="/splash-screen-640x1136.png" /> 

添加智能 App 广告条：
<meta name="apple-itunes-app" content="app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL">

iPhone 6对应的图片大小是750×1294，iPhone 6 Plus 对应的是1242×2148 。
<link rel="apple-touch-startup-image" href="launch6.png" media="(device-width: 375px)">
<link rel="apple-touch-startup-image" href="launch6plus.png" media="(device-width: 414px)">

Android：
meta 标签，增加 theme-color来控制选项卡颜色。
<meta name="theme-color" content="#db5945">

Windows 8 磁贴颜色
<meta name="msapplication-TileColor" content="#000"/>

Windows 8 磁贴图标
<meta name="msapplication-TileImage" content="icon.png"/>

rss订阅
<link rel="alternate" type="application/rss+xml" title="RSS" href="/rss.xml" />

favicon icon
<link rel="shortcut icon" type="image/ico" href="/favicon.ico" />

关闭chrome浏览器下翻译插件
<meta name="google" value="notranslate" />


