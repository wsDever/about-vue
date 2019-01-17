## 我的水货vue
##### 最近在看vue源码部。之前有零零碎碎的看过一些文档，但是项目中一直没怎么用到这个框架。这次也算是边看（其实也是跟着幕课网）边学吧。
先看一把演示吧。
[点击这里](http://wshome.bid/main/vue/vue-code/wue.html)
首先，关于vue的介绍就不多说了。我的理解，vue这个框架的作用就是把我们开发的一个个的组件，进行整合，形成一个完成的项目（页面），同时它可以监听（管理）各个组件。
那么就先写个组件吧，这个组件中有data，动态设置data，更新UI等。

```
var wComponent = { };
```
添加一个原本的data：
```
var wComponent = {
      data: {
       name: "WuSun"
       }
 };
```
添加对data数据的动态修改：
```
var wComponent = {
      data: {
       name: "WuSun"
       }，
      setData: function(key,val){
    var _this = this ;
    _this.data[key] = val ;
  },
 };
```
有了data，有了可以修改data的，先来把这个data渲染出来吧，给它添加个render函数，但是在此之前要给它加个元素elm，指定渲染到哪里，这个元素要在body里面添加。

```
html :
<div id="app"></div>
js:
var wComponent = {
      elm: document.getElementById("app"),
      data: {
      name: "WuSun"
      }，
      setData: function(key,val){
      var _this = this ;
      _this.data[key] = val ;
    },
      render: function(){
        var _this = this;
        var template = `<div>hello, world!</div>`;
        _this.elm.innerHTML = template;
      }
 };
```
至此，这个组件已经可以跑起来了。调用渲染函数试试：

```
    wComponent.render();
```
页面上的效果就是：

```
<body>
  <div id="app">
     hello, world!
  </div>
</body>
```
但是这个渲染出来的是一个固定的字符串，所以要在render函数里面做修改，使其能够渲染组件里面的data.
```
var wComponent = {
      elm: document.getElementById("app"),
      data: {
      name: "WuSun"
      }，
      setData: function(key,val){
      var _this = this ;
      _this.data[key] = val ;
    },
      render: function(){
        var _this = this;
        var template = '<div>hello, '+ _this.data.name +'</div>';
        _this.elm.innerHTML = template;
      }
 };
```
嗯，这样就可以了。
但是要怎样才能把data中的变量映射到一个字符模版中，允许进行插值替换的呢？还是要修改render函数的。
在此之前，先把html里面修改一下。

```
html :
<div id="app">
    hello, ｛name｝!
</div>
var wComponent = {
      elm: document.getElementById("app"),
      data: {
      name: "WuSun"
      }，
      setData: function(key,val){
      var _this = this ;
      _this.data[key] = val ;
    },
      compile: function(str){
      var _this = this ;
      return str.replace(/\{.*\}/g, function(res){
       var _res = res.replace(/\s+/g,"") ;
       var _key = _res.substring(1,_res.length-1);
      return _this.data[_key];
       })
     },
      render: function(){
        var _this = this;
        var template = _this.compile(_this.ele.innerHTML);
        _this.elm.innerHTML = template;
      }
 };
```
同时为了易读，为组件添加了一个编译的函数，其作用就是将html中带有插值的字符串编译（转化）成带有data值的普通字符串，然后通过render()函数渲染出来。
再次执行

```
    wComponent.render();
```
页面上的效果就是：

```
<body>
  <div id="app">
     hello, WuSun!
  </div>
</body>
```
到目前为止，我们还并没有使用上setData。
这个简单，设置一个定时器模拟一下就可以了，同时也要修改一下setData函数，要调用一下render()函数。

```
    wComponent.render();
    setTimeout(function () {
       wComponent.setData('name', '老吴');
   }, 3000);
```
至此，这个组件的基本功能算是实现了。但是vue中的组件是有生命周期期的，也就是在dom生成的不同阶段做不同的事，这里很明显，对data的修改，就需要在dom完成之后 。继续对组件进行修改，添加一个ready函数吧，对dom的操作就放在这个函数中。对html也进行一下改造。

```
<body>
  <div id="app">
    <div>
    Hello ,{ name } 
    </div>
    <input type="button" id="btn" value="点击" name="">
  </div>
</body>

 var wComponent = {
      elm: document.getElementById("app"),
      data: {
      name: "WuSun"
      }，
     ......
     ready: function(){
      var _this = this ;
      document.getElementById("btn").onclick = function(){
         _this.setData("name","小吴");
          }
        },
   }
```
这样我们只需要在render()函数后面调用ready函数就可以了。说了这么多好像跟vue没什么关系呀，马上就来了。
vue是干嘛用的，管这些组件的，那在项目中肯定不是直接调用组件的相关函数的。

这里我们再定义另一个 对象，就叫作Wue吧，在它的函数里面我们来调用这个组件的相关函数（vue就是这么干的）。
（这里就直接把代码一次全部放出来吧）
```
    function Wue(options){
    if(this instanceof Wue)
      this.init(options);
  };
  Wue.prototype.init = function(options){
    var Wm = this;
    Wm.opts = options;
    Wm.initState(Wm);
    Wm.initRender(Wm);
    Wm.initLifecycle(Wm);
    Wm.initEvents(Wm);
  }
  Wue.prototype.initState = function(wm){
    var _des = wm.opts.component.data ;
    var _src = wm.opts.data ;
    for (var item in _des) {
            _des[item] = _src[item] || _des[item];
        }
  }
  Wue.prototype.initLifecycle = function(wm){
    wm.opts.component.ready();
  }
  Wue.prototype.initRender = function(wm){
    wm.opts.component.render();
  }
  Wue.prototype.initEvents = function(wm){

  }
  
  new Wue({
    component: wComponent,
    data:{
      name: "老吴"
    },
    motheds:{}
  });
```
可以看到在Wue的init阶段，进行了很多的初始化（实际上vue的比这里的还要多），像生命周期，事件，渲染，状态等。然后在各个初始化的时候，调用组件的各个方法。

至此，这个水货vue也算基本完了，当然这个肯定是不能在项目中使用的，但对于认识vue还是有帮助的。下次将开始学习一下关于v-dom的知识了。

###### 最后放全全部代码，以供学习。

```
<!DOCTYPE html>
<html lang="zh-cn">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, maximum-scale=1, minimum-scale=1, user-scalable=no, minimal-ui">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black">
  <meta name="format-detection" content="telephone=no" />
  <title>WUE</title>
</head>
<body>
  <div id="app">
    <div>
    Hello ,{ name } 
    </div>
    <input type="button" id="btn" value="点击" name="">
  </div>
</body>
<script type="text/javascript">
  var wComponent = {
    ele: document.getElementById("app"),
    data: {
      name: "WuSun"
    },
    setData: function(key,val){
      var _this = this ;
      _this.data[key] = val ;
      _this.render();
    },
    compile: function(str){
      var _this = this ;
      return str.replace(/\{.*\}/g, function(res){
        var _res = res.replace(/\s+/g,"") ;
        var _key = _res.substring(1,_res.length-1);
        return _this.data[_key];
      })
    },
    methods:{

    },
    ready: function(){
      var _this = this ;
      document.getElementById("btn").onclick = function(){
        _this.setData("name","小吴");
      }
    },
    render: function(){
      var _this = this ;
      if(!_this._source){
        _this._source = _this.ele.innerHTML;
      }
      var template = _this.compile(_this._source);
      _this.ele.innerHTML = template ;
    },
    _source:''
  };
  


  function Wue(options){
    if(this instanceof Wue)
      this.init(options);
  };
  Wue.prototype.init = function(options){
    var Wm = this;
    Wm.opts = options;
    Wm.initState(Wm);
    Wm.initRender(Wm);
    Wm.initLifecycle(Wm);
    Wm.initEvents(Wm);
  }
  Wue.prototype.initState = function(wm){
    var _des = wm.opts.component.data ;
    var _src = wm.opts.data ;
    for (var item in _des) {
            _des[item] = _src[item] || _des[item];
        }
  }
  Wue.prototype.initLifecycle = function(wm){
    wm.opts.component.ready();
  }
  Wue.prototype.initRender = function(wm){
    wm.opts.component.render();
  }
  Wue.prototype.initEvents = function(wm){

  }
  
  new Wue({
    component: wComponent,
    data:{
      name: "老吴"
    },
    motheds:{}
  });
</script>
</html>

```