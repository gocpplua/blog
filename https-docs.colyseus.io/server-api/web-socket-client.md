# Client

## Web套接字客户端 <a id="web-socket-client"></a>

该`client`实例存在于：

* [`Room#clients`](https://docs.colyseus.io/server/room/#clients-websocket)
* [`Room#onJoin()`](https://docs.colyseus.io/server/room/#onjoin-client)
* [`Room#onLeave()`](https://docs.colyseus.io/server/room/#onleave-client-consented)
* [`Room#onMessage()`](https://docs.colyseus.io/server/room/#onmessage-type-callback)

> 这是来自[`ws`](https://www.npmjs.com/package/ws)程序包的原始WebSocket连接。有更多方法不建议与Colyseus一起使用。

### 属性 <a id="properties"></a>

#### `auth: any` <a id="auth-any"></a>

您在期间返回的自定义数据[`onAuth()`](https://docs.colyseus.io/server/room/#onauth-client-options-request)。

### 方法 <a id="methods"></a>

#### `send(type, message)` <a id="sendtype-message"></a>

发送消息给客户端一种消息。消息使用MsgPack编码，并且可以包含任何JSON可序列化的数据结构。

`type`可以是一个`string`或一个`number`。

**发送消息：**

```text
//
// sending message with a string type ("powerup")
//
client.send("powerup", { kind: "ammo" });

//
// sending message with a number type (1)
//
client.send(1, { kind: "ammo"});
```

> [查看如何在客户端处理这些消息。](https://docs.colyseus.io/client/room/#onmessage)

#### `leave(code?: number)`[¶](https://docs.colyseus.io/server/client/#leavecode-number) <a id="leavecode-number"></a>

强制断开`client`与房间的连接。

> 这将在客户端触发room.onLeave事件。

#### `error(code, message)`[¶](https://docs.colyseus.io/server/client/#errorcode-message) <a id="errorcode-message"></a>

将错误与代码和消息发送给客户端。客户可以继续处理[`onError`](https://docs.colyseus.io/client/room/#onerror)

