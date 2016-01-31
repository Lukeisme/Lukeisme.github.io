---
layout: post
title:  "node-webkit中的JavaScript"
date:   2016-02-01 16:24:18
categories: tech
---

##1.单线程
在使用nwjs编写桌面客户端时，通常会有nodejs和传统浏览器前端的两部分js代码。有nodejs的帮助，就可以比较方便和操作系统进行交互，比如最常见的文件操作，数据库查询等。虽然nwjs中有这两部分的js代码，但是，但是它并没有两个线程在同时运行。nwjs在尽量不影响原来环境的运行逻辑的前提下，对nodejs和chromium进行了整合。对它们的整合只做两件事：

- 主循环的融合
- 上下文的打通

nodejs和chromium的运行，都有他们的主循环。而nwjs的一个重要特性就是，可以在浏览器的运行环境中，可以直接调用nodejs的相关方法、属性。这就需要对nodejs的主循环和chromium的渲染进程进行整合。整合之后，nodejs和chromium会使用相同的V8引擎，但是会将他们的内容放到两个不同的上下文环境中，以保证命名空间不相互污染。

##2.直接调用

在前端代码当中，可以通过

{% highlight javascript %}
process.mainModule.exports
{% endhighlight %}

来对nodejs入口模块中exports出来的内容直接进行访问和执行。所以如非必要，不要在程序中启动一个server来供客户端的UI内容进行使用。因为在DOM的环境中就可以很方便地对nodejs的内容进行访问，而不需要http请求或者web socket来完成这件事情。

- 路由：或许在开发的过程中会希望使用多个页面切换，来组成客户端应用。但是在客户端的编写上，最好不要去改变window.location或者开启一个新的窗口。而是采用单页面web应用的开发方式，作DOM的替换和更新。这种方式不仅有更好的用户体验，同时，js运行的上下文环境也能得以保留。
- 模板引擎：在传统的web开发当中，也许会习惯于在node端进行模板渲染之后再返回给浏览器渲染。这样做主要是可以提高网页的性能，缩短首屏渲染的时间。但是在客户端的应用中，这就显得没有必要了。直接渲染模板就行，不需要使用一个server来完成这项任务。

##3.nwjs相关API

###3.1 nw.gui
在nwjs中，对原生客户端组件进行了封装，如菜单，以便于在js中进行操作。可以通过

{% highlight javascript %}
var gui = require('nw.gui') 
{% endhighlight %}

进行引用。值得注意的是，只能在前端js代码当中去引用，也就是window的上下文中。在nodejs中引用会出错，造成程序无法运行。同时，原生UI模块的不当使用，会直接造成客户端的crash，并且不会有报错信心，所以使用过程中，要小心谨慎。
相关原生UI组件的[文档](https://github.com/nwjs/nw.js/wiki/Native-UI-API-Manual)

> ps:在mac系统下，需要应用mac的内建菜单，才能进行复制，粘贴和全选等操作。可以通过以下代码来创建一个mac内建菜单。
>
{% highlight javascript %}
var gui = require('nw.gui')
var win = gui.Window.get()
var nativeMenuBar = new gui.Menu({ type: "menubar" })
try {
  nativeMenuBar.createMacBuiltin("My App")
  win.menu = nativeMenuBar
} catch (ex) {
  console.log(ex.message)
}
{% endhighlight %}

###3.2 事件

在nwjs中，每个UI元素都继承了nodejs的[EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter)，所以可以很方便地给DOM元素绑定事件，自定义事件。

{% highlight javascript %}
div.on('click', function() {
  console.log('clicked')
})
{% endhighlight %}

###3.3 本地文件
出于安全考虑。在浏览器的web页面当中是不能对用户的本地文件进行相关的操作的。但是作为客户端，有时候对本地文件的访问和操作的需求还是存在的。所以在nwjs中，可以获取file类型的input的真实路径。

{% highlight javascript %}
fileInput.files[0].path //真实文件路径
{% endhighlight %}
同时还可以设置文件input的值

{% highlight javascript %}
var f = new File('/path/to/file', 'name')
var files = new FileList()
files.append(f)
document.getElementById('input0').files = files
{% endhighlight %}
而且在js脚本当中，你还可以触发file类型input的click事件，用来打开一个文件选择对话框。这可以更加方便地制作一个个性化的交互的文件选择器。

以上就是我在做了一段时间的nwjs的开发之后，觉得编写代码的过程中需要注意的，比较不同的地方 ~