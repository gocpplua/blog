---
description: Overview 概述
---

# Overview

## 状态处理[¶](https://docs.colyseus.io/state/overview/#state-handling) <a id="state-handling"></a>

在Colyseus中，房间处理程序是**有状态**的。每个房间都有自己的状态。状态的突变会自动同步到所有连接的客户端。

### 序列化方法[¶](https://docs.colyseus.io/state/overview/#serialization-methods) <a id="serialization-methods"></a>

* [Schema](https://docs.colyseus.io/state/schema/) （默认）
* [Fossil Delta](https://docs.colyseus.io/state/fossil-delta/) （已弃用）

### 状态同步时[¶](https://docs.colyseus.io/state/overview/#when-the-state-is-synchronized) <a id="when-the-state-is-synchronized"></a>

* 当用户成功加入会议室后，他将从服务器接收到完整状态。
* 在每个[patchRate](https://docs.colyseus.io/server/room/#patchrate-number)，状态的二进制补丁会发送到每个客户端（默认为`50ms`）
* 从服务器收到每个补丁后，都会在客户端调用[`onStateChange`](https://docs.colyseus.io/client/room/#onstatechange) 
* 每种序列化方法都有其自己的特殊方式来处理进入状态补丁。

