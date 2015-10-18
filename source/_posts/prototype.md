title: prototype
date: 2015-09-06 12:56:19
tags:
---
下面三者的关系：

一、构造函数
如
返回Object的Object.constructor构造函数
返回Object.constructor.prototype
返回Object.prototype

二、原型
1、任何一个对象（除null）都有一个__proto__指向这个对象的原型链。
2、function的实例属性__proto__指向function的prototype属性，实例的constructor属性指向该function

三、实例
查检实例中的属性：
obj.hasOwnProperty('proto')——检查实例中的属性，反应不出原型中的属性。
proto in obj——反应出实例中的属性，包括原型中的属性。

obj.isPrototypeOf()------对象是否是另一个对象的原型
obj.propertyIsEnumerable()-----检查指定的属性是否能枚举

判断是否是某个类的实例instanceof： a instanceof RegExp
>是否是正则实例

四、对象数据属性：
1、数据配置：
Configurable——1、是否能通过delete删除属性 2、修改属性的特性3、修改访问属性
Enumerable——1、能否枚举
Writable——1、能否修改属性值
Value——从这个位置读取、写入属性值。

2、定义数据属性：
Object.defineProperty(obj，'属性'，{数据属性})
Object.defineProperties(obj，{
  属性：{
      数据属性
  },
  get:function(){},
  set:function(){}
})

