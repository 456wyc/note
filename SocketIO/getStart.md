原文链接：http://socket.io/get-started/chat/
#开始：聊天应用
本节将创建一个聊天应用，几乎不需要有对NodeJS或者SocketIO有
什么了解，所以适合所有用户！

##介绍：
使用如 LAMP(PHP) 之类的流行web应用栈技术搭建一个聊天应用一直
很麻烦，它包括轮询服务器的变化,跟踪时间戳,这让它的反应速度比
我们要求的要慢许多。

对于大多数现存的实时聊天系统来说，Socket算是一个传统的解决方
案，他提供了一个客户端和服务器端之间的双向通信通道。

这意味着服务器可以推送信息到客户端。当你写了一条聊天信息时，
我们的想法是，服务器会得到它并且把它发送到所有其他已连接的客
户端。

##web框架：
我们最初要弄一个简单的HTML页面，页面包含一个表单和信息列表。
我们将使用NodeJS的web框架express来完成这一步。此前要确保
已安装[NodeJS](http://nodejs.org/)。

首先，我们创建一个package.json 清单文件来描述我们的项目。
建议你把它放到专用的空目录（我们将称之为chat-example）。

    {
    "name": "socket-chat-example",
    "version": "0.0.1",
    "description": "my first socket.io app",
    "dependencies": {}
    }

现在为了更容易的填写我们所需要的东西的 dependencies(依赖性)，
我们需要使用:

    npm install --save:
    npm install --save express

express已经安装好了，我们可以创建一个将用来启动应用的index.js文件。

    var app = require('express')();
    var http = require('http').Server(app);

    app.get('/', function(req, res){
    res.send('<h1>Hello world</h1>');
    });

    http.listen(3000, function(){
    console.log('listening on *:3000');
    });

以上代码解释如下：
>   1. express初始化app作为一个函数处理器供我们的HTTP
        服务器使用（见第二行）。
>   2. 定义一个路由处理器/ ，当网站主页被访问时它会被呼叫。
>   3. 让http服务器监听3000端口。

如果你运行`node index.js`，你会看到如下情况：

![效果图](https://i.cloudup.com/-LsMcTduUg.png)


如果你用浏览器打开：[http://localhost:3000](http://localhost:3000),会看到:

![效果图](https://i.cloudup.com/AOuGSHy7QM.png)

##服务HTML：
到目前为止,我们在 *index.js* 里调用了`res.send`并传递给它
一个HTML字符串。但是如果我们将应用的所有HTML代码都放到这里，
我们的代码将变得混乱不堪。因此，我们将创建一个 *index.html*
文件并且为它提供服务。

下边我们通过 sendfile 重构一下我们的路由器：

    app.get('/', function(req, res){
    res.sendFile('index.html');
    });

然后向index.html中填充如下代码：

    <!doctype html>
    <html>
    <head>
      <title>Socket.IO chat</title>
      <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font: 13px Helvetica, Arial; }
        form { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
        form input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
        form button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
        #messages { list-style-type: none; margin: 0; padding: 0; }
        #messages li { padding: 5px 10px; }
        #messages li:nth-child(odd) { background: #eee; }
      </style>
    </head>
    <body>
      <ul id="messages"></ul>
      <form action="">
        <input id="m" autocomplete="off" /><button>Send</button>
      </form>
    </body>
    </html>

如果你重启一下进程（通过 Ctrl+C 结束，然后node index重启）并刷
新网页，将看到这样的效果：
![效果图](https://i.cloudup.com/985FgSH2HQ.png)


##集成Socket.IO
Socket.IO由两部分组成：
>  * 一个与Node.JS HTTP服务器集成(或加载在上)的服务器：socket.io
>  * 一个在浏览器端加载的客户端库：socket.io-client

在开发过程中，我们会看到 *socket.io* 自动为客户端提供服务，
现在我们只需要安装一个模块：

    npm install --save socket.io

这将会安装模块并添加依赖到`package.json`。现在我们编辑
*index.js* ，并添加如下代码：

    var app = require('express')();
    var http = require('http').Server(app);
    var io = require('socket.io')(http);

    app.get('/', function(req, res){
    res.sendfile('index.html');
    });

    io.on('connection', function(socket){
    console.log('a user connected');
    });

    http.listen(3000, function(){
    console.log('listening on *:3000');
    });

注意我们在这里通过`http`对象（HTTP服务器）初始化了一个新的
`socket.io` 的实例。接着，我们为进来的`sockets`监听了
`connection`事件，并且在控制台输出了日志。

现在在 *index.html* 的 `</body>` 前添加如下片段：

    <script src="/socket.io/socket.io.js"></script>
    <script>
    var socket = io();
    </script>

这是用来加载`socket.io-client`的，它暴露了一个全局的`io`，
然后连接它。

注意，我在调用io()时没有指定任何URL，因为它默认会尝试连接提
供该页面服务的主机。

如果你现在重新加载服务器和网站，你应该会看到一个输出
“a user connected”。
尝试多开一些页面，你将看到一些消息：

![效果图](https://i.cloudup.com/F5EBcTVDcd.png)

每个socket 也会触发特殊的`disconnet`事件；

    io.on('connection', function(socket){
    console.log('a user connected');
    socket.on('disconnect', function(){
      console.log('user disconnected');
    });
    });

然后多次刷新一个tab，就会看到它有动作：

![效果图](https://i.cloudup.com/bOmy6xrJmi.png)


##发出事件：
*Socket.IO* 的主要思想就是你可以发送和接收你想要做的任何事件以
及任何数据。任何对象都可以被编码成JSON，同时二进制数据也支持。

现在我们要这样做，当用户输入一个消息时，服务器获取到消息并作为一个
`chat message`事件。*index.html*里的脚本片段看起来应该是下
边这样：

    <script src="/socket.io/socket.io.js"></script>
    <script src="http://code.jquery.com/jquery-1.11.1.js"></script>
    <script>
    var socket = io();
    $('form').submit(function(){
      socket.emit('chat message', $('#m').val());
      $('#m').val('');
      return false;
    });
    </script>

我们在index.js里边打印出chat message事件：

    io.on('connection', function(socket){
    socket.on('chat message', function(msg){
      console.log('message: ' + msg);
    });
    });
结果看起来应该是这样：
[效果](https://i.cloudup.com/transcoded/zboNrGSsai.mp4)

<video autoplay="true" loop="" width="100%" controls="">
  <source src="https://i.cloudup.com/transcoded/zboNrGSsai.mp4">
</video>


##广播：
我们接下来的目标就是把事件发送到其他用户。

为了发送事件给每个人，Socket.IO 给我们提供了`io.emit`：

    io.emit('some event', { for: 'everyone' });

如果你想给除了某个socket外的所有用户发消息，我们有broadcast标签：

    io.on('connection', function(socket){
        socket.broadcast.emit('hi');
    });

在这种情况下,为了简单起见,我们将发送消息给所有人,包括发送方。

    io.on('connection', function(socket){
        socket.on('chat message', function(msg){
          io.emit('chat message', msg);
        });
    });

在客户端，当我们捕获到一个chat message事件，我们就会把它插
入页面。现在客户端所有脚本如下：

    <script>
        var socket = io();
        $('form').submit(function(){
          socket.emit('chat message', $('#m').val());
          $('#m').val('');
          return false;
        });
        socket.on('chat message', function(msg){
          $('#messages').append($('<li>').text(msg));
        });
    </script>

我们整个聊天应用大概就只用了20行代码！看起来会是怎样的呢？
[效果](https://i.cloudup.com/transcoded/J4xwRU9DRn.mp4)

<video autoplay="" loop="" width="100%" controls="">
<source src="https://i.cloudup.com/transcoded/J4xwRU9DRn.mp4">
</video>


##课后练习：
这里有一些增强该应用的想法：
  1. 当某人上线或下线时广播给所有用户！
  2. 添加昵称支持。
  3. 不发送同样的消息给用户自己，而是当用户按下发送按钮时直接在页面添加消息。
  4. 添加"{用户}正在输入"的功能。
  5. 显示谁在线。
  6. 添加私信。
  7. 分享你的改进！

##获取这个例子：
你可以在[GitHub](https://github.com/guille/chat-example)
上找到

    $ git clone https://github.com/guille/chat-example.git
