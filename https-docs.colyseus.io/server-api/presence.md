---
description: Presence　状态
---

# Presence

当您需要在多个进程和/或机器上扩展服务器时，需要向Server提供[`Presence`](https://docs.colyseus.io/server/api/#optionspresence)选项。的目的`Presence`是允许在不同进程之间进行通信和共享数据，尤其是在进行匹配时。`Presence`目的是允许在不同进程之间进行通信和共享数据，尤其是在进行匹配时。

* [`LocalPresence`](https://docs.colyseus.io/server/presence/#localpresence) （默认）
* [`RedisPresence`](https://docs.colyseus.io/server/presence/#redispresence-clientopts)

`presence`实例在每个`Room`处理程序上也可用。您可以使用其[API](https://docs.colyseus.io/server/presence/#api)保留数据并通过PUB / SUB在房间之间进行通信。



#### `LocalPresence`[¶](https://docs.colyseus.io/server/presence/#localpresence) <a id="localpresence"></a>

这是默认选项。它用于在单个进程中运行Colyseus时使用。

#### `RedisPresence (clientOpts?)`[¶](https://docs.colyseus.io/server/presence/#redispresence-clientopts) <a id="redispresence-clientopts"></a>

在多个进程和/或计算机上运行Colyseus时，请使用此选项。

**参数：**

* `clientOpts`：redis客户端选项（主机/凭证）。[查看选项的完整列表](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/redis/index.d.ts#L28-L52)。

```text
import { Server, RedisPresence } from "colyseus";

// This happens on the slave processes.
const gameServer = new Server({
    // ...
    presence: new RedisPresence()
});

gameServer.listen(2567);
```

### API <a id="api"></a>

`Presence`API高度基于Redis的API，它是一个键值数据库。

每个[`Room`](https://docs.colyseus.io/server/room)实例都有一个[`presence`](https://docs.colyseus.io/server/room/#presence-presence)属性，该属性实现以下方法：

#### `subscribe(topic: string, callback: Function)` <a id="subscribetopic-string-callback-function"></a>

订阅给定的`topic`。只要在`topic上`[公布](https://docs.colyseus.io/server/presence/#publishtopic-string-data-any)　消息，`callback`将被触发。

#### `unsubscribe(topic: string)`[¶](https://docs.colyseus.io/server/presence/#unsubscribetopic-string) <a id="unsubscribetopic-string"></a>

取消订阅`topic`。

#### `publish(topic: string, data: any)`[¶](https://docs.colyseus.io/server/presence/#publishtopic-string-data-any) <a id="publishtopic-string-data-any"></a>

将消息发布到给定的消息`topic`。

#### `exists(key: string): Promise<boolean>`[¶](https://docs.colyseus.io/server/presence/#existskey-string-promiseboolean) <a id="existskey-string-promiseboolean"></a>

返回键是否存在。

#### `setex(key: string, value: string, seconds: number)`[¶](https://docs.colyseus.io/server/presence/#setexkey-string-value-string-seconds-number) <a id="setexkey-string-value-string-seconds-number"></a>

设置键以保留字符串值，并将键设置为在给定的秒数后超时。

#### `get(key: string)`[¶](https://docs.colyseus.io/server/presence/#getkey-string) <a id="getkey-string"></a>

获取密钥的值。

#### `del(key: string): void`[¶](https://docs.colyseus.io/server/presence/#delkey-string-void) <a id="delkey-string-void"></a>

删除指定的密钥。

#### `sadd(key: string, value: any)`[¶](https://docs.colyseus.io/server/presence/#saddkey-string-value-any) <a id="saddkey-string-value-any"></a>

将指定的成员添加到存储在键处的集合中。已是此集合成员的指定成员将被忽略。如果键不存在，则在添加指定成员之前会创建一个新集。

`smembers(key: string)`[¶](https://docs.colyseus.io/server/presence/#smemberskey-string)

返回键处存储的设置值的所有成员。

#### `sismember(member: string)`[¶](https://docs.colyseus.io/server/presence/#sismembermember-string) <a id="sismembermember-string"></a>

返回是否`member`是存储在key中的集合的成员

**返回值**

* `1` 如果元素是集合的成员。
* `0` 如果元素不是集合的成员，或者键不存在。

#### `srem(key: string, value: any)`[¶](https://docs.colyseus.io/server/presence/#sremkey-string-value-any) <a id="sremkey-string-value-any"></a>

#### 从key处存储的集合中删除指定的成员。不是该集合成员的指定成员将被忽略。如果key不存在，则将其视为空集，并且此命令返回0。 <a id="sremkey-string-value-any"></a>

#### `scard(key: string)`[¶](https://docs.colyseus.io/server/presence/#scardkey-string) <a id="scardkey-string"></a>

返回键处存储的集合的集合基数（元素数）。

#### `sinter(...keys: string[])`[¶](https://docs.colyseus.io/server/presence/#sinterkeys-string) <a id="sinterkeys-string"></a>

返回所有给定集合的交集所得的集合成员。

#### `hset(key: string, field: string, value: string)`[¶](https://docs.colyseus.io/server/presence/#hsetkey-string-field-string-value-string) <a id="hsetkey-string-field-string-value-string"></a>

将存储在键处的哈希中的字段设置为value。如果密钥不存在，则会创建一个包含哈希的新密钥。如果哈希中已经存在该字段，则将其覆盖。

#### `hincrby(key: string, field: string, value: number)`[¶](https://docs.colyseus.io/server/presence/#hincrbykey-string-field-string-value-number) <a id="hincrbykey-string-field-string-value-number"></a>

按增量递增存储在键处存储的哈希中字段中存储的数字。如果密钥不存在，则会创建一个包含哈希的新密钥。如果该字段不存在，则在执行操作之前将值设置为0。

#### `hget(key: string, field: string): Promise<string>`[¶](https://docs.colyseus.io/server/presence/#hgetkey-string-field-string-promisestring) <a id="hgetkey-string-field-string-promisestring"></a>

返回与存储在key处的哈希中的field关联的值。

#### `hgetall(key: string): Promise<{[field: string]: string}>`[¶](https://docs.colyseus.io/server/presence/#hgetallkey-string-promisefield-string-string) <a id="hgetallkey-string-promisefield-string-string"></a>

返回存储在键处的哈希的所有字段和值。

#### `hdel(key: string, field: string)`[¶](https://docs.colyseus.io/server/presence/#hdelkey-string-field-string) <a id="hdelkey-string-field-string"></a>

从存储在key处的哈希中删除指定的字段。该哈希中不存在的指定字段将被忽略。如果key不存在，则将其视为空哈希，并且此命令返回0。

#### `hlen(key: string): Promise<number>`[¶](https://docs.colyseus.io/server/presence/#hlenkey-string-promisenumber) <a id="hlenkey-string-promisenumber"></a>

返回键处存储的哈希中包含的字段数

#### `incr(key: string)`[¶](https://docs.colyseus.io/server/presence/#incrkey-string) <a id="incrkey-string"></a>

将键处存储的数字加1。如果密钥不存在，则在执行操作之前将其设置为0。如果键包含错误类型的值或包含不能表示为整数的字符串，则返回错误。此操作仅限于64位带符号整数。

#### `decr(key: string)`[¶](https://docs.colyseus.io/server/presence/#decrkey-string) <a id="decrkey-string"></a>

将键处存储的数字减一。如果密钥不存在，则在执行操作之前将其设置为0。如果键包含错误类型的值或包含不能表示为整数的字符串，则返回错误。此操作仅限于64位带符号整数。  


