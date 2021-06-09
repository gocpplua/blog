# 【pomelo】master服务和connector等服务的创建流程

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



#### 六：子进程创建

上面是master服务器的启动流程，那么对于其他服务，其他服务器是在什么时候启动的呢？ 我们先说下app.type的设置，这个在后面有用。 我们在`Application.init`中，调用`appUtil.defaultConfiguration`,然后在`appUtil.processArgs`中设置`app.type = Constants.RESERVED.ALL;`:

```text
var processArgs = function(app, args) {
...
  var type = args.type || Constants.RESERVED.ALL;
  var startId = args.startId;

  app.set(Constants.RESERVED.MAIN, args.main, true);
  app.set(Constants.RESERVED.SERVER_TYPE, serverType, true);
  app.set(Constants.RESERVED.SERVER_ID, serverId, true);
  app.set(Constants.RESERVED.MODE, mode, true);
  app.set(Constants.RESERVED.TYPE, type, true);
...
};
```

回到主流程中，我们会调用各个组件的start函数，当我们在调用`master`组件时候:

```text
// pomelo\lib\components\master.js
pro.start = function (cb) {
  this.master.start(cb);
};
```

上面的`this.master.start`,实际上是在调用:

```text
// Epomelo\lib\master\master.js
Server.prototype.start = function(cb) {
  moduleUtil.registerDefaultModules(true, this.app, this.closeWatcher);
  moduleUtil.loadModules(this, this.masterConsole);

  var self = this;
  // start master console
  this.masterConsole.start(function(err) {
   // 其实是:node_modules\pomelo-admin\lib\consoleService.js
    if(err) {
      process.exit(0);
    }
    moduleUtil.startModules(self.modules, function(err) {
    // 经过几次回调以后，会进入这里！！！！
      if(err) {
        utils.invokeCallback(cb, err);
        return;
      }

      if(self.app.get(Constants.RESERVED.MODE) !== Constants.RESERVED.STAND_ALONE) {
        starter.runServers(self.app);
      }
      utils.invokeCallback(cb);
    });
  });

....
};
```

上面的start函数，最后会进入到`moduleUtil.startModules`的回调:

```text
 if(err) {
        utils.invokeCallback(cb, err);
        return;
      }

      if(self.app.get(Constants.RESERVED.MODE) !== Constants.RESERVED.STAND_ALONE) {
        starter.runServers(self.app);
      }
      utils.invokeCallback(cb);
```

由于不是独立模式，会进入:`starter.runServers(self.app);`。

```text
// pomelo\lib\master\starter.js
/**
 * Run all servers
 *
 * @param {Object} app current application  context
 * @return {Void}
 */
 starter.runServers = function(app) {
  console.trace("runServer")
  var server, servers;
  var condition = app.startId || app.type;
  switch(condition) {
    case Constants.RESERVED.MASTER:
    break;
    case Constants.RESERVED.ALL:
    servers = app.getServersFromConfig();
    for (var serverId in servers) {
      this.run(app, servers[serverId]);
    }
    break;
    default:
    server = app.getServerFromConfig(condition);
    if(!!server) {
      this.run(app, server);
    } else {
      servers = app.get(Constants.RESERVED.SERVERS)[condition];
      for(var i=0; i<servers.length; i++) {
        this.run(app, servers[i]);
      }
    }
  }
};
```

这里我们的`condition`实际上就是`Constants.RESERVED.ALL`,就会执行:

```text
// pomelo\lib\master\starter.js
    servers = app.getServersFromConfig();
    for (var serverId in servers) {
      this.run(app, servers[serverId]);
    }
```

上面就是获取servers.json中的配置，然后依次运行，我们看下`run`函数:

```text
// pomelo\lib\master\starter.js
/**
 * Run server
 *
 * @param {Object} app current application context
 * @param {Object} server
 * @return {Void}
 */
starter.run = function(app, server, cb) {
  console.log(`run ${process.pid} `)
  env = app.get(Constants.RESERVED.ENV);
  var cmd, key;
  if (utils.isLocal(server.host)) {
...
    starter.localrun(process.execPath, null, options, cb);
  } else {
...
    starter.sshrun(cmd, server.host, cb);
  }
};
```

上面根据host，区分是不是本机地址，调用不同的接口:`starter.localrun`和`starter.sshr.n`。这两个函数，最后都会调用`starter.spawnProcess`:

```text
/**
 * Fork child process to run command.
 *
 * @param {String} command
 * @param {Object} options
 * @param {Callback} callback
 *
 */
var spawnProcess = function(command, host, options, cb) {
  console.trace("spawnProcess", command, options)
  var child = null;

  if(env === Constants.RESERVED.ENV_DEV) {
    child = cp.spawn(command, options);
    ...
  } else {
    child = cp.spawn(command, options, {detached: true, stdio: 'inherit'});
    console.log(`spawnProcess3 ${process.pid} `, command, options)
    child.unref();
  }

 ...
};
```

上面最后就是调用nodejs的child\_process模块的spawn函数,创建新进程:

```text
var cp = require('child_process');
cp.spawn(command, options,...)
```

我调试了下，得到`command` 和 `option`参数：

```text
command:'D:\\Program Files\\nodejs\\node.exe'
option:
[
  "e:\\3rdparty\\pomelo_proj\\HelloWorld\\game-server\\app.js",
  "env=development",
  "id=connector-server-1",
  "host=127.0.0.1",
  "port=3150",
  "clientHost=127.0.0.1",
  "clientPort=3010",
  "frontend=true",
  "serverType=connector",
]
```

从中我们可以看出，其实有执行了一次app.js，只是现在的参数不一样了，例如:`serverType`。

以上就是master进程和子进程的创建流程。



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



我后来在自己电脑上全局搜索了下，在node源码路径找到：node-v14.16.1\deps\npm\lib\utils\lifecycle.js

```text
exports = module.exports = runLifecycle

const lifecycleOpts = require('../config/lifecycle')
const lifecycle = require('npm-lifecycle')

function runLifecycle (pkg, stage, wd, moreOpts, cb) {
  if (typeof moreOpts === 'function') {
    cb = moreOpts
    moreOpts = null
  }

  const opts = lifecycleOpts(moreOpts)
  lifecycle(pkg, stage, wd, opts).then(cb, cb)
}
```

查看上面 lifecycleOpts 对应的文件:

```text
'use strict'

const npm = require('../npm.js')
const log = require('npmlog')

module.exports = lifecycleOpts

let opts

function lifecycleOpts (moreOpts) {
  if (!opts) {
    opts = {
      config: npm.config.snapshot,
      dir: npm.dir,
      failOk: false,
      force: npm.config.get('force'),
      group: npm.config.get('group'),
      ignorePrepublish: npm.config.get('ignore-prepublish'),
      ignoreScripts: npm.config.get('ignore-scripts'),
      log: log,
      nodeOptions: npm.config.get('node-options'),
      production: npm.config.get('production'),
      scriptShell: npm.config.get('script-shell'),
      scriptsPrependNodePath: npm.config.get('scripts-prepend-node-path'),
      unsafePerm: npm.config.get('unsafe-perm'),
      user: npm.config.get('user')
    }
  }

  return moreOpts ? Object.assign({}, opts, moreOpts) : opts
}
```

想要了解 npm 中的npm-lifecycle，请戳[链接](https://github.com/npm/npm-lifecycle)

接下来我们回到: Application.start，进入到appUtil.startByType的匿名回调函数，进入到插件的start阶段。



