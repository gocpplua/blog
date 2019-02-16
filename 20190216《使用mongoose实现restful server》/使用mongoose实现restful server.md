通过上一篇文章[《使用mongoose实现本地http服务器》](http://www.gocpplua.com/index.php/archives/34.html)，我们可以通过下载文章中源码，进行尝试编译运行，就可以基本了解mongoose。
本篇文章主要使用mongoose实现现restful server：


1. 将客户端上传的两个数字相加，并且返回相加后的值
2. 将客户端上传的字符串进行打印

核心代码代码如下:
![](https://i.imgur.com/eiYqFcl.png)

上图中红色比较是我根据官方源码[链接](https://github.com/cesanta/mongoose/blob/master/examples/restful_server/restful_server.c)自己新增的。我的源码下载地址:[链接](https://github.com/gocpplua/minilibiary/releases/tag/v0.0.2)

大家下载后编译运行(我是使用VS2017)，如下图：
![](https://i.imgur.com/KFbJBqh.png)


接着我们使用postman来模拟客户端发送http消息：
![](https://i.imgur.com/rI5A2Xs.png)

我们接着查看服务端日志：
![](https://i.imgur.com/q8voB2T.png)

客户端postman收到结果：
![](https://i.imgur.com/Ftioely.png)

我们可以看到，首先是创建了一个http连接，然后收到http请求。这里有一个疑惑，在我印象中，http请求默认是短连接，那为什么服务端日志中没有看到断开http请求的打印:"MG_EV_CLOSE"。当我们关闭postman时，才出现"MG_EV_CLOSE"。经过一番追踪，终于发现问题了。
postman工具可以把请求参数生成对应的代码(划重点)，见下图:
![](https://i.imgur.com/swpqRWe.png)

我们可以看到，postman发送的http请求是:HTTP/1.1起。接着我们查看HTTP/1.0和HTTP/1.1。
> 在HTTP/1.0中默认使用短连接。也就是说，客户端和服务器每进行一次HTTP操作，就建立一次连接，任务结束就中断连接。当客户端浏览器访问的某个HTML或其他类型的Web页中包含有其他的Web资源（如JavaScript文件、图像文件、CSS文件等），每遇到这样一个Web资源，浏览器就会重新建立一个HTTP会话。
而从HTTP/1.1起，默认使用长连接，用以保持连接特性。

问题得到很好的解答，有兴趣的朋友可以通过关键字搜索下：http 短连接 长连接。
同理，我们可以通过postman验证接口:http://127.0.0.1:8000/printcontent,此处就不再进行阐述，如有问题请留言。

码字不易，如果解决了你的问题的话，欢迎打赏:
![](https://i.imgur.com/y1Kh5wx.png)

实现功能参考文章:
https://blog.csdn.net/hnzwx888/article/details/85012644
postman 和 fiddler 区别:https://blog.csdn.net/cmzhuang/article/details/80734731
如何使用postman：https://www.cnblogs.com/qiaoyeye/p/5521156.html
http中get和post差别:http://www.w3school.com.cn/tags/html_ref_httpmethods.asp
http post中 form-data,x-www-form-unlencoded,raw,binary等区别