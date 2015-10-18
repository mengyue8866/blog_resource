title: create view and switch view
date: 2015-08-31 14:05:50
tags: MVC
---
pageView扩展backbone
cAbstractApp定义view加载、切换、回退、跳转---webApp/cWebViewApp/hybirdApp为其子类
-----
1、cWebApp扩展了父类的bindEvents,start,goTo,goBack,jump,judgeForward
2、webviewApp扩展了父类的bindEvents,_getCurrentView,start,goTo,goBack,startObserver,endObserver,judgeForward
3、hybridApp扩展了父类的bindEvents,start,loadFromRoute,_getCurrentView,goTo,goBack,jump,startObserver,endObserver,judgeForward
cBaseInit根据Lizard.pdConfig返回实例化abstractApp的方法。
不同环境加载的文件:
在APP，但不是hybrid方式：加载cHybridAppInit、cStatic、cBaseInit
Hybrid：加载cHybridAppInit、cBaseInit
其他方式：cWebAppInit

/**cWebInit的作用
 * define(['cBaseInit', 'cWebMember'], function(initFunc, Member){
  Member.autoLogin({
    callback: function(){
      require(['cStatic'], function(){
        initFunc();
      });     
    }
  });
})
 */
一、backbone view的实现
{% codeblock %}
/**
 * backbone view构造函数，
 * @param {[type]} options [description]
 */
var viewOptions = ['model', 'collection', 'el', 'id', 'attributes', 'className', 'tagName', 'events'];
var View = Backbone.View = function(options) {
  this.cid = _.uniqueId('view');
  options || (options = {});
  _.extend(this, _.pick(options, viewOptions));
  this._ensureElement();
  this.initialize.apply(this, arguments);
};
{% endcodeblock %}
>1、_.uniqueId('c/view')生成唯一cid；
2、配置参数options；
3、创建View实例；
4、初始化。

