# 【pomelo】components组件加载原理

文档官方中有一个简单在添加组件组件实例:[给pomelo加个组件](https://github.com/NetEase/pomelo/wiki/%E7%BB%99pomelo%E5%8A%A0%E4%B8%AA%E7%BB%84%E4%BB%B6)

那组件添加的原理是怎么样子的呢？请往下看。

在app.js中，第一行用于加载pomelo.js:

```text
var pomelo = require('pomelo');
```

在pomelo.js的下述代码，自动加载捆绑的组件\(可以看到，组件都在 pomelo\lib\components下面\)：

```text
fs.readdirSync(__dirname + '/components').forEach(function (filename) {
  if (!/\.js$/.test(filename)) {
    return;
  }
  var name = path.basename(filename, '.js');
  var _load = load.bind(null, './components/', name);
  Pomelo.components.__defineGetter__(name, _load);
  Pomelo.__defineGetter__(name, _load);
});
```

这里我们来看下load.bind和\_\__defineGetter_\_\_:

> **`bind()`** 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

> **`__defineGetter__`** 方法可以将一个函数绑定在当前对象的指定属性上，当那个属性的值被读取时，你所绑定的函数就会被调用。

下面我们以第一个组件`backendSession`为例。 根据\_\_**defineGetter\_\_** 可知，我们在调用Pomelo.backendSession，就是在调用

