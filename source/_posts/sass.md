title: sass
date: 2015-09-02 06:28:55
tags: css
---
定义变量：(常用关键字!global   !default  !optional)
$mainColor:  #efefef;
$borderDirect: top!default;

引用定义的变量为其一部分：
border-#{$borderDirect}

选择器的嵌套规则：
{% codeblock %}
body{
    div{
        p{}
    }
}
{% endcodeblock %}
属性的嵌套规则：
{% codeblock %}
font:{
    family:
    size:
    weight:
    padding:{
	    left:
	    right:
	  }
}
{% endcodeblock %}

数据类型：
数字：如1.2  13  13px
数学函数：
percentage($value)------转换成百分比
round($value)------四舍五入成一个整数
ceil($value)------大于自己的下一位整数
floor($value)------去除小数部分
abs($value)------绝对值
min($number……)------找出数值之间的最小值
max($numbers……)------找出数值之间的最大值

文本字符串：'foo'  "bar"  baz
字符串函数：
unquote($string)-------删除字符串中的引号
quote($string)----------给字符串引号

颜色：blue #00ffff 
颜色函数：
rgba(255,2,4,0.3)  rgb()
lighten($color , $amount) darken($color , $amount) 改变亮度，变亮/暗
red($color) green($color) blue($color)  从一个颜色中获取其中某颜色色值
mix($color-1,$color-2,[$weight]) 把两种颜色混合在一起
hsla($hue,$saturation,$lightness,$alpha) 色相、饱和度、亮度、透明度
hue($color)  saturation($color) lightness($color) 获取颜色中的相关值
adjust-hue($color,$degrees) saturate($color,$amount) desaturate($color,$amount)改变相关值，获取新颜色
grayscale($color) 颜色变灰,相当于desaturate($color,100%)
complement($color) 返回一个补充色,相当于adjust-hue($color,180deg)
invert($color) 反回一个反相色，红、绿、蓝色值倒过来

布尔：true  false
空值：null

List类型：
一维：如$font-size:1.5em 1em 0 2em
多维：如$borderWidth: 5px 10px, 20px 30px 或者(5px 10px) (20px 30px)
List函数：
length($list)
nth($list , $n)-----返回指定列表中的某个值
join($list1 , $list2 , [$separator])------将两个列表连接在一起，成一个列表
append($list , $val , [$separator])
zip($lists……)-------将多个合并成一个多维列表
index($list , $value)---------返回一个值在列表中的位置
一维循环：
{% codeblock %}
@each $var in  list{
	...
}
{% endcodeblock %}
多维循环：
{% codeblock %}
@each $var1，$var2 in list{
	...
}
{% endcodeblock %}

Map类型：如$map: (key1:value1, key2:value2)
Map函数：
map-get($map , $key)
map-merge($map1 , $map2)
map-keys($map)
map-value($map)
{% codeblock %}
@each $key, $value in $map{
    #{$key} {
        font-size:$value;
    }
}
{% endcodeblock %}

功能性函数：
type-of($value)
unit($unmber)-----返回单位
unitless($unmber)-----是否有单位
comparable($unmber1 , $unmber2)------判断两个值是否可以进行加、减及合并操作


@if语句：
{% codeblock %}
@if  condition {
	...
} @else if condition {
	...
} @else {
	...
}
{% endcodeblock %}
@for语句：
{% codeblock %}
	@for $var from start through end{ 
		...
	}
{% endcodeblock %}
@each in语句：
@each $var in <list or map>

定义函数：
{% codeblock %}
@function funName([paramList]){
    return value;
}
{% endcodeblock %}
调用：funName();

分享的十二栅格的实现：
{% codeblock %}
@mixin make-grid-columns{
	@for $i from (1) through $grid-columns{
		$list:"#{$list},.col-xs-#{$i},.col-sm-#{$i},.col-md-#{$i},.col-lg-#{$i}";
	}
	#{$list}{
		position: relative;
		min-height: 1px;
		padding-left:($grid-gutter-width/2);
		padding-right:($grid-gutter-width/2);
	}
}
@mixin float-grid-columns($class){
	@for $i from (1) through $grid-columns{
		$list:"#{$list},.col-#{$class}-#{$i}";
	}
	#{$list}{
		float:left;
	}
}

