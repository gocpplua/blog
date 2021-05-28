# 【Pomelo】tcp 连接建立和数据发送流程

> 先学习之前几篇文章，不然这篇文章看起来会有些困难。

服务器启动后，用终端发送:

```text
$> echo -n -e "\x01\x00\x00\x09\x7b\x22\x73\x79\x73\x22\x3a\x31\x7d" | nc 1
27.0.0.1 3010
```

在nodejs原生模块net.js中，如果建立建立连接，就会发送`'connection'消息:`

```text
// nodejs原生模块net.js
function onconnection(err, clientHandle) {
  const handle = this;
  const self = handle[owner_symbol];

  debug('onconnection');

  if (err) {
    self.emit('error', errnoException(err, 'accept'));
    return;
  }

  if (self.maxConnections && self._connections >= self.maxConnections) {
    clientHandle.close();
    return;
  }

  const socket = new Socket({
    handle: clientHandle,
    allowHalfOpen: self.allowHalfOpen,
    pauseOnCreate: self.pauseOnConnect,
    readable: true,
    writable: true
  });

  self._connections++;
  socket.server = self;
  socket._server = self;

  DTRACE_NET_SERVER_CONNECTION(socket);
  self.emit('connection', socket);
}
```

我们服务器中Switcher模块监听到`'connection':`

```text
var Switcher = function(server, opts) {
  ...
  if (!opts.ssl) {
    this.server.on('connection', this.newSocket.bind(this));　// 这里监听到的
  } else {
    this.server.on('secureConnection', this.newSocket.bind(this));
    this.server.on('clientError', function(e, tlsSo) {
      logger.warn('an ssl error occured before handshake established: ', e);
      tlsSo.destroy();
    });
  }

  this.wsprocessor.on('connection', this.emit.bind(this, 'connection'));
  this.tcpprocessor.on('connection', this.emit.bind(this, 'connection'));　// 这个需要注意下

  this.state = ST_STARTED;
};
```

Switcher在tcp连接的时候。可以监听到，说明已经初始化了，那么是什么时候初始化的呢？

首先在pomelo/lib/connectors/hybrid/switcher.js的构造函数填写堆栈打印:

```text
var Switcher = function(server, opts) {
  console.trace("Here!")　// 此方案很原始，但是也很有效
 ...
};
```

然后重启服务器，得到下述堆栈:

```text
[2021-05-21 15:19:58.789] [ERROR] console - Trace: Here!
    at new Switcher (/data/gocpplua/pomelo/pomelo_prj/HelloWorld/game-server/node_modules/pomelo/lib/connectors/hybrid/switcher.js:22:11)
    at Connector.start (/data/gocpplua/pomelo/pomelo_prj/HelloWorld/game-server/node_modules/pomelo/lib/connectors/hybridconnector.js:69:19)
    at Component.pro.afterStart (/data/gocpplua/pomelo/pomelo_prj/HelloWorld/game-server/node_modules/pomelo/lib/components/connector.js:74:18)
    at /data/gocpplua/pomelo/pomelo_prj/HelloWorld/game-server/node_modules/pomelo/lib/util/appUtil.js:111:19
    at iterate (/data/gocpplua/pomelo/pomelo_prj/HelloWorld/game-server/node_modules/async/lib/async.js:123:13)
    at processTicksAndRejections (internal/process/task_queues.js:75:11)
```

从而我们可以看到构造函数入参中的`'server',实际上是:`

```text
Connector.prototype.start = function(cb) {
 ...
  if(!this.ssl) {
    this.listeningServer = net.createServer(); // 非ssl情况下
  } else {
    this.listeningServer = tls.createServer(this.ssl); // ssl情况下
  }
  this.switcher = new Switcher(this.listeningServer, self.opts);

  this.switcher.on('connection', function(socket) {
    gensocket(socket);
  });
...
};
```

这里我就不展开了，有兴趣想要深入了解的小伙伴可以自己去调试下。上面为什么会跑到hybridconnector.js去，是因为我们app.js如下:

```text
app.configure('production|development', 'connector', function(){
  app.set('connectorConfig',
    {
      connector : pomelo.connectors.hybridconnector, // !!! 就是这个
      heartbeat : 3,
      useDict : true,
      useProtobuf : true
    });
});
```

我们接着往下讲，`'connection' 监听以后，`然后执行回调`newSocket`:

```text
Switcher.prototype.newSocket = function(socket) {
  if(this.state !== ST_STARTED) {
    return;
  }

  socket.setTimeout(this.timeout, function() {
     logger.warn('connection is timeout without communication, the remote ip is %s && port is %s',
       socket.remoteAddress, socket.remotePort);
     socket.destroy();
  });

  var self = this;

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
};
```

由于是tcp，就执行processTcp:

```text
var processTcp = function(switcher, processor, socket, data) {
  processor.add(socket, data);
};
```

```text
// pomelo/lib/connectors/hybrid/tcpprocessor.js
Processor.prototype.add = function(socket, data) {
  if(this.state !== ST_STARTED) {
    return;
  }
  var tcpsocket = new TcpSocket(socket, {headSize: HEAD_SIZE,
                                         headHandler: utils.headHandler,
                                         closeMethod: this.closeMethod});
  this.emit('connection', tcpsocket); 　// 第一步
  socket.emit('data', data);　// 第二步
};
```

## 第一步：发送`'connection'事件`

`被Switcher监听:`

```text
var Switcher = function(server, opts) {
  ...
  this.tcpprocessor.on('connection', this.emit.bind(this, 'connection'));　// 这个需要注意下
  ...
};
```

然后`Switcher这个对象也发送'connection'事件，那么我们看哪里进行监听了:`

