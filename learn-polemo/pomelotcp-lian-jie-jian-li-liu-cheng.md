# 【Pomelo】tcp 连接建立流程

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
    this.server.on('connection', this.newSocket.bind(this));
  } else {
    this.server.on('secureConnection', this.newSocket.bind(this));
    this.server.on('clientError', function(e, tlsSo) {
      logger.warn('an ssl error occured before handshake established: ', e);
      tlsSo.destroy();
    });
  }

  this.wsprocessor.on('connection', this.emit.bind(this, 'connection'));
  this.tcpprocessor.on('connection', this.emit.bind(this, 'connection'));

  this.state = ST_STARTED;
};
```

然后出发回调:

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
  this.emit('connection', tcpsocket);
  socket.emit('data', data);
};
```





