#Client API([原文链接:http://socket.io/docs/client-api/](http://socket.io/docs/client-api/))
##IO(url:String, opts:Object):Socket
使用独立构建（例如:`/socket.io/socket.io.js`或CDN)或者
调用`require('socket.io-client')返回的结果在`window`
全局暴露`io`。

当被调用时，会为给定的URL创建一个新的`Manager`，并且在以后调用时
会尝试重用已存在的`Manager`，除非为`multiplex`选项传递`false`，
传递这个选项就相当于传递`'force new connection':true`。

剩下的options传递给`Manager`的构造函数（详情参见下边）。

为URL里的path所指定的命名空间返回一个`socket`实例，默认是`/`。
例如，如果`url`是`http://localhost/users`，一个传输链接将被
建立到`http://localhost`,并且一个Socket.IO连接将被建立到`／user`。

##IO#protocol
这个客户端所工作的Socket.io协议修订编号。

##IO#Socket
`Socket`的构造函数的引用。

##IO#Manager
`Manager`的构造函数的引用。

##IO#Emitter
`Emitter`的构造函数的引用。

##Manager(url:String, opts:Object)
一个`Manager`代表到给定的Socket.IO 服务器的一个链接。
一个或者更多的`Socket`实例和manager关联。可以通过每个
`Socket`实例的`io`属性来访问manager。
`opt`也会在`Socket`初始化后传递给`engine.io`.

选项：
- `reconnection`是否自动重连接（`true`）
- `reconnectionDelay` 等待多久尝试重新连接（1000）
- `reconnectionDelayMax` 两次重连接间的最大等待时间（5000）。
每次尝试重连都增加`reconnectionDelay`指定的时间。
- `timeout` 在`connect_error`和`connect_timeout`
事件发射之前的连接超时（20000）。

事件：
*   `connect` 成功连接后触发。
*   `connect_error` 连接错误时触发。
    *   `Object`    错误对象。
*   `connect_timeout` 连接超时时触发。
*   `reconnect` 重连接成功是触发。
    *   `Number` 重连接尝试次数。
*   `reconnect_attempt` 尝试重连时触发。
*   `reconnecting` 尝试重连是触发。
    *   `Number` 尝试重连次数。
*   `reconnect_error` 重连接出错时触发。
    *   `Object` 错误对象。
*   `reconnet_failed` 在`reconnectionAttempts`无法连接是触发。

以上事件被发送到依赖这个`Manager`进行重连的`socket`。

##Manager#reconnection(v:Boolean):Manager
设置`reconnection`选项，如果不传参则返回该选项。

##Manager#reconnectionAttempts(v:Boolean):Manager
设置`reconnectionAttempts`选项，如果不传参则返回该选项。

##Manager#reconnectionDelay(v:Boolean):Manager
设置`reconectionDelay`选项，如果不传参则返回该选项。

##Manager#reconnectionDelayMax(v:Boolean):Manager
设置`reconnectionDelayMax`选项，如果不传参则返回该选项。

##Manager#timeout(v:Boolean):Manager
设置`timeout`选项，如果不传参则返回该选项。

##Socket
事件：
*   `connect`. 连接是触发。
*   `error`. 连接出错时触发。
    Parameters:
    * `Object` 错误数据。
*   `disconnect`. 断开是触发。
*   `reconnect`. 重连成功是触发。
    Parameters:
    * `Number` 尝试重连的次数
*   `reconnect_attempt`. 尝试重连时触发。
*   `reconnecting`. 尝试重连时触发。
    Parameters:
    *   `Number` 重连尝试次数。
*   `reconnect_error`. 尝试重连出错时触发。
    Parameters:
    *`Object` 错误对象
*   `reconnect_failed`.在reconnectionAttempts无法连接是触发。
