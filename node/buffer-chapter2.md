##### buf.toJSON()
将Buffer实例转换成JSON形式的数组。

```js
var bfJson = new Buffer('hello');

console.log(bfJson.toJSON()); // [ 104, 101, 108, 108, 111 ]

console.log(new Buffer(bfJson.toJSON()).toString()); // hello
```

##### Buffer.isBuffer(obj)
判断对象是否是Buffer实例

```js
console.log(Buffer.isBuffer({})); // false
console.log(Buffer.isBuffer(new Buffer(3))); // true
console.log(Buffer.isBuffer(new Buffer('test'))); // true
console.log(Buffer.isBuffer('world')); // false
console.log(Buffer.isBuffer(123)); // false
```

##### Buffer.byteLength(string, [encoding])
获取字符串的字节数

```js
console.log(Buffer.byteLength('hello')); // 5
console.log(Buffer.byteLength('你好')); // 6
```

##### Buffer.concat(list, [totalLength])
拼接Buffer实例

```js
var buf1 = new Buffer('小明');
var buf2 = new Buffer('今年');
var buf3 = new Buffer('5');
var buf4 = new Buffer('岁');

var bufList = [];

bufList.push(buf1);
bufList.push(buf2);
bufList.push(buf3);
bufList.push(buf4);

var bufTotal = Buffer.concat(bufList, buf1.length + buf2.length + buf3.length + buf4.length);

console.log(bufTotal.toString()); // 小明今年5岁
```

##### buf.copy(targetBuffer, [targetStart], [sourceStart], [sourceEnd])
将Buffer实例拷贝到目标对象上

```js
var bfSource = new Buffer('可可是个乖宝宝');
var bfTarget = new Buffer(bfSource.length);

bfSource.copy(bfTarget, 0, 0, bfSource.length);

console.log(bfTarget.toString()); // 可可是个乖宝宝
```

##### buf.slice([start], [end])
取Buffer实例片段（不是将原始数据中片段复制出来，而是直接引用。所有当原数据改变时，取出的片段也将随之改变）

```js
var bufFull = new Buffer('abcde', 'ascii');
var bufSlice = bufFull.slice(2, 3);

console.log(bufSlice.toString('ascii')); // c

bufFull[2] = 97;

console.log(bufSlice.toString('ascii')); // a
```

上面的代码中，我们在调用slice之后，又改变了bufFull的第三个元素值，之后我们打印bufSlice发现bufSlice中的值也随之改变了。

##### buf.readUInt8(offset, [noAssert])
读取一个8位的无符号整型

```js
var bfRead = new Buffer(1);

bfRead[0] = 0x12;

console.log(bfRead.readUInt8(0)); // 18

console.log(bfRead.readUInt8(1, true)); // undefined

console.log(bfRead.readUInt8(1)); // 程序出错
```

buf.readUInt8只读取一个8位无符号整数，并且可以指定读取的位置。第二个参数（noAssert）表示如果读取位置超过Buffer实例的长度时是否抛出错误，默认为false。如果为false，即使超过Buffer.length，系统也不会抛错，这是之后返回undefined。

##### 其他的read方法

+ buf.readUInt16LE(offset, [noAssert])
+ buf.readUInt16BE(offset, [noAssert])
+ buf.readUInt32LE(offset, [noAssert])
+ buf.readUInt32BE(offset, [noAssert])
+ buf.readInt8(offset, [noAssert])
+ buf.readInt16LE(offset, [noAssert])
+ buf.readInt16BE(offset, [noAssert])
+ buf.readInt32LE(offset, [noAssert])
+ buf.readInt32BE(offset, [noAssert])
+ buf.readFloatLE(offset, [noAssert])
+ buf.readFloatBE(offset, [noAssert])
+ buf.readDoubleLE(offset, [noAssert])
+ buf.readDoubleBE(offset, [noAssert])

以上方法与buf.readUInt8类似，就不多做赘述。

##### buf.writeUInt8(value, offset, [noAssert])
将值写入指定Buffer实例的指定位置

```js
var bfWrite = new Buffer(1);

bfWrite.writeUInt8(0x12, 0);

console.log(bfWrite); // <Buffer 12>
```

##### 其他的write方法

+ buf.writeUInt16LE(value, offset, [noAssert])
+ buf.writeUInt16BE(value, offset, [noAssert])
+ buf.writeUInt32LE(value, offset, [noAssert])
+ buf.writeUInt32BE(value, offset, [noAssert])
+ buf.writeInt8(value, offset, [noAssert])
+ buf.writeInt16LE(value, offset, [noAssert])
+ buf.writeInt16BE(value, offset, [noAssert])
+ buf.writeInt32LE(value, offset, [noAssert])
+ buf.writeInt32BE(value, offset, [noAssert])
+ buf.writeFloatLE(value, offset, [noAssert])
+ buf.writeFloatBE(value, offset, [noAssert])
+ buf.writeDoubleLE(value, offset, [noAssert])
+ buf.writeDoubleBE(value, offset, [noAssert])

以上方法与buf.writeUInt8类似，就不多做赘述。

##### buf.fill(value, [offset], [end])
向Buffer实例中填充指定的值

```js
var bfFill1 = new Buffer(1);
var bfFill2 = new Buffer(6);

bfFill1.fill(97, 0, 1);

console.log(bfFill1.toString()); // a

bfFill2.fill(97);

console.log(bfFill2.toString()); // aaaaaa
```

如果不指定offset和end，那么fill方法会把当前Buffer实例填满。

##### 总结
好了，Buffer模块介绍到这里就介绍完了。此模块的方法很多，但是大多数都是读取和写入的方法，形式都差不多。只要我们掌握了一个，其他的稍稍看下API就可以轻松使用了。