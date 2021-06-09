# 【pomelo】启动时几个阶段详解

pomelo总体流程是下述四个阶段:

```text
/**
 * Application states
 */
var STATE_INITED  = 1;  // app has inited -> Application.init 
var STATE_START = 2;  // app start ->  Application.start
var STATE_STARTED = 3;  // app has started -> Application.afterStart
var STATE_STOPED  = 4;  // app has stoped -> Application.stop
```

这个是插件加载流程:

```text
RESERVED: {
...
    START: 'start',
    AFTER_START: 'afterStart',
...
  },
```

#### 一、项目入口:app.js

每一个项目创建后，会生成入口:`app.js`。

```text
var pomelo = require('pomelo');
var app = pomelo.createApp();
app.set('name', '$');
...
// start app
app.start();
...
```

#### 二、STATE\_INITED

在`pomelo.createApp()`中会进行初始化，然后设置`Application.state = STATE_INITED;`

#### 三、STATE\_START

接着执行`app.start();`。这里我们看到会进入`appUtil.startByType`，然后进入匿名回调。进入以后首先会加载组件，加载成功后，再去判断是否存在`beforeFun`:

```text
var beforeFun = self.lifecycleCbs[Constants.LIFECYCLE.BEFORE_STARTUP];
    if(!!beforeFun) {
      beforeFun.call(null, self, startUp);
    } else {
      startUp();
    }
```

经过我的调试，发现是不存在`beforeFun`的\(关于lifecycleCbs具体的分析见此文附录\),于是就会进入到startUp:

```text
    var startUp = function() {
      appUtil.optComponents(self.loaded, Constants.RESERVED.START, function(err) {
        self.state = STATE_START;
        if(err) {
          utils.invokeCallback(cb, err);
        } else {
          logger.info('%j enter after start...', self.getServerId());
          self.afterStart(cb);
        }
      });
    };
```

因为已经加载过相关组件，这边`appUtil.optComponents`用来运行组件的`start`函数:

> Constants.RESERVED.START -&gt; start

当所有的都执行完毕，进入回调。先设置`self.state = STATE_START;`,然后如果没有出错，就会执行:`self.afterStart(cb);`。

#### 四、STATE\_STARTED

从上面步骤，先去执行组件的`afterStart`函数，所有组件都执行完毕后，进入回调:

```text
self.state = STATE_STARTED;
    var id = self.getServerId();
    if(!err) {
      logger.info('%j finish start', id);
    }
    if(!!afterFun) {
      afterFun.call(null, self, function() {
        utils.invokeCallback(cb, err);
      });
    } else {
      utils.invokeCallback(cb, err);
    }
    var usedTime = Date.now() - self.startTime;
    logger.info('%j startup in %s ms', id, usedTime);
    self.event.emit(events.START_SERVER, id);
```

这里最后会发型一个`START_SERVER`给`monitorwatcher.js`。

#### 五、STATE\_STOPED

在`Application.stop`中会设置:

```text
this.state = STATE_STOPED;
```

那么是谁调用`Application.stop`。 如下:

pomelo\lib\modules\console.js中Module.prototype.monitorHandler，msg.signal为`'stop'`和`'restart'`。 其他的就自己找一下吧。

### 附录：

下面是npm-lifecycle相关的各个阶段

```text
// pomelo\lib\util\constants.js
  LIFECYCLE: {
    BEFORE_STARTUP: 'beforeStartup', ->  Application.start
    BEFORE_SHUTDOWN: 'beforeShutdown', -> Application.stop
    AFTER_STARTUP: 'afterStartup', -> Application.afterStart
    AFTER_STARTALL: 'afterStartAll' -> monitorwatcher.js (startOver)
  },
```

我们从`BEFORE_STARTUP`开始，全文搜索后`BEFORE_STARTUP`，发现pomelo就一个地方使用:

```text
Application.start = function(cb) {
...

  var self = this;
  appUtil.startByType(self, function() {
    appUtil.loadDefaultComponents(self);
    var startUp = function() {
...
    };
    var beforeFun = self.lifecycleCbs[Constants.LIFECYCLE.BEFORE_STARTUP]; // 此处
    if(!!beforeFun) {
      beforeFun.call(null, self, startUp);
    } else { // lifecycleCbs 不存在，所有都是执行startUp
      startUp();
    }
  });
};
```

然后看下`self.lifecycleCbs`:

```text
// pomelo\lib\util\appUtil.js
/**
 * Load lifecycle file.
 *
 */
var loadLifecycle = function(app) {
  var filePath = path.join(app.getBase(), Constants.FILEPATH.SERVER_DIR, app.serverType, Constants.FILEPATH.LIFECYCLE);
  if(!fs.existsSync(filePath)) {
    return;
  }
  var lifecycle = require(filePath);
  for(var key in lifecycle) {
    if(typeof lifecycle[key] === 'function') {
      app.lifecycleCbs[key] = lifecycle[key];
    } else {
      logger.warn('lifecycle.js in %s is error format.', filePath);
    }
  }
};
```

我调试了一把，得到 filePath:

> 'XXXXXXX\game-server\app\servers\master\lifecycle.js'

上面就是 master 服务器对应的 lifecycle.js 路径。不过没这个文件。所有我们的pomelo程序一直都是执行startUp。

我后来在自己电脑上全局搜索了下，在node源码路径找到：

> node-v14.16.1\deps\npm\lib\utils\lifecycle.js
>
> \`\`\` exports = module.exports = runLifecycle

const lifecycleOpts = require\('../config/lifecycle'\) const lifecycle = require\('npm-lifecycle'\)

function runLifecycle \(pkg, stage, wd, moreOpts, cb\) { if \(typeof moreOpts === 'function'\) { cb = moreOpts moreOpts = null }

const opts = lifecycleOpts\(moreOpts\) lifecycle\(pkg, stage, wd, opts\).then\(cb, cb\) }

```text
查看上面 lifecycleOpts 对应的文件:
```

'use strict'

const npm = require\('../npm.js'\) const log = require\('npmlog'\)

module.exports = lifecycleOpts

let opts

function lifecycleOpts \(moreOpts\) { if \(!opts\) { opts = { config: npm.config.snapshot, dir: npm.dir, failOk: false, force: npm.config.get\('force'\), group: npm.config.get\('group'\), ignorePrepublish: npm.config.get\('ignore-prepublish'\), ignoreScripts: npm.config.get\('ignore-scripts'\), log: log, nodeOptions: npm.config.get\('node-options'\), production: npm.config.get\('production'\), scriptShell: npm.config.get\('script-shell'\), scriptsPrependNodePath: npm.config.get\('scripts-prepend-node-path'\), unsafePerm: npm.config.get\('unsafe-perm'\), user: npm.config.get\('user'\) } }

return moreOpts ? Object.assign\({}, opts, moreOpts\) : opts }

\`\`\` 想要了解 npm 中的npm-lifecycle，请戳[链接](https://github.com/npm/npm-lifecycle)

接下来我们回到: Application.start，进入到appUtil.startByType的匿名回调函数，进入到插件的start阶段。