{% codeblock %}
_.extend(View.prototype, Events, {
  // view的默认标签
  tagName: 'div',
  // 重定义$方法，在当前view查找元素
  $: function(selector) {
    return this.$el.find(selector);
  },
  // 初始化
  initialize: function(){},
  /**
   * 
   * @return {[type]} [description]
   */
  render: function() {
    return this;
  },
  /**
   * _ensureElement 创建View结构
   * this._createElement：创建View结构
   * this.setElement：对DOM元素进行事件解绑，定义this.$el与this.el，绑定
   */
  _ensureElement: function() {
    if (!this.el) {
      var attrs = _.extend({}, _.result(this, 'attributes'));
      if (this.id) attrs.id = _.result(this, 'id');
      if (this.className) attrs['class'] = _.result(this, 'className');
      this.setElement(this._createElement(_.result(this, 'tagName')));
      this._setAttributes(attrs);
    } else {
      this.setElement(_.result(this, 'el'));
    }
  },
  _createElement: function(tagName) {
    return document.createElement(tagName);
  },
  _setAttributes: function(attributes) {
    this.$el.attr(attributes);
  },
  /**
   * 对View的DOM进行定义与事件处理
   */
  setElement: function(element) {
    this.undelegateEvents();
    this._setElement(element);
    this.delegateEvents();
    return this;
  },
  /**
   * Backbone.$ = $
   * el是Backbone.$的实例
   */
  _setElement: function(el) {
    this.$el = el instanceof Backbone.$ ? el : Backbone.$(el);
    this.el = this.$el[0];
  },
  /**
   * events格式为{"event selector": "callback"}
   * 循环events，如果method不是function，取this对象的method属性方法
   * 不存在method，跳过此次循环
   * match取出evnet与selector，_.bind的用法？？？
   */
  delegateEvents: function(events) {
    if (!(events || (events = _.result(this, 'events')))) return this;
    this.undelegateEvents();
    for (var key in events) {
      var method = events[key];
      if (!_.isFunction(method)) method = this[events[key]];
      if (!method) continue;
      // delegateEventSplitter = /^(\S+)\s*(.*)$/;
      var match = key.match(delegateEventSplitter);
      this.delegate(match[1], match[2], _.bind(method, this));
    }
    return this;
  },
  /**
   * 解除View事件委托，根据.delegateEvents与cid
   */
  undelegateEvents: function() {
    if (this.$el) this.$el.off('.delegateEvents' + this.cid);
    return this;
  },
  /**
   * 为View添加事件委托eventName，根据.delegateEvents与cid
   */
  delegate: function(eventName, selector, listener) {
    this.$el.on(eventName + '.delegateEvents' + this.cid, selector, listener);
  },
  /**
   * 解除View事件委托eventName，根据.delegateEvents与cid
   */
  undelegate: function(eventName, selector, listener) {
    this.$el.off(eventName + '.delegateEvents' + this.cid, selector, listener);
  },
  /**
   * 移除this.$el，调用Events模块的stopListening
   */
  remove: function() {
    this._removeElement();
    this.stopListening();
    return this;
  },
  _removeElement: function() {
    this.$el.remove();
  },
}
{% endcodeblock %}

二、基于backbone View的扩展
define(['libs','header', 'cGuiderService'],
  function (libs, Header, Guider) {
    var PageView = Backbone.View.extend({
      // 滚动条位置
      scrollPos: { x: 0, y: 0 },
      // 标题组件
      header: null,
      // web 环境下使用pageid
      pageid: 0,
      // hybrid 环境下使用hpageid
      hpageid: 0,
      // 页面切换时，是否要滚动至顶部
      scrollZero: true,
      // 页面切换时，是否执行onHide
      triggerShow: true,
      // 页面切换时，是否执行onHide
      triggerHide: true,
      initialize: function () {
        this.id = this.$el.attr("id");
        this.create();
      },
      // 生成头部
      _createHeader: function () {
        var hDom = $('#headerview');
        this.header = this.headerview = new Header({ 'root': hDom });
      },
      // create 方法,View首次初始化是调用
      create: function () {
        //调用子类onCreate
        this.onCreate && this.onCreate();
      },
      // view 销毁方法
      destroy: function () {
        this.$el.remove();
      },
      // View 显示时调用的方法
      show: function () {
        // fix ios 页面切换键盘不消失的bug
        document.activeElement && document.activeElement.blur();
        //生成头部
        this._createHeader();
        //调用子类onShow方法
        !this.switchByOut && this.$el.show();

        this.triggerShow && this.onShow && this.onShow();

        if (this.onBottomPull) {
          this._onWidnowScroll = $.proxy(this.onWidnowScroll, this);
          this.addScrollListener();
        }

        if (this.scrollZero) {
          window.scrollTo(0, 0);
        }
        
        this.triggerShow = true;
        this.triggerHide = true;

        //如果定义了addScrollListener,说明要监听滚动条事,此方法在cListView中实现
        this.addScrollListener && this.addScrollListener();
      },
      // View 隐藏
      hide: function () {
        //调用子类onHide方法
        this.triggerHide && this.onHide && this.onHide();
        this.removeScrollListener && this.removeScrollListener();
        this.$el.hide();
      },

      // 跨频道跳转
      jump: function (opt) {
        if (_.isString(opt)) {
          window.location.href = opt;
        } else {
          Guider.jump(opt);
        }
      },
      // 前进
      forward: function (url, opt) {
        Lizard.forward.apply(null, arguments);
      },
      // 回退至前一页面
      back: function (url, opt) {
        Lizard.back.apply(null, arguments);
      },
      // 刷新页面
      refresh: function () {

      },
      turning: function(){

      },
      // 唤醒App,要求返回一个app接受的字符串
      getAppUrl: function () {
        return "";
      },
      // 返回URL中参数的值
      getQuery: function (key) {
        return Lizard.P(key);
      },
      // 保存滚动条位置
      saveScrollPos: function () {
        this.scrollPos = {
          x: window.scrollX,
          y: window.scrollY
        };
      },

      // 恢复原滚动条位置
      restoreScrollPos: function () {
        window.scrollTo(this.scrollPos.x, this.scrollPos.y);
      },
      // 获得页面Url,hyrbid会增加一个虚拟域名
      _getViewUrl: function () {
        var url = this._hybridUrl(location.href);
        return url;
      },
      
      _hybridUrl: function(url) {
        if (Lizard.isInCtripApp)
        {
          return 'http://hybridm.ctrip.com' + this.$el.attr('page-url');
        } else {
          return url;
        }
      }
    })
    return PageView;

  });

三、view调用与切换：
{% codeblock %}
// cPageModelProcessor负责处理viewCache
define(['cPageModelProcessor', 'cUtilPerformance', 'cUtilCommon', 'UILoadingLayer', (Lizard.isHybrid || Lizard.isInCtripApp) ? 'cHybridHeader' : 'UIHeader', 'UIWarning404', 'UIAlert', 'UIToast', 'cMessageCenter', 'UIAnimation', 'cPageParser'],
  function (callModels, cperformance, utils, Loading, Header, Warning404, Alert, Toast, MessageCenter, animation) {
// cBaseInit实例化app，循环调用interface
function APP(options) {
  this.initialize(options)
}
APP.subclasses = [],

APP.defaults = {
  "mainRoot": '#main',
  "header": 'header',
  "viewport": '.main-viewport',
  "animForwardName": 'slideleft',
  "animBackwardName": 'slideright',
  "isAnim": false,
  //是否开启动画
  "maxsize": 10
}
APP.prototype = {
  ctnrViewNames: ['lizardHisCtnrView'],
  // 订阅viewReady
  viewReady: function (handler) {
    //TODO subscribe viewReady message
    MessageCenter.subscribe('viewReady', handler);
  },
  initProperty: function initProperty(options) {
    var opts = _.extend({}, APP.defaults, options || {});
    return opts;
  },
  bindEvents: function () {
    //l_wang提升响应速度
    $.bindFastClick && $.bindFastClick();
    //处理a标签
    this._handleLink();
  },
  initialize: function initialize(options) {
    // 初始化配置
    var opts = this.initProperty(options);
    this.options = opts;
    this.firstState = null;
    this.mainRoot = $(opts.mainRoot);
    this.header = $(opts.header);
    this.viewport = this.mainRoot.find(opts.viewport);
    this.curView = null;
    this.lastView = null;
    //实例化cathViews组件
    this.maxsize = opts.maxsize;
    this.animForwardName = opts.animForwardName;
    this.animBackwardName = opts.animBackwardName;
    this.isAnim = opts.isAnim;
    this.animAPIs = animation;
    this.animatName = this.animForwardName;
    // 定义基础UI
    this._loading = new Loading();
    this._alert = new Alert();
    this._confirm = new Alert();
    this._toast = new Toast();
    this._warning404 = new Warning404();
    //是否开启hashchange,false为不开启
    this.observe = false;
    this.headerView = new Header({ 'root': $('#headerview') });
    // 绑定事件
    this.bindEvents();
    this.views = {};
    this.start();
    // 订阅switchview
    MessageCenter.subscribe('switchview', function (inView, outView) {
      inView.$el.show();
    }, this);
  },
  // 链接跳转处理
  _handleLink: function _handleLink() {
    if (!Lizard.isHybrid && !utils.isSupportPushState) return;
    $('body').on('click', $.proxy(function (e) {
      var el = $(e.target);
      var needhandle = false;

      while (true) {
        if (!el[0]) break;
        if (el[0].nodeName == 'BODY') break;
        if (el.hasClass('sub-viewport')) break;

        if (el[0].nodeName == 'A') {
          needhandle = true;
          break;
        }
        el = el.parent();
      }

      if (needhandle) {
        var href = el.attr('href');
        var opts = {};
        var lizard_data = el.attr('lizard-data');

        if ((el.attr('lizard-catch') == 'off') || (href && utils.isExternalLink(href))) {
          return true;
        }
        e.preventDefault();
        if (lizard_data) {
          opts.data = JSON.parse(lizard_data);
        }
        if (el.attr('data-jumptype') == 'back') {
          this.back(el.attr('href'), opts);
        } else if (el.attr('data-jumptype') == 'forward') {
          this.goTo(el.attr('href'), opts);
        }
      }
    },
  this));
  },
  // 切换view
  switchView: function switchView(inView, outView) {
    if (outView && !document.getElementById(outView.id) && (inView && !inView.switchByOut)) {
      outView.$el.appendTo(this.viewport);
      outView.$el.hide();
    }
    if (inView && !document.getElementById(inView.id)) {
      inView.$el.appendTo(this.viewport);
      inView.$el.hide();
    }
    //inView.$el.show();
    //动画切换时执行的回调
    var switchFn;

    //此处有问题，如果inView不再的话，应该由firstState生成默认页面
    if (!inView) throw 'inview 未被实例化';

    //将T 、P的值重新设置回去
    Lizard.T.lizTmpl = inView.lizTmpl;
    Lizard.P.lizParam = inView.lizParam;

    //outView不存在的情况下就不释放动画接口
    if (outView) {
      outView.saveScrollPos();
      if (this.isAnim) {
        switchFn = this.animAPIs[this.animatName];
      }
      //switchFn = this.animAPIs[this.animatName];
      //未定义的话便使用默认的无动画
      //l_wang 此段代码需要做一个包裹，或者需要回调，否则不会执行应该执行的代码!!!
      inView.fromView = outView.config.viewName;
      if (_.indexOf(this.ctnrViewNames, inView.config.viewName) > -1) {
        MessageCenter.publish('showHisCtnrView');
        outView.hideWarning404 = function () {
          Lizard.goBack();
        }
      }
      if (switchFn && _.isFunction(switchFn)) {
        switchFn(inView, outView, $.proxy(function (inView, outView) {
          this._onSwitchEnd(inView, outView);
        },
      this));
      } else {
        inView && !inView.switchByOut && outView.hide();            
        inView.show();
        this._onSwitchEnd(inView, outView);
      }
    } else {
      //这里开始走view的逻辑，我这里不予关注
      if (_.indexOf(this.ctnrViewNames, inView.config.viewName) > -1) {
        MessageCenter.publish('showHisCtnrView');
      }
      inView.show();
      this._onSwitchEnd(inView, outView);
    }
  },
  //l_wang 既然使用消息机制，就应该全部使用，后期重构
  _onSwitchEnd: function (inView, outView) {
    if (outView != inView && !inView.switchByOut) {
      setTimeout(function () {
        outView && outView.$el && outView.$el.hide()
      }, 10);
    }
    inView.sendUbt(true);
    MessageCenter.publish('viewReady', [inView]);
  },

  showView: function (data) {
    this.loadView(data.url, data.text, data.options);
  },        
  // 根据访问的URL是webapp或者html5不同，响应的数据也不同
  // 根据服务器响应的数据，解析路由配置text/lizard-config，加载view
  loadView: function (url, html, options) {
    // 显示加载UI
    if ((Lizard.config && Lizard.config.isHideAllLoading) || options.hideloading) {
      this.hideLoading();
    }
    else {
      this.showLoading();
    }
    Lizard.loadingView = true;
    
    if (url) {
      // view是this.curView或者加载新页面
      var view = this.curView || {
          hpageid: '',
          pageid:'',
          _getViewUrl: function () {
            return "";
          },
          _hybridUrl: function() {
            if (Lizard.isInCtripApp)
            {                  
              return 'http://hybridm.ctrip.com' + url;
            } else {
              return ubtURL;
            }
          }
      };
      var ubtURL = window.location.protocol + '//' + window.location.host + (url.indexOf(Lizard.appBaseUrl) == 0?url: Lizard.appBaseUrl + url)
      if (!window.__bfi) window.__bfi = [];                    
        window.__bfi.push(['_unload', {
          page_id: Lizard.isHybrid?view.hpageid: view.pageid,
          url: view._hybridUrl(),
          refer: view.$el?view._hybridUrl(window.location.protocol + '//' + window.location.host + view.$el.attr('page-url')):document.referrer
      }]);
    }
    // cPageParser模块中的方法_initParser，负责找到script中的text/lizard-config
    // 解析出viewName与controller、参数、模板、pageUrl
    var pageConfig = Lizard._initParser(url, html);
    // 根据pageConfig获取models
    callModels(pageConfig, _.bind(function (datas, pageConfig) {
      if (_.isFunction(this.judgeForward) && !this.judgeForward(url)) {
        return;
      }
      Lizard.viewHtmlMap[renderObj.config.viewName] = html;
      this.header.html(renderObj.header);
      var renderNode = $('<DIV></DIV>').css({display: 'none'});
      // 服务端，renderObj？
      if (options.renderAt == 'server') {
        this.hideLoading(); 
      } else {
        renderNode = $(renderObj.viewport).css({display: 'none'});
      }

      if (renderObj.config.showfake && (!this.views[renderObj.config.viewName] || this.views[renderObj.config.viewName].$el.attr('page-url') != url)) {
        if (_.isObject(renderObj.config.showfake) && renderObj.config.showfake.hideloading) {
          this.hideLoading();
        }
        // renderNode与renderObj.viewport及this.viewport
        Lizard.__fakeViewNode = renderNode.appendTo(this.viewport);
        // 当前View
        this.curView && this.curView.$el.hide();
      }
      // 加载controller
      require([renderObj.config.controller||'cPageView'], _.bind(function (View) {
        if (_.isFunction(this.judgeForward) && !this.judgeForward(url)) {
          return;
        }
        // 把this.curView存入this.lastView
        if (this.curView) this.lastView = this.curView;
        if (renderObj.config.viewName && this.views[renderObj.config.viewName]) {
          // 把要加载的view，存入this.curView
          this.curView = this.views[renderObj.config.viewName];
          if (this.curView.$el.attr('page-url') != url) {
            this.curView.$el.remove();
            !renderObj.config.showfake && renderNode.appendTo(this.viewport);
            // 定义view节点
            this.curView.$el = renderNode;
            // 执行pageview中的onCreate
            this.curView.onCreate && this.curView.onCreate();
            this.curView.delegateEvents();
          }
        }
        else {
          !renderObj.config.showfake && renderNode.appendTo(this.viewport);
          // 新的View
          this.curView = new View({
            el: (options.renderAt == 'server') ? this.viewport.children().first() : renderNode
          });
          // 设置view节点上的属性url
          this.curView.$el.attr('page-url', url);
        }
        if (options.renderAt == 'server') renderNode.remove();
        // __fakeViewNode??
        Lizard.__fakeViewNode = null;
        // 定义this.curView的属性
        this.curView.text = html;
        _.extend(this.curView, _.pick(renderObj, ['datas', 'config', 'lizTmpl', 'lizParam']));
        // switchByOut？？
        if (this.curView && this.curView.switchByOut) {
          var self = this;
          // 重新定义turning
          this.curView.turning = function () {
            this.hideLoading();
            // 隐藏上一个view，发布switchview，显示当前view
            self.lastView && self.lastView.hide();
            MessageCenter.publish('switchview', [self.curView, self.lastView]);
            self.curView.$el.show();
          }
        }else {
          this.hideLoading();
          MessageCenter.publish('switchview', [this.curView, this.lastView]);
        }
        this.curView.lastViewId = this.curView.referrer = (this.lastView && this.lastView.config.viewName);
        this.switchView(this.curView, this.lastView);
        if (renderObj.config.viewName) {
          this.views[renderObj.config.viewName] = this.curView;
        }
      }, this))
    }, this), _.bind(function (datas, errorBack) {
      this.hideLoading();
      var errorData =
      {
        callback: function () {
          this.hideWarning404();
          Lizard.goTo(url);
        },
        headData: {
          title: '网络不给力',
          back: true,
          events: {
            returnHandler: function () {
              Lizard.back();
            }
          }
        }
      };
      if (errorBack) errorData = _.extend(errorData, errorBack(datas));
      this.showWarning404(_.bind(errorData.callback, this), pageConfig, errorData);
    }, this));
  },
  // 显示一个全局遮盖层
  showHisCtnrView: function (onShow, onHide, options) {
    if (!this.curView && !options.pageConfig) return;
    if (!this.curView && options.pageConfig) options.addToHistory = false;
    var oldAnimFlag = this.isAnim, oldAnimName = this.animatName;
    if (this.curView) {
      this.curView.triggerShow = this.curView.triggerHide = options?(!options.triggerFlag):true;
      this.curView.triggerHide = options && ('triggerHide' in options) ?(options.triggerHide):true;
    }
    this.isAnim = (options && options.isAnim)?true:this.isAnim;
    if (this.isAnim) {
      this.animatName = this.animForwardName;
    }
    var config = _.clone(options.pageConfig?options.pageConfig:this.curView.config);
    config.model.apis = [];
    config.view = { viewport: '' };
    config.controller = 'cPageView';
    config.viewName = options && options.viewName?options.viewName:'lizardHisCtnrView';
    if (_.indexOf(this.ctnrViewNames, config.viewName) == -1)
    {
      this.ctnrViewNames.push(config.viewName);
    }
    var url = options.pageConfig?options.pageConfig.pageUrl:this.curView.config.pageUrl;
    if (!options || options.addToHistory !== false) {
      if (Lizard.isHybrid || Lizard.isInCtripApp)
      {
        this.endObserver();   
        window.location.hash = '/lizardHisCtnrView';
      } else {
        history.pushState({url: url, text: ' <SCRIPT type="text/lizard-config">' + JSON.stringify(config) + '<' + '/SCRIPT>', options: {pushState: true}}, document.title, url);
      }
    }
    this.loadView(url, ' <SCRIPT type="text/lizard-config">' + JSON.stringify(config) + '<' + '/SCRIPT>', { pushState: true, hideloading: true });
    if (Lizard.isHybrid || Lizard.isInCtripApp)
    {
      setTimeout(_.bind(this.startObserver, this), 1);
    }
    var headData = {};
    MessageCenter.unsubscribe('showHisCtnrView');
    MessageCenter.subscribe('showHisCtnrView', function () {
      var self = this;
      this.lizardHisCtnrView = this.curView;
      this.curView.onShow = function () {
        if (self.lastView && !_.isEmpty(self.lastView.header.datamodel)) {
          headData = _.clone(self.lastView.header.datamodel);
          headData.events.returnHandler = function () { history.back(); };
        }
        this.header.set(headData);
        onShow && onShow.apply(this, arguments);
        setTimeout(function(){
          self.animatName = self.animBackwardName;
        }, 10);            
      };
      this.curView.onHide = function () {
        onHide && onHide.apply(this, arguments);
        setTimeout(function(){
          self.isAnim = oldAnimFlag;
          self.animatName = oldAnimName;
        }, 10)
      };
      this.curView.show();
    }, this)
  },
  // 隐藏遮盖层
  hideHisCtnrView: function()
  {
    history.back();
  },
  
  interface: function () {
    return {
      'viewReady': this.viewReady,
      // 'showMessage': this.showMessage,
      // 'hideMessage': this.hideMessage,
      // 'showConfirm': this.showConfirm,
      // 'hideConfirm': this.hideConfirm,
      // 'showWarning404': this.showWarning404,
      // 'hideWarning404': this.hideWarning404,
      // 'showToast': this.showToast,
      // 'hideToast': this.hideToast,
      // 'showLoading': this.showLoading,
      // 'hideLoading': this.hideLoading,
      'showHisCtnrView': this.showHisCtnrView,
      'hideHisCtnrView': this.hideHisCtnrView,
      "goTo": this.goTo,
      "goBack": this.goBack,
      "forward": this.goTo,
      "back": this.goBack,
      "go": this.go
    }
  }
}
})
{% endcodeblock %}