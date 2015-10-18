title: javascript发布与订阅
date: 2015-09-07 15:38:51
tags: 设计模式
---
define([], function () {
	// 消息存储键值对
  var messageQueue = {};
  var MessageCenter = {
    /**
     * 发布消息
     * @method Common.cMessageCenter.publish
     * @param {string} message 消息标示
     * @param {array} args 参数
     */
    publish: function(message, args)
    {
      if (messageQueue[message])
      {
        _.each(messageQueue[message], function(item){
          item.handler.apply(item.scope?item.scope: window, args);  
        });
      }  
    },
    // loadView方法中，发布切换view,MessageCenter.publish('switchview', [self.curView, self.lastView]);
    // switchView方法中，MessageCenter.publish('showHisCtnrView');
    // MessageCenter.publish('showHisCtnrView');
    // _onSwitchEnd方法中，MessageCenter.publish('viewReady', [inView]);

    /**
     * 订阅消息
     * @method Common.cMessageCenter.subscribe
     * @param {string} message 消息标示
     * @param {function} handler 消息处理
     * @param {object} [scope] 函数作用域
     */
    subscribe: function(message, handler, scope)
    {
      if (!messageQueue[message]) messageQueue[message] = [];
      messageQueue[message].push({scope: scope, handler: handler});
    },
    // viewReady方法中订阅MessageCenter.subscribe('viewReady', handler);
    // initialize方法中订阅MessageCenter.subscribe('switchview', fn)

    /**
     * 取消订阅
     * @method Common.cMessageCenter.subscribe
     * @param {string} message 消息标示
     * @param {function} handler 消息处理函数句柄
     */
    unsubscribe: function(message, handler)
    {
      if (messageQueue[message]) {
        if (handler) {
          messageQueue[message] = _.reject(messageQueue[message], function(item){ return item.handler !=  handler});
        } else {
          delete messageQueue[message];
        }
      }
    }
  }
  // showHisCtnrView方法中，取消MessageCenter.unsubscribe('showHisCtnrView');
  // 并从新订阅MessageCenter.subscribe('showHisCtnrView',fn)
  return MessageCenter;
});