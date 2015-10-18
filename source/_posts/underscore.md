title: underscore算法与实现
date: 2015-08-30 09:00:47
tags: js库
---
偏函数的使用:

一、循环中的callback参数处理：
1、如：each的callback,都由optimizeCb方法根据参数个数返回一个方法,如下参数为3个的情况：

{% codeblock %}
return function(value, index, collection) {
	return func(context,value,index,collection);
}
{% endcodeblock %}

2.cb方法提供回调(第2个参数)是null、function、Object及其他情况的处理。

4.group(behavior)方法：返回function，统一参数obj,iteratee,context；定义一个对象result,cb一个iteratee，each obj,执行iteratee获得key，调用behavior(result, value, key);返回result.*使用者：groupBy indexBy countBy.*


二、对象中的方法实现：
1、property(key)：返回一个函数，参数为obj，如果obj为null，返回void 0；否则返回对应的值。

2、propertyOf(obj)：参数是null，返回一个空函数；否则返回一个函数，参数为Key，返回其相应的值。

3、keys：不是obj，返回[]，nativeKeys，返回nativeKeys(obj)，for in obj，_.has(obj, key)，将key存入keys数组，hasEnumBug

4、allKeys：没有nativeKeys项，其余与keys处理相同。

5、values：获取obj所有的key，循环keys将value存入数组values，返回values。

6、identity(value)：return value。

7、has

8、constant(value)：返回一个函数，函数返回value。

9、matcher(attrs)：返回一个函数，处理后续传入obj与attrs的isMath。

10、times：

11、random(min, max)：一个参数max就是min，min是0，否则不变；return min+Math.floor(Math.random()*(max-min+1));

12、extendOwn：

13、isMacth(object,attrs)：取attrs的keys，如果object为null，返回keys.length取反；obj = Object(object)；循环keys.length，如果attrs[key]不等于obj[key]或者key in obj取反为真，return false；循环结束return true。

14、isArguments、isFunction，isString，isNumber，isDate，isRegExp，isError：都用toString(obj)来判断。

15、isNull(obj)：return obj === null。

16、isBoolean(obj)：return obj === true || obj === false || toString.call(obj) === '[object Boolean]'。

17、isNaN(obj)：return isNumber(obj) && obj != +obj。

18、isUndefined(obj)：return obj === void 0。

19、isFinite：!isNaN(parseFloat(obj))且isFinite(obj)

20、isFunction：兼容性(IE11或者safari8)

21、isArguments(obj)：_.has(obj,'callee')。


三、集合中的方法实现：
1、each判断是类数组，直接循环，否则取对象的keys进行循环，直接执行生成的iteratee方法。
2、map判断是类数组，直接循环，否则取对象的keys进行循环，执行生成的iteratee方法，将执行结果存入数组，循环完扫返回数组。
3、reduce, reduceRight
4、find：调用findIndex与findKey，根据返回的key或者index，返回该值。
5、findIndex/findLastIndex：调用createIndexFinder(1/-1)，cb一个predicate，如果传入的参数是1就从0开始循环数组,否则从lenth-1开始，步长为传入的参数；执行predicate返回为true，直接return index。
6、findKey：根据条件predicate，cb一个callback，循环中调用predicate返回的是true，直接return key。
7、filter：cb一个predicate，each传入的obj，将满足predicate的元素push到results数组，循环结束后返回results数组。
8、where:调用filter(obj,matcher(attrs))
9、findWhere:调用find(obj,matcher(attrs))
10、reject
11、every：与map类似,循环数组或者对象的Keys时predicate执行为false，return false，否则return true。
12、some：与map类似，循环数组或者对象的Keys时predicate执行为true，return true，否则返回false。
13、contains
14、indexOf
15、invoke(obj, method)：_map(obj,fn)，fn中定义func如果method是function，取method,否则取obj中的method; func如果是null，返回func，否则返回func执行结果。
16、pluck(obj, key)：return _map(obj, _.property(key))。
17、max
18、min
19、shuffle：返回一个随机乱序的list副本。
20、sample：返回一个list中随机的n个元素。
21、sortBy
22、groupBy
23、indexBy
24、countBy
25、toArray：参数错误，返回[]；参数是数组，slice返回该数组；类数组,map返回该数组；values返回该数组

size：返回类数组的长度或者Object的keys的长度

partition：cb一个predicate方法，each每个元素执行predicate，将满足的存入pass数组，不满足的存入fail数组；合并成一个数组返回


四、数组中的方法实现：
first,last：返回第一个(到第n个)/最后一个(从n个到最后一个) initial, rest

compact：去除数组中的false(false, null, '', undefined, NaN),使用filter其中的identity

flatten:

difference:without

uniq：
object：根据给的key与value列表[key,key],[value,value]或者[[key,value],[key,value]]，创建对象。

findIndex, findLastIndex：返回某一值的索引

indexOf, lastIndexOf:返回某一索引的值
range(start, stop, step)：创建一个Math.ceil((stop-start)/step)长度的数组。

五、函数中方法的实现：
1、negate(predicate)：返回一个函数，返回predicate执行取反。

2、partial

3、bind

4、bindAll

5、wrap(func, wrapper)

6、memoize

7、delay

8、defer

9、throttle

10、debounce

11、compose

12、after

13、before

14、once