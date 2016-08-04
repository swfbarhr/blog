![nodejs，祝老婆大人、爸妈、岳父岳母身体健康，宝宝健康成长。新年快乐！](../file/nodejs.jpg)
##### 前言
在开始node.js（以后简称node）之前，我犹豫了好久：到底我第一篇用什么主题开始，是node的整体框架还是Hello world还是一堆废话呢。最后我决定还是直接上API，简单粗暴有效果，因为大家用node除了node有强大的社区之外，还因为node的API简单易用，容易上手。OK，下面我们就开始node之旅。

##### HTTP
大部分的node使用者，都是用node来做Web API的，而HTTP模块是提供Web API的基础。为了支持所有的HTTP应用，node中的HTTTP模块提供的API是偏向底层化的。利用HTTP模块，我们可以简单快速搭建一个Web Server。

##### 引入HTTP模块
在node[官方API](https://nodejs.org/dist/latest-v5.x/docs/api)中，第一句话是这么说的：“To use the HTTP server and client one must require('http')”，如果想要使用HTTP的服务器端和客户端的方法，首先第一步我们需要使用“require('http')”引入HTTP模块。

```js
var http = require('http');
```

我们引入了HTTP模块，并且将结果赋值给了http变量，这样http就拥有了HTTP模块的所有方法和对象（关于requirejs的原理，与主题没什么太大关系，就略过。但是记住，require的过程是一个同步的过程，而前端的引入define是一个异步过程）。

##### 创建一个简单的Web Server
在HTTP模块中，node提供给我们一个createServer方法，来创建一个Web Server。代码可以这样写：

```js
var http = require('http');
var server = http.createServer();

server.on('request', function(request, response) {
  console.log('有人请求了服务器！');

  response.writeHead(200, {
    'Content-Type': 'text/plain'
  });
  response.write('OK');
  response.end();
});

server.listen(88, function() {
  console.log('服务器已经开始监听88端口');
});
```

上面的代码：

+ 我们首先用http.createServer（从express4.X时代进入的同学可能有点陌生，express在4.X后把该方法封装到了其内部，但是我们在创建HTTPS连接时，还是要用到此方法）函数创建了一个服务器对象。

+ 然后监听了该对象的“request”事件（了解过node的同学应该知道，node是事件驱动的，一般事件触发和事件接收是成对出现的），这边我们定义接收事件。也就是说，如果当前服务器对象监听的端口一旦被请求，那么就会触发该事件。“request”的回调中，我们首先调用了response.writeHead方法：该方法的第一个参数表示HTTP的响应状态（200）表示一切正常；第二个参数是“Content-Type”，表示我响应给客户端的内容类型。然后我们调用了response.write方法，写入我们需要传递给客户端的内容。最后一步我们调用了response.end，表示此次请求已处理完成。

+ 最后我们调用了server.listen函数，此函数有两个参数，第一个参数表示我们需要监听的端口，第二个参数是回调函数（其实是listening事件），当监听开启后立刻触发。所以当我们用node命令来启动当前的js文件后，在cmd窗口会马上出现“服务器已经开始监听88端口”的字样，表明服务器正常且已经开始监听88端口。

当我们请求了服务器的88端口后，服务器会响应给我们“OK”字符串。到此我们的简单Web Server已经搭建完成，但是此时我们的代码还不具备任何业务能力，但是我们只要把代码稍稍修改下，就可以入眼了：

```js
var url = require('url');
var http = require('http');
var server = http.createServer();

function bizLogic(url) {

  if (url.path.toLowerCase() === '/login') {
    return {
      msg: '登录成功！'
    }
  } else if (url.path.toLowerCase() === '/user') {
    return {
      msg: [{
        name: '小明',
        class: '一年级1班'
      }, {
        name: '小红',
        class: '一年级3班'
      }]
    }
  } else {
    return {
      msg: 404
    }
  }
}

server.on('request', function(request, response) {

  if ('url' in request) {
    response.writeHead(200, {
      'Content-Type': 'application/json; charset=utf-8'
    });

    response.write(JSON.stringify(bizLogic(url.parse(request.url))), 'utf8');
  } else {
    response.writeHead(404, {
      'Content-Type': 'application/json; charset=utf-8'
    });

    response.write('找不到页面');
  }

  response.end();
});

server.listen(88, function() {
  console.log('服务器已经开始监听88端口');
});
```

+ 首先我们引入了一个新的模块：“url”。这个模块主要帮助我们将整理请求url中的信息即url.parse方法（如果有兴趣，你可以去研究下url的所有API，其实就几个）。url.parse转换完成之后，url参数信息就变成这样了：

```js
{
    protocol: null,
    slashes: null,
    auth: null,
    host: null,
    port: null,
    hostname: null,
    hash: null,
    search: null,
    query: null,
    pathname: '/login',
    path: '/login',
    href: '/login'
}
```

这里我们取了我们需要的参数：path，简单的根据path来判断接口。然后响应客户端不同的内容。

+ 如果我们直接“response.write(bizLogic(url.parse(request.url)), 'utf8');”系统就会报错，response.write第一个参数必须是string或者Buffer，所以我们在这里将object转换成了string，然后接着write。

+ 最后我们直接结束连接（response.end）。

好了，一个简单的业务已经处理完成了，但是实际开过过程没有那么简单：HTTP方法有“GET”、“POST”、“PUT”等等。这也是要我们来处理的，所以如果我们用node来做Web API的话，一般会选择第三方的Web框架如：express、koa、restify等等（好吧，这是后话了，以后有机会我会去研究下他们各自的源码给大家分享）。

##### http.request
上面讲了那么多，其实都是在将HTTP Server的内容，既然node那么强大，当然除了可以作为HTTP Server之外，也可以做HTTP Client。现在就以我们刚刚建立的服务端为服务器，利用http.request来请求刚刚的服务器，看看会发生什么事情。

```js
var http = require('http');

var option, req;

option = {
  host: '127.0.0.1',
  port: 88,
  path: '/login'
};

req = http.request(option, function(res){

  if(res.statusCode === 200) {
    console.log('ok');
  }

  res.on('data', function(chunk){
    console.log('the result is ', chunk.toString());
  });
});

req.on('error', function(error){
  console.log('something is wrong:', error.message);
});

req.end();
```

+ 首先我们当然要引入http模块，之后又定义了option和req对象。option是我们一会需要请求服务器的一些参数，req是请求对象用于接收request返回的结果（并不是请求的结果）

+ option参数是一个对象，使我们对此次请求设置的参数，包括：我们要请求的主机地址（host，注意此处不需要“http://”）、请求的请求的端口号（port）、请求的路由及参数（path）

+ 调用http.request请求，第一个参数就是我们刚刚设置的option，第二个参数是回调函数（用于返回我们请求的结果）

+ 回调函数有一个参数，这个参数是http.IncomingMessage的实例。字面意思就是“进来的消息”，也就是请求收到的响应结果对象。http.IncomingMessage实现了可读流接口，也就是说我们的“data”事件被触发时，回调函数中的结果是Buffer（关于Buffer我以后会详细解释---另一篇博客）对象，所以我在打印的时候调用了Bufer对象的toString方法把Buffer对象转换成字符串打印出来。

+ 最后我们监听了req的错误事件（在node中，我们如果出错不定义error事件的话，程序就会直接把错误抛出，如果没有做错误捕获的话，node程序就会崩掉），用于记录错误信息。最后我们关闭了请求。

经过这几个简单地步骤，我们此次请求就完成了，如果服务端没有错误的话，使用node命令运行我们刚刚写好的客户端代码，在cmd窗口先会打印“ok”，然后带打印“the result is  {"msg":"登录成功！"}”。一旦请求出错，就会直接出发我们的“error”事件。当然如果不想使用原生的HTTP模块的话，你也可以使用第三方的包：[request](https://github.com/request/request)、[superagent](https://github.com/visionmedia/superagent)等都是非常优秀且方便的第三方包。（原生的HTTP模块可能会遇到各种奇葩的问题，使用第三方包的话吗，坑会少一些）。

##### 其他方法和事件

+ http.get是http.request的“GET”方法的方便版本，只能执行“GET”操作。

+ http.ClientRequest的“connection”事件，当TCP（HTTP其实也是TCP）的连接通道建立之后触发的事件。

+ http.ClientRequest的“close”事件，当服务端关闭连接时触发。

+ http.ServerResponse的setTimeout方法，设置响应超时。如果没有设置的话，只能等待socket或服务器超时。（还有很多方法和事件。就不一一列举了。）

##### 总结
在node v0.10.x时代，HTTP模块的稳定系数就已经是3了（一般node的API有4个级别，分别是：Deprecated-0、Experimental-1、Stable-2、Locked-3）.所以说HTTP模块的API非常基础和稳定的。我们使用HTTP模块时，可以很放心的使用其方法和事件，不必担心版本兼容问题。需要提到的是，学习node的最好最捷径的方法就是学会读官方API，官方API虽然是英文的，但是里面的解释和英文单词都是很戳中要点和易于理解的。
