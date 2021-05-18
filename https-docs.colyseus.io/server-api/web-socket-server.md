# Server

## 服务器

[`Server`](https://docs.colyseus.io/server/api/#server)负责提供WebSocket的服务器，使服务器和客户机之间的通信。

### `constructor (options)`

#### **options.server**

将WebSocket服务器绑定到的HTTP服务器。您也可以将其[`express`](https://www.npmjs.com/package/express)用于服务器。

```text
// Colyseus (barebones)
import { Server } from "colyseus";
const port = process.env.port || 3000;

const gameServer = new Server();
gameServer.listen(port);
```

使用express

```text
// Colyseus + Express
import { Server } from "colyseus";
import { createServer } from "http";
import express from "express";
const port = Number(process.env.port) || 3000;

const app = express();
app.use(express.json());

const gameServer = new Server({
  server: createServer(app)
});

gameServer.listen(port);
```

JS版本见[原文](https://docs.colyseus.io/server/api/#server)。

#### **options.pingInterval**

服务器“ ping”客户端的毫秒数。默认：`3000`

如果客户端在[pingMaxRetries](https://docs.colyseus.io/server/api/#optionspingMaxRetries)重试后无法响应，将被强制断开连接。

#### **options.pingMaxRetries**

无响应的最大ping次数。默认值：`2`。

#### **options.verifyClient**

此方法在WebSocket握手之前发生。如果`verifyClient`未设置，则握手被自动接受。

* `info` \(Object\)
  * `origin` \(String\) 客户端指示的Origin标头中的值.
  * `req` \(http.IncomingMessage\) 客户端HTTP GET请求.
  * `secure` \(Boolean\) 如果设置了req.connection.authorized或req.connection.encrypted，则为true。.
* `next` \(Function\)用户在检查info字段时必须调用的回调。此回调中的参数为:
  * `result` \(Boolean\) 是否接受握手.
  * `code` \(Number\) 如果`result`是`false,`则此字段确定要发送到客户端的HTTP错误状态代码。.
  * `name` \(String\) 如果`result`是`false`这个字段确定HTTP原因短语.

```text
import { Server } from "colyseus";

const gameServer = new Server({
  // ...

  verifyClient: function (info, next) {
    // validate 'info'
    //
    // - next(false) will reject the websocket handshake
    // - next(true) will accept the websocket handshake
  }
});
```

#### **options.presence**

通过多个进程/机器扩展Colyseus时，您需要提供一个状态服务器。阅读有关[可伸缩性的](https://docs.colyseus.io/scalability/)更多信息，以及[`Presence API`](https://docs.colyseus.io/server/presence/#api)。

```text
import { Server, RedisPresence } from "colyseus";

const gameServer = new Server({
  // ...
  presence: new RedisPresence()
});
```

当前可用的状态服务器是

* `RedisPresence` （在单个服务器和多个服务器上缩放）

#### **options.gracefullyShutdown**

自动注册关闭例程。 默认为true。 如果禁用，则应在关机过程中手动调用gracefullyShutdown（）方法。

### `define (name: string, handler: Room, options?: any)`

定义一个新的房间处理程序。

**参数:**

* `name: string` - 房间的公共名称。从客户端加入会议室时，将使用此名称。.
* `handler: Room` - 引用`Room`处理程序类.
* `options?: any` - 房间初始化的自定义选项.

```text
// Define "chat" room
gameServer.define("chat", ChatRoom);

// Define "battle" room
gameServer.define("battle", BattleRoom);

// Define "battle" room with custom options
gameServer.define("battle_woods", BattleRoom, { map: "woods" });
```

> #### 多次定义同一个房间处理程序
>
> 您可以使用不同的多次定义同一个房间处理程序`options`当[Room＃的onCreate（）](https://docs.colyseus.io/server/room/#oncreate-options)被调用时，`options`将包含合并你指定的值[Server＃defiine\(\)](https://docs.colyseus.io/server/api/#define-name-string-handler-room-options-any)+在创建房间时提供的选项。

### **Matchmaking filters: filterBy\(options\)**

**参数**

* `options: string[]` - 选项名称列表

每当通过create（）或joinOrCreate（）方法创建房间时，仅将由filterBy（）方法定义的选项存储在内部，并用于在进一步的join（）或joinOrCreate（）调用中过滤掉房间。

**示例：**允许使用不同的“游戏模式”。

```text
gameServer
  .define("battle", BattleRoom)
  .filterBy(['mode']);
```

无论何时创建房间，该`mode`选项都将在内部存储。

```text
client.joinOrCreate("battle", { mode: "duo" }).then(room => {/* ... */});
```

您可以在onCreate（）and/or onJoin（）中处理提供的选项，以在房间实现中实现请求的功能。

```text
class BattleRoom extends Room {
  onCreate(options) {
    if (options.mode === "duo") {
      // do something!
    }
  }
  onJoin(client, options) {
    if (options.mode === "duo") {
      // put this player into a team!
    }
  }
}
```

**示例：**通过内置过滤`maxClients`

maxClients是存储用于配对的内部变量，也可以用于过滤。

```text
gameServer
  .define("battle", BattleRoom)
  .filterBy(['maxClients']);
```

然后客户可以要求加入一个能够容纳一定数量玩家的房间

```text
client.joinOrCreate("battle", { maxClients: 10 }).then(room => {/* ... */});
client.joinOrCreate("battle", { maxClients: 20 }).then(room => {/* ... */});
```

**Matchmaking priority: sortBy\(options\)**

您还可以根据创建时加入房间的信息为加入房间赋予不同的优先级。

options参数是一个键值对象，在左侧包含字段名称，在右侧包含排序方向。 排序方向可以是以下值之一：-1，“ desc”，“ descending”，1，“ asc”或“ ascending”。

**示例：**按内置排序`clients`

客户端是为配对而存储的内部变量，其中包含当前已连接客户端的数量。 在以下示例中，连接最多客户端的房间将具有优先权。 使用-1，“ desc”或“ descending”降序：

```text
gameServer
  .define("battle", BattleRoom)
  .sortBy({ clients: -1 });
```

要按最少的玩家人数进行排序，您可以做相反的事情。 对于升序，请使用1，“升序”或“升序”：

```text
gameServer
  .define("battle", BattleRoom)
  .sortBy({ clients: 1 });
```

### 启用大厅的实时房间列表

要允许`LobbyRoom`接收来自特定房间类型的更新，您应该在启用实时列表的情况下对其进行定义：

```text
gameServer
  .define("battle", BattleRoom)
  .enableRealtimeListing();
```

[进一步了解 `LobbyRoom`](https://docs.colyseus.io/builtin-rooms/lobby/)

### 监听房间实例事件

`define`方法将返回已注册的处理程序实例，您可以从房间实例范围之外监听匹配事件。如：

* `"create"` -创建房间后
* `"dispose"` -处置房间后
* `"join"` -当客户加入房间时
* `"leave"` -客户离开房间时
* `"lock"` -锁定房间时
* `"unlock"` -房间解锁时

**用法：**

```text
gameServer
  .define("chat", ChatRoom)
  .on("create", (room) => console.log("room created:", room.roomId))
  .on("dispose", (room) => console.log("room disposed:", room.roomId))
  .on("join", (room, client) => console.log(client.id, "joined", room.roomId))
  .on("leave", (room, client) => console.log(client.id, "left", room.roomId));
```

> 不鼓励通过这些事件来操纵房间的状态。而是在您的房间处理程序中使用[抽象方法](https://docs.colyseus.io/server/room/#abstract-methods)。

#### `simulateLatency (milliseconds: number)`

这是一种便捷的方法，适用于您希望本地测试“落后”客户端的行为而不必将服务器部署到远程云的情况。

```text
// Make sure to never call the `simulateLatency()` method in production.
if (process.env.NODE_ENV !== "production") {

  // simulate 200ms latency between server and client.
  // 模拟服务器和客户端之间的200毫秒延迟。
  gameServer.simulateLatency(200);
}
```

#### `attach (options: any)`

> 您通常不需要调用此函数。仅当您有非常特定的理由时才使用它。

附加或创建WebSocket服务器。

* `options.server`：用于附加WebSocket服务器的HTTP服务器。
* `options.ws`：要重用的现有WebSocket服务器。

> ```text
> // http.createserver
> import http from "http";
> import { Server } from "colyseus";
>
> const httpServer = http.createServer();
> const gameServer = new Server();
>
> gameServer.attach({ server: httpServer });
> ```
>
> ```text
> // websocket.server
> import http from "http";
> import express from "express";
> import ws from "ws";
> import { Server } from "colyseus";
>
> const app = express();
> const server = http.createServer(app);
> const wss = new WebSocket.Server({
>     // your custom WebSocket.Server setup.
> });
>
> const gameServer = new Server();
> gameServer.attach({ ws: wss });
> ```

#### `listen (port: number)` <a id="listen-port-number"></a>

将WebSocket服务器绑定到指定的端口。

#### `onShutdown (callback: Function)`

注册一个应在进程关闭之前调用的回调。有关更多详细信息，请参见[正常关闭](https://docs.colyseus.io/server/graceful-shutdown/)。

#### `gracefullyShutdown (exit: boolean)` <a id="gracefullyshutdown-exit-boolean"></a>

关闭所有房间并清理其缓存的数据。 返回在清理完成后将兑现的承诺。

除非在Server构造函数中提供了gracefulShutdown：false，否则将自动调用此方法。

