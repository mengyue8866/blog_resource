title: Router and History
date: 2015-09-07 14:20:27
tags: MVC
---
一、pushState是为了实现局部刷新，同时改变浏览器URL，且支持浏览器的回退
>-之前做法：会注册hashChange事件，支持浏览器的跳转。
>-pushState：
{% codeblock %}
window.history.pushState({
  title: title,
  url: url,
  XXX: xxx
}, document.title, url)
// 或者replaceState，不会新增节点；
// URL改变了，但没有跳转
window.addEventListener("popstate", function (e) {
  var state = history.state;//e.state
  document.title = state.title;
  $('#city').html(state.title);
  var s = '';
});
{% endcodeblock %}
与之对应的是popState事件，该事件会在浏览器前进后退时候触发，他会捕捉当前URL中的data。

但pushState只支持到IE10，对于winphone为IE9的手机，兼容方案：使用iframe

基本知识普及：
>-http:\/\/为协议，后面为host，接下来为path：此部分的变化会触发服务器请求，刷新页面。
>-hash部分：不会触发服务器请求，所以最初的单页面应用以hashChange为主，监控hashChange。
>-而pushState实现了改变前面的，也不引发服务器请求，popState变化引起回调。

Backbone做了兼容性检测：支持便使用pushState，不支持便还是使用hashChange。


二、路由/View的切换

>猫点（hash）片段（#page）可以被用来提供这种链接，随着 History API 的到来，猫点已经可以用于处理标准 URLs （/page）Backbone.Router 为客户端路由提供了许多方法，并能连接到指定的动作（actions）和事件（events）,对于不支持 History API 的旧浏览器，路由提供了优雅的回调函数并可以透明的进行 URL 片段的转换。
需要调用 Backbone.history.start()，或 Backbone.history.start({pushState: true}) 来确保驱动初始化 URL 的路由。

{% codeblock %}

/**
 * 创建router，定义URL匹配到触发actions
 * Backbone.Router.extend({

  routes: {
    "help":                 "help",    // #help
    "search/:query":        "search",  // #search/kiwis
    "search/:query/p:page": "search"   // #search/kiwis/p7
  },

  help: function() {
    ...
  },

  search: function(query, page) {
    ...
  }
 */
/**
 * routes 将带参数的 URLs 映射到路由实例的方法上
 * 如果匹配一个路由，此时会触发一个基于动作名称的 event， 其它对象可以监听这个路由并接收到通知。
 *
routes: {
  "help/:page":         "help",
  "download/*path":     "download",
  "folder/:name":       "openFolder",
  "folder/:name-:mode": "openFolder"
}
router.on("route:help", function(page) {
  ...
});
或者初始化方法直接调用原型方法route，定义路由，后定义的会覆盖前面定义的。
this.route("page/:number", "page", function(number){ ... });
 */

/**
 * Router构造函数
 * 1、定义this.routes，根据参数的routes项
 * 2、绑定
 * 3、初始化
 */
var Router = Backbone.Router = function(options) {
  options || (options = {});
  if (options.routes) this.routes = options.routes;
  this._bindRoutes();
  this.initialize.apply(this, arguments);
};

