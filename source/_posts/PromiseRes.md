title: Promise的实现
date: 2015-09-01 07:57:40
tags: js 模式
---
实现如下的流程：
{% blockquote %}
new Promise(ready).then(getTpl).then(getData).then(makeHtml).resolve();
{% endblockquote %}

先将要事务按照执行顺序依次 push 到事务队列中，push 完了之后再通过 resolve 函数启动整个流程。

整个流程的操作模型如下：
{% codeblock %}
promise(ok).then(ok_1).then(ok_2).then(ok_3).reslove(value)------+
         |         |          |          |                       |
         |         |          |          |        +=======+      |
         |         |          |          |        |       |      |
         |         |          |          |        |       |      |
         +---------|----------|----------|--------→  ok() ←------+
                   |          |          |        |   ↓   |
                   |          |          |        |   ↓   |
                   +----------|----------|--------→ ok_1()|
                              |          |        |   ↓   |
                              |          |        |   ↓   |
                              +----------|--------→ ok_2()|
                                         |        |   ↓   |
                                         |        |   ↓   |
                                         +--------→ ok_3()-----+
                                                  |       |    |       
                                                  |       |    ↓
@ Created By Barret Lee                           +=======+   exit
{% endcodeblock %}
在 resolve 之前，promise的每一个then都会将事务压入队列，返回另外一个promise对象，resolve 后，将 resolve 的值送给队列的第一个函数，第一个函数执行完毕后，将执行结果再送入下一个函数，依次执行完队列。

Promise实现代码：
{% codeblock %}
function Promise(resolver){
  // 定义队列
  this.handlers = [];
  this.status = 'pending';
  this.value = null;
  // 
  this._doPromise.call(this,resolver);
}
Promise.prototype = {
  _doPromise:function(resolver){
    var called = false, self = this;
    try{
      resolver(function(value){
          !called && (called = !0, self.resolve(value));
      }, function(reason){
          !called && (called = !0, self.reject(reason));
      });
    } catch(e) {
      !called && (called = !0, self.reject(e));
    }
  },
  resolve: function(value) {
    try{
      if(this === value){
        throw new TypeError("Promise connot be resolved by itself.");
      } else {
        value && value.then && this._doPromise(value.then);
      }
      this.status = "fulfilled";
      this.value = value;
      this._dequeue();
    } catch(e) {
      this.reject(e);
    }
  },
  reject:function(reason){
    this.status = 'rejected';
    this.value = reason;
    this._dequeue();
  },
  // 循环取出队列中的fn
  _dequeue:function(){
    var handler;
    while(this.handlers.length){
      handler = this.handlers.shift();
      this._handle(handler.thenPromise, handler.fulfilledFn, handler.rejectedFn);
    }
  },
  _handler:function(thenPromise, fulfilledFn, rejectedFn){
    var self = this;
    setTimeout(function() {
      var callback = self.status == "fulfilled" ? fulfilledFn : rejectedFn;

      if (typeof callback === 'function') {
        try {
            self.resolve.call(thenPromise, callback(self.value));
        } catch(e) {
            self.reject.call(thenPromise, e);
        }
        return;
      }

      self.status == "fulfilled" ? self.resolve.call(thenPromise, self.value) 
                       : self.reject.call(thenPromise, self.value);
    }, 1);
  },
  /**
   * 返回promise对象
   * @param  {[fn]} fulfilledFn [成功回调]
   * @param  {[fn]} rejectedFn  [失败回调]
   * @return {[object]}             [promise对象]
   */
  then: function(fulfilledFn, rejectedFn){
    var thenPromise = new Promise(function(){});
    if(this.status === 'pending'){
      var handler = {
        fulfilled: fulfilledFn,
        rejected: rejectedFn,
        deferred: thenPromise
      }
      this.handlers.push(callback);
    }else{
      this._handler(thenPromise,fulfilledFn,rejectedFn);
    }
    return thenPromise;
  }
}
// pending状态到fulfilled或者rejected状态的转换
function sleep(ms){
  return function(v){
    var p = Promise();
    setTimeout(function(){
      p.fulfilledFn(v);
    },ms||500);
    return p;
  }
}

{% endcodeblock %}


Deferred用于创建promise
{% codeblock %}
function Defferred = function(){
  this.promise = new Promise();
}
Defferred.prototype.fulfilled = function(){

}
{% endcodeblock %}