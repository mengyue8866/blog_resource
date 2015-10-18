title: 淘宝的rem响应式
date: 2015-10-15 10:46:04
tags: css
---
1、在HTML标签上定义data-dpr与font-size，计算原理：设计图为750px，把其分成100份来看，根元素为75px，dpr不同，但viewport获取到的是375px；此时我们写的1px实际是2px；
    >-不同的屏目，HTML的data-dpr，font-size的值不同，viewport的initial-scale等的值不同
    >-iphone4,5的data-dpr为2，font-size为64，viewport的initial-scale等的值0.5
    >-iphone6的data-dpr为2，font-size为75，viewport的initial-scale等的值0.5
    >-iphone6 plus的data-dpr为3，font-size为124.2，viewport的initial-scale等的值0.33333333333
    >-Android的data-dpr为1，font-size随屏目大小变化，viewport的initial-scale等的值1

    >-随着data-dpr与viewport的initial-scale等的值的变化，body的font-size也会不同，Android为12,iphone4-6为24,6plus为36，body的宽高也不同。
    >-每个元素的宽、高、行高的计算方式：

2、meta标签，通过其name与content属性，来实现一些定义。
3、meta的name为data-spm，content来记录加载页面所用的时间
4、link标签的rel为dns-prefetch，其中的href的作用？？？rel为apple-touch-icon-precomposed，为Shortcut Icon
且type为image/x-icon的作用？？？
5、script的async
6、文件的压缩、加载：携程url?f=/path/a.js,b.js；淘宝：用script的filePath加载js,url??path/a.js,b.js；但其他所有路径以//开始？？？
7、设备的逻辑像素与物理像素DPR，iphone的逻辑像素为375px，物理像素为750px
