##### JavaScript不足
JavaScript原生支持Unicode，并且对Unicode非常友好。但是，纯JavaScript对于处理二进制数据却是一个弱项。但是node既然是后端，那么不免要和二进制数据打交道，这样Buffer类就应运而生了。

##### 全局变量
与一般的node模块不同，Buffer在node中是全局存在的（这也说明了Buffer模块在node中的重要性）。既然Buffer是全局变量，那么我们不需要显示的引用，直接使用即可：

```js
var bfData = new Buffer(4);
```

##### 格式转换
我们在Buffer与JavaScript字符串之间转换时，需要显示的指定格式。当前支持的格式有：

+ ascii 目前只支持7比特的ASCII数据（需要注意的是，在此编码中，node会把null转换成空格）
+ utf8
+ utf16le、ucs2
+ base64
+ hex

```js
var bfASCII = new Buffer('abcd', 'ascii');
console.log(bfASCII); // <Buffer 61 62 63 64>
console.log(bfASCII.toString()); // abcd

var bfHex = new Buffer('b4c127c3d7', 'hex');
console.log(bfHex); // <Buffer b4 c1 27 c3 d7>
console.log(bfHex.toString('hex')); // b4c127c3d7
```

##### 创建一个Buffer实例
我们可以有3中创建Buffer实例的方法。

+ new Buffer(size) 创建指定大小的Buffer实例

```js
var bfSize = new Buffer(6);

bfSize.write('abcdef', 0, 6, 'utf8');

console.log(bfSize); // <Buffer 61 62 63 64 65 66>
console.log(bfSize.toString('utf8')); // abcdef
```

+ new Buffer(array) 给定指定数组创建Buffer实例

```js
var bfArray = new Buffer(['97', 98]);

console.log(bfArray.toString('utf8')); // ab
```

+ new Buffer(str, [encoding]) 使用指定的字符串和编码格式（默认为utf8）创建新的Buffer实例

```js
var bfStr = new Buffer('你好', 'utf8');

console.log(bfStr); // <Buffer e4 bd a0 e5 a5 bd>
console.log(bfStr.toString('utf8')); // 你好
```

##### Buffer.isEncoding(encoding)
此方法可以方便的知晓给定的字符编码是否在node支持范围内：

```js
console.log(Buffer.isEncoding('utf8')); // true
console.log(Buffer.isEncoding('ascii')); // true
console.log(Buffer.isEncoding('hex')); // true
console.log(Buffer.isEncoding('base64')); // true
console.log(Buffer.isEncoding('ucs2')); // true
console.log(Buffer.isEncoding('binary')); // true
console.log(Buffer.isEncoding('utf108')); // false
```

##### buf.write(string, [offset], [length], [encoding])
将指定字符串写入到Buffer实例，并且可以指定从偏移量、写入长度和编码

```js
var bfWrite = new Buffer(6);

bfSize.write('abcdef', 0, 5, 'utf8');

console.log(bfWrite); // <Buffer 61 62 63 64 65 00>
console.log(bfWrite.toString('utf8')); // abcde
```

##### buf.toString([encoding], [start], [end])
将Buffer实例转换成字符串

```js
var bfToStr = new Buffer(2);

bfToStr[0] = 108;
bfToStr[1] = 109;

console.log(bfToStr.toString('ascii')); // lm
```

##### 未完待续
本文也会在我的[博客](https://www.sunweifeng.cn/node-buffer-chapter1)上同步更新，欢迎访问。
