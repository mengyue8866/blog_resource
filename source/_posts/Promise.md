title: Promise
date: 2015-08-31 15:10:52
tags: ES6
---
一、ES6中的promise
1、所谓Promise，就是一个对象，用来传递异步操作的消息。它代表了某个未来才会知道结果的事件（通常是一个异步操作），并且这个事件提供统一的API，可供进一步处理。

2、promise的状态：Pending，Resolved(Fulfilled)，rejected，只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态；一旦状态改变，就不会再变，任何时候都可以得到这个结果。

3、Promise也有一些缺点。首先，无法取消Promise，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。第三，当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

Promise提供的方法：
{% blockquote %}
Promise.prototype.then()
Promise.prototype.catch()  //用于指定发生错误时的回调函数，返回一个promise对象
Promise.all() //将多个Promise实例，包装成一个新的Promise实例
Promise.race() //将多个Promise实例，包装成一个新的Promise实例
Promise.resolve() //将现有对象转成Promise对象
Promise.reject() //返回一个新的Promise实例，实例状态为rejected
{% endblockquote%}

promise实例
{% codeblock %}
var promise = new Promise(function(resolve, reject) {
  // ... some code
  // 状态由pending变为resolved时执行，并将异步操作的结果传递出去
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
{% endcodeblock %}
promise实例生成以后，用then方法分别指定resolved和reject状态的回调函数。
{% codeblock %}
promise.then(function(value) {
  // success
}, function(value) {
  // failure
});
{% endcodeblock %}
then方法可以接受两个可选回调函数作为参数。第一个回调函数是Promise对象的状态变为Resolved时调用，第二个回调函数是Promise对象的状态变为Reject时调用。

一个Promise简单的实例：
{% codeblock %}
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
{% endcodeblock %}

异步加载图片的例子：
{% codeblock %}
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    var image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
{% endcodeblock %}

Promise实现Ajax的例子：
{% codeblock %}
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

    function handler() {
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
{% endcodeblock %}

resolve函数的参数除了正常的值以外，还可能是另一个Promise实例，表示异步操作的结果有可能是一个值，也有可能是另一个异步操作：
{% codeblock %}
var p1 = new Promise(function(resolve, reject){
  // ...
});

var p2 = new Promise(function(resolve, reject){
  // ...
  resolve(p1);
})
{% endcodeblock%}
注：p1和p2都是Promise的实例，p2的resolve将p1作为参数。即p1的状态就会传递给p2，也决定了p2的状态，所以p2的回调函数就会等待p1的状态改变；如果p1的状态已经是Resolved或者Rejected，那么p2的回调函数将会立刻执行。

4、then方法实现的是为Promise实例添加状态改变时的回调函数，第一个参数是Resolved状态的回调函数，第二个参数（可选）是Rejected状态的回调函数。

then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法。

{% codeblock %}
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("Resolved: ", comments);
}, function funcB(err){
  console.log("Rejected: ", err);
});
{% endcodeblock%}

第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数。采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。

5、catch及其他方法实例：
{% codeblock %}
getJSON("/posts.json").then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
{% endcodeblock %}
{% codeblock %}
var promise = new Promise(function(resolve, reject) {
  throw new Error('test')
});
promise.catch(function(error) { console.log(error) });
// Error: test
{% endcodeblock%}
Promise抛出一个错误，就被catch方法指定的回调函数捕获

{% codeblock %}
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
{% endcodeblock %}
Promise对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止，前面两个then方法之中任何一个抛出的错误，都会被最后一个catch捕获。

all方法：只有数组中每个状态都为fulfilled，promise才会为fulfilled，有一项为rejected，结果就为rejected。
{% codeblock %}
// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function(id){
  return getJSON("/post/" + id + ".json");
});

Promise.all(promises).then(function(posts) {
  // ...
}).catch(function(reason){
  // ...
});
{% endcodeblock %}

race方法：只要一个实例率先改变状态，P的状态就会改变;那个率先改变的Promise实例的返回值，就会传递给P的回调函数。

{% codeblock %}
var p = Promise.race([p1,p2,p3]);
{% endcodeblock%}

resolve方法：
{% codeblock %}
var jsPromise = Promise.resolve($.ajax('/whatever.json'));
jsPromise.then(function(result){
  console.log(result);
})
{% endcodeblock%}

reject方法：
{% codeblock %}
var p = Promise.reject('出错了');

p.then(null, function (s){
  console.log(s)
});
// 出错了
{% endcodeblock %}