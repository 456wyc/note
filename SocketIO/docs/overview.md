#概览（[原文链接:http://socket.io/docs/](http://socket.io/docs/)）
##如何使用
---
###安装
    $ npm install socket.io
---
###通过Node的*http server*来使用
#####服务器端（app.js）
    var app = require('http').createServer(handler)
    var io = require('socket.io')(app);
    var fs = require('fs');

    app.listen(80);

    function handler (req, res) {
      fs.readFile(__dirname + '/index.html',
      function (err, data) {
        if (err) {
          res.writeHead(500);
          return res.end('Error loading index.html');
        }

        res.writeHead(200);
        res.end(data);
      });
    }

    io.on('connection', function (socket) {
      socket.emit('news', { hello: 'world' });
      socket.on('my other event', function (data) {
        console.log(data);
      });
    });
#####客户端（index.html）
    <script src="/socket.io/socket.io.js"></script>
    <script>
      var socket = io('http://localhost');
      socket.on('news', function (data) {
        console.log(data);
        socket.emit('my other event', { my: 'data' });
      });
    </script>
---
###通过*Express 3/4*来使用
#####服务器端（app.js）
    var app = require('express').createServer();
    var io = require('socket.io')(app);

    app.listen(80);

    app.get('/', function (req, res) {
      res.sendfile(__dirname + '/index.html');
    });

    io.on('connection', function (socket) {
      socket.emit('news', { hello: 'world' });
      socket.on('my other event', function (data) {
        console.log(data);
      });
    });
#####客户端（index.html）
    <script src="/socket.io/socket.io.js"></script>
    <script>
      var socket = io.connect('http://localhost');
      socket.on('news', function (data) {
        console.log(data);
        socket.emit('my other event', { my: 'data' });
      });
    </script>
---
###通过*Express框架*来使用
#####服务器端（app.js）
    var app = require('express').createServer();
    var io = require('socket.io')(app);

    app.listen(80);

    app.get('/', function (req, res) {
      res.sendfile(__dirname + '/index.html');
    });

    io.on('connection', function (socket) {
      socket.emit('news', { hello: 'world' });
      socket.on('my other event', function (data) {
        console.log(data);
      });
    });
#####客户端（index.html）
    <script src="/socket.io/socket.io.js"></script>
    <script>
      var socket = io.connect('http://localhost');
      socket.on('news', function (data) {
        console.log(data);
        socket.emit('my other event', { my: 'data' });
      });
    </script>
---
###发送和接收事件
Socket.IO 允许你发送和接收自定义事件，出了`connect`,`message`
和`disconnect`,你可以发送自定义事件：
####服务器端
    // note, io.listen(&lt;port&gt;) will create a http server for you
    var io = require('socket.io')(80);

    io.on('connection', function (socket) {
      io.emit('this', { will: 'be received by everyone'});

      socket.on('private message', function (from, msg) {
        console.log('I received a private message by ', from, ' saying ', msg);
      });

      socket.on('disconnect', function () {
        io.sockets.emit('user disconnected');
      });
    });
---
###限定你的命名空间
如果你要控制特定应用的所有信息和事件的发送，请使用默认的命名空间*/*来
工作。如果你想要利用第三方代码，或者和他人分享代码,socket.IO提供了一
种方法来为一个socket指定命名空间。
这里看看单连接复用的优点，它用一个连接取代了socket.io的两个*WebSocket*连接。
#####服务器端
    var io = require('socket.io').listen(80);
    var chat = io
      .of('/chat')
      .on('connection', function (socket) {
        socket.emit('a message', {
            that: 'only'
          , '/chat': 'will get'
        });
        chat.emit('a message', {
            everyone: 'in'
          , '/chat': 'will get'
        });
      });

    var news = io
      .of('/news')
      .on('connection', function (socket) {
        socket.emit('item', { news: 'item' });
      });
#####客户端
    <script>
      var chat = io.connect('http://localhost/chat')
        , news = io.connect('http://localhost/news');

      chat.on('connect', function () {
        chat.emit('hi!');
      });

      news.on('news', function () {
        news.emit('woot');
      });
    </script>

---
###发送不稳定的消息
有时可以删除某些消息。假设您有一个应用程序,展示了关键字
*bieber* 的实时tweets。
如果某个客户端没有准备好就收信息（由于网速慢或其他问题，
或者因为他通过长轮询连接并且正好在请求响应周期的中间），
如果他不接收所有与bieber有关的tweets
在这种情况下,您可能想要发送这些消息是动荡的消息。
#####服务器端
    var io = require('socket.io').listen(80);

    io.sockets.on('connection', function (socket) {
      var tweets = setInterval(function () {
        getBieberTweet(function (tweet) {
          socket.volatile.emit('bieber tweet', tweet);
        });
      }, 100);

      socket.on('disconnect', function () {
        clearInterval(tweets);
      });
    });
---
###发送和接收数据([acknowledgements(ACKs)](http://baike.baidu.com/view/204040.htm?fr=aladdin))
有时候，当客户端确认消息接收后你可能想要调用一个回调函数。
为此，只需为*.send* , *.emit* 传递一个函数作为最后一个参数即可。
更重要的是，当你使用*.emit* 时，如果你已经确认ACK，就意味着你也可
以一起传递数据。
#####服务器端
    var io = require('socket.io').listen(80);

    io.sockets.on('connection', function (socket) {
      socket.on('ferret', function (name, fn) {
        fn('woot');
      });
    });
#####客户端
    <script>
      var socket = io(); // TIP: io() with no args does auto-discovery
      socket.on('connect', function () { // TIP: you can avoid listening on `connect` and listen on events directly too!
        socket.emit('ferret', 'tobi', function (data) {
          console.log(data); // data will be 'woot'
        });
      });
    </script>
---
###广播信息
对于广播，只需在调用*sned*， *emit* 时添加一个*broadcast* 标记。
广播意味着向除了发送广播的socket之外的其他socket都发送一个消息。
#####服务器端
    var io = require('socket.io').listen(80);

    io.sockets.on('connection', function (socket) {
      socket.broadcast.emit('user connected');
    });
---
###把它当跨浏览器WebSocket使用
如果你仅仅是希望使用WebSocket语义，你也可以这样做，简单的利用
*send* 并且监听*message* 事件：
#####服务器端(app.js)
    var io = require('socket.io').listen(80);

    io.sockets.on('connection', function (socket) {
      socket.on('message', function () { });
      socket.on('disconnect', function () { });
    });
#####客户端(index.html)
    <script>
      var socket = io('http://localhost/');
      socket.on('connect', function () {
        socket.send('hi');

        socket.on('message', function (msg) {
          // my msg
        });
      });
    </script>
如果你不关心重新连接逻辑等，可以看看[Engine.IO]()，这是
Socket.IO使用的WebSocket语义传输层。
