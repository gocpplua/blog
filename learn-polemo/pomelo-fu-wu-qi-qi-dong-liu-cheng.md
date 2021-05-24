# 【Pomelo】pomelo start 启动流程

我们执行: pomelo start，然后服务器启动。

直接打开pomelo/bin/pomelo:

```text
program.command('start')
  .description('start the application')
  .option('-e, --env <env>', 'the used environment', DEFAULT_ENV)
  .option('-D, --daemon', 'enable the daemon start')
  .option('-d, --directory, <directory>', 'the code directory', DEFAULT_GAME_SERVER_DIR)
  .option('-t, --type <server-type>,', 'start server type')
  .option('-i, --id <server-id>', 'start server id')
  .action(function(opts) {
    start(opts);
  });
```

看到如果输入start,最后会执行start\(ops\):

```text
// pomelo/bin/pomelo
function start(opts) {
  var absScript = path.resolve(opts.directory, 'app.js');
  if (!fs.existsSync(absScript)) {
    abort(SCRIPT_NOT_FOUND);
  }

  var logDir = path.resolve(opts.directory, 'logs');
  if (!fs.existsSync(logDir)) {
    mkdir(logDir);
  }
  
  var ls;
  var type = opts.type || constants.RESERVED.ALL;
  var params = [absScript, 'env=' + opts.env, 'type=' + type];
  if(!!opts.id) {
    params.push('startId=' + opts.id);
  }
  
  //　spawn　创建子进程
  if (opts.daemon) {
    ls = spawn(process.execPath, params, {detached: true, stdio: 'ignore'});
    ls.unref();
    console.log(DAEMON_INFO);
    process.exit(0);
  } else {
    ls = spawn(process.execPath, params);
    ls.stdout.on('data', function(data) {
      console.log(data.toString());
    });
    ls.stderr.on('data', function(data) {
      console.log(data.toString());
    });
  }
}
```

所以，最后启动的是app.js。

```text
var pomelo = require('pomelo');

/**
 * Init app for client.
 */
var app = pomelo.createApp();
app.set('name', '$');

// app configuration
app.configure('production|development', 'connector', function(){
  app.set('connectorConfig',
    {
      connector : pomelo.connectors.hybridconnector, // 设置连接器
      heartbeat : 3,
      useDict : true, // 用户指令:dictionary.json 文件进行配置
      useProtobuf : true
    });
});

// start app
app.start();

process.on('uncaughtException', function (err) {
  console.error(' Caught exception: ' + err.stack);
});
```

在`'start()'函数中，使用`child\_process._spawn_\(\) 　异步生成子进程。

其中参数`process.execPath,` 是 Node.js 进程的可执行文件的绝对路径。

我修改了pemelo源码，将参数打印出来:

```text
$ pomelo start
undefined
[
  '/data/XXX/pomelo/pomelo_prj/HelloWorld/game-server/app.js',
  'env=development',
  'type=all'
]

```

所以pomelo　start 其实就是执行:

```text
>$ node /data/gocpplua/pomelo/pomelo_prj/HelloWorld/game-server/app.js env=development type=all
```

在app.js中:

```text
 Application.start = function(cb) {
  this.startTime = Date.now();
  if(this.state > STATE_INITED) {
    utils.invokeCallback(cb, new Error('application has already start.'));
    return;
  }
  
  var self = this;
  appUtil.startByType(self, function() {
    appUtil.loadDefaultComponents(self);
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
    var beforeFun = self.lifecycleCbs[Constants.LIFECYCLE.BEFORE_STARTUP];
    if(!!beforeFun) {
      beforeFun.call(null, self, startUp);
    } else {
      startUp();
    }
  });
};
```

接下来的细节就不在这边展开了。

