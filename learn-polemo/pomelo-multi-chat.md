# 【pomelo】multi chat和broadcast

multi chat可以参考官方[\[扩充服务器\]](https://github.com/NetEase/pomelo/wiki/%E6%89%A9%E5%85%85%E6%9C%8D%E5%8A%A1%E5%99%A8)教程，感觉还是可以看懂的。下面就讲解下广播。

关于如何选择connector和chat直接省略。我们直接从进入Join开始。客户端点击Join以后，服务器会进入到master服务器下述函数:

```text
// chatofpomelo-websocket/game-server/app/servers/connector/handler/entryHandler.js

/**
 * New client entry chat server.
 *
 * @param  {Object}   msg     request message
 * @param  {Object}   session current session object
 * @param  {Function} next    next stemp callback
 * @return {Void}
 */
handler.enter = function(msg, session, next) {
	...
	//put user into channel
	self.app.rpc.chat.chatRemote.add(session, uid, self.app.get('serverId'), rid, true, function(users){
		next(null, {
			users:users
		});
	});
};
```

然后通过rpc调用\(想要了解rpc的，自行处理\)。

在下述函数选择对应的chat服务器:

```text
// pomelo/lib/components/proxy.js
var defaultRoute = function(session, msg, app, cb) {
  var list = app.getServersByType(msg.serverType);
  if (!list || !list.length) {
    cb(new Error('can not find server info for type:' + msg.serverType));
    return;
  }

  var uid = session ? (session.uid || '') : '';
  var index = Math.abs(crc.crc32(uid.toString())) % list.length;
  utils.invokeCallback(cb, null, list[index].id);
};
```

进入到chat服务器：

```text
// chatofpomelo-websocket/game-server/app/servers/chat/remote/chatRemote.js
/**
 * Add user into chat channel.
 *
 * @param {String} uid unique id for user
 * @param {String} sid server id
 * @param {String} name channel name
 * @param {boolean} flag channel parameter
 *
 */
ChatRemote.prototype.add = function(uid, sid, name, flag, cb) {　// flag = true
	var channel = this.channelService.getChannel(name, flag);
	var username = uid.split('*')[0];
	var param = {
		route: 'onAdd',
		user: username
	};
	channel.pushMessage(param);

	if( !! channel) {
		channel.add(uid, sid);  // 将玩家加channel
		cb(this.get(name, flag));
};
```

我们首先会尝试获取channel：

```text
// omelo/lib/common/service/channelService.js
/**
 * Get channel by name.
 *
 * @param {String} name channel's name
 * @param {Boolean} create if true, create channel
 * @return {Channel}
 * @memberOf ChannelService
 */
ChannelService.prototype.getChannel = function(name, create) {// create = true
  var channel = this.channels[name];
  if(!channel && !!create) {
    channel = this.channels[name] = new Channel(name, this);
    addToStore(this, genKey(this), genKey(this, name));
  }
  return channel;
};
```

到获取就返回channel，获取不到就创建一个。

```text
// pomelo/lib/common/service/channelService.js
var addToStore = function(self, key, value) {
  if(!!self.store) {
    self.store.add(key, value, function(err) {
      if(!!err) {
        logger.error('add key: %s value: %s to store, with err: %j', key, value, err.stack);
      }
    });
  }
};
```

回到`ChatRemote.prototype.add`,获取成功后，将消息发送给channel中所有玩家,调用`Channel.prototype.pushMessage`。

```text
/**
 * Push message to all the members in the channel
 *
 * @param {String} route message route
 * @param {Object} msg message that would be sent to client
 * @param {Object} opts user-defined push options, optional
 * @param {Function} cb callback function
 */
Channel.prototype.pushMessage = function(route, msg, opts, cb) {
  if(this.state !== ST_INITED) {
    utils.invokeCallback(new Error('channel is not running now'));
    return;
  }

  if(typeof route !== 'string') {
    cb = opts;
    opts = msg;
    msg = route;
    route = msg.route;
  }

  if(!cb && typeof opts === 'function') {
    cb = opts;
    opts = {};
  }

  sendMessageByGroup(this.__channelService__, route, msg, this.groups, opts, cb);
};
```

sendMessageByGroup实现如下:

```text
/**
 * push message by group
 *
 * @param route {String} route route message
 * @param msg {Object} message that would be sent to client
 * @param groups {Object} grouped uids, , key: sid, value: [uid]
 * @param opts {Object} push options
 * @param cb {Function} cb(err)
 *
 * @api private
 */
var sendMessageByGroup = function(channelService, route, msg, groups, opts, cb) {
  ...
    var rpcCB = function(serverId) {
    return function(err, fails) {
      if(err) {
        logger.error('[pushMessage] fail to dispatch msg to serverId: ' + serverId + ', err:' + err.stack);
        latch.done();
        return;
      }
      if(fails) {
        failIds = failIds.concat(fails);
      }
      successFlag = true;
      latch.done();
    };
  };
  ...
  var sendMessage = function(sid) {
    return (function() {
      if(sid === app.serverId) {
        channelService.channelRemote[method](route, msg, groups[sid], opts, rpcCB(sid));
      } else {
        app.rpcInvoke(sid, {namespace: namespace, service: service,
          method: method, args: [route, msg, groups[sid], opts]}, rpcCB(sid));
      }
    })();
  };

  var group;
  for(var sid in groups) {
    group = groups[sid];  // 查找对应的group，然后发送消息
    if(group && group.length > 0) {
      sendMessage(sid);
    } else {
      // empty group
      process.nextTick(rpcCB(sid));
    }
  }
};
```

注意上面代码的rpcCB函数，在发送消息以后，就会进入rpcCB。

关于rpc内部如何进行我们就不管了，最后回到next调用:

```text
// chatofpomelo-websocket/game-server/app/servers/connector/handler/entryHandler.js
handler.enter = function(msg, session, next) {
	...

	//put user into channel
	self.app.rpc.chat.chatRemote.add(session, uid, self.app.get('serverId'), rid, true, function(users){
		next(null, {
			users:users
		});
	});
};
```

然后respond给客户端:

```text
// pomelo/lib/server/server.js
var doHandle = function(server, msg, session, routeRecord, cb) {
  var originMsg = msg;
  msg = msg.body || {};
  msg.__route__ = originMsg.route;

  var self = server;

  var handle = function(err, resp, opts) {
    if(err) {
      // error from before filter
      handleError(false, self, err, msg, session, resp, opts, function(err, resp, opts) {
        response(false, self, err, msg, session, resp, opts, cb);
      });
      return;
    }

    self.handlerService.handle(routeRecord, msg, session, function(err, resp, opts) {
      if(err) {
        //error from handler
        handleError(false, self, err, msg, session, resp, opts, function(err, resp, opts) {
          response(false, self, err, msg, session, resp, opts, cb);
        });
        return;
      }

      response(false, self, err, msg, session, resp, opts, cb);
    });
  };  //end of handle

  beforeFilter(false, server, msg, session, handle);
};
```

[https://itbilu.com/nodejs/npm/EknY6k0FX.html\#source](https://itbilu.com/nodejs/npm/EknY6k0FX.html#source)

[https://www.cnblogs.com/zhongweiv/p/nodejs\_pomelo.html](https://www.cnblogs.com/zhongweiv/p/nodejs_pomelo.html)

[https://blog.csdn.net/fjslovejhl/article/details/11703651](https://blog.csdn.net/fjslovejhl/article/details/11703651)



