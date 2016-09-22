# 前言
消失了快2个月，俺又回来了。最近换比较忙，好久没写博客，但是学习的脚步一直没停下。前段时间在[cnode](https://www.cnodejs.org)上看到一个关于微信access_token的问题：[高并发如何保证微信token的有效](https://cnodejs.org/topic/57b330b6670139eb7bf6fd4c#57b51330d6124db37b1d13cb)。其中本人也在下面回复了一下，但是觉得解决方案还是不够好，于是就有了本篇：本文主要以渐进的方式，来一步一步解决高并发情况下access_token的获取与保存。

# 前提条件
由于本文讨论是基于微信公众平台开发展开的，所以如果对微信公众平台开发不熟悉的同学可以先去看下微信公众平台的[开发文档](https://mp.weixin.qq.com/wiki/home/)

# 需要解决的问题
本文讨论的其实是access_token获取与保存在高并发情况下的边界问题：
+ 1.在node单进程情况下 第一个请求（请求A）过来，程序发现access_token过期，这时就会去向微信服务器获取新的access_token，然后更新到全局变量中。这个过程本身没有问题，但是如果请求A在向微信服务器请求新的access_token期间，来了第二个请求（请求B），因为这时新的access_token还未更新到全局变量中，请求B就会认为access_token已过期，也会同请求A一样，去微信服务器请求新的access_token，这样就会导致请求A得到的access_token失效。想象一下，如果在请求A后面有来了N个请求，并且此时请求A还未成功更新access_token，那么后面的所有请求都会去微信服务器获取新的access_token。以上的情况，不仅会导致浪费带宽，而且会导致最后一个请求之前获取的access_token都失效。那我们应该如何做控制，在高并发情况下仅请求一次微信服务器呢？

+ 2.在node多进程模式下 如果我们的项目开启了node多进程，情况将更加复杂：我们不仅会遇到在单进程情况下的问题，而且由于node进程之间是不共享内存的，也就是说上面说到的全局变量是无法使用的，如果第一个请求由进程1处理，而此时access_token已过期，然后第二个请求由进程2处理。即使在进程1成功更新了access_token，但是由于进程1与进程2内存不共享，所以如果不借助外部存储的话，后面一个请求也无法得知access_token已更新。那么，这种情况有该如何解决呢？

# （单进程模式下）思路
首先我们来讨论，如何解决单进程模式下高并发遇到的问题。
+ 第一步我们需要引入一个新的全局变量，用来标记已经有请求在获取新的access_token
+ 第二步就是将后续的请求缓存到node的事件队列中去
+ 第三步统一触发事件

具体实现代码：

```js
var Emitter = require('events').Emitter;
var util = require('util');

var access_token, flag;

function TokenEmitter() {
  Emitter.call(this);
}
util.inherit(TokenEmitter, Emitter);

myEmitter = new TokenEmitter();
// 消除警告
myEmitter.setMaxListeners(0);

function getAccessToken(appID, appSecret, callback) {
  // 将callback缓存到事件队列中，等待触发
  myEmitter.once('token', callback);

  // 判断access_token是否过期
  if (!isValid(access_token) && !flag) {
    // 标记已经向微信服务器发送获取access_token的请求
    flag = true;

    // 向微信服务器请求新的access_token
    requestForAccessToken(appID, appSecret, function(err, newToken) {
      if (err) {
        // 通知出错
        return myEmitter.emit('token', err);
      }

      // 更新access_token
      access_token = newToken;
      // 触发所有监听回调函数
      myEmitter.emit('token', null, newToken.access_token);
      // 还原标记
      flag = false;
    });
  } else {
    process.nextTick(function(){
      callback(null, access_token.access_token);
    });
  }
}
```

以上代码主要的思路就是利用，node自带的事件监听器，也就是代码中的'myEmitter.once()'方法，在access_token失效的情况下把所有调用的回调方法添加为'token'事件监听函数。并且只有第一个调用者可以去更新access_token的值（主要用flag来控制）。当获得新的access_token后，以新access_token为参数，去触发'token'事件。此时，所有监听了'token'事件的函数都会被调用，也就是说，所有调用者的回调函数都会被调用。这样，我们就实现了高并发情况下，防止access_token被多次更新的问题，也就是解决了问题1。

# （多进程模式下）思路
解决了单进程模式下的问题，可以说我们多进程问题也解决了一部分。在多进程模式下，我们的主题思路还是与单进程一直，将调用缓存到事件队列中。但是，多进程的各个进程是不共享内存的，所以我们的access_token和flag标记不可以存储在变量中，所以我们需要引入外部存储--redis。使用redis作为外部存储有以下几个原因：
+ 1.在redis中统一存储access_token，各个进程都可以自由访问
+ 2.利用redis作为锁媒介（相当于单进程中的flag）
+ 3.利用redis发布订阅功能来触发事件（相当于单进程中的emit）

## 统一存储access_token
这一点大家都应该没什么疑问，access_token统一存储的好处就是不需要面对复杂的进程见通信。

## 锁媒介
当我们标记“正在请求微信服务器的”的flag标志不可以放在代码的变量中时，那就要寻求代码之外的解决方法，其实我们可以存在mongodb、mysql等等可以存储的媒介中，甚至可以存放在文本文件中。但是为了保证速度，我还是考虑将其存放在速度更快的redis中。

## redis发布订阅功能
当然，如果我们的程序使用的是node的cluster模块开启的多进程模式，进程间通信还是相对容易一些：每个worker都可以向master发送message，利用这一点把master当做中心，来交换数据。但是如果我们是使用pm2开启了多实例，pm2虽然提供了进程间通信的API，但是使用起来各种不顺畅，最终选择redis来作为各个进程接受通知的发起方。

以上思路的实现代码大致如下：

1.第一步需要坐的就是从判断access_token是否过期（为了方便起见，直接用appID + appSecret作为存储access_token的键）：从redis获取键为access_token的内容，因为我们在设置access_token时，是将其设为了过期键（设置过程涉及到锁，将在之后给出），所以只要能取到值，就说明access_token没有过期。代码如下：

```js
function isValid(appID, appSecret, callback) {
  redis.get(appID + appSecret, function(err, token) {
    if (err) {
      return callback(err);
    }

    // 可以取到值
    if (tokenInfo) {
      return callback(null, token);
    }

    // 未取到值
    callback(null);
  });
}
```

2.如果在第一步的判断中，我们得出结论：access_token已经过期，那么我们需要做的下一步就是设置一个代码级别的锁，防止之后的程序访问之后的代码：
```js
function aquireLock(callback) {
  redis.setnx('lock', callback);
}

function releaseLock(callback) {
  redis.del('lock', callback);
}
```
这2个函数，一个用于设置锁，一个用于释放锁。我们设置锁是利用了redis的setnx命令原理：setnx只可以设置不存在的key，即使同一时间有多个setnx命令来设置同一个key，由于redis是单线程的，最终只有一个客户端可以成功设置'lock'键，也就是说只有一个请求获得了锁的权限。这样就控制了并发锁产生的问题。

3.最后我们将所有程序写入主函数中：
* 1).主函数中一进来，首先添加监听器
* 2).第二步我们需要订阅redis的new_token和new_token_err频道
* 3).第三步判断access_token是否过期
* 4).如果过期，就尝试去获取锁权限
* 5).如果获取锁失败，就什么都不做，等待事件触发；如果获取锁权限成功后，就可以请求微信服务器来获取新的access_token，当获得新的access_token之后将其更新到redis中，并且设置合理的过期过期时间
* 6).释放锁并且发出通知
```js
function getAccessToken(appID, appSecret, callback) {
  // 将callback缓存到事件队列中，等待触发
  myEmitter.once('token', callback);

  // 处理订阅消息
  subscribe.on('message', (channel, message) => {
    switch (channel) {
      case 'new_token':
        myEmitter.emit('token', null, message);
        break;
      case 'new_token_err':
        myEmitter.emit('token', new Error(message));
        break;
      default:
        break;
    }
  });

  // 判断access_token是否过期
  isValid(appID, appSecret, function(err, token) {
    // 出错
    if (err) {
      return myEmitter.emit('token', err);
    }

    // token正常
    if (token) {
      return myEmitter.emit('token', null, token.access_token);
    }

    // token已过期，获取锁
    aquireLock(function(err, result) {
      // 如果获取锁成功，则开始更新access_token，如果未得到锁，等待'token'触发
      if (result) {
        // 向微信服务器请求新的access_token
        requestForAccessToken(appID, appSecret, function(err, newToken) {
          if (err) {
            // 释放锁标记
            releaseLock();
            // 通知出错
            return myEmitter.emit('token', err);
          }

          // 更新access_token，将新的access_token保存到redis，并且提前5分钟过期
          redis.setex(appID + appSecret, (newToken.expires_in - 300), newToken.access_token);
          // 发布更新
          publish.publish('new_token', newToken.access_token);
          // 释放锁标记
          releaseLock();
        });
      }
    });

    // 订阅
    subscribe.subscribe('new_token');
  });
}
```

