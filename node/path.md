##### 介绍
path模块其实就是对路径的处理，而路径实际就是字符串。那么，说到底path模块就是对字符串的一些特殊处理。这个模块非常简单，拥有很少的API，但是非常实用，在实际开发过程中，如果没有了解path模块或者不够熟悉的话，可能会走很多弯路（这也说明了基础API的重要性）。

##### path.normalize(p)
我们知道，一般而言在程序中“..”和“.”表示上一级文件夹路径和当前路径。而path.normalize方法就是专注于处理这两个符号，并返回对应的路径。

```js
var path = require('path');

console.log(path.normalize('/a/b/./c')); // \a\b\c
console.log(path.normalize('/a/b/../c')); // \a\c
console.log(path.normalize('/a/b/c/../')); // \a\b\
console.log(path.normalize('/a/b//c')); // \a\b\c
console.log(path.normalize('/a/b/c/.../')); // \a\b\c\...\
```

根据以上代码我们可以看出，此方法只是对“..”和“.”进行处理，并且返回处理结果。需要注意的是，如果是多个斜杠（比如上述代码第四个log），path.normalize会把其处理成一个斜杠。

##### path.join([path1], [path2], [...])
把多个路径参数拼接成一个路径，并且会用path.normalize处理拼装结果。

```js
var path = require('path');

console.log(path.join('/a', '/n', '/b', '/h')); // \a\n\b\h
console.log(path.join('/a', '/n', '..', '/h')); // \a\h
console.log(path.join('/a', '/../../', '/h')); // \h
```

path.join的所有参数都是字符串，但是如果你非要捣蛋的话（比如我传入一个object），那会出现什么情况呢。

```js
var path = require('path');

console.log(path.join('/a', {}, '/h')); // throw new TypeError('Arguments to path.join must be strings');
```

好吧，报错了！看来还是不能瞎搞的！

##### path.resolve([from ...], to)
这个方法在官方有两种解释，第一种解释我噼里啪啦讲了半天都没看懂，然后看到下面才发现有第二种解释（简单容易理解）。

```js
var path = require('path');

console.log(path.resolve('/foo/bar', '/bar/faa', '..', 'a/../c')); // C:\bar\c
```

以上的代码就相当于一堆shell脚本

```
cd /foo/bar
cd /bar/faa
cd ..
cd a/../c
pwd
```

代码与shell脚本的区别在于，代码所针对的路径不一定要存在并且可以是文件：

```js
var path = require('path');

console.log(path.resolve('/foo/bar', '/bar/faa', '..', 'a/../c.txt')); // C:\bar\c.txt
```

##### path.relative(from, to)
返回从from到to的相对路径，比如我们要在from对应的文件夹中引用to文件夹中的文件，那么我们就可以使用这个方法，例如：

首先我们有一个test文件夹，里面有一个文件：test.js（F:\jsresearch\test\test.js）

```js
function showLevel(){
  return 'The level is 5';
}

exports.showLevel = showLevel;
```

然后我们在与test文件夹平级的nodejs文件夹中引用test.js文件，我们可以这么做

```js
var path = require('path');
var test = require(path.relative(__dirname, 'F:\\jsresearch\\test\\test.js'));

console.log(test.showLevel()); // The level is 5
```

当我们调用showLevel方法时，就可以正常调用了。其实以上的代码等价于：

```js
var path = require('path');
var test = require('../test/test.js');

console.log(test.showLevel()); // The level is 5
```

##### path.dirname(p)
获取指定路径的目录

```js
var path = require('path');

console.log(path.dirname('/ab/cd/a.txt')); // /ab/cd
console.log(path.dirname('/ab/cd/a')); // /ab/cd
console.log(path.dirname('/ab/cd/a/')); // /ab/cd
```

##### path.basename(p, [ext])
获取指定路径文件的文件名

```js
var path = require('path');

console.log(path.basename('/ab/c.txt')); // c.txt
console.log(path.basename('/ab/c.txt', '.txt')); // c
console.log(path.basename('/ab/c.txt', 'txt')); // c.
console.log(path.basename('/ab/c.txt', 'xt')); // c.t
```

第一个参数表示路径，第二个参数表示需要除去的结尾字符。举个栗子：如果第二个参数是“xt”那么我们得到的结果就会删除“xt”字符。

##### path.extname(p)
获取对应文件的扩展名

```js
var path = require('path');

console.log(path.extname('/ab/c.txt')); // .txt
console.log(path.extname('/ab/c.html')); // .html
console.log(path.extname('/ab/c.md')); // .md
console.log(path.extname('/ab/c.')); // .
```

##### path.sep、path.delimiter
+ path.sep 路径分隔符号，一般在Windows是反斜杠，在linux是斜杠
+ path.delimiter 路径分割符号，一般在Windows是分号，在linux是冒号，用于分割多个路径

##### 总结
path模块非常简单，此模块专注于路径（也就是字符串）的特殊处理。
