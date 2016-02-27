---
layout: post
title:  "从promise到async"
date:   2016-02-26 16:40:18
categories: tech
---

在使用js编程的过程中，特别是nodejs快速发展之后，js的异步编程困扰了越来越多的开发者。es6中Promise的出现可以说很好的缓解了大部分遇到的问题，尤其是回调金字塔问题，层层嵌套的回调，然代码的开发和维护都变得困难。但是Promise还远称不上完美的解决方案，它依然需要回调函数来实现异步编程，同时，在遇到较为复杂的情况时，Promise编写的代码也会变得冗余而繁琐。

async的出现，可以说弥补了promise的不足，让异步编程更加简洁优雅。开发者在进行代码的编写时，还是倾向于同步代码的编写。而async恰好满足了这方面的需要。可以通过简单异步操作来对比一下promise和async的写法。
先看看promise的写法


{% highlight javascript %}
function doAsyncThing () {
	//执行一个异步操作，然后调用then方法传入回调函数
    return asyncOperation().then(function(val) {
        console.log(val)
        return val
    })
}
{% endhighlight %}

再看async的写法


{% highlight javascript %}
async function doAsyncThing () {
	//等待异步操作asyncOperation完成，再执行后续操作
    var val = await asyncOperation()
    console.log(val)
    return val
}
{% endhighlight %}


通过对比可以看出来，async的写法更接近同步编程的写法，结构也更加的清晰。同时，promise的写法由于需要用then来传入回调函数，也让代码显得繁琐。另一个让人不适的方面就是，代码中有多个return，很容易让人对最后return的东西产生困惑，进而容易产生bug。

### 链式操作
promise最为吸引人的一个方面就是可以通过链式操作来避免回调嵌套。看看用promise来进行一系列异步操作的代码：

{% highlight javascript %}
function doAsyncThing () {
    return asyncOperation1().then(function(val) {
        return asyncOperation2(val)
    }).then(function(val) {
        return asyncOperation3(val)
    }).then(function(val) {
        return asyncOperation4(val)
    })
}
{% endhighlight %}


而使用async来完成


{% highlight javascript %}
async function doAsyncThing () {
    var val = await asyncOperation1();
    val = await asyncOperation2(val);
    val = await asyncOperation3(val);
    return await asyncOperation4(val);
}
{% endhighlight %}


### 异常处理
在promise的执行过程中，异步操作可以被rejected，被rejected了的promise可以在then方法的第二个参数中传入异常处理函数进行处理，也可以使用catch方法来进行处理。在使用async时，则用try-catch的方式来进行异常处理。


{% highlight javascript %}
function doAsyncThing () {
    return asyncOperation1().then(function(val) {
        return asyncOperation2(val)
    }).then(function(val) {
        return asyncOperation3(val)
    }).catch(function(err) {
        console.error(err)
    })
}
{% endhighlight %}


async异常处理的代码


{% highlight javascript %}
async function doAsyncThing () {
    try {
      var val = await asyncOperation1()
      val = await asyncOperation2(val)
      return await asyncOperation3(val)
    } catch (err) {
      console.err(err)
    }
}
{% endhighlight %}

在异常处理上，async的写法相对于promise虽然并没有什么优势。但是这样的写法与进行同步代码的编写时，习惯是一样。

### 使用async
尽管离原生环境支持async还有很长的一段时间，但是，运用一些工具可以将代码编译成兼容es5的模式。比如：

- 命令行工具：[traceur](https://github.com/google/traceur-compiler/)
- Gulp插件：[gulp-traceur](https://www.npmjs.com/package/gulp-traceur)
- Grunt插件：[grunt-traceur](https://www.npmjs.com/package/grunt-traceur)
