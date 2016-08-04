# 前言
前面2个部分，分别介绍了Stream的基础使用和自定义进阶。为了更好的帮助大家理解这个模块的，node官方还有第三部分，可以使我们进一步理解Stream模块的内部实现原理

# 缓冲
可读流和写入流各自都有一个缓冲区，分别存放在内部对象\_writableState和\_readableState的'buffer'属性中。在可读流中，自定义实现时我们可以通过stream.push方法将数据推入此属性中（'buffer'属性），在通过stream.read方法来消费缓冲区的内容；在写入流中，我们通过stream.write方法将我们的数据块写入到'buffer'属性中

### stream.read(0)
当我们不需要读取流中内容，但是又需要触发内部刷新机制。这时，我们可以在调用stream.read方法时传入参数0，此调用的返回值永远是null并且不会消耗任何缓冲信息。这种调用的应用场景并不常见，但是在node源码中确实有这样的调用方式。

### stream.push('')
当我们调用stream.push的时候传入空字符串或者空Buffer时，会产生一个副作用：终止读取过程。

```js
var Readable = require('stream').Readable;
var util = require('util');
util.inherits(Prefix, Readable);

function Prefix(pre, opt) {
  Readable.call(this, opt);

  this._prefix = pre || 'node:';
  this._count = 0;
  this._max = 9;
}

Prefix.prototype._read = function() {

  if (this._count > this._max) {
    this.push(null);
  } else if (this._count === 2) {
    this.push('');
  } else {
    this.push(new Buffer(this._prefix + this._count.toString()));
  }

  this._count++;
}

var preObj = new Prefix('node v6:');

console.log(preObj.read().toString()); // node v6:0
console.log(preObj.read().toString()); // node v6:1
console.log(preObj.read().toString()); // TypeError: Cannot call method 'toString' of null
```

可以看到当我们往缓冲区中push了空字符串之后，会导致调用read方法返回null值。其实，需要用到此场景的地方极其少，如果非要使用stream.push('')，最好还是先考虑另择他法。

# 兼容老版本
在node v0.10之前，可读流（Readable）接口非常简单，功能也简单:

+ 老版本中不会等待read方法调用之后才会输出数据，而是会立即触发'data'事件，开始推送数据。
+ pause方法只是建议型的方法，不作出任何数据保证。我们还是需要使用'data'事件来接收数据，即使当前状态是'暂停状态'

为了兼容老版本，node v0.10中一旦我们添加了'data'事件的处理函数或者调用了pause或resume方法，此时stream会切换至flowing-mode，数据会尽快推送至'data'事件中。而在node v0.10我们就再也不用担心数据丢失的问题了。

# Object Mode
当我们在创建一个stream实例的时候，将配置参数objectMode设置为true时，此时流就可以推送JavaScript对象（一般情况下流是专注于处理String和Buffer）。当我们处于obejct mode模式下，stream.read(size)中的size参数会被忽略，而每次读取一个单独的值、stream.write(data, encoding)则会忽略编码。

# State Objects
一般的，可读流在内部会有一个内置的对象：\_readableState、而写入流也会相应的有一个内置对象：\_writableState。当我们在自定义子类的时候，应该避免修改这2个内置对象，否则会导致不可预料的后果。

#总结
这一篇基本是关于流模块的一些收尾工作以及对于流的进一步认识。进一步了解流内部的机制以及实现原理，可以更好的帮助我们在以后用到此模块的时候更加得心应手。好了，关于流的介绍到这里就画上一个句号。
