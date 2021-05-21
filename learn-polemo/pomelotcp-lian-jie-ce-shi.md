# Pomelo　tcp 连接测试

> 接着上一讲《Pomelo点击:Test Game Server 按钮后流程》。

pomelo提供了hybirdconnector和sioconnector，hybirdconnector支持TCP、WebSocket，sioconnector支持socket.io。

我们创建项目的时候，选择的是: 1 for websocket\(native socket\)，那么项目还是可以支持tcp连接的，我们这边就使用原项目测试。



测试tcp的话，我们只要启动game-server即可:

```text
$> polemo start
```

启动日志中我们主要关注clientPort:

> \[2021-05-20 20:30:02.478\] \[INFO\] pomelo - \[/data/gocpplua/pomelo/pomelo\_prj/HelloWorld/game-server/node\_modules/pomelo/lib/master/starter.js\] Executing /home/SENSETIME/chenqi1/.nvm/versions/node/v14.16.1/bin/node --inspect=127.0.0.1:16772,/data/gocpplua/pomelo/pomelo\_prj/HelloWorld/game-server/app.js,env=development,id=connector-server-1,host=127.0.0.1,port=3150,clientHost=127.0.0.1,`clientPort=3010`,frontend=true,args= --inspect=127.0.0.1:16772,serverType=connector locally

然后使用nc\(全名叫 netcat\)工具，充当客户端，去连接3010端口。

我当时去发送数据的数据，调试了很一会，因为不知道tcp对应的包的格式，以及如何在命令行发送１６进制数据，那这边我就直接剧透了吧。

* 一、协议

假设我发送的数据如下:

> {"sys":1}

那么对应16进制:

> \x7b\x22\x73\x79\x73\x22\x3a\x31\x7d

这边需要注意:

1. 需要是Json数据
2. 数据中需要"sys"字段
3. 发送的数据格式说明

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7B2C;&#xFF11;&#x5B57;&#x8282;</th>
      <th style="text-align:left">&#x7B2C;&#xFF12;&#xFF5E;&#xFF14;&#x5B57;&#x8282;</th>
      <th style="text-align:left">&#x7B2C;&#xFF15;&#x5B57;&#x8282;&#x4EE5;&#x53CA;&#x5176;&#x540E;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x534F;&#x8BAE;&#x7C7B;&#x578B;</td>
      <td style="text-align:left">&#x6570;&#x636E;&#x957F;&#x5EA6;</td>
      <td style="text-align:left">&#x6570;&#x636E;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>Package.TYPE_HANDSHAKE = 1;</p>
        <p>Package.TYPE_HANDSHAKE_ACK = 2;</p>
        <p>Package.TYPE_HEARTBEAT = 3;</p>
        <p>Package.TYPE_DATA = 4;</p>
        <p>Package.TYPE_KICK = 5;</p>
      </td>
      <td style="text-align:left">&#x7565;</td>
      <td style="text-align:left">&#x7565;</td>
    </tr>
  </tbody>
</table>

所以，我们最后组装出来的数据就是:

> "\x01\x00\x00\x09\x7b\x22\x73\x79\x73\x22\x3a\x31\x7d"

* 二、十六进制发送

我们可以将十六进制写到文件中发送，也可以在终端发送，我以终端为例:

```text
$> echo -n -e "\x01\x00\x00\x09\x7b\x22\x73\x79\x73\x22\x3a\x31\x7d" | nc 1
27.0.0.1 3010
```

服务器返回数据:

> {"code":200,"sys":{"heartbeat":3,"dict":{"connector.entryHandler.entry":1,"connector.entryHandler.publish":2,"connector.entryHandler.subscribe":3},"routeToCode":{"connector.entryHandler.entry":1,"connector.entryHandler.publish":2,"connector.entryHandler.subscribe":3},"codeToRoute":{"1":"connector.entryHandler.entry","2":"connector.entryHandler.publish","3":"connector.entryHandler.subscribe"},"dictVersion":"vhs1TJfFaUAZcUaS9t3zTQ==","useDict":true,"protos":{"server":{},"client":{},"version":"xT9OvpsqULwrUv2IpdUD4Q=="},"useProto":true}}

如果你打算自己Debug的话，可以在tcpsocket.js中的ondata函数中添加断点,函数如下:

```text
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

具体如何生成工程和调试，请参照前面两篇文章。

