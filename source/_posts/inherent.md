title: inheritance
date: 2015-08-31 09:11:01
tags: js库
---

一、John Resig's inheritance

{% codeblock %}
(function(){
  var initializing = false,
    // 支持function toString,用正则匹配到
    fnTest = /xyz/.test(function(){xyz;})? /\b_super\b/ : /.*/;
  // 全局化定义Class
  this.Class = function(){};
  Class.extend = function(prop){
    // 父类原型属性
    var _super = this.prototype;
    initializing = true;
    // 父类原型实例
    var prototype = new this();
    initializing = true;
    for(var name in prop){
      /*
       *定义prototype，如果prop的该属性是function,父类的该属性也是function,
       *且prop的该属性中包含_super
       */
      prototype[name] = typeof prop[name] == 'function' && 
      typeof _super[name] == 'function' && fnTest.test(prop[name]) ?
      (function(key, fn){
        return function(){
          // 如果子类属性中名为_super的暂存。
          var temp = this._super;
          // 取父类中的同名属性
          this._super = _super[name];
          var ret = fn.apply(this, arguments);
          this._super = temp;
          return ret;
        }
      })(name, prop[name]) : prop[name];
    }
    function Class(){
      if(!initializing && this.init){
        this.init.apply(this, arguments);
      }
    }
    Class.newInstance = function(paramArr){
      initializing = true;
      var obj = new Class();
      initializing = false;
      obj.init.apply(obj, paramArr);
      return obj;
    }
    Class.prototype = prototype;
    Class.prototype.constructor = Class;
    Class.extend = arguments.callee;
    return Class;
  }
})
{% endcodeblock %}

二、ctrip's inheritance
{% codeblock %}
(function(){
  this.Core = function(){};
  var slice = [].slice;
  Core.Class = function(){
    if(arguments.length == 0 || arguments.length >2) throw '参数错误';
    // 定义父类
    var parent = null;
    var properties = slice.call(arguments);
    if(typeof properties[0] === 'function') parent = properties.shift();
    properties = properties[0];
    function klass = function(){
      this.__propertys__();
      this.initializing.apply(this,arguments);
    };
    klass.superclass = parent;
    klass.subclasses = [];
    var sup__propertys__ = function(){};
    // 存储配置属性
    var sub__properyts__ = properties.__propertys__ || function(){};
    if(parent){
      // 父类原型上的配置属性
      if(parent.prototype.__propertys__) sup__propertys__ = parent.prototype.__propertys__;
      // 原型继承
      var subclass = function(){};
      subclass.prototype = parent.prototype;
      klass.prototype = new subclass;
      // 将当前类，存入到父类的subclasses属性中
      parent.subclasses.push(klass);
    }
    // 父类原型属性
    var ancestor = klass.superclass && klass.superclass.prototype;
    for(var name in properties){
      var value = properties[name];
      // 个人认为在这里判断ancestor[name]存在，会更好
      if(ancestor && typeof value === 'function'){
        // 取出value的参数
        var argList = /^\s*function\s*\(([^\(\)]*?)\)\s*?\{/i.exec(value.toString())[i].replace(/\s/i,'').split(',');
        // 满足条件，重定义原型属性的方法
        if(argList[0] === '$super' && ancestor[name]){
          value = (function(methodName,fn){
            return function(){
              var scope = this;
              // 定义第一个参数，返回父类原型的同名方法
              var args = [function(){
                return ancestor[methodName].apply(scope,arguments);
              }];
              return fn.apply(this.args.concat(slice.call(arguments)));
            }
          })(name,value)
        }
      }
      // 有$super参数的，重定义该方法
      klass.prototype[name] = value;
    }
    // 如果原型上没有初始化方法，定义
    if(!klass.prototype.initialize){
      klass.prototype.initialize = function(){};
    }
    // 定义原型的配置属性
    klass.prototype.__propertys__ = function(){
      // 调用父类的配置及参数传的配置
      sup__propertys__.call(this);
      sub__properyts__.call(this);
    }
    // 继承父类的非原型属性
    for(var key in parent){
      if(parent.hasOwnProperty(key) && key !== 'prototype' && key !== 'superclass'){
        klass[key] = parent[key];
      }
    }
    klass.prototype.constructor = klass;
    return klass;
  }
});
{% endcodeblock %}