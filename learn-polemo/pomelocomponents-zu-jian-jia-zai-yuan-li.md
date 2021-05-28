# 【pomelo】components组件加载原理

文档官方中有一个简单的添加组件组件实例:[给pomelo加个组件](https://github.com/NetEase/pomelo/wiki/%E7%BB%99pomelo%E5%8A%A0%E4%B8%AA%E7%BB%84%E4%BB%B6)

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

function load(path, name) {
  if (name) {
    return require(path + name);
  }
  return require(path);
}
```

这里我们来看下load.bind和\_\__defineGetter_\_\_:

> **`bind()`** 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。
>
> **`__defineGetter__`** 方法可以将一个函数绑定在当前对象的指定属性上，当那个属性的值被读取时，你所绑定的函数就会被调用。

下面我们以 pomelo\lib\components下面第一个组件`backendSession`为例。 根据\_\_**defineGetter\_\_** 可知，我们在调用Pomelo.backendSession，就是在调用\__load，而\_load_ 就是调用require\(./component/backendSession\)。

在Application.start 接口中，我们看到:

backendSession.js如下:

```text
 appUtil.loadDefaultComponents(self);
```

上面函数如下:

```text
module.exports.loadDefaultComponents = function(app) {
  var pomelo = require('../pomelo');
  // load system default components
  if (app.serverType === Constants.RESERVED.MASTER) {
    app.load(pomelo.master, app.get('masterConfig'));
  } else {
...
    app.load(pomelo.backendSession, app.get('backendSessionConfig')); // 这里
    app.load(pomelo.channel, app.get('channelConfig'));
    app.load(pomelo.server, app.get('serverConfig'));
  }
  app.load(pomelo.monitor, app.get('monitorConfig'));
};
```

接下来就解析:

```text
app.load(pomelo.backendSession, app.get('backendSessionConfig'));
```

* pomelo.backendSession

> 相当于：require\(./component/backendSession\)

* app.get\('backendSessionConfig'\)

> > 代码中没有相关set，所以是undefined

接着就调用app.load:

pomelo/lib/components/backendSession.js如下:

```text
/**
 * Load component
 *
 * @param  {String} name    (optional) name of the component
 * @param  {Object} component component instance or factory function of the component
 * @param  {[type]} opts    (optional) construct parameters for the factory function
 * @return {Object}     app instance for chain invoke
 * @memberOf Application
 */
Application.load = function(name, component, opts) {
  if(typeof name !== 'string') {
    opts = component;
    component = name;
    name = null;
    if(typeof component.name === 'string') {
      name = component.name;
    }
  }

  if(typeof component === 'function') {
    component = component(this, opts);
  }

  if(!name && typeof component.name === 'string') {
    name = component.name;
  }

  if(name && this.components[name]) {
    // ignore duplicat component
    logger.warn('ignore duplicate component: %j', name);
    return;
  }

  this.loaded.push(component);
  if(name) {
    // components with a name would get by name throught app.components later.
    this.components[name] = component;
  }

  return this;
};
```

我们分析下入参:

* name:require\(./component/backendSession\)
* comonent:undefined
* opt:undefined

所以先进入`if(typeof name !== 'string')，`最后会走到:

```text
  if(typeof component === 'function') {
    component = component(this, opts);
  }
```

上面代码就是调用backendSession.js函数，如下:

```text
var BackendSessionService = require('../common/service/backendSessionService');

module.exports = function(app) {
  var service = new BackendSessionService(app);
  service.name = '__backendSession__';
  // export backend session service to the application context.
  app.set('backendSessionService', service, true);　// 1. set　函数自行查看

  // for compatibility as `LocalSession` is renamed to `BackendSession` 
  app.set('localSessionService', service, true);

  return service;  // 2. 返回　BackendSessionService
};
```

接着回到`Application.load:`

```text
this.loaded.push(component);
  if(name) {
    // components with a name would get by name throught app.components later.
    this.components[name] = component;　// name = '__backendSession__'
  }
```

接下来，我们就可以通过以下代码来访问BackendSessionService实例:

```text
方式一、app.get('backendSessionService')
方式二、app.backendSessionService
```

组件如果存在:start 和 afterStart方法，通过 `appUtil.optComponents`调用,参数如下:

```text
Constants.RESERVED.START
Constants.RESERVED.AFTER_START
```

Good Joob!!!

如有疑问或错误，或者觉得我哪里漏说明，请联系我！

