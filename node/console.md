##### 调式
众所周知，node的调试是一个令人头疼的工作，著名的node大神TJ也抱怨过。由此见得，想要像.NET那样方便的调试是很困难的（微软最近出了VSCode倒是可以很好的调式，有时间可以试试）。其实node最快捷、方便的调试方式还是控制台输出日志信息。今天我们就来研究下如何使用node中的console模块，console模块是大多数noder的调试神器。

##### console.log
console.log（console是个全局变量）是node最常用的一个日志打印函数，此函数与C和C++中的printf很像，可以设置占位符，并且与前2者一致。

```js
console.log('娜塔莎是个%s', '好孩子'); // 娜塔莎是个好孩子

var info = {
  status: 200,
  msg: 'everything is ok'
}

console.log(info); // { status: 200, msg: 'everything is ok' }
```

console.log的参数是可变的，可以有很多个，就像这样：

```js
var a = 'hello Peter';
var b = 'hi Lily';

console.log(a, b); // hello Peter hi Lily
```

这样就可以打印多个变量或者表达式的结果，当我们在调试程序的时候（特别是需要同时打印多个变量的值）特别有用。就如上面演示的一样，console.log方法可以打印数字、字符串、对象。可以说console.log方法非常强大，但是它不是万能的，当我们打印的对象的树层次太深的话就会出现意料之外的结果。

```js
var info = {
  status: 200,
  msg: {
    user:{
      name: 'Peter',
      country: 'Japan',
      company: {
        name: '全球无限皮包公司',
        ceo: {
          name: 'Lucy'
        }
      }
    }
  }
}

console.log(info); // { status: 200,  msg: { user: { name: 'Peter', country: 'Japan', company: [Object] } } }
```

可以看到，当我们打印info变量时，到第四层级“company”只打印出了一个[object]。说明console.log在打印object的时候，最多只支持到第三层级，如果需要打印对象所有层级的话，我们可以与util.inspect方法配合使用（之后我们讲到util模块的时候会讲到这个方法）。

```js
var util = require('util');

var info = {
  status: 200,
  msg: {
    user:{
      name: 'Peter',
      country: 'Japan',
      company: {
        name: '全球无限皮包公司',
        ceo: {
          name: 'Lucy'
        }
      }
    }
  }
}


console.log(util.inspect(info, {depth: null})); // { status: 200,
                                                //   msg:
                                                //   { user:
                                                //      { name: 'Peter',
                                                //        country: 'Japan',
                                                //        company: { name: '全球无限皮包公司',
                                                //                   ceo: { name: 'Lucy'
                                                //                   }
                                                //        }
                                                //      }
                                                //   }
                                                // }
```

此时，我们就可以打印出info中所有的层级了。

##### console.error
此方法如果我们没有在运行时指定输出文件（包括一般输出和错误输出），在我们运行node的时候可以手动指定日志和错误的输出文件，console.error方法就会输出到错误文件中。

```js
node app.js 2> error.log | tee info.log
```

运行了以上代码，我们的错误信息都会记录到error.log文件中，普通日志信息都会记录到info.log文件中。如果我们在运行的时候没有指定输出文件，那么所有的信息都会输出到控制台，这时我们调用console.log和console.error是一样的效果。

##### console.warn、console.info
console.warn的实现与console.error是一样的，console.info的实现与console.log是一样的。

##### console.dir
console.dir方法就是与我们在上面介绍的util.inspect方法的效果一致。

```js
var info = {
  status: 200,
  msg: {
    user:{
      name: 'Peter',
      country: 'Japan',
      company: {
        name: '全球无限皮包公司',
        ceo: {
          name: 'Lucy'
        }
      }
    }
  }
}


console.dir(info); // { status: 200,  msg: { user: { name: 'Peter', country: 'Japan', company: [Object] } } }
```

##### console.time、console.timeEnd
一般console.time与console.tiemEnd成对出现，用于记录两者之间的代码运行时间

```js
console.time('for-time');

for(var i = 0;i < 10000000;i++){
  ;
}

console.timeEnd('for-time'); // for-time: 173ms
```

以上代码说明，在我的电脑上循环0到10000000需要耗费173毫秒。

##### console.trance
此方法会输出到错误文件中，在日志前添加“Trace:”字符串。

##### console.assert
此方法是一个断言函数，第一个参数是布尔值，如果第一个参数的结果为真，那么什么事情都不做；如果第一个布尔值为假，那么系统就会抛出错误，并且把后面的参数打印出来，并且会打印出调用栈。

#### 总结
console模块是一个非常简单地模块，所有的API我都已经解释或者演示过了。非常容易理解，这些方法在我们平常的开发过程中是很常见的，给noder提供了单接单并不唯一的调试node程序的方法（还有其他调试方法：node-inspector、vscode等，我以后有机会也会开博演示）。