@mixin make-grid($class){
	@include float-grid-columns($class);
	@for $i from 0 through $grid-columns{
		@include calc-grid-column($i,$class,width);
	}
	@for $i from 0 through $grid-columns{
		@include calc-grid-column($i,$class,push);
	}
	@for $i from 0 through $grid-columns{
		@include calc-grid-column($i,$class,pull);
	}
	@for $i from 0 through $grid-columns{
		@include calc-grid-column($i,$class,offset);
	}
}

@mixin calc-grid-column($index,$class,$type){
	@if (($type==width) and ($index>0)){
		.col-#{$class}-#{$i}{
			width:percentage($index/$grid-columns);
		}
	}
	@if (($type==push) and ($index>0)){
		.col-#{$class}-#{$type}-#{$i}{
			left:percentage($index/$grid-columns);
		}
	}
	@if (($type==pull) and ($index>0)){
		.col-#{$class}-#{$type}-#{$i}{
			right:percentage($index/$grid-columns);
		}
	}
	@if (($type==offset) and ($index>0)){
		.col-#{$class}-#{$type}-#{$i}{
			margin-left:percentage($index/$grid-columns);
		}
	}
}
{% endcodeblock %}

@extend：
{% codeblock %}
.message {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}
.error {
  @extend .message;
  border-color: red;
}
// 通用定义
%clearfix::after {
    content: '';
    display: table;
    clear: both;
}
@mixin clearfix($extend: true) {
    @if $extend == true {
        @extend %clearfix;
    } @else {
        &::after {
            content: '';
            display: table;
            clear: both;
        }
    }
}

%clearfix {
    @include clearfix($extend: false);
}
{% endcodeblock %}
相当于.message .error
@mixin定义组件：
{% codeblock %}
@mixin clearfix($A, $b : '0px'){
	……
	&:after{}

}
{% endcodeblock %}
引用@mixin定义组件：@include 组件名
以下为在分享项目中的代码：
{% codeblock %}

{% endcodeblock %}
@content：将内容块放到@mixin中
{% codeblock %}
@mixin apply-to-ie6-only {
  * html {
    @content
  }
}

@include apply-to-ie6-only {
  #logo {
    background-image: url(/logo.gif);
  }
}
//编译之后
* html #logo {
  background-image: url(/logo.gif);
}
{% endcodeblock %}

{% codeblock %}
@mixin create-context($classes...) {
  @each $class in $classes {
    .#{$class} & {
      @content;
  }
}

@mixin context--alternate-template {
  @include create-context(about, blog) {
    @content
  }
}

.header {
  height: 12em;
  background: red;

  @include context--alternate-template {
    background: green;
  }
}
//编译之后
.header {
    height: 12em;
    background: red;
  }

  .about .header {
    background: green;
  }

  .blog .header {
    background: green;
  }
{% endcodeblock%}
{% codeblock %}
@mixin button() {
    display: block;
    font-size: 20px;
    text-decoration: none;
    @content;
}

.alert {
    @include button {
        color: #F00;
    }
}
.cancel {
    @include button {
        border: solid 1px #999;
    }   
}
//编译之后
.alert {
    display: block;
    font-size: 20px;
    text-decoration: none;
    color: #F00;
}
.cancel {
    display: block;
    font-size: 20px;
    text-decoration: none;
    border: solid 1px #999;
}
{% endcodeblock %}
@media 媒体查询：
{% codeblock %}
.container{
	@include container-fixed;
	@media (min-width: $screen-sm-min){
		width: $container-sm;
	}
	@media (min-width: $screen-md-min){
		width: $container-md;
	}
	@media (min-width: $screen-lg-min){
		width: $container-lg;
	}
}
{% endcodeblock %}

@import文件：
{% codeblock %}
@import "define";
@import "mixin-common";
@import "mixin-gridFramework";
@import "mixin-selector";
@import "adjust";
{% endcodeblock %}

编译工具：gulp
比grunt配置更加方便，以管道式的编程方式：
{% codeblock %}
var gulp = require('gulp');
var sass = require('gulp-sass');

var sass_path = './share_sass/root.scss';
var css_path = './css';

gulp.task('css',function(){
	gulp.src(sass_path)
	.pipe(sass())
	.pipe(gulp.dest(css_path));
});

{% endcodeblock %}