# 进一步思考
到此，一个简单多进程控制token并发的解决方法已经呈现在眼前，但是我们还需要考虑一下边界情况：
+ 1.当我们获得到锁权限的进程在未知因素下崩溃了，并且此时用于标记锁状态的'lock'已被成功设置
+ 2.微信服务器崩溃了（无法正常提供服务）
以上2个情况，都会导致锁状态永远无法释放，针对这一类问题，我们可以引入超时的概念，在设置'lock'的时候给'lock'键设置一个过期时间，如果过了指定的时间还没有释放锁，那么锁权限就会被收回。代码如下：

```js
function aquireLock(callback) {
  redis.watch('lock');
  redis.multi().setnx('lock', callback).expire('lock', 2).exec();
}
```
由于设置锁和设置锁的过期时间需要同一时间完成，所以这里我使用了redis的事务来保证了原子性。

# 更进一步的思考
虽然我们解决了锁问题，但是此时所有未获得锁的请求还处于pending状态，等待这access_token的到来，但是由于获取锁的请求已经走在天堂的路上，已经无法再来给这些个请求触发事件了。所以为了解决此类问题，我们需要引入另一个超时，那就是函数调用超时，在一定时间内未完成的话，我们就回调一个超时错误给调用者：
```js
function getAccessToken(appID, appSecret, callback) {
  // 将callback缓存到事件队列中，等待触发
  myEmitter.once('token', callback);

  // 设置函数调用超时
  setTimeout(function () {
    callback(null, new Error('time out'));
  }, 2000);
}
```

# 总结
其实在使用redis的订阅功能之前，我还考虑过[tj](https://github.com/tj)的[axon](https://github.com/tj/axon)作为进程通信的手段，但是由于axon初始化过程有一定的延迟，不符合我的预期，所以放弃了。但是不得不说axon是一个非常好的项目，有条件的话可以用在项目当中。好了，以上就是我对高并发下处理access_token的一些自己的看法。
