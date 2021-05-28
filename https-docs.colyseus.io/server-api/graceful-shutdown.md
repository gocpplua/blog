---
description: 正常关机
---

# Graceful Shutdown

默认情况下，Colyseus提供了正常的关闭机制。这些操作将在进程终止自身之前执行：

* 异步断开所有已连接的客户端（`Room#onLeave`）
* 异步处理所有生成的房间（`Room#onDispose`）
* 在关闭进程之前执行可选的异步回调 `Server#onShutdown`

如果您要在`onLeave`/上执行异步任务`onDispose`，则应返回`Promise`，并在任务准备就绪时对其进行解析。同样适用于`onShutdown(callback)`。

## 返回一个 `Promise`[¶](https://docs.colyseus.io/server/graceful-shutdown/#returning-a-promise) <a id="returning-a-promise"></a>

通过返回 `Promise`，服务器将在杀死工作进程之前等待它们完成。

```text
import { Room } from "colyseus";

class MyRoom extends Room {
    onLeave (client) {
        return new Promise((resolve, reject) => {
            doDatabaseOperation((err, data) => {
                if (err) {
                    reject(err);
                } else {
                    resolve(data);
                }
            });
        });
    }

    onDispose () {
        return new Promise((resolve, reject) => {
            doDatabaseOperation((err, data) => {
                if (err) {
                    reject(err);
                } else {
                    resolve(data);
                }
            });
        });
    }
}
```

## 使用 `async`[¶](https://docs.colyseus.io/server/graceful-shutdown/#using-async) <a id="using-async"></a>

`async`关键字将使你的函数在后台返回一个`Promise`。[阅读有关Async / Await的更多信息](https://basarat.gitbooks.io/typescript/content/docs/async-await.html)。

```text
import { Room } from "colyseus";

class MyRoom extends Room {
    async onLeave (client) {
        await doDatabaseOperation(client);
    }

    async onDispose () {
        await removeRoomFromDatabase();
    }
}
```

## 进程关闭回调[¶](https://docs.colyseus.io/server/graceful-shutdown/#process-shutdown-callback) <a id="process-shutdown-callback"></a>

您还可以通过设置`onShutdown`回调来监听进程关闭。

```text
import { Server } from "colyseus";

let server = new Server();

server.onShutdown(function () {
    console.log("master process is being shut down!");
});
```

