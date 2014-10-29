#从0.9迁移（[原文链接:http://socket.io/docs/migrating-from-0-9/](http://socket.io/docs/migrating-from-0-9/)）
对于大多是应用来说，过度到1.0应该是完全无缝的，没有问题的。
也就是说，我们已经做了一些努力来简化一些API，并且我们已经
改变了一些内部构件，所以推荐大多数现有用户读一度这篇文章。

##验证上的差异
Socket.IO现在使用的是中间件。
你可以通过`io.use()`方法给一个Socket.IO服务器任意的函数，
该方法会在一个socket创建时运行。看看这个例子：

    var srv = http();
    var sio = require('socket.io')(srv);
    var run = 0;
    sio.use(function(socket, next){
      run++; // 0 -> 1
      next();
    });
    sio.use(function(socket, next) {
      run++; // 1 -> 2
      next();
    });
    var socket = require('socket.io-client')();
    socket.on('connect', function(){
      // run == 2 here
    });
…现在通过中间件来做验证是如此干净。

旧的`io.set()`和`io.get()`方法是不赞成的，他们仅仅被
用于向后兼容。这里有一个老的认证例子翻译到中间件风格：

    io.set('authorization', function (handshakeData, callback) {
      // make sure the handshake data looks good
      callback(null, true); // error first, 'authorized' boolean second
    });

vs.

    io.use(function(socket, next) {
      var handshakeData = socket.request;
      // make sure the handshake data looks good as before
      // if error do this:
        // next(new Error('not authorized');
      // else just call next
      next();
    });

命名空间认证？

    io.of('/namespace').use(function(socket, next) {
      var handshakeData = socket.request;
      next();
    });


##日志差异
现在日志基于debug
要打印所有debug日志，设置环境变量**DEBUG**为*****。
如`DEBUG=* node index.js`

只打印socket.io相关日志：
`DEBUG=socket.io:* node index.js`。

只打印来自socket对象的日志：
`DEBUG=socket.io:socket node index.js`

This pattern should hopefully be making sense
at this point. The names of the files in
socket.io/lib are equivalent to their debug names.

调试也在浏览器进行，日志保存在本地。
用法：打开开发控制台并键入
`localStorage.debug = 'socket.io:*'`（或任何其他
调试级别），然后刷新页面。所有事情都被记入日志，知道你
运行`localStorage.debug = ''`。

在[这里](https://www.npmjs.org/package/debug)
查看更多调试文档。


##快捷方式
通常，这里有一些新的常见事情的快捷方式。老版本应该继续
工作，但是快捷方式是好的。

####向默认命名空间的所有客户读广播
*以前*

    io.sockets.emit('eventname', 'eventdata');

*现在*

    io.emit('eventname', 'eventdata');

优雅！注意这两种情况，这些信息到达所有已连接到默认的`/`
命名空间的客户端，但是不会到其他命名空间的客户端。

####启动服务器
*以前*

    var io = require('socket.io');
    var socket = io.listen(80, { /* options */ });

*现在*

    var io = require('socket.io');
    var socket = io({ /* options */ });

##配置差异
io.set 没有了。
取而代之的在服务器端进行配置的方法是像这样初始化：

    var socket = require('socket.io')({
      // options go here
    });

像log-level这样的选项也没有了。为了向后兼容，
`io.set('transports')`,`io.set('heartbeat interval')`,
`io.set('heartbeat timeout'`,`io.set('resource')`
仍然支持。

####设置资源路径
以前的`resource`选项等价于新的`path`选项，但是在开头需要
一个`/`。例如，以下的配置：

    var socket = io.connect('localhost:3000', {
      'resource': 'path/to/socket.io';
    });
变成

    var socket = io.connect('localhost:3000', {
      'path': '/path/to/socket.io';
    });


##解析器／协议差异
这只跟更新事情有关，如其他语言实现的socket.io、自定义的
socket.io客户端等等。

####差异1——数据包编码
现在基于类和异步解析。而不是返回一个单一的编码字符串。
编码调用仅有一个编码数组作为参数回调函数。每个编码应该按顺序
写入传输通道。这样更加灵活，而且是使用二进制数据工作，如下：

    var encoding = parser.encode(packet);
    console.log(encoding); // fully encoded packet
VS.

    var encoder = new parser.Encoder();
    encoder.encode(packet, function(encodings) {
      for (var i = 0; i < encodings.length; i++) {
        console.log(encodings[i]); // encoded parts of the packet
      }
    });

####差异2——数据包解码
解码更进一步，是基于事件的，这样做是因为一些对象
（binary-containing）在多个部分编码和解码，以下例子应该有
帮助：

    var packet = parser.decode(decoding);
    console.log(packet); // formed socket.io packet to handle

VS.

    var decoder = new parser.Decoder();
    decoder.on('decoded', function(packet) {
      console.log(packet); // formed socket.io packet to handle
    });
    decoder.add(encodings[0]); // say encodings is array of two encodings received from transport
    decoder.add(encodings[1]); // after adding the last element, 'decoded' is emitted from decoder
