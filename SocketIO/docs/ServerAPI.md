#Server API（[原文链接:http://socket.io/docs/server-api/](http://socket.io/docs/server-api/)）
##Server
通过require('socket.io')暴露出来。

##Server()

通过*new* 或者不通过两种方式创建一个新的Server:

    var io = require('socket.io')();
    // or
    var Server = require('socket.io');
    var io = new Server();
##Server(opts:Object)
可选的，`Server`构造函数的第一、二个参数都是一个可选的对象（如下）。
下列选项是被支持的：
* `serveClient` 为Server#serveClient()设置值。
* `path` 为Server#path()设置值。
传递给socket.io的选项总是同样的传递给`engine.io` `Server`。
参见[engine.io选项](https://github.com/learnboost/engine.io#methods-1).

##Server(srv:http#Server, opts:Object)
创建一个新`Server`并且附加到给定的`srv`。`opt`是可选的。

##Server(port:Number, opts:Object)
绑定socket.io到监听`port`端口的新`http.Server`。

##Server#serveClient(v:Boolean):Server
如果`v`是`true`,被绑定的server(参见`Server#attach`)将为
客户端提供文件。默认是`true`。
当`attach`被调用后并不对该方法产生影响。

    // pass a server and the `serveClient` option
    var io = require('socket.io')(http, { serveClient: false });

    // or pass no server and then you can call the method
    var io = require('socket.io')();
    io.serveClient(false);
    io.attach(http);

如果不提供参数，方法将返回当前值。

##Server#path(v:String):Server
设置`engine.io`下的路径`v`并且保存静态文件。默认值是`/socket.io`。

##Server#adapter(v:Adapter):Server
