# 使用mongoose实现本地http服务器 \#

注意：这里是使用mongoose实现HTTP服务，非数据库使用。 在[项目](https://github.com/gocpplua/minilibiary)中需要实现一个http service的功能。在同事的推荐下选择了mongoose的代码。

> Mongoose是一个用C编写的网络库。它为客户端和服务器模式实现TCP，UDP，HTTP，WebSocket，CoAP，MQTT的事件驱动的非阻塞API。

mongoose的代码着实轻量，先看看它的特点： 1. 在整个的实现是使用C语言编写 2. 整个代码也只有一个mongoose.c和mongoose.h两个文件， 从引入第三方的考虑上也着实不多。 3. 实现的功能还是非常多的，从使用的层面上来说功能还是比较全面。只不过不知道是否是为了第三方使用的方便还是怎么地，它的代码只用了两个源文件罢了。诸多的功能也大以宏的开始与结束来区分。 4. 示例非常齐全，所有的功能都有单独的示例

mongoose github路径如下：[链接](https://github.com/cesanta/mongoose)

接下来我们直接从代码入手： ![](https://i.imgur.com/SlawNgv.png)

源码下载地址：[链接](https://github.com/gocpplua/minilibiary/releases/tag/v0.0.1) 大家下载后编译运行，如下图： ![](https://i.imgur.com/oe6eYIK.png)

然后我们就可以在浏览器中输入对应的地址，这边特别需要注意的是端口号，我默认使用:8000! 

码字不易，如果解决了你的问题的话，欢迎打赏: ![](https://i.imgur.com/gALqni9.png)

