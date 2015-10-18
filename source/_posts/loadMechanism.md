title: loadMechanism
date: 2015-09-06 20:32:43
tags: load
---
一、定义L
{% codeblock %}
  L = {
    version: '3.0',
    isHybrid: !!(window.LlocalRoute),
    isInApp: !!(navigator.userAgent.match(/agentdefine/i) && (window.location.protocol != 'file:')),
    viewReady: function(){},
    notpackaged: typeof require == 'undefined'
  }
{% endcodeblock %}

初始化L配置initLizardConfig：
1、通过script引入框架Lizard.seed.js标签属性：
>-pdConfig定义每个BU的Lizard.pdConfig
>-lizardConfig定义每个BU的lizard.config


不同环境加载不同文件loadRes：
1、队列：basescripts
notpackaged的情况下，将require.js存入队列。
否则，不在Hybrid且不在App时，将Lizard.web.js存入队列。
2、在Hybrid或者APP中，定义方法Lizard.mutileLoad属性方法：
{% codeblock %}
Lizard.mutileLoad = function () {
  mutileLoad(basescripts, amdLoaderLoaded);
};
{% endcodeblock %}
否则直接执行mutileLoad(basescripts, amdLoaderLoaded)。

3、多文件加载：
{% codeblock %}
function mutileLoad(scripts, callback) {
  var len = scripts.length;
  var no = 0;
  if (!len) {
    end();
    return;
  }
  for (var i = 0; i < len; i++) {
    var url = scripts[i];
    loadScript(url, end);
  }

  function end() {
    no++;
    if (no >= len) {
      callback();
    }
  }
}
{% endcodeblock %}

4、加载AMD模块文件amdLoaderLoaded：
定义配置队列：configModel = L.notpackaged ? ['config.js'] : ['config']
{% codeblock %}
require(configModel, function(){
  var reqs = [];
  // 不同环境将不同的文件的配置项存入队列
  // 不在Hybrid但在app，cHybridAppInit与cStatic
  // 否则存入cWebAppInit
  // Hybrid存入cHybridAppInit
  // !notpackaged，存入cBaseInit
  // define，异步加载_，$，B，F
  require(['_','$'],function(){
    // 根据html的name属性指定加载资源的类型，content为资源类型的地址
    // 根据name属性，定义lizard不同的属性，指向资源地址。
    require(reqs, function () {
      if (_.isFunction(arguments[arguments.length - 1])) {
        arguments[arguments.length - 1]();
      }
    });
  })
})
{% endcodeblock%}


5、打包之后的加载：
定义cdnComboUrl = '/res/concat?f='。
comboModels = ['core.js']，根据环境将不同的文件放入队列。
将loading，warning等基础组件存入队列。
cdnComboUrl += comboModules.join(",");
document.write("<script src='" + cdnComboUrl + "'></script>");

域名+'/res/concat?f=/code/lizard/2.1/web/lizard.core.js,/code/lizard/2.1/web/lizard.web.js,/ResCRMOnline/R5/basewidget/ui.loadFailed.js,/ResCRMOnline/R5/basewidget/ui.loading.js'
去加载其他js静态资源。



safari回退js不执行的问题：
定义全局变量shown = false；
定义方法：
{% codeblock %}
window.onpageshow = function(){
  if(!shown){
    location.reload();
    show = true;
  }
}
// 为各BU提供的方法
{% endcodeblock %}