```text
// pomelo/lib/connectors/hybridconnector.js
Connector.prototype.start = function(cb) {。
...
  var gensocket = function(socket) {
    var hybridsocket = new HybridSocket(curId++, socket);
    hybridsocket.on('handshake', self.handshake.handle.bind(self.handshake, hybridsocket));
    hybridsocket.on('heartbeat', self.heartbeat.handle.bind(self.heartbeat, hybridsocket));
    hybridsocket.on('disconnect', self.heartbeat.clear.bind(self.heartbeat, hybridsocket.id));
    hybridsocket.on('closing', Kick.handle.bind(null, hybridsocket));
    self.emit('connection', hybridsocket); // !!! 他也发送'connection'事件
  };
  ...
  this.switcher.on('connection', function(socket) {
    gensocket(socket);
  });
...
};
```

Connector分发`'connection'事件后，进入回调:`

```text
// game-server/node_modules/pomelo/lib/components/connector.js
pro.afterStart = function(cb) {
  this.connector.start(cb);
  this.connector.on('connection', hostFilter.bind(this, bindEvents));
};


var hostFilter = function(cb, socket) {
  if(!this.useHostFilter) {
    return cb(this, socket); //　执行回调:bindEvents
  }

...
};

var bindEvents = function(self, socket) {
...
  //create session for connection
  var session = getSession(self, socket);
  var closed = false;

  socket.on('disconnect', function() {
    ...
  });

  socket.on('error', function() {
  ...
  });

  // new message
  socket.on('message', function(msg) {
    ...
    handleMessage(self, session, dmsg);
  }); //on message end
};
```

所以我们就明白了，客户端发送上来的消息都是会先经过connector.js中。

## 第二步：socket.emit\('data', data\)

我们知道socket 其实就是 Switcher，而现在Switcher已经不监听`'data'消息。`

`我们看下socket去构造`TcpSocket发生了什么:

```text
// pomelo/lib/connectors/hybrid/tcpsocket.js
var Socket = function(socket, opts) {
  ...
  this._socket = socket;
 ...

  // bind event form the origin socket
  this._socket.on('data', ondata.bind(null, this));
  this._socket.on('end', onend.bind(null, this));
  this._socket.on('error', this.emit.bind(this, 'error'));
  this._socket.on('close', this.emit.bind(this, 'close'));

  this.state = ST_HEAD;
};
```

焕然大悟，TcpSocket监听了`'data'，然后执行回调:`

```text
// pomelo/lib/connectors/hybrid/tcpsocket.js
var ondata = function(socket, chunk) {
  if(socket.state === ST_CLOSED) {
    throw new Error('socket has closed');
  }

  if(typeof chunk !== 'string' && !Buffer.isBuffer(chunk)) {
    throw new Error('invalid data');
  }

  if(typeof chunk === 'string') {
    chunk = new Buffer(chunk, 'utf8');
  }

  var offset = 0, end = chunk.length;

  while(offset < end && socket.state !== ST_CLOSED) {
    if(socket.state === ST_HEAD) {
      offset = readHead(socket, chunk, offset);
    }

    if(socket.state === ST_BODY) {
      offset = readBody(socket, chunk, offset);
    }
  }

  return true;
};
```

在`'readBody'中，分发'message'`:

```text
// pomelo/lib/connectors/hybrid/tcpsocket.js
var readBody = function(socket, data, offset) {
  ...

  if(socket.packageOffset === socket.packageSize) {
    // if all the package finished
    var buffer = socket.packageBuffer;
    socket.emit('message', buffer); // 这里
    reset(socket);
  }

  return dend;
};
```

对这个socket分析可以得到，就是hybridsocket，

```text
// pomelo/lib/connectors/hybridsocket.js
var Socket = function(id, socket) {
  ...
  socket.on('message', function(msg) {
    if(msg) {
      msg = Package.decode(msg);
      handler(self, msg);
    }
  });

...
};
```

解析msg，type = 1

```text
handlers[Package.TYPE_HANDSHAKE] = handleHandshake; // Package.TYPE_HANDSHAKE = 1
handlers[Package.TYPE_HANDSHAKE_ACK] = handleHandshakeAck;
handlers[Package.TYPE_HEARTBEAT] = handleHeartbeat;
handlers[Package.TYPE_DATA] = handleData;

var handle = function(socket, pkg) {
  var handler = handlers[pkg.type];
  if(!!handler) {
    handler(socket, pkg);
  } else {
    logger.error('could not find handle invalid data package.');
    socket.disconnect();
  }
};

var handleHandshake = function(socket, pkg) {
  if(socket.state !== ST_INITED) {
    return;
  }
  try {
    socket.emit('handshake', JSON.parse(protocol.strdecode(pkg.body)));
  } catch (ex) {
    socket.emit('handshake', {});
  }
};
```

然后分发`'handshake'，执行:`

```text
// pomelo/lib/connectors/commands/handshake.js
Command.prototype.handle = function(socket, msg) {
  ...
  process.nextTick(function() {
    response(socket, opts);
  });
};
```

最后在上述函数中调用`'respond':`

```text
// pomelo/lib/connectors/commands/handshake.js
var response = function(socket, sys, resp) {
  var res = {
    code: CODE_OK,
    sys: sys
  };
  if(resp) {
    res.user = resp;
  }
  socket.handshakeResponse(Package.encode(Package.TYPE_HANDSHAKE, new Buffer(JSON.stringify(res))));
};
```

handshakeResponse 实现:

```text
// pomelo/lib/connectors/hybridsocket.js
Socket.prototype.handshakeResponse = function(resp) {
  if(this.state !== ST_INITED) {
    return;
  }

  this.socket.send(resp, {binary: true});
  this.state = ST_WAIT_ACK;
};
```

Good Job！！ 从建立TCP到收发数据，搞定！！