_.extend(Router.prototype, Events, {
  initialize: function(){},
  /**
   * [_bindRoutes description]
   * @return {[type]} [description]
   */
  _bindRoutes: function() {
    if (!this.routes) return;
    this.routes = _.result(this, 'routes');
    var route, routes = _.keys(this.routes);
    while ((route = routes.pop()) != null) {
      this.route(route, this.routes[route]);
    }
  },
  /**
   * 创建路由匹配，路由器发器。
   * @param  {[type]}   route    [page/:number]
   * @param  {[type]}   name     [page]
   * @param  {Function} callback [fn]
   * @return {[type]}            [router对象]
   */
  route: function(route, name, callback) {
    // 
    if (!_.isRegExp(route)) route = this._routeToRegExp(route);
    // 两项参数
    if (_.isFunction(name)) {
      callback = name;
      name = '';
    }
    // 不存在callback参数
    if (!callback) callback = this[name];
    var router = this;
    // history的route
    Backbone.history.route(route, function(fragment) {
      var args = router._extractParameters(route, fragment);
      if (router.execute(callback, args, name) !== false) {
        router.trigger.apply(router, ['route:' + name].concat(args));
        router.trigger('route', name, args);
        Backbone.history.trigger('route', router, name, args);
      }
    });
    return this;
  },
  /**
   * optionalParam = /\((.*?)\)/g;  获取括号内的内容
   * namedParam    = /(\(\?)?:\w+/g;  (?及后面的所有内容
   * splatParam    = /\*\w+/g;   *及后面的内容
   * escapeRegExp  = /[\-{}\[\]+?.,\\\^$|#\s]/g;
   * [_routeToRegExp description]
   * @param  {[type]} route [description]
   * @return {[type]}       [description]
   */
  _routeToRegExp: function(route) {
    route = route.replace(escapeRegExp, '\\$&')
                 .replace(optionalParam, '(?:$1)?')
                 .replace(namedParam, function(match, optional) {
                   return optional ? match : '([^/?]+)';
                 })
                 .replace(splatParam, '([^?]*?)');
    return new RegExp('^' + route + '(?:\\?([\\s\\S]*))?$');
  },
  _extractParameters: function(route, fragment) {
    var params = route.exec(fragment).slice(1);
    return _.map(params, function(param, i) {
      // Don't decode the search params.
      if (i === params.length - 1) return param || null;
      return param ? decodeURIComponent(param) : null;
    });
  },
  /**
   * 覆盖它来执行自定义解析或包装路由
   */
  execute: function(callback, args, name) {
    if (callback) callback.apply(this, args);
  },
  /**
   * 你想保存为一个URL，  可以调用navigate以更新的URL。 
   * 如果您也想调用路由功能， 设置trigger选项设置为true。 
   * 无需在浏览器的历史记录创建条目来更新URL，  设置 replace选项设置为true。
   * app.navigate("help/troubleshooting", {trigger: true, replace: true});
   * @param  {[type]} fragment [url]
   * @param  {[type]} options  [配置]
   */
  navigate: function(fragment, options) {
    Backbone.history.navigate(fragment, options);
    return this;
  }
}

{% endcodeblock %}

{% codeblock %}
/**
 * start方法来开始路由监听
 * History构造函数：处理 hashchange 事件或 pushState，创建了键值对的路由，history会自动创建。
 * 1、定义handlers
 * 2、绑定checkUrl
 * 定义location及history
 */
var History = Backbone.History = function() {
  this.handlers = [];
  _.bindAll(this, 'checkUrl');

  // Ensure that `History` can be used outside of the browser.
  if (typeof window !== 'undefined') {
    this.location = window.location;
    this.history = window.history;
  }
};
_.extend(History.prototype, Events, {
  interval: 50,
  /**
   * [checkUrl description]
   * @param  {[type]} e [description]
   * @return {[type]}   [description]
   */
  checkUrl: function(e) {
    var current = this.getFragment();
    // iframe方式
    if (current === this.fragment && this.iframe) {
      current = this.getHash(this.iframe);
    }
    if (current === this.fragment) return false;
    if (this.iframe) this.navigate(current);
    this.loadUrl();
  },
  /**
   * 根据pushState与hashchange，返回不同的URL
   */
  getFragment: function(fragment) {
    if (fragment == null) {
      if (this._hasPushState || !this._wantsHashChange) {
        fragment = this.getPath();
      } else {
        fragment = this.getHash();
      }
    }
    // routeStripper = /^[#\/]|\s+$/g
    return fragment.replace(routeStripper, '');
  },
  /**
   * 返回pushState情况下的路径
   */
  getPath: function() {
    var path = decodeURI(this.location.pathname + this.getSearch());
    var root = this.root.slice(0, -1);
    if (!path.indexOf(root)) path = path.slice(root.length);
    return path.slice(1);
  }, 
  /**
   * 获取锚点
   */
  getHash: function(window) {
    var match = (window || this).location.href.match(/#(.*)$/);
    return match ? match[1] : '';
  },
  /**
   * 删除锚点，获取参数。
   */
  getSearch: function() {
    var match = this.location.href.replace(/#.*/, '').match(/\?.+/);
    return match ? match[0] : '';
  },
  start: function(options) {
    if (History.started) throw new Error('Backbone.history has already been started');
    History.started = true;
    // 初始化配置
    this.options          = _.extend({root: '/'}, this.options, options);
    this.root             = this.options.root;
    this._wantsHashChange = this.options.hashChange !== false;
    this._hasHashChange   = 'onhashchange' in window;
    this._wantsPushState  = !!this.options.pushState;
    this._hasPushState    = !!(this.options.pushState && this.history && this.history.pushState);
    this.fragment         = this.getFragment();

    //定义事件监听
    var addEventListener = window.addEventListener || function (eventName, listener) {
      return attachEvent('on' + eventName, listener);
    };

    // 确保root以/开始结束，rootStripper = /^\/+|\/+$/g
    this.root = ('/' + this.root + '/').replace(rootStripper, '/');

    // 如果不支持hashchange事件，或者用户想用hashchange，但没有设置pushState
    // 用iframe代理处理location
    if (!this._hasHashChange && this._wantsHashChange && (!this._wantsPushState || !this._hasPushState)) {
      var iframe = document.createElement('iframe');
      iframe.src = 'javascript:0';
      iframe.style.display = 'none';
      iframe.tabIndex = -1;
      var body = document.body;
      // 如果document没有准备好，IE9以下用appendChild会报错
      this.iframe = body.insertBefore(iframe, body.firstChild).contentWindow;
      // 更新URL
      this.navigate(this.fragment);
    }

    // 1、支持pushState
    // 2、使用hashchange，也支持，且不存在iframe
    // 3、想要用hashChange，隔一时间去检查URL
    if (this._hasPushState) {
      addEventListener('popstate', this.checkUrl, false);
    } else if (this._wantsHashChange && this._hasHashChange && !this.iframe) {
      addEventListener('hashchange', this.checkUrl, false);
    } else if (this._wantsHashChange) {
      this._checkUrlInterval = setInterval(this.checkUrl, this.interval);
    }

    if (this._wantsHashChange && this._wantsPushState) {

      // 设置了使用 `pushState`，便浏览器不支持，重定向到新的URL
      if (!this._hasPushState && !this.atRoot()) {
        this.location.replace(this.root + '#' + this.getPath());
        return true;

      // Or if we've started out with a hash-based route, but we're currently
      // in a browser where it could be `pushState`-based instead...
      } else if (this._hasPushState && this.atRoot()) {
        this.navigate(this.getHash(), {replace: true});
      }

    }

    if (!this.options.silent) return this.loadUrl();
  },
  /**
   * 修正路径，并返回是否在根目录
   */
  atRoot: function() {
    // 如果路径的最后不是以/结束，要加上
    var path = this.location.pathname.replace(/[^\/]$/, '$&/');
    return path === this.root && !this.getSearch();
  },
  // Disable Backbone.history, perhaps temporarily. Not useful in a real app,
  // but possibly useful for unit testing Routers.
  stop: function() {
    // Add a cross-platform `removeEventListener` shim for older browsers.
    var removeEventListener = window.removeEventListener || function (eventName, listener) {
      return detachEvent('on' + eventName, listener);
    };

    // Remove window listeners.
    if (this._hasPushState) {
      removeEventListener('popstate', this.checkUrl, false);
    } else if (this._wantsHashChange && this._hasHashChange && !this.iframe) {
      removeEventListener('hashchange', this.checkUrl, false);
    }

    // Clean up the iframe if necessary.
    if (this.iframe) {
      document.body.removeChild(this.iframe.frameElement);
      this.iframe = null;
    }

    // Some environments will throw when clearing an undefined interval.
    if (this._checkUrlInterval) clearInterval(this._checkUrlInterval);
    History.started = false;
  },
  route: function(route, callback) {
    this.handlers.unshift({route: route, callback: callback});
  },
  // Attempt to load the current URL fragment. If a route succeeds with a
  // match, returns `true`. If no defined routes matches the fragment,
  // returns `false`.
  loadUrl: function(fragment) {
    fragment = this.fragment = this.getFragment(fragment);
    return _.any(this.handlers, function(handler) {
      if (handler.route.test(fragment)) {
        handler.callback(fragment);
        return true;
      }
    });
  },
  /**
   * [navigate description]
   * @param  {[type]} fragment [description]
   * @param  {[type]} options  [description]
   * @return {[type]}          [description]
   */
  navigate: function(fragment, options) {
    if (!History.started) return false;
    if (!options || options === true) options = {trigger: !!options};

    var url = this.root + (fragment = this.getFragment(fragment || ''));

    // Strip the hash and decode for matching.
    fragment = decodeURI(fragment.replace(pathStripper, ''));

    if (this.fragment === fragment) return;
    this.fragment = fragment;

    // Don't include a trailing slash on the root.
    if (fragment === '' && url !== '/') url = url.slice(0, -1);

    // If pushState is available, we use it to set the fragment as a real URL.
    if (this._hasPushState) {
      this.history[options.replace ? 'replaceState' : 'pushState']({}, document.title, url);

    // If hash changes haven't been explicitly disabled, update the hash
    // fragment to store history.
    } else if (this._wantsHashChange) {
      this._updateHash(this.location, fragment, options.replace);
      if (this.iframe && (fragment !== this.getHash(this.iframe))) {
        // Opening and closing the iframe tricks IE7 and earlier to push a
        // history entry on hash-tag change.  When replace is true, we don't
        // want this.
        if(!options.replace) this.iframe.document.open().close();
        this._updateHash(this.iframe.location, fragment, options.replace);
      }

    // If you've told us that you explicitly don't want fallback hashchange-
    // based history, then `navigate` becomes a page refresh.
    } else {
      return this.location.assign(url);
    }
    if (options.trigger) return this.loadUrl(fragment);
  },

  // Update the hash location, either replacing the current entry, or adding
  // a new one to the browser history.
  _updateHash: function(location, fragment, replace) {
    if (replace) {
      var href = location.href.replace(/(javascript:|#).*$/, '');
      location.replace(href + '#' + fragment);
    } else {
      // Some browsers require that `hash` contains a leading #.
      location.hash = '#' + fragment;
    }
  }
}

{% endcodeblock %}
