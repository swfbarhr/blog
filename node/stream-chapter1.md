# 前言
今天要讲的模块（stream，下称流）比较特殊，在node官网的API文档中分成了3个部分：第一部分是介绍流的用法，第二部分是介绍如何实现自定义的流，第三部分是介绍流的原理与内部实现机制。针对这3个部分，我会分3篇博客来讲，现在就来讲讲流的用法（第一部分，也是最基础的部分）

# 概念
流其实不是一个实质的对象或方法，流是一些抽象的接口，只有这些接口被实现了，流才有了真正的意义。所有的流都是EventEmitter的实例，也就是说所有的流都可以触发、响应事件。

# 分类
从功能上讲，流一般可以分为“读取流”、“写入流”、“复杂流”：读取流用来读取，写入流用来写入，复杂流可以读写。

# stream.Readable
stream.Readable是一个类，利用这个类，我们可以读取流里面的内容。可读流有2中模式：flowing-mode、non-flowing-mode

+ flowing-mode 此模式会尽快把数据读取出来

+ non-flowing-mode 此模式需要显示调用读取方法，才会把数据读取出来

### 事件 'readable'
当流中有可读取的数据块时，就会触发该事件。

```js
var fs = require('fs');

var readableStream = fs.createReadStream('file.txt');

readableStream.on('readable', function(){
  console.log('有数据进来！');
});
```

如果读取到file.txt的数据，那么'readable'就会被触发，在终端上回打印“有数据进来！”

### 事件 'data'
一旦监听了'data'事件，会将流的模式转换成“flowing-mode”，也就是说数据会被尽快读出。

```js
var fs = require('fs');

var readableStream = fs.createReadStream('file.txt');

readableStream.on('data', function(data){
  console.log('data:%s', data);
});
```

### 事件 'end'
当没有可以读数据之后，会触发此事件

```js
var fs = require('fs');

var readableStream = fs.createReadStream('file.txt');

readableStream.on('data', function(data){
  console.log('data:%s', data);
});

readableStream.on('end', function(){
  console.log('所有数据读取完毕！');
});
```

### 事件 'close'
当基础资源被关闭之后，会触发此事件。

### 事件 'error'
当接收数据出错时，触发此事件

```js
var fs = require('fs');

var readableStream = fs.createReadStream('/NotExistPath/file.txt');

readableStream.on('error', function(err){
  console.log('错误:%s', err.message); // 错误:ENOENT, open 'F:\NotExistPath\file.txt'
});
```

### readable.read([size])
显示读取数据（此方法只能在“non-flowing-mode”使用），此方法有一个可选参数size，表示一次要读取的数据。如果不传入size的话，表示读取所有数据，另外如果没有数据的话此方法会返回null。

```js
var fs = require('fs');

var readableStream = fs.createReadStream('file.txt');

readableStream.on('readable', function(){
  console.log('有数据进来！数据：' + readableStream.read()); // 有数据进来！数据：我到底二不二
});
```

### readable.setEncoding(encoding)
设置可读流的读出数据的编码格式，并且会强制将数据转换成字符串

### readable.resume()
恢复推送'data'事件，并且将模式转换成“flowing-mode”

```js
var fs = require('fs');

var readableStream = fs.createReadStream('file.txt');

readableStream.resume();

readableStream.on('end', function(){
  console.log('所有数据读取完毕！');
});
```

### readable.pause()
暂停触发'data'事件

```js
var fs = require('fs');

var readableStream = fs.createReadStream('file.txt');

readableStream.pause();

readableStream.on('data', function(data){
  console.log('data:%s', data); // 此处不会得到数据，因为我们调用了readableStream.pause
});
```

### readable.pipe(destination, [options])
此方法会将可读流里的所有数据读取出来，并写入destination（通常是一个写入流对象）。此方法还有一个可选的配置参数options：

+ end 默认为true，表示数据读取结束之后是否需要关闭写入流。

