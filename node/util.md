##### 引子
话说大家在做框架的时候，除了引用大量的引入第三方类库之外，不免自己要实现一些。例如：判断对象是否是某个类型等。其实node也提供了一部分实用的函数（需要引入“util”模块），下面我们就来一一介绍。

##### util.format(format, [...])
此方法与C语言中的prinf类似，第一个参数是一个字符串，后面是可变参数（可以有任意多个参数）

+ %s String 可变参数对应位置是字符串。

```js
var util = require('util');

console.log(util.format('%s%s', '你好！')); // 你好！%s
console.log(util.format('Hello %s', 'World!')); // Hello World!
```

+ %d Number 可变参数对应位置是数字（整型或浮点型）

```js
var util = require('util');

console.log(util.format('你的幸运数字是%d', 6)); // 你的幸运数字是6
```

+ %j JSON 可变参数为JSON对象

```js
var util = require('util');

console.log(util.format('这个接口需要的参数为：%j，正确的返回结果为：%j', {type: 'pet'}, {status: 200})); // 这个接口需要的参数为：{"type":"pet"}，正确的返回结果为：{"status":200}
```

需要注意的是，如果第一个参数不是拥有占位符的字符串，那么此方法会把所有参数拼接成一个字符串，并且每个参数之间有空格分开

```js
var util = require('util');

console.log(util.format(1,2,3); // 1 2 3
```

##### util.debug(string)
在打印的字符串之前添加“DEBUG:”字样，就如我在[Node.js之console](console.md)一文中提到的一样，util.debug会将输出的信息记录到错误日志文件中去，如果没有指定错误文件，那么就会直接输出到控制台

```js
var util = require('util');

util.debug('这是一条调试信息'); // DEBUG: 这是一条调试信息
```

##### util.error([...])
与util.debug一样，日志会记录到错误日志文件中去，但是不带“DEBUG:”，并且可以记录多个参数

```js
var util = require('util');

util.error('error1', 'error2',  'error2'); // error1
                                           // error2
                                           // error2
```

##### util.puts([...])、util.print([...])
util.puts与util.print功能一致，会将日志信息记录到正常的日志文件中，两者唯一的区别在于：util.puts会在新行输出每一个参数，而util.print将所有参数输出到一行。

```js
var util = require('util');

uti.puts('arg1', 'arg2', 'arg3'); // error1
                                  // error2
                                  // error2
console.log('======优雅的分割线========='); // ======优雅的分割线=========
uti.print('arg1', 'arg2', 'arg3'); // arg1arg2arg3
```

##### util.log(string)
从node源码可以看出util.log在内部内部其实就是调用了console.log方法，只不过在日志前加入了时间点。

```js
function timestamp() {
  var d = new Date();
  var time = [pad(d.getHours()),
              pad(d.getMinutes()),
              pad(d.getSeconds())].join(':');
  return [d.getDate(), months[d.getMonth()], time].join(' ');
}

// log is just a thin wrapper to console.log that prepends a timestamp
exports.log = function() {
  console.log('%s - %s', timestamp(), exports.format.apply(exports, arguments));
};
```

以上是util模块的node源码：timestamp是产生时间字符串的函数，当我们调用util.log时，timestamp()产生的字符串会被填充到原日志的前面（更多的模块源码，我以后会有专门的博客介绍）。下面我们来看下util.log(string)用法与效果：

```js
var util = require('util');

util.log('我是一条日志！'); // 29 Feb 15:50:34 - 我是一条日志！
```

##### util.inspect(object, [options])
这个函数我们在[Node.js之console](console.md)一文中也提到过，功能我就不多做介绍。第一个参数应该没什么异议，第二个参数是可选配置项，通过传入不同的option值，会有不同的效果，下面就来介绍下这个option：

+ showHidden表示是否要返回不可枚举选项，我们在[JavaScript之object](../object.md)提到过，object对象的属性可以设置为“不可枚举”，这样我们的for...in结构就不能访问了。这里也是一样，showHidden默认是false，也就是说使用util.inspect时，默认不返回不可枚举属性的：

```js
var util = require('util');

var obj = Object.create({}, {
  'enumerable_prop':{
    value: 'a',
    enumerable: true
  },
  'non_enumerable_prop':{
    value: 'b',
    enumerable: false
  }
});

var strObjShow = util.inspect(obj, {showHidden: true});
var strObjHide = util.inspect(obj, {showHidden: false});

console.log('strObjShow is %s', strObjShow); // strObjShow is { enumerable_prop: 'a', [non_enumerable_prop]: 'b' }
console.log('strObjHide is %s', strObjHide); // strObjHide is { enumerable_prop: 'a' }
```

+ depth表示深度，我们在[Node.js之console](console.md)介绍过，这里就不再赘述。

+ colors 可以在控制台输出有颜色的对象（并且颜色是可以自定义的）

```js
var util = require('util');

console.log(util.inspect({a:1, b:'123'}, {colors: true})); // { a: 1, b: '123' }
```

以上代码的实际效果如下：

![inspect-colors](../file/inspect_colors.PNG)

+ customInspect如果我们在对象像定义inspect方法，那么util.inspect就会直接调用该对象的inspect方法，如果将customInspect设置为false，则将依旧调用node定义的方法：

```js
var util = require('util');

var obj = {
  name: '张三',
  inspect: function(){
    return 'inspect is called!';
  }
}

console.log(util.inspect(obj)); // inspect is called!
console.log(util.inspect(obj, {customInspect: false})); // { name: '张三', inspect: [Function] }
```

##### util.isArray(object)、util.isRegExp(object)、util.isDate(object)、util.isError(object)
这几个方法从英文语义上就可以知道功能了，我就不多述了。（需要提到的是在以前，判断一个对象是否是数组，很长时间没有人给出优雅的解决方法。直到有人利用调用原型上的方法才给出了优雅的解决方案，这就告诉我们，群众的力量是无穷的）。

##### util.pump(readableStream, writableStream, [callback])
这个方法在v0.10.42时代就被废弃了，就不介绍了。

##### util.inherits(constructor, superConstructor)
虽然我们可以手动做JavaScript的“继承”，但是node还是给出了实用的工具函数。但是我在这里不做介绍，我会把此方法放到events模块一起介绍。

##### 总结
总的来说，介绍API是比较枯燥的（我也觉得）。大家可以作为知识点学习学习，熟悉熟悉，为以后工作中遇到的问题打打基础。另外之前我在[v站](http://v2ex.com)上发的帖子很多人回复，非常感谢大家给我的建议，我会坚持写技术博客的。
