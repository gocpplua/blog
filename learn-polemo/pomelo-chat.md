---
description: 'Chat源码下载与安装:'
---

# 【Pomelo】chat

[\[官方\]Chat源码下载与安装](https://github.com/NetEase/pomelo/wiki/chat%E6%BA%90%E7%A0%81%E4%B8%8B%E8%BD%BD%E4%B8%8E%E5%AE%89%E8%A3%85)

执行:sh npm-install.sh

出现下面错误:

```text
npm ERR! code 1
npm ERR! path /data/gocpplua/pomelo/pomelo_prj/chatofpomelo-websocket/game-server/node_modules/heapdump
npm ERR! command failed
npm ERR! command sh -c node-gyp rebuild
npm ERR! make: Entering directory '/data/gocpplua/pomelo/pomelo_prj/chatofpomelo-websocket/game-server/node_modules/heapdump/build'
npm ERR!   CXX(target) Release/obj.target/addon/src/heapdump.o
npm ERR! addon.target.mk:109: recipe for target 'Release/obj.target/addon/src/heapdump.o' failed
npm ERR! make: Leaving directory '/data/gocpplua/pomelo/pomelo_prj/chatofpomelo-websocket/game-server/node_modules/heapdump/build'
npm ERR! gyp info it worked if it ends with ok
npm ERR! gyp info using node-gyp@7.1.2
npm ERR! gyp info using node@14.16.1 | linux | x64
npm ERR! gyp info find Python using Python version 3.5.2 found at "/usr/bin/python3"
npm ERR! gyp info spawn /usr/bin/python3
npm ERR! gyp info spawn args [
npm ERR! gyp info spawn args   '/home/SENSETIME/chenqi1/.nvm/versions/node/v14.16.1/lib/node_modules/npm/node_modules/node-gyp/gyp/gyp_main.py',
npm ERR! gyp info spawn args   'binding.gyp',
npm ERR! gyp info spawn args   '-f',
npm ERR! gyp info spawn args   'make',
npm ERR! gyp info spawn args   '-I',
npm ERR! gyp info spawn args   '/data/gocpplua/pomelo/pomelo_prj/chatofpomelo-websocket/game-server/node_modules/heapdump/build/config.gypi',
npm ERR! gyp info spawn args   '-I',
npm ERR! gyp info spawn args   '/home/SENSETIME/chenqi1/.nvm/versions/node/v14.16.1/lib/node_modules/npm/node_modules/node-gyp/addon.gypi',
npm ERR! gyp info spawn args   '-I',
npm ERR! gyp info spawn args   '/home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/common.gypi',
npm ERR! gyp info spawn args   '-Dlibrary=shared_library',
npm ERR! gyp info spawn args   '-Dvisibility=default',
npm ERR! gyp info spawn args   '-Dnode_root_dir=/home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1',
npm ERR! gyp info spawn args   '-Dnode_gyp_dir=/home/SENSETIME/chenqi1/.nvm/versions/node/v14.16.1/lib/node_modules/npm/node_modules/node-gyp',
npm ERR! gyp info spawn args   '-Dnode_lib_file=/home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/<(target_arch)/node.lib',
npm ERR! gyp info spawn args   '-Dmodule_root_dir=/data/gocpplua/pomelo/pomelo_prj/chatofpomelo-websocket/game-server/node_modules/heapdump',
npm ERR! gyp info spawn args   '-Dnode_engine=v8',
npm ERR! gyp info spawn args   '--depth=.',
npm ERR! gyp info spawn args   '--no-parallel',
npm ERR! gyp info spawn args   '--generator-output',
npm ERR! gyp info spawn args   'build',
npm ERR! gyp info spawn args   '-Goutput_dir=.'
npm ERR! gyp info spawn args ]
npm ERR! gyp info spawn make
npm ERR! gyp info spawn args [ 'BUILDTYPE=Release', '-C', 'build' ]
npm ERR! In file included from ../src/heapdump.cc:17:
npm ERR! ../src/compat-inl.h: In static member function ‘static void compat::CpuProfiler::StartCpuProfiling(v8::Isolate*, v8::Local<v8::String>)’:
npm ERR! ../src/compat-inl.h:300:19: error: ‘class v8::Isolate’ has no member named ‘GetCpuProfiler’; did you mean ‘GetHeapProfiler’?
npm ERR!   300 |   return isolate->GetCpuProfiler()->StartProfiling(title, record_samples);
npm ERR!       |                   ^~~~~~~~~~~~~~
npm ERR!       |                   GetHeapProfiler
npm ERR! ../src/compat-inl.h:300:73: error: return-statement with a value, in function returning ‘void’ [-fpermissive]
npm ERR!   300 |   return isolate->GetCpuProfiler()->StartProfiling(title, record_samples);
npm ERR!       |                                                                         ^
npm ERR! ../src/compat-inl.h: In static member function ‘static const v8::CpuProfile* compat::CpuProfiler::StopCpuProfiling(v8::Isolate*, v8::Local<v8::String>)’:
npm ERR! ../src/compat-inl.h:310:19: error: ‘class v8::Isolate’ has no member named ‘GetCpuProfiler’; did you mean ‘GetHeapProfiler’?
npm ERR!   310 |   return isolate->GetCpuProfiler()->StopProfiling(title);
npm ERR!       |                   ^~~~~~~~~~~~~~
npm ERR!       |                   GetHeapProfiler
npm ERR! ../src/compat-inl.h: In static member function ‘static v8::Local<v8::String> compat::String::NewFromUtf8(v8::Isolate*, const char*, compat::String::NewStringType, int)’:
npm ERR! ../src/compat-inl.h:341:46: error: ‘NewStringType’ in ‘class v8::String’ does not name a type
npm ERR!   341 |       isolate, data, static_cast<v8::String::NewStringType>(type), length);
npm ERR!       |                                              ^~~~~~~~~~~~~
npm ERR! ../src/heapdump.cc: At global scope:
npm ERR! ../src/heapdump.cc:52:11: error: ‘v8::Handle’ has not been declared
npm ERR!    52 | using v8::Handle;
npm ERR!       |           ^~~~~~
npm ERR! ../src/heapdump.cc: In function ‘compat::ReturnType {anonymous}::WriteSnapshot(const ArgumentType&)’:
npm ERR! ../src/heapdump.cc:106:46: error: no matching function for call to ‘v8::String::Utf8Value::Utf8Value(v8::Local<v8::Value>)’
npm ERR!   106 |     String::Utf8Value filename_string(args[0]);
npm ERR!       |                                              ^
npm ERR! In file included from /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:67,
npm ERR!                  from ../src/heapdump.cc:15:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3287:5: note: candidate: ‘v8::String::Utf8Value::Utf8Value(v8::Isolate*, v8::Local<v8::Value>)’
npm ERR!  3287 |     Utf8Value(Isolate* isolate, Local<v8::Value> obj);
npm ERR!       |     ^~~~~~~~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3287:5: note:   candidate expects 2 arguments, 1 provided
npm ERR! ../src/heapdump.cc: In function ‘void {anonymous}::InvokeCallback(const char*)’:
npm ERR! ../src/heapdump.cc:144:32: warning: ‘v8::Local<v8::Value> node::MakeCallback(v8::Isolate*, v8::Local<v8::Object>, v8::Local<v8::Function>, int, v8::Local<v8::Value>*)’ is deprecated: Use MakeCallback(..., async_context) [-Wdeprecated-declarations]
npm ERR!   144 |                      argc, argv);
npm ERR!       |                                ^
npm ERR! In file included from ../src/heapdump.cc:15:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:191:50: note: declared here
npm ERR!   191 |                 NODE_EXTERN v8::Local<v8::Value> MakeCallback(
npm ERR!       |                                                  ^~~~~~~~~~~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:108:42: note: in definition of macro ‘NODE_DEPRECATED’
npm ERR!   108 |     __attribute__((deprecated(message))) declarator
npm ERR!       |                                          ^~~~~~~~~~
npm ERR! ../src/heapdump.cc:144:32: warning: ‘v8::Local<v8::Value> node::MakeCallback(v8::Isolate*, v8::Local<v8::Object>, v8::Local<v8::Function>, int, v8::Local<v8::Value>*)’ is deprecated: Use MakeCallback(..., async_context) [-Wdeprecated-declarations]
npm ERR!   144 |                      argc, argv);
npm ERR!       |                                ^
npm ERR! In file included from ../src/heapdump.cc:15:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:191:50: note: declared here
npm ERR!   191 |                 NODE_EXTERN v8::Local<v8::Value> MakeCallback(
npm ERR!       |                                                  ^~~~~~~~~~~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:108:42: note: in definition of macro ‘NODE_DEPRECATED’
npm ERR!   108 |     __attribute__((deprecated(message))) declarator
npm ERR!       |                                          ^~~~~~~~~~
npm ERR! ../src/heapdump.cc: In function ‘compat::ReturnType {anonymous}::Configure(const ArgumentType&)’:
npm ERR! ../src/heapdump.cc:157:55: error: no matching function for call to ‘v8::Value::Int32Value()’
npm ERR!   157 |   PlatformInit(args.GetIsolate(), args[0]->Int32Value());
npm ERR!       |                                                       ^
npm ERR! In file included from /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:67,
npm ERR!                  from ../src/heapdump.cc:15:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:2869:40: note: candidate: ‘v8::Maybe<int> v8::Value::Int32Value(v8::Local<v8::Context>) const’
npm ERR!  2869 |   V8_WARN_UNUSED_RESULT Maybe<int32_t> Int32Value(Local<Context> context) const;
npm ERR!       |                                        ^~~~~~~~~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:2869:40: note:   candidate expects 1 argument, 0 provided
npm ERR! ../src/heapdump.cc: In function ‘void {anonymous}::Initialize(v8::Local<v8::Object>)’:
npm ERR! ../src/heapdump.cc:164:51: error: no matching function for call to ‘v8::Object::Set(v8::Local<v8::String>, v8::Local<v8::Integer>)’
npm ERR!   164 |                C::Integer::New(isolate, kForkFlag));
npm ERR!       |                                                   ^
npm ERR! In file included from /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:67,
npm ERR!                  from ../src/heapdump.cc:15:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3670:37: note: candidate: ‘v8::Maybe<bool> v8::Object::Set(v8::Local<v8::Context>, v8::Local<v8::Value>, v8::Local<v8::Value>)’
npm ERR!  3670 |   V8_WARN_UNUSED_RESULT Maybe<bool> Set(Local<Context> context,
npm ERR!       |                                     ^~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3670:37: note:   candidate expects 3 arguments, 2 provided
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3673:37: note: candidate: ‘v8::Maybe<bool> v8::Object::Set(v8::Local<v8::Context>, uint32_t, v8::Local<v8::Value>)’
npm ERR!  3673 |   V8_WARN_UNUSED_RESULT Maybe<bool> Set(Local<Context> context, uint32_t index,
npm ERR!       |                                     ^~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3673:37: note:   candidate expects 3 arguments, 2 provided
npm ERR! ../src/heapdump.cc:166:53: error: no matching function for call to ‘v8::Object::Set(v8::Local<v8::String>, v8::Local<v8::Integer>)’
npm ERR!   166 |                C::Integer::New(isolate, kSignalFlag));
npm ERR!       |                                                     ^
npm ERR! In file included from /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:67,
npm ERR!                  from ../src/heapdump.cc:15:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3670:37: note: candidate: ‘v8::Maybe<bool> v8::Object::Set(v8::Local<v8::Context>, v8::Local<v8::Value>, v8::Local<v8::Value>)’
npm ERR!  3670 |   V8_WARN_UNUSED_RESULT Maybe<bool> Set(Local<Context> context,
npm ERR!       |                                     ^~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3670:37: note:   candidate expects 3 arguments, 2 provided
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3673:37: note: candidate: ‘v8::Maybe<bool> v8::Object::Set(v8::Local<v8::Context>, uint32_t, v8::Local<v8::Value>)’
npm ERR!  3673 |   V8_WARN_UNUSED_RESULT Maybe<bool> Set(Local<Context> context, uint32_t index,
npm ERR!       |                                     ^~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:3673:37: note:   candidate expects 3 arguments, 2 provided
npm ERR! ../src/heapdump.cc:168:74: error: no matching function for call to ‘v8::FunctionTemplate::GetFunction()’
npm ERR!   168 |                C::FunctionTemplate::New(isolate, Configure)->GetFunction());
npm ERR!       |                                                                          ^
npm ERR! In file included from /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:67,
npm ERR!                  from ../src/heapdump.cc:15:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:6482:46: note: candidate: ‘v8::MaybeLocal<v8::Function> v8::FunctionTemplate::GetFunction(v8::Local<v8::Context>)’
npm ERR!  6482 |   V8_WARN_UNUSED_RESULT MaybeLocal<Function> GetFunction(
npm ERR!       |                                              ^~~~~~~~~~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:6482:46: note:   candidate expects 1 argument, 0 provided
npm ERR! ../src/heapdump.cc:170:78: error: no matching function for call to ‘v8::FunctionTemplate::GetFunction()’
npm ERR!   170 |                C::FunctionTemplate::New(isolate, WriteSnapshot)->GetFunction());
npm ERR!       |                                                                              ^
npm ERR! In file included from /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:67,
npm ERR!                  from ../src/heapdump.cc:15:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:6482:46: note: candidate: ‘v8::MaybeLocal<v8::Function> v8::FunctionTemplate::GetFunction(v8::Local<v8::Context>)’
npm ERR!  6482 |   V8_WARN_UNUSED_RESULT MaybeLocal<Function> GetFunction(
npm ERR!       |                                              ^~~~~~~~~~~
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/v8.h:6482:46: note:   candidate expects 1 argument, 0 provided
npm ERR! In file included from ../src/heapdump.cc:15:
npm ERR! ../src/heapdump.cc: At global scope:
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:758:43: warning: cast between incompatible function types from ‘void (*)(v8::Local<v8::Object>)’ to ‘node::addon_register_func’ {aka ‘void (*)(v8::Local<v8::Object>, v8::Local<v8::Value>, void*)’} [-Wcast-function-type]
npm ERR!   758 |       (node::addon_register_func) (regfunc),                          \
npm ERR!       |                                           ^
npm ERR! /home/SENSETIME/chenqi1/.cache/node-gyp/14.16.1/include/node/node.h:792:3: note: in expansion of macro ‘NODE_MODULE_X’
npm ERR!   792 |   NODE_MODULE_X(modname, regfunc, NULL, 0)  // NOLINT (readability/null_usage)
npm ERR!       |   ^~~~~~~~~~~~~
npm ERR! ../src/heapdump.cc:173:1: note: in expansion of macro ‘NODE_MODULE’
npm ERR!   173 | NODE_MODULE(addon, Initialize)
npm ERR!       | ^~~~~~~~~~~
npm ERR! make: *** [Release/obj.target/addon/src/heapdump.o] Error 1
npm ERR! gyp ERR! build error 
npm ERR! gyp ERR! stack Error: `make` failed with exit code: 2
npm ERR! gyp ERR! stack     at ChildProcess.onExit (/home/SENSETIME/chenqi1/.nvm/versions/node/v14.16.1/lib/node_modules/npm/node_modules/node-gyp/lib/build.js:194:23)
npm ERR! gyp ERR! stack     at ChildProcess.emit (events.js:315:20)
npm ERR! gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:277:12)
npm ERR! gyp ERR! System Linux 4.15.0-142-generic
npm ERR! gyp ERR! command "/home/SENSETIME/chenqi1/.nvm/versions/node/v14.16.1/bin/node" "/home/SENSETIME/chenqi1/.nvm/versions/node/v14.16.1/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
npm ERR! gyp ERR! cwd /data/gocpplua/pomelo/pomelo_prj/chatofpomelo-websocket/game-server/node_modules/heapdump
npm ERR! gyp ERR! node -v v14.16.1
npm ERR! gyp ERR! node-gyp -v v7.1.2
npm ERR! gyp ERR! not ok
npm timing npm Completed in 5451ms

npm ERR! A complete log of this run can be found in:
npm ERR!     /home/SENSETIME/chenqi1/.npm/_logs/2021-05-24T12_40_11_114Z-debug.log
============   game-server npm installed ============

```

然后网上说执行下面命令即可:

> npm install --global --production build-tools

可是我操作以后还是不行。

由于自己误操作，执行了:

> polemo start

既然可以正常运行，于是上面错误我就不管了。

然后启动 web-server以后，通过浏览器登录两个账号，就可以聊天了。

> 注意:enter your channel 　需要填写一样。

通过命令查看pomelo服务:

```text
XXX\XXX@cn0314000510l:~$ pomelo list
try to connect 127.0.0.1:3005
(node:371) Warning: Accessing non-existent property 'padLevels' of module exports inside circular dependency
(Use `node --trace-warnings ...` to show where the warning was created)
serverId           serverType pid   rss(M) heapTotal(M) heapUsed(M) uptime(m) 
chat-server-1      chat       31392 45.67  12.90        11.77       3.50      
connector-server-1 connector  31391 48.51  15.02        13.55       3.50      
gate-server-1      gate       31393 47.05  14.27        12.92       3.50      
master-server-1    master     31380 44.75  12.40        11.20       3.50 
```

可以看到，除了master 和　connector　服务，还多了chat和gate。

这些服务都是通过 servers.json 里面进行配置，下面就是servers.json 文件:

```text
{
    "development":{
        "connector":[
             {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true}
         ],
        "chat":[
             {"id":"chat-server-1", "host":"127.0.0.1", "port":6050}
        ],
        "gate":[
	       {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
	    ]
    },
    "production":{
           "connector":[
             {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true}
         ],
        "chat":[
             {"id":"chat-server-1", "host":"127.0.0.1", "port":6050}
        ],
        "gate":[
           {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true}
        ]
  }
}

```



接着启动web-server，然后打开浏览器，输入`http://127.0.0.1:3001/index.html`, 输入一个用户名和一个房间名，点击　Join按钮。

首先在下述代码中收到消息:

```text
// pomelo/lib/connectors/hybrid/switcher.js
socket.once('data', function(data) {
    // FIXME: handle incomplete HTTP method
    if(isHttp(data)) {
      processHttp(self, self.wsprocessor, socket, data);
    } else {
      if(!!self.setNoDelay) {
        socket.setNoDelay(true);
      }
      processTcp(self, self.tcpprocessor, socket, data);
    }
  });
```

收到'connection'消息，然后到:

```text
  // pomelo/lib/connectors/hybridconnector.js
  this.switcher.on('connection', function(socket) {
    gensocket(socket);
  });
```

在gensocket中，设置了id,并且发送'connection'事件:

```text
    // pomelo/lib/connectors/hybridconnector.js
  var gensocket = function(socket) {
    var hybridsocket = new HybridSocket(curId++, socket);
    hybridsocket.on('handshake', self.handshake.handle.bind(self.handshake, hybridsocket));
    hybridsocket.on('heartbeat', self.heartbeat.handle.bind(self.heartbeat, hybridsocket));
    hybridsocket.on('disconnect', self.heartbeat.clear.bind(self.heartbeat, hybridsocket.id));
    hybridsocket.on('closing', Kick.handle.bind(null, hybridsocket));
    self.emit('connection', hybridsocket);
  };
```

在connector.js中监听了'connection'事件:

```text
pro.afterStart = function(cb) {
  this.connector.start(cb);
  this.connector.on('connection', hostFilter.bind(this, bindEvents));
};
```

然后执行bindEvents:

```text
// /pomelo/lib/components/connector.js
var bindEvents = function(self, socket) {
  var curServer = self.app.getCurServer();
  var maxConnections = curServer['max-connections'];
  if (self.connection && maxConnections) {
    self.connection.increaseConnectionCount();
    var statisticInfo = self.connection.getStatisticsInfo();
    if (statisticInfo.totalConnCount > maxConnections) {
      logger.warn('the server %s has reached the max connections %s', curServer.id, maxConnections);
      socket.disconnect();
      return;
    }
  }
```

在pomelo/lib/common/service/sessionService.js中创建一个session.其中入参sid就是socket id:

```text
// pomelo/lib/common/service/sessionService.js
SessionService.prototype.create = function(sid, frontendId, socket) {
  var session = new Session(sid, frontendId, socket, this);
  this.sessions[session.id] = session;

  return session;
};
```

接着会收到‘meaasge’消息:

```text
  // /pomelo/lib/connectors/hybridsocket.js
  socket.on('message', function(msg) {
    if(msg) {
      msg = Package.decode(msg);
      handler(self, msg);
    }
  });
```

在上述接口中一次收到TYPE\_HANDSHAKE、TYPE\_HANDSHAKE\_ACK、TYPE\_HEARTBEAT，最后收到TYPE\_DATA数据,于是进入回调:

```text
// pomelo/lib/connectors/common/handler.js
var handleData = function(socket, pkg) {
  if(socket.state !== ST_WORKING) {
    return;
  }
  socket.emit('message', pkg);
};
```

connector收到`'message'事件(客户端发送:`'gate.gateHandler.queryEntry'`)，并且处理:`

```text
//  /pomelo/lib/components/connector.js
var bindEvents = function(self, socket) {
 ...
  // new message
  socket.on('message', function(msg) {
   ...

    handleMessage(self, session, dmsg);
  }); //on message end
};
```

然后调用组件server.js的globalHandle接口:

```text
// pomelo/lib/components/server.js
/**
 * Proxy server global handle
 */
pro.globalHandle = function(msg, session, cb) {
	this.server.globalHandle(msg, session, cb);
};
```

再调用下述接口\(入参msg:{id: 1, type: 0, compressRoute: 0, route: 'gate.gateHandler.queryEntry', body: {…}, …}\):

```text
// /pomelo/lib/server/server.js
pro.globalHandle = function(msg, session, cb) {
...
  var dispatch = function(err, resp, opts) {
    if(err) {
      handleError(true, self, err, msg, session, resp, opts, function(err, resp, opts) {
        response(true, self, err, msg, session, resp, opts, cb);
      });
      return;
    }

    if(self.app.getServerType() !== routeRecord.serverType) {
      doForward(self.app, msg, session, routeRecord, function(err, resp, opts) {
        response(true, self, err, msg, session, resp, opts, cb);
      });
    } else {
      doHandle(self, msg, session, routeRecord, function(err, resp, opts) {
        response(true, self, err, msg, session, resp, opts, cb);
      });
    }
  };
  beforeFilter(true, self, msg, session, dispatch);
};
```

最后会queryEntry，得到对应的connector对应的host和port。

```text
// app/servers/gate/handler/gateHandler.js
handler.queryEntry = function(msg, session, next) {
	...
	// here we just start `ONE` connector server, so we return the connectors[0] 
	var res = connectors[0];
	next(null, {
		code: 200,
		host: res.host,
		port: res.clientPort
	});
};
```

接着客户端发送进房间消息\(路由:'connector.entryHandler.enter'\)，服务器将用户添加到对应的channel:

```text
// app/servers/connector/handler/entryHandler.js
handler.enter = function(msg, session, next) {
	...

	//put user into channel
	self.app.rpc.chat.chatRemote.add(session, uid, self.app.get('serverId'), rid, true, function(users){
		next(null, {
			users:users
		});
	});
};
```

然后进入ChatRemote.prototype.add:

```text
// app/servers/chat/remote/chatRemote.js
ChatRemote.prototype.add = function(uid, sid, name, flag, cb) {
	var channel = this.channelService.getChannel(name, flag);
	var username = uid.split('*')[0];
	var param = {
		route: 'onAdd',
		user: username
	};
	channel.pushMessage(param);

	if( !! channel) {
		channel.add(uid, sid);
	}

	cb(this.get(name, flag));
};

```

接着进入回调，返回到entryHandler.js的next　回调,

```text
	//put user into channel
	self.app.rpc.chat.chatRemote.add(session, uid, self.app.get('serverId'), rid, true, function(users){
		next(null, {
			users:users
		});
	});
```

最后调用:

```text
// pomelo/lib/server/server.js
/**
 * Send response to client and fire after filter chain if any.
 */

var response = function(isGlobal, server, err, msg, session, resp, opts, cb) 
```

