# Pomelo点击:Test Game Server 按钮后流程

前提:搭建好HelloWorld环境

点击:点击:Test Game Server 按钮，客户端向服务器发送连接消息。

服务器:switcher.js 中　Switcher.prototype.newSocket接口，进入:

```text
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

由于是http消息。所以会进入:processHttp:

```text
var processHttp = function(switcher, processor, socket, data) {
  processor.add(socket, data);
};
```

他会分发'data'消息:

```text
Processor.prototype.add = function(socket, data) {
  if(this.state !== ST_STARTED) {
    return;
  }
  this.httpServer.emit('connection', socket);
  if(typeof socket.ondata === 'function') {
    // compatible with stream2
    socket.ondata(data, 0, data.length);
  } else {
    // compatible with old stream
    socket.emit('data', data); // -> 此处
  }
};
```

通过vscode调试发现，分发的‘data’事件，会转换为'upgrade'，这个转换是在node内部　\_http\_server.js中onParserExecuteCommon接口实现:

```text
function onParserExecuteCommon(server, socket, parser, state, ret, d) {
...
// 就在下面这行
    const eventName = req.method === 'CONNECT' ? 'connect' : 'upgrade';
    if (eventName === 'upgrade' || server.listenerCount(eventName) > 0) {
      debug('SERVER have listener for %s', eventName);
      const bodyHead = d.slice(ret, d.length);

      socket.readableFlowing = null;

      // Clear the requestTimeout after upgrading the connection.
      clearRequestTimeout(req);

      server.emit(eventName, req, socket, bodyHead);
    } else {
...
    }
...
}
```

所以就看下，是哪一个socket监听'upgrade'事件:

```text
    this._onServerUpgrade = function(req, socket, upgradeHead) {
      //copy upgradeHead to avoid retention of large slab buffers used in node core
      var head = new Buffer(upgradeHead.length);
      upgradeHead.copy(head);

      self.handleUpgrade(req, socket, head, function(client) {
        self.emit('connection'+req.url, client);
        self.emit('connection', client);
      });
    };
    this._server.on('upgrade', this._onServerUpgrade);
```

上面函数使用了buffer的copy函数，将upgradeHead内存拷贝到head中:

```text
// nodejs中的buffer.js
Buffer.prototype.copy =
  function copy(target, targetStart, sourceStart, sourceEnd) {
    return _copy(this, target, targetStart, sourceStart, sourceEnd);
  };
```

接下来调用栈:

```text
WebSocketServer.prototype.handleUpgrade ->
completeHybiUpgrade1 ->
completeHybiUpgrade2 -> new WebSocket
```

最后在　completeHybiUpgrade2　中，创建客户端WebSocket。

在Websocket构造函数中，去建立连接:

```text
function establishConnection(ReceiverClass, SenderClass, socket, upgradeHead) {
...
  // ensure that the upgradeHead is added to the receiver
  function firstHandler(data) {
    if (called || self.readyState === WebSocket.CLOSED) return;

    called = true;
    socket.removeListener('data', firstHandler);
    ultron.on('data', realHandler);

    if (upgradeHead && upgradeHead.length > 0) {
      realHandler(upgradeHead);
      upgradeHead = null;
    }

    if (data) realHandler(data);
  }

  // subsequent packets are pushed straight to the receiver
  function realHandler(data) {
    self.bytesReceived += data.length;
    self._receiver.add(data);
  }

  ultron.on('data', firstHandler);

  process.nextTick(firstHandler);

  // receiver event handlers
  self._receiver.ontext = function ontext(data, flags) {
    flags = flags || {};

    self.emit('message', data, flags);
  };

  self._receiver.onbinary = function onbinary(data, flags) {
    flags = flags || {};

    flags.binary = true;
    self.emit('message', data, flags);
  };

...

  this.readyState = WebSocket.OPEN;
  this.emit('open');
}
```

在Websocket构造函数中，去emit消息:

```text
 self.emit('connection'+req.url, client);
 self.emit('connection', client);
```

监听‘connection’的回调一通执行，然后执行到hostFilter:

```text
pro.afterStart = function(cb) {
  this.connector.start(cb);
  this.connector.on('connection', hostFilter.bind(this, bindEvents));
};
```

然后执行bindEvents:

```text
var bindEvents = function(self, socket) {
 ...

  //create session for connection
  var session = getSession(self, socket);
  var closed = false;

 ...
  // new message
  socket.on('message', function(msg) {
    ...
    if (self.decode) {
      dmsg = self.decode(msg, session);
    } else if (self.connector.decode) {
      dmsg = self.connector.decode(msg, socket);
    }
    ...
    handleMessage(self, session, dmsg);
  }); //on message end
};
```

上面可以看到监听了'message'，我们客户端发送上来的消息最后就会到这里，经过解包，然后调用connector.js中handleMessage，此函数中调用self.server.globalHandle，会执行:

```text
// game-server/app/servers/connector/handler/entryHandler.js
Handler.prototype.entry = function(msg, session, next) {
  next(null, {code: 200, msg: 'game server is ok.'});
};
```

并在pro.globalHandle会respond给客户端，客户端收到消息后弹出窗口。



