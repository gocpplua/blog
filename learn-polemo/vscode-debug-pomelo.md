---
description: The most detailed version online
---

# vscode debug pomelo

## vscode debug pomelo

### 1、安装pomelo

npm install pomelo -g

### ２、创建HelloWorld 项目：\([中文](https://github.com/NetEase/pomelo/wiki/pomelo%E7%9A%84HelloWorld) \|  [English](https://github.com/NetEase/pomelo/wiki/HelloWorld-of-Pomelo)\)

2.1 使用pomelo的命令行工具可以快速创建一个项目，命令如下：

```text
$ pomelo init ./HelloWorld
```

选择: 1 for websocket\(native socket\)

2.2 进入到HelloWorld文件夹，安装依赖包：$ sh npm-install.sh

> 备注:我在执行sh的时候，发现进不去web\_server文件夹去安装依赖，那么我们手动执行npm install -d安装就行

### 3、修改 servers.json

在game-server/config/servers.json,配置中添加 args 参数，此参数为node开启此服务器时命令参数

```text
   "args": " --inspect=127.0.0.1:16772"
```

文件如下:

```text
{
  "development":{
    "connector": [
    {"id": "connector-server-1", "host": "127.0.0.1", "port": 3150, "clientHost": "127.0.0.1", "clientPort": 3010, "frontend": true, "args": " --inspect=127.0.0.1:16772"}
    ]
  },
  "production":{
    "connector": [
    {"id": "connector-server-1", "host": "127.0.0.1", "port": 3150, "clientHost": "127.0.0.1", "clientPort": 3010, "frontend": true}
    ]
  }
}
```

### ４、配置launch.json

使用vscode打开项目，配置launch.json,并且配置:"Node.js:Attach"。

> 因为我是本地调试所以这么做，如果是远程的话就配置:"Node.js:Attach to Remote Program"
>
> ```text
> {
>   "version": "0.2.0",
>   "configurations": [
>     {
>       "name": "Attach",
>       "port": 16772,
>       "request": "attach",
>       "skipFiles": [
>         "<node_internals>/**"
>       ],
>       "type": "pwa-node"
>     }
>   ]
> }
> ```
>
> ### 5、启动game-server服务器：
>
> ```text
> $ cd game-server
> $ pomelo start
> ```

### ６、启动web-server服务器：

```text
$ cd web-server
$ node app
```

可以看到： Web server has started. Please log on [http://127.0.0.1:3001/index.html](http://127.0.0.1:3001/index.html)

７、在 HelloWorld/game-server/app/servers/connector/handler/entryHandler.js的entry中加上断点:

```text
Handler.prototype.entry = function(msg, session, next) {
  next(null, {code: 200, msg: 'game server is ok.'});  // --> 此处加上断点
};
```

### ８、Debug

vscode的Debug选项选择为:Attach后，按下F5。连接上后可以在VS中看到相关日志，也可以在game-server中看到日志:Debugger attached.

### ９、打开浏览器

浏览器中输入:[http://127.0.0.1:3001](http://127.0.0.1:3001) -&gt; 这里的端口和web-server启动的时候一致

### 10、连接服务器

在浏览器页面上，点击:Test Game Server 按钮，就可以看到vs code 中断点命中

`Good Job!!!`

> 还有一种方法是:在vscode中创建一个launch.json\(Launch Program\)，然后将program指定为app.js即可。

\`\`

