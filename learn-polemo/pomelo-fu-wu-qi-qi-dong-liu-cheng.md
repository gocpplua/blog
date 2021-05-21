# 【Pomelo】服务器启动流程

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

接下来的细节就不在这边展开了。

