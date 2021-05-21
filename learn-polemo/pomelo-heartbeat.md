---
description: '参考文章:https://www.jianshu.com/p/b392b75409f9'
---

# Pomelo Heartbeat

在lib/connectors/hybridconnector.js中，看到了对心跳消息'heartbeat'的监听:

```text
hybridsocket.on('heartbeat', self.heartbeat.handle.bind(self.heartbeat, hybridsocket));
```

在game-server/node\_modules/pomelo/lib/connectors/common/handler.js中，看到心跳消息的分发:

```text
var handleHeartbeat = function(socket, pkg) {
  if(socket.state !== ST_WORKING) {
    return;
  }
  socket.emit('heartbeat');
};
```

默认情况下客户端每个３秒中发送一个心跳，服务器进入'message'事件回调:

```text
  // lib/connectors/hybridsocket.js
  socket.on('message', function(msg) {
    if(msg) {
      msg = Package.decode(msg);
      handler(self, msg);
    }
  });
```

解析msg后，获取到type:3

```text
// lib/connectors/common/handler.js
handlers[Package.TYPE_HANDSHAKE] = handleHandshake;
handlers[Package.TYPE_HANDSHAKE_ACK] = handleHandshakeAck;
handlers[Package.TYPE_HEARTBEAT] = handleHeartbeat;
handlers[Package.TYPE_DATA] = handleData;
```

```text
  // node_modules/pomelo-protocol 模块　protocol.js
  Package.TYPE_HANDSHAKE = 1;
  Package.TYPE_HANDSHAKE_ACK = 2;
  Package.TYPE_HEARTBEAT = 3;
  Package.TYPE_DATA = 4;
  Package.TYPE_KICK = 5;
```

执行回调 handleHeartbeat,分发'heartbeat'事件:

```text
var handleHeartbeat = function(socket, pkg) {
  if(socket.state !== ST_WORKING) {
    return;
  }
  socket.emit('heartbeat');
};
```

hybridsocket　收到　`'heartbeat'` 消息后，执行回调:

```text
// lib/connectors/commands/heartbeat.js
Command.prototype.handle = function(socket) {
  if(!this.heartbeat) {
    // no heartbeat setting
    return;
  }

  var self = this;

  if(!this.clients[socket.id]) {
    // clear timers when socket disconnect or error
    this.clients[socket.id] = 1;
    socket.once('disconnect', clearTimers.bind(null, this, socket.id));
    socket.once('error', clearTimers.bind(null, this, socket.id));
  }

  // clear timeout timer
  if(self.disconnectOnTimeout) {
    this.clear(socket.id);
  }

// 回复给客户端(sendRaw实现在hybridsocket.js中)
  socket.sendRaw(Package.encode(Package.TYPE_HEARTBEAT));

  if(self.disconnectOnTimeout) {
    self.timeouts[socket.id] = setTimeout(function() {
      logger.info('client %j heartbeat timeout.', socket.id);
      socket.disconnect();
    }, self.timeout);
  }
};
```

默认 self.disconnectOnTimeout = true，所以上述会设置超时回调:

```text
self.timeouts[socket.id] = setTimeout(function() {
      logger.info('client %j heartbeat timeout.', socket.id);
      socket.disconnect();
    }, self.timeout);
  }
```

disconnect实现在hybridsocket.js中，如下:

```text
Socket.prototype.disconnect = function() {
  if(this.state === ST_CLOSED) {
    return;
  }

  this.state = ST_CLOSED;
  this.socket.emit('close');
  this.socket.close();
};
```

发送`'close'事件，关闭socket。`　

`我们在监听'close'事件看到，写法都是:`

```text
socket.once('close', ...)
```

> Socket.once仅适用于事件侦听器-当您只想在下次事件发生时收到通知，而不是在事件的后续发生时得到通知。

这是一个很好的写法，因为socket只会close一次。

