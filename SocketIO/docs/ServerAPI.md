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
设置适配器`v`。默认为一个socket.io附带的基于内存的`Adapter`实例。
参见[socket.io-adapter](https://github.com/learnboost/socket.io-adapter)。

##Server#origins(v:String):Server
设置起源为允许的值`v`。默认为任何被允许的起源。
如果没有传递参数，返回值为当前值。

##Server#sockets:Namespace
默认的的（`／`）命名空间。

##Server#attach(srv:http#Server, opts:Object):Server
绑定`Server`到一个`srv`上的engine.io的实例，
同时可以提供`opt`(可选)。

##Server#attach(port:Number, opts:Object):Server
绑定`Server`到一个与`port`绑定的engine.io实例，
同时可提供`opt`(可选)。

##Server#listen
和`Server#attach`一样。

##Server#bind(srv:engine#Server):Server
仅作高级用法。绑定server到制定的engine.io`Server`(或兼容的API)实例。

##Server#onconnection(socket:engine#Socket):Server
仅作高级用法。从传入的engine.io(或兼容API)`socket`创建
一个新`socket.io`客户端。

##Server#of(nsp:String):Namespace
通过路径名`nsp`初始化并检索给定的`Namespace`。
如果命名空间已经初始化，那么就立刻返回它。

##Server#emit
发送一个事件到所有已连接的客户端，以下两个是等价的：

    var io = require('socket.io')();
    io.sockets.emit('an event sent to all connected clients');
    io.emit('an event sent to all connected clients');
其他可用方法，参见下边的`Namespace`。

##Server#use
参见下边的`Namespace#use`。

##Namespace
代表一个通过路径名标识(如：*/char*)的给定范围下的套接字连接池。
默认情况下客户端总是连接到`/`。
事件：
* connection/connect.触发一个连接。
    * `Socket`进来的套接字。

##Namespace#name:String
命名空间的标识符。

##Namespace#connected:Object
通过`id`索引的已连接到这个命名空间的`Socket`hash散列表。

##Namespace#use(fn:Function):Namespace
注册一个中间件，它是一个函数,每个`socket`进来时都会被执行，
并且将该socket和一个函数作为参数接收，该函数可用于选择推迟
到下一个注册的中间件执行。

    var io = require('socket.io')();
    io.use(function(socket, next){
      if (socket.request.headers.cookie) return next();
      next(new Error('Authentication error'));
    });
传递给中间件回调函数的错误被作为特殊`error`数据包发送给客户端。

##Socket
`socket`是与浏览器交互的基本类，`socket`属于某个`Namespace`(默认是`/`),
并且使用一个底层的`client`进行通信。

##Socket#rooms:Array
一个标识该socket所在的房间的字符串的列表。

##Socket#client:Client
底层的`client`对象的引用。

##Socket#conn:Socket
底层的`client`传输连接(engine.io`Socket`对象)的引用。

##Socket#request:Request
一个getter代理，它返回来自底层的engine.io`Client`的`request`的
请求的引用。用来访问请求头信息，如`Cookie`或`User-Agent`。

##Socket#id:String
socket session的唯一标识符，来自底层的`Client`。

##Socket#emit(name:String[, â¦]):Socket
发送事件到`name`标识的socket.任何其他的参数都可以包含进来。
所有的数据结构都支持，包括`Buffer`。JavaScript函数也可以
被序列化和反序列化。

    var io = require('socket.io')();
    io.on('connection', function(socket){
      socket.emit('an event', { some: 'data' });
    });

##Socket#join(name:String[, fn:Function]):Socket
添加该socket到`room`,并且在`err`产生时触发可选的回调函
数`fn`(如果有的话)。

该socket会自动的成为以其seesion id（参见`Socket#id`）
标识的房间的一员。

加入房间的机制由已经配置的`Adapter`(参见前面的`Server#adapter`)
来处理。默认为[socket.io-adapter](https://github.com/socket.io/socket.io-adapter).

##Socket#leave(name:String[, fn:Function]):Socket
将socket从`room`移除，并且在`err`产生是触发可选的回调
函数`fn`(如果有的话)。

**当断开时自动离开房间。**

离开房间的机制由已经配置的`Adapter`(参见前面的`Server#adapter`)
来处理。默认为[socket.io-adapter](https://github.com/socket.io/socket.io-adapter).


##Socket#to(room:String):Socket
##Socket#in(room:String):Socket
为后续事件发送设置一个修饰符，这个事件将只能被 *广播* 到那些已加
入到给定的`room`的socket。（就是向指定room的所有socket广播事件）

要发送到多个房间,你可以调用`to`多次。

    var io = require('socket.io')();
    io.on('connection', function(socket){
      socket.to('others').emit('an event', { some: 'data' });
    });

##Client
`Client`类代表一个进来的传输连接（engine.io）。一个`Client`
可以关联任何属于不同名民空间的多路复用的`Socket`。

##Client#conn
底层的`engine.io``Socket`连接的引用。

##Client#request
一个getter代理，它返回来自底层的engine.io连接的`request`
的引用。用于访问请求头信息例如`Cookie`或`User-Agent`。
