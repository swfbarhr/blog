# 前言
[上一篇](https://github.com/swfbarhr/blog/blob/master/node/stream-chapter1.md)博客中，我们介绍了stream的基本用法，今天我们来讲讲stream接口如何实现，之后我们就可实现自己的stream类了。

# stream.Readable
简单来说，如果需要自定义stream.Readable类，那么我们需要做的就是一件事：重写_read方法。

```js
var Readable = require('stream').Readable;
var util = require('util');
util.inherits(Prefix, Readable);

function Prefix(pre, opt){
  Readable.call(this, opt);

  this._prefix = pre || 'node:';
  this._count = 0;
  this._max = 9;
}

Prefix.prototype._read = function(){

  if(this._count++ >this._max) {
    this.push(null);
  } else {
    this.push(new Buffer(this._prefix + this._count.toString()));
  }
}

var preObj = new Prefix('node v6:');

console.log(preObj.read().toString()); // node v6:1
console.log(preObj.read().toString()); // node v6:2
console.log(preObj.read().toString()); // node v6:3
```

### new stream.Readable([options])
当我们实现了Readable，就可以使用new来创建一个实例对象，我们可以传入一个可选的options配置参数。options有3项：

+ highWaterMark 表示内部buffer中最大的字节数
+ encoding 如果指定了此参数，buffer会被按照此格式转换成字符串
+ objectMode 如果此参数设置为true，当我们调用read方法时，每次只能读取一个值即使我们传入了size参数

以上面的Prefix为例：

```js
var preObj1 = new Prefix('node v6:');

console.log(preObj1.read()); // <Buffer 6e 6f 64 65 20 76 36 3a 31>
console.log(preObj1.read(1).toString()); // n
console.log(preObj1.read().toString()); // ode v6:2node v6:3

console.log('---------------超华丽分割线----------------');

var preObj2 = new Prefix('node v6:', {highWaterMark:10, encoding:'utf8', objectMode:true});

console.log(preObj2.read()); // node v6:1
console.log(preObj2.read(8)); // node v6:2
console.log(preObj2.read()); // node v6:3
```

通过对比可以看出：

+ 当我们没有设置encoding时，调用read方法返回的是buffer对象
+ 当我们设置objectMode为true时，每次只返回单个值（不可以根据size大小返回值）

### readable._read(size)
这是一个接口方法，此方法是用来实现的并且不可以直接调用（一般默认以下划线开始的方法为内部方法）。具体用法在上面第一个代码示例中已经展示，这里就不多赘述。

### readable.push(chunk, [encoding])
此方法是向读取队列中添加数据，当我们插入null时，表示数据插入完成。此方法有2个参数：

+ chunk chunk参数接收Buffer或者String或者null

+ encoding 表示当chunk为String时的编码格式

```js
var Readable = require('stream').Readable;
var util = require('util');
util.inherits(PushTest, Readable);

function PushTest(options) {
  Readable.call(this, options);

  // 假设souce对象一有数据就会调用data方法
  // 并且数据全部消费完之后调用end方法
  this._source = source;

  this._source.data = function(chunk) {
    if (!self.push(chunk)){
      this._source.readStop();
    }
  }

  this._source.end = function(){
    this.push(null);
  }
}

PushTest.prototype._read = function(size) {
  this._source.readStart();
}
```

# stream.Writable
与stream.Readable相似，stream.Writable也是一个抽象类，其中Writable.\_write方法不可以直接被调用，Writable.\_write需要被实现。

### new stream.Writable([options])
与stream.Readable一样，我们自定义的写入流类需要使用new来创建一个实例，当我们创建实例的时候，需要传入一个可选的options，此options同样有3个配置项：

+ highWaterMark 表示写入的缓冲区大小，我们调用write方法时单次写入的大小如果超过设定的阈值，就会返回false，highWaterMark就是这个阈值（默认大小16kb）
+ decodeStrings 在写入之前（调用内部_write之前），是否将字符串转成Buffer（默认为true）
+ objectMode 是否可以写入任意类型的数据，而不仅仅是Buffer或String（默认为false）

### writable._write(chunk, encoding, callback)
同样的，如果我们需要实现自定义的写入流，我们需要实现一个内部接口，此接口在node源码中是这样的：

```js
Writable.prototype._write = function(chunk, encoding, cb) {
  cb(new Error('not implemented'));
};
```

由此可知，如果我们需要实现自定义的写入流（可读流也是一样），就必须实现\_write方法，否则就会报错。\_write有三个参数：

+ chunk 需要写入的数据
+ encoding 如果写入数据是字符串，那么此参数表示该数据的编码格式
+ callback 回调函数（数据写入完成后触发）

```js
var util = require('util');
var Writable = require('stream').Writable;
var fs = require('fs');

util.inherits(CustomFS, Writable);

function CustomFS(path, options) {
  Writable.call(this, options);

  this._path = path;
}

CustomFS.prototype._write = function(chunk, encoding, cb) {

  // 判断文件是否存在
  fs.exists(this._path, function(exists) {

    if (exists) {
      fs.writeFile(this._path, +new Date() + ':' + chunk + '\n', cb);
    } else {
      cb('找不到文件！');
    }
  }.bind(this));
}

// 创建实例
var writeFile = new CustomFS('./file.txt', {
  highWaterMark: 1024
});

// 写入数据
writeFile.write('窗前明月光，地上鞋两双！', function(err){
  if(err) {
    return console.log('报错啦！错误：' + err.message);
  }

  console.log('写入完成！');
});
```

以上代码在每次写入文本之前，都会在最前面加入时间戳。

# stream.Duplex类
“复杂”类型的流，在我们实现它时，既要实现可读流的\_read方法，又要实现写入流的\_write方法。也就是说，只有同时实现了可读、可写两个功能，才可以叫做“复杂”型流即'duplex'。具体可读和写入流的自定义实现代码我们在上面已经介绍过了，这里就不多讲。

# stream.Transform类
'transform'类型的流属于“复杂”类型，但是我们不需要实现\_read和\_write，但是我们要实现另一个方法：_transform。'transform'类型的流的特点在于输入决定输出。就像crypto（加密模块）一样，输入决定了输出（比如哈希值）。

### new stream.Transform([options])
当然，如果要创建一个'transform'流的实例，还是要通过new关键字（这样可以保证正确的初始化实例）。同样，新建实例时也需要传入配置参数，此配置参数就是只读流和写入流配置项的和。

### transform._transform(chunk, encoding, callback)
'_transform'是我们需要实现一个自定义'transform'流的必须要实现的一个方法，其参数意义与“写入流”的_write方法类似。

### transform._flush(callback)
此方法会在调用end方法之后，但是在触发'end'或'finish'事件之前调用，便于我们做一些事情。一下是一个'transform'类型流例子：

```js
var util = require('util');
var Transform = require('stream').Transform;

util.inherits(Trans, Transform);

function Trans(options) {
  Transform.call(this, options);
}

Trans.prototype._transform = function(chunk, encoding, callback) {

  this.push(new Buffer('Transform: ' + chunk.toString()));

  callback();
}

Trans.prototype._flush = function(callback) {
  console.log('flushed!');
  callback();
}

var myTrans = new Trans();

myTrans.write('你好，小强！', function(err) {
  if (err) {
    return console.log('报错啦！错误：' + err.message);
  }

  console.log('写入完成！');
  console.log(myTrans.read().toString());
  myTrans.end();
});

myTrans.on('finish', function() {
  console.log('finished!');
});
```

# 总结
关于stream第二部分，到这里就算是介绍完了。自定义的stream类只需要我们按照实现规则，去丰富不同的内部方法而已。这些对于混迹于JavaScript界的老司机来说，是一件非常轻松的事情，但是我们需要理解stream并融会贯通之后再来做这些事情的话，又会有另一层理解。另外本文也会在我的[博客](https://www.sunweifeng.cn/nodejs-stream-chapter2/)上同步更新，欢迎访问。