```js
var fs = require('fs');

var readableStream = fs.createReadStream('file.txt');
var writableStream = fs.createWriteStream('file1.txt');

readableStream.pipe(writableStream);
```

需要注意的是，此方法的返回值还是一个可读的流，所以可以链式调用。

```js
var fs = require('fs');
var zlib = require('zlib');

var readableStream = fs.createReadStream('file.txt');
var writableStream = fs.createWriteStream('file1.txt');
var zStream = zlib.createGzip();

readableStream.pipe(zStream).pipe(writableStream);
```

### readable.unpipe([destination])
此操作为readable.pipe的逆操作，调用之后readable中读取出来的数据块不会再写入destination，destination是可选参数。如果我们没有指定destination，那么readable数据块不会任何一个destination中。

### readable.unshift(chunk)
这个方法比较牛逼，我一开始很不理解，因为它颠覆了我的观念。一般从流中读取是“单向”的，也就是说读出来的数据是不可以再回去的。但是readable.unshift居然能把“吐”出来的东西又“吃”回去（感觉有点恶心），但是有的场景还是很有用的。

### readable.wrap(stream)
此方法是为了兼容老版本而设的，老版本stream没有实现现有的API，我们就可以使用此方法来弥补，这里引用node官方一个例子：

```js
var OldReader = require('./old-api-module.js').OldReader;
var oreader = new OldReader;
var Readable = require('stream').Readable;
var myReader = new Readable().wrap(oreader);

myReader.on('readable', function() {
  myReader.read(); // etc.
});
```

OldReader是老版本的类库，使用wrap方法包裹之后返回的myReader就可以使用当前所有的API了

# stream.Writable
可写流表示数据最终目的地的抽象。

### writable.write(chunk, [encoding], [callback])
写入数据，writable.write有三个参数，第一个参数是需要写入的数据、第二个参数表示写入的编码格式、第三个参数表示写入完成之后的回调。此方法的返回值表示数据是否已经被处理完成，这样就可以判断是否需要继续写入数据。

```js
var fs = require('fs');

var writableStream = fs.createWriteStream('file.txt');

var result = writableStream.write('test write method' , 'utf-8', function() {
  console.log('complete'); // complete
  console.log(result); // true
});
```

### 事件 'drain'
 writable.write返回false，那么数据再次被写入时，事件'drain'就会触发

```js
var fs = require('fs');

var writableStream = fs.createWriteStream('file.txt');

writableStream.on('drain', function() {
  console.log('data is drain!');
});

var result = writableStream.write('big data');

console.log(result); // false
```

以上代码中，如果我们给出的数据足够大，导致一次处理完成不了，我们得到的result将会是false。然后，在剩余数据继续被处理开始时，就会触发'drain'事件。

### writable.end([chunk], [encoding], [callback])
写入数据并且结束流通道（此参数与writable.write方法参数一致，这里就不做重复解释）。

### 事件 'finish'
当调用writable.end方法之后，所有数据都被写入基础系统中，就会触发此事件

```js
var fs = require('fs');

var writableStream = fs.createWriteStream('file.txt');

writableStream.write('test data line1');
writableStream.end('test data line2');

writableStream.on('finish', function() {
  console.log('line1 and line2 finished!');
});
```

### 事件 'pipe'、'unpipe'、'error'

+ pipe 调用readable.pipe到此可写流时触发
+ unpipe 调用readable.unpipe到此可写流时触发
+ error 写入或者pipe数据时出错时，触发此事件（是可读流的'error'事件类似）

# 总结
stream在整个node的核心模块运用都非常广泛，例如：http模块、File System模块等。这说明此模块地位非常重要，并且当我们需要处理大文件时，通常第一个想到的也是stream模块。所以，少年！好好学习，天天向上吧！另外本文也会在我的[博客](https://www.sunweifeng.cn/nodejs-stream-chapter1/)上同步更新，欢迎访问。
