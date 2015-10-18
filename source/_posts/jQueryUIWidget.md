title: jQueryUIWidget
date: 2015-09-06 17:09:48
tags: jQueryUIWidget
---
jquery.ui.widget工作原理：
-----
1、jQuery.widget 被调用时，它会为你的插件创建一个构造函数， 并以插件定义体作为其 prototype。
2、所有默认提供的方法来自于一个基础的 widget prototype， 定义在 jQuery.Widget.prototype。
3、当插件实例化时， 它会被用jQuery.data 的方式保存在原来的 DOM 元素里， 以插件名作为 key 值。

定义有状态的插件。
-----
无状态插件：与插件进行交互的就限于调用插件时的那一组对象。
widget factory：管理状态，允许通过一个插件暴露多个函数，并提供多个扩展点，对应 jQuery.widget，可以独立于 jQuery UI 使用的。
参数讲解：
/**
 * name 创建的小部件名称
 * base 继承的基础小部件，必须是一个可以使用 `new` 关键词实例化的构造函数。
 * prototype 作为小部件原型使用的对象
 */
jQuery.widget( name [, base ], prototype )
{% codeblock %}
//下面的两个参数：插件名字(必须包含一个命名空间)和带有具体实现方法的对象
//_开头的，被jquery UI当作私有方法，控件工厂会阻止$.fn调用私有方法
$.widget("nmk.progressbar", {
  // default options
  options: {
      value: 0
  },
  _create: function() {
    // this.element
    // this.options
    this.element.addClass("progressbar");
    this._update();
  },
  // 修改或者获取参数
  _setOption: function(key, value) {
    this.options[key] = value;
    this._update();
  },
  _update: function() {
    var progress = this.options.value + "%";
    this.element.text(progress);
    if (this.options.value == 100) {
      // 触发回调事件，event.preventDefault() 或者 return false，去撤销回调和相关的事件。
      // 如果用户撤销了回调，_trigger 方法会返回 false，
      this._trigger("complete", null, { value: 100 });
    }
  },
  // 可以设置回调函数，无回调函数，事件也会触发
  complete: function(event, data) {},
  // destory：应该取消你的插件能造成的所有修改，初始化过程中或者后面的使用中造成的。
  //  destroy 方法在 DOM 删除时会被自动调用，所以它可以用于垃圾回收。
  //  默认的destroy 方法会删掉 DOM 元素与插件实例直接的连接， 所以在覆盖它时是调用原先插件提供的基础 destroy 方法，是很重要的。
  destroy: function() {
    this.element
        .removeClass("progressbar")
        .text("");

    // call the base destroy function
    $.Widget.prototype.destroy.call(this);
  }
});

// 调用插件
var bar = $("<div></div>").appendTo( "body" ).progressbar({ value: 20 });
// 获取当前值
bar.progressbar('value');
// 更新值
bar.progressbar('value',10);
// 事件——事件类型 = 插件名称+回调名称
var bar = $("<div></div>").appendTo( "body" ).progressbar({ value: 20 }).bind("progressbarcomplete", function(event, data) {});
// 可以直接该插件实例，可以不用传方法名，直接调用实例方法或者实例属性
bar.option('value',50)
console.log(bar.options.value);
// 只给prototype添加方法，实例上都拥有此方法
$.nmk.progressbar.prototype.reset = function() {
    this._setOption("value", 0);
};
{% endcodeblock %}
> widget factory 给我们提供了两个属性：
1、this.element——它指向一个只包含一个元素的 jQuery 对象，如果插件是由包含多个元素的 jQuery 对象调用时，会给其中的每一个元素都分配一个插件实例， 并让this.element 指向它；
2、this.options， 是包含键值对形式的插件参数的 hash 对象，插件的参数就是像这样传递进来的。

jquery.plugin的定义：
----
