# Room

考虑到您已经[设置了服务器](https://docs.colyseus.io/server/api)，现在是时候注册Room处理程序并开始接受用户的连接了。

您将定义房间处理程序，以创建从`Room`扩展的类。

```text
import http from "http";
import { Room, Client } from "colyseus";

export class MyRoom extends Room {
    // When room is initialized
    onCreate (options: any) { }

    // Authorize client based on provided options before WebSocket handshake is complete
    onAuth (client: Client, options: any, request: http.IncomingMessage) { }

    // When client successfully join the room
    onJoin (client: Client, options: any, auth: any) { }

    // When a client leaves the room
    onLeave (client: Client, consented: boolean) { }

    // Cleanup callback, called after there are no more clients in the room. (see `autoDispose`)
    onDispose () { }
}
```

## Room lifecycle

这些方法对应于房间的生命周期。

### `onCreate (options)` <a id="oncreate-options"></a>

房间初始化后被调用一次。您可以在注册房间处理程序时指定自定义初始化选项。

> Tip
>
> 这些选项将包含您在[Server＃define（）](https://docs.colyseus.io/server/api/#define-name-string-handler-room-options-any)上指定的合并值+ [client.joinOrCreate（](https://docs.colyseus.io/client/client/#joinorcreate-roomname-string-options-any)）或[client.create（）](https://docs.colyseus.io/client/client/#create-roomname-string-options-any)提供的选项

### `onAuth (client, options, request)` <a id="onauth-client-options-request"></a>

onAuth（）方法将在onJoin（）之前执行。 它可以用来验证加入房间的客户的真实性。

* 如果`onAuth()`返回true，`onJoin()`则将使用返回值作为第三个参数来调用。
* 如果`onAuth()`返回false，则客户端将立即被拒绝，从而导致客户端的匹配函数调用失败。
* 您也可以抛出`ServerError`来公开要在客户端处理的自定义错误。

如果不执行，它将始终返回`true`-允许任何客户端进行连接。

> 获取玩家IP地址
>
> 您可以使用该`request`变量来检索用户的IP地址，http标头等。例如：`request.headers['x-forwarded-for'] || request.connection.remoteAddress`

**实现示例**

* ASYNC / AWAIT

```text
//ASYNC / AWAIT
import { Room, ServerError } from "colyseus";

class MyRoom extends Room {
  async onAuth (client, options, request) {
    /**
     * Alternatively, you can use `async` / `await`,
     * which will return a `Promise` under the hood.
     */
    const userData = await validateToken(options.accessToken);
    if (userData) {
        return userData;

    } else {
        throw new ServerError(400, "bad access token");
    }
  }
}
```

* SYNCHRONOUS

```text
import { Room } from "colyseus";

class MyRoom extends Room {
  onAuth (client, options, request): boolean {
    /**
     * You can immediatelly return a `boolean` value.
     */
     if (options.password === "secret") {
       return true;

     } else {
       throw new ServerError(400, "bad access token");
     }
  }
}
```

* PROMISES

```text
import { Room } from "colyseus";

class MyRoom extends Room {
  onAuth (client, options, request): Promise<any> {
    /**
     * You can return a `Promise`, and perform some asynchronous task to validate the client.
     */
    return new Promise((resolve, reject) => {
      validateToken(options.accessToken, (err, userData) => {
        if (!err) {
          resolve(userData);
        } else {
          reject(new ServerError(400, "bad access token"));
        }
      });
    });
  }
}
```

**客户端示例**

在客户端，您可以使用您选择的某些身份验证服务（例如Facebook）中的令牌调用匹配方法（join，joinOrCreate等）：

```text
client.joinOrCreate("world", {
  accessToken: yourFacebookAccessToken

}).then((room) => {
  // success

}).catch((err) => {
  // handle error...
  err.code // 400
  err.message // "bad access token"
});
```

### `onJoin (client, options, auth?)` <a id="onjoin-client-options-auth"></a>

**参数：**

* `client`：[`client`](https://docs.colyseus.io/server/client)实例。
* `options`：将[Server＃define（）](https://docs.colyseus.io/server/api/#define-name-string-handler-room-options-any)上指定的值与[`client.join()`](https://docs.colyseus.io/client/client/#join-roomname-string-options-any)上客户端提供的选项合并
* `auth`：（可选）[`onAuth`](https://docs.colyseus.io/server/room/#onauth-client-options-request)方法返回的身份验证数据。

当客户端成功加入会议室之后`requestJoin`，`onAuth`成功之后被调用。

### `onLeave (client, consented)` <a id="onleave-client-consented"></a>

当客户离开房间时被调用。如果断开连接是[由客户端发起的](https://docs.colyseus.io/client/room/#leave)，则`consented`参数将为`true`，否则为`false`。

您可以将此函数定义为`async`。查看[正常关机](https://docs.colyseus.io/server/graceful-shutdown)

* SYNCHRONOUS

```text
onLeave(client, consented) {
    if (this.state.players.has(client.sessionId)) {
        this.state.players.delete(client.sessionId);
    }
}
```

* ASYNCHRONOUS

```text
async onLeave(client, consented) {
    const player = this.state.players.get(client.sessionId);
    await persistUserOnDatabase(player);
}
```

### `onDispose ()` <a id="ondispose"></a>

在销毁房间之前调用onDispose（）方法，这种情况发生在`:`

* 房间中没有其他客户，并且`autoDispose`已将其设置为`true`（默认）
* 您手动调用[`.disconnect()`](https://docs.colyseus.io/server/room/#disconnect)。

您可以将async onDispose（）定义为异步方法，以便将某些数据持久化在数据库中。 实际上，这是在比赛结束后将玩家数据保留在数据库中的好地方。

请参见[正常关机](https://docs.colyseus.io/server/graceful-shutdown)。

### Example room <a id="example-room"></a>

这个例子演示了整个房间实现`onCreate`，`onJoin`和`onMessage`方法。

* typescript

```text
import { Room, Client } from "colyseus";
import { Schema, MapSchema, type } from "@colyseus/schema";

// An abstract player object, demonstrating a potential 2D world position
export class Player extends Schema {
  @type("number")
  x: number = 0.11;

  @type("number")
  y: number = 2.22;
}

// Our custom game state, an ArraySchema of type Player only at the moment
export class State extends Schema {
  @type({ map: Player })
  players = new MapSchema<Player>();
}

export class GameRoom extends Room<State> {
  // Colyseus will invoke when creating the room instance
  onCreate(options: any) {
    // initialize empty room state
    this.setState(new State());

    // Called every time this room receives a "move" message
    this.onMessage("move", (client, data) => {
      const player = this.state.players.get(client.sessionId);
      player.x += data.x;
      player.y += data.y;
      console.log(client.sessionId + " at, x: " + player.x, "y: " + player.y);
    });
  }

  // Called every time a client joins
  onJoin(client: Client, options: any) {
    this.state.players.set(client.sessionId, new Player());
  }
}
```

* javascript

```text
const colyseus = require('colyseus');
const schema = require('@colyseus/schema');

// An abstract player object, demonstrating a potential 2D world position
exports.Player = class Player extends schema.Schema {
    constructor() {
        super();
        this.x = 0.11;
        this.y = 2.22;
    }
}
schema.defineTypes(Player, {
    x: "number",
    y: "number",
});

// Our custom game state, an ArraySchema of type Player only at the moment
exports.State = class State extends schema.Schema {
    constructor() {
        super();
        this.players = new schema.MapSchema();
    }
}
defineTypes(State, {
    players: { map: Player }
});

exports.GameRoom = class GameRoom extends colyseus.Room {
  // Colyseus will invoke when creating the room instance
  onCreate(options) {
    // initialize empty room state
    this.setState(new State());

    // Called every time this room receives a "move" message
    this.onMessage("move", (client, data) => {
      const player = this.state.players.get(client.sessionId);
      player.x += data.x;
      player.y += data.y;
      console.log(client.sessionId + " at, x: " + player.x, "y: " + player.y);
    });
  }

  // Called every time a client joins
  onJoin(client, options) {
    this.state.players.set(client.sessionId, new Player());
  }
}
```

## 公开方法

### `onMessage (type, callback)` <a id="onmessage-type-callback"></a>

注册一个回调以处理客户端发送的消息类型。所述`type`参数可以是`string`或`number`。

**回调特定类型的消息**

```text
onCreate () {
    this.onMessage("action", (client, message) => {
        console.log(client.sessionId, "sent 'action' message: ", message);
    });
}
```

**回调所有消息**

您可以注册一个回调来处理所有其他类型的消息。

```text
onCreate () {
    this.onMessage("action", (client, message) => {
        //
        // Triggers when 'action' message is sent.
        //
    });

    this.onMessage("*", (client, type, message) => {
        //
        // Triggers when any other type of message is sent,
        // excluding "action", which has its own specific handler defined above.
        //
        console.log(client.sessionId, "sent", type, message);
    });
}
```

### `setState (object)` <a id="setstate-object"></a>

设置新的房间状态实例。有关状态对象的更多详细信息，请参见[状态处理](https://docs.colyseus.io/state/overview/)。强烈建议使用新的[架构序列化器](https://docs.colyseus.io/state/schema/)来处理您的状态。

> Warning
>
> 不要在房间状态下调用此方法进行更新。 每次调用它时，都会重置二进制补丁算法。
>
> Tip
>
> 通常，您在房间处理程序中的onCreate（）过程中只会调用一次此方法。

### `setSimulationInterval (callback[, milliseconds=16.6])` <a id="setsimulationinterval-callback-milliseconds166"></a>

（可选）设置可以更改游戏状态的模拟间隔。模拟间隔是您的游戏循环。默认模拟间隔：16.6毫秒（60fps）

```text
onCreate () {
    this.setSimulationInterval((deltaTime) => this.update(deltaTime));
}

update (deltaTime) {
    // implement your physics or world updates here!
    // this is a good place to update the room state
}
```

### `setPatchRate (milliseconds)` <a id="setpatchrate-milliseconds"></a>

设置频率，应将修补状态发送给所有客户端。默认值为`50ms`（20fps）

### `setPrivate (bool)` <a id="setprivate-bool"></a>

将房间列表设置为私人房间（如果提供了false，则恢复为公共房间）。

私人房间未在getAvailableRooms（）方法中列出。

### `setMetadata (metadata)` <a id="setmetadata-metadata"></a>

将元数据设置为此房间。每个房间实例可能都附有元数据-附加元数据的唯一目的是从客户端获取可用房间列表时，将一个房间与另一个房间区分开`roomId`，并使用[`client.getAvailableRooms()`](https://docs.colyseus.io/client/client/#getavailablerooms-roomname)进行连接。

```text
// server-side
this.setMetadata({ friendlyFire: true });
```

现在，一个房间已附加了元数据，例如，客户端可以检查哪个房间具有friendlyFire，并通过roomId直接连接到该房间。

```text
// client-side
client.getAvailableRooms("battle").then(rooms => {
  for (var i=0; i<rooms.length; i++) {
    if (room.metadata && room.metadata.friendlyFire) {
      //
      // join the room with `friendlyFire` by id:
      //
      var room = client.join(room.roomId);
      return;
    }
  }
});
```

### `setSeatReservationTime (seconds)` <a id="setseatreservationtime-seconds"></a>

设置`Room`可以等待客户端有效加入Room的秒数。您应该考虑[`onAuth()`](https://docs.colyseus.io/server/room/#onauth-client-options-request)需要等待多长时间才能设置不同的座位预定时间。默认值为15秒。

`COLYSEUS_SEAT_RESERVATION_TIME`如果您想全局更改座位预订时间，则可以设置环境变量。

### `send (client, message)` <a id="send-client-message"></a>

> `this.send()`已不推荐使用。请[`client.send()`改用](https://docs.colyseus.io/server/client/#sendtype-message)。

### `broadcast (type, message, options?)` <a id="broadcast-type-message-options"></a>

向所有连接的客户端发送消息。

可用的选项有：

* **`except`**：一个[`Client`](https://docs.colyseus.io/server/client/)实例，不向其发送消息
* **`afterNextPatch`**：等到下一个补丁广播消息

**广播示例**

向所有客户广播消息：

```text
onCreate() {
    this.onMessage("action", (client, message) => {
        // broadcast a message to all clients
        this.broadcast("action-taken", "an action has been taken!");
    });
}
```

向发件人以外的所有客户端广播消息。

```text
onCreate() {
    this.onMessage("fire", (client, message) => {
        // sends "fire" event to every client, except the one who triggered it.
        this.broadcast("fire", message, { except: client });
    });
}
```

仅在应用状态更改后，才向所有客户端广播消息：

```text
onCreate() {
    this.onMessage("destroy", (client, message) => {
        // perform changes in your state!
        this.state.destroySomething();

        // this message will arrive only after new state has been applied
        this.broadcast("destroy", "something has been destroyed", { afterNextPatch: true });
    });
}
```

广播模式编码的消息：

```text
class MyMessage extends Schema {
  @type("string") message: string;
}

// ...
onCreate() {
    this.onMessage("action", (client, message) => {
        const data = new MyMessage();
        data.message = "an action has been taken!";
        this.broadcast(data);
    });
}
```

> Tip
>
> [查看如何在客户端处理这些onMessage（）。](https://docs.colyseus.io/client/room/#onmessage)

### `lock ()`[¶](https://docs.colyseus.io/server/room/#lock) <a id="lock"></a>

锁定房间会将其从可用房间池中删除，以供新客户端连接。

### `unlock ()`[¶](https://docs.colyseus.io/server/room/#unlock) <a id="unlock"></a>

解锁房间将其返回到可用房间池中，以供新客户端连接。

### `allowReconnection (client, seconds?)`[¶](https://docs.colyseus.io/server/room/#allowreconnection-client-seconds) <a id="allowreconnection-client-seconds"></a>

允许指定的客户[`reconnect`](https://docs.colyseus.io/client/client/#reconnect-roomid-string-sessionid-string)进入房间。必须在内部[`onLeave()`](https://docs.colyseus.io/server/room/#onleave-client)方法中使用。

如果**`seconds`**提供，则将在提供的秒数后取消重新连接。

```text
async onLeave (client: Client, consented: boolean) {
  // flag client as inactive for other users
  this.state.players[client.sessionId].connected = false;

  try {
    if (consented) {
        throw new Error("consented leave");
    }

    // allow disconnected client to reconnect into this room until 20 seconds
    await this.allowReconnection(client, 20);

    // client returned! let's re-activate it.
    this.state.players[client.sessionId].connected = true;

  } catch (e) {

    // 20 seconds expired. let's remove the client.
    delete this.state.players[client.sessionId];
  }
}
```

或者，您可能不提供自动拒绝重新连接并使用自己的逻辑自己拒绝重新连接的秒数。

```text
async onLeave (client: Client, consented: boolean) {
  // flag client as inactive for other users
  this.state.players[client.sessionId].connected = false;

  try {
    if (consented) {
        throw new Error("consented leave");
    }

    // get reconnection token
    const reconnection = this.allowReconnection(client);

    //
    // here is the custom logic for rejecting the reconnection.
    // for demonstration purposes of the API, an interval is created
    // rejecting the reconnection if the player has missed 2 rounds,
    // (assuming he's playing a turn-based game)
    //
    // in a real scenario, you would store the `reconnection` in
    // your Player instance, for example, and perform this check during your
    // game loop logic
    //
    const currentRound = this.state.currentRound;
    const interval = setInterval(() => {
      if ((this.state.currentRound - currentRound) > 2) {
        // manually reject the client reconnection
        reconnection.reject();
        clearInterval(interval);
      }
    }, 1000);

    // allow disconnected client to reconnect
    await reconnection;

    // client returned! let's re-activate it.
    this.state.players[client.sessionId].connected = true;

  } catch (e) {

    // 20 seconds expired. let's remove the client.
    delete this.state.players[client.sessionId];
  }
}
```

### `disconnect ()`[¶](https://docs.colyseus.io/server/room/#disconnect) <a id="disconnect"></a>

断开所有客户端的连接，然后处理。

### `broadcastPatch ()` <a id="broadcastpatch"></a>

> 您可能不需要这个！
>
> 该方法由框架自动调用。

此方法将检查状态中是否发生了突变，并将其广播给所有连接的客户端。

如果您想控制何时广播补丁，可以通过禁用默认补丁间隔来实现：

```text
onCreate() {
    // disable automatic patches
    this.setPatchRate(null);

    // ensure clock timers are enabled
    this.setSimulationInterval(() => {/* */});

    this.clock.setInterval(() => {
        // only broadcast patches if your custom conditions are met.
        if (yourCondition) {
            this.broadcastPatch();
        }
    }, 2000);
}
```

## 公共属性

### `roomId: string`[¶](https://docs.colyseus.io/server/room/#roomid-string) <a id="roomid-string"></a>

房间的唯一的，自动生成的9个字符长的ID。

您可以`this.roomId`在期间更换`onCreate()`。您需要确保`roomId`唯一。

### `roomName: string`[¶](https://docs.colyseus.io/server/room/#roomname-string) <a id="roomname-string"></a>

您作为的第一个参数提供的房间名称[`gameServer.define()`](https://docs.colyseus.io/server/api/#define-name-string-handler-room-options-any)。

### `state: T`[¶](https://docs.colyseus.io/server/room/#state-t) <a id="state-t"></a>

您提供给的状态实例[`setState()`](https://docs.colyseus.io/server/room/#setstate-object)。

### `clients: Client[]`[¶](https://docs.colyseus.io/server/room/#clients-client) <a id="clients-client"></a>

连接的客户端阵列。请参阅[Web-Socket Client](https://docs.colyseus.io/server/client)。

### `maxClients: number`[¶](https://docs.colyseus.io/server/room/#maxclients-number) <a id="maxclients-number"></a>

允许连接到房间的最大客户端数。当房间达到此限制时，它将自动锁定。除非您通过[lock（）](https://docs.colyseus.io/server/room/#lock)方法明确锁定了房间，否则一旦客户端断开与房间的连接，房间就会被解锁。

### `patchRate: number`[¶](https://docs.colyseus.io/server/room/#patchrate-number) <a id="patchrate-number"></a>

将房间状态发送到连接的客户端的频率（以毫秒为单位）。默认值为`50`ms（20fps）

### `autoDispose: boolean`[¶](https://docs.colyseus.io/server/room/#autodispose-boolean) <a id="autodispose-boolean"></a>

当最后一个客户端断开连接时，自动处理房间。默认是`true`

### `locked: boolean` （只读）[¶](https://docs.colyseus.io/server/room/#locked-boolean-read-only) <a id="locked-boolean-read-only"></a>

该属性将在以下情况下更改：

* 已达到允许的最大客户数量（`maxClients`）
* 您使用[`lock()`](https://docs.colyseus.io/server/room/#lock)或手动锁定或解锁了房间[`unlock()`](https://docs.colyseus.io/server/room/#unlock)。

### `clock: ClockTimer`[¶](https://docs.colyseus.io/server/room/#clock-clocktimer) <a id="clock-clocktimer"></a>

一个[`ClockTimer`](https://github.com/gamestdio/timer#api)实例，用于 [计时事件](https://docs.colyseus.io/server/timing-events)。

### `presence: Presence`[¶](https://docs.colyseus.io/server/room/#presence-presence) <a id="presence-presence"></a>

该`presence`实例。有关更多详细信息，请检查[Presence API](https://docs.colyseus.io/server/presence)。

