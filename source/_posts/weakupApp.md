title: weakupApp
date: 2015-09-06 13:20:40
tags:
---
实现APP唤醒：
-------
body里插入一个
iframe，width,height=1
frameBorder=0
style.poisition='absolute'
style.left='-9999px'
style.top='-9999px'
append到body
iframe.src = url