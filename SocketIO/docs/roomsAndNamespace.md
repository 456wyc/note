[原文链接:http://socket.io/docs/rooms-and-namespaces/](http://socket.io/docs/rooms-and-namespaces/)
#Namespace(命名空间)
Socket.IO允许你为你的socket指定命名空间，这本质上意味着
分配不同的*节点*或*路径*。

这是一个非常有用的特点，它可以让资源（TCP连接）数量最小化，并且
同时通过分离信道来分离你的应用的关注点。

##默认命名空间
我们把`／` 作为默认命名空间，默认情况下Socket.IO客户端连接到
它，服务器也默认监听它。

这个命名空间被`io.sockets`或简单的`io`所标识。

    // the following two will emit to all the sockets connected to `/`
    io.sockets.emit('hi', 'everyone');
    io.emit('hi', 'everyone'); // short form

每个命名空间发出一个`connection`事件，该事件接收每个`Socket`
实例作为参数。

    io.on('connection', function(socket){
      socket.on('disconnect', function(){ });
    });


##自定义命名空间
你可以在服务器端通过调用`of`函数老设置一个自定义命名空间。

    var nsp = io.of('/my-namespace');
    nsp.on('connection', function(socket){
      console.log('someone connected'):
    });
    nsp.emit('hi', 'everyone!');

在客户端，你需要告诉Socket.IO客户端去链接到那个命名空间：

    var socket = io('/my-namespace');

**特别注意：**命名空间是Socket.IO的具体实现，它和真实的
底层传输的URL不相关，默认为`/socket.io/…`。

---

#Rooms(房间)
对于每个命名空间，你还可以定义任意的通道，从而让套接字
`join`和`leave`。

##Join和leaving(加入和离开)
你可以调用`join`来订阅套接字到给定的通道：

    io.on('connection', function(socket){
      socket.join('some room');
    });

然后在广播过着发送时简单的使用`to`或`in`(两个一样):

    io.to('some room').emit('some event'):

要离开一个通道，你只用像调用`join`一样调用`leave`即可。

##默认room
Socket.IO中的每个`Socket`都被一个随机的、不可猜测的、
唯一的标识符`Socket#id`所标识。为你方便，每个socket会
自动加入一个以该id标识的房间。

这使得广播信息给其他socket变得容易：

    io.on('connection', function(socket){
      socket.on('say to someone', function(id, msg){
        socket.broadcast.to(id).emit('my message', msg);
      });
    });

##断开链接
在断开连接时，socket会自动离开他们所属的所有的通道，并且不
需要你做其他特殊工作。

---
#发送外界信息
在某些时候，你可能想要发送事件到你的Socket.IO进程以外的
Socket.IO的命名空间或房间的socket。

这里有一些办法处理这个问题，像实现你自己的通道去发送信息
到那个进程。

为了帮助用例，我们创建了两个模块：
* [socket.io-redis](http://github.com/automattic/socket.io-redis)
* [socket.io-emitter](http://github.com/automattic/socket.io-emitter)

通过实现Redis`Adapter`:

    var io = require('socket.io')(3000);
    var redis = require('socket.io-redis');
    io.adapter(redis({ host: 'localhost', port: 6379 }));

你可以`emit`来自任何其他进程的信息到任何通道

    var io = require('socket.io-emitter')();
    setInterval(function(){
      io.emit('time', new Date);
    }, 5000);
