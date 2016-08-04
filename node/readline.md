##### 前言
近一段时间工作太忙，并且在研究docker相关的技术，好久没更新博客了。今天趁任务完成的早，在公司补上一篇。今天要讲的这个模块，让我瞬间就回到了刚学C#的时候，做一些小程序逗同学的年代。今天我要介绍的模块是Readline（记得C#考试的时候Console.ReadLine这个方法我是运用了我的英语知识拼出来的，现在想想都是泪呀！）

##### Readline
这个模块可以读取stream，比如说process.stdin。以前对stdin也是不太理解，最近研究docker就对stdin概念稍稍了解了一下下。std是standard（标准）的缩写，in就是输入，组合起来解释就是标准输入，而stdout我的理解就是标准输出（见识浅薄，大神勿喷）。而Readline可以读取process.stdin，我们就可以利用这一点，使得程序通过控制台的标准输入输出与用户进行简单的交互。

```js
var readline = require('readline');

var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.question('你是2B吗？', function(answer){

  if(answer === '是'){
    console.log('好吧，再见！2B！');
    rl.close();
  }

  if(answer === '不是'){
    console.log('no no no，显然你就是2B，有本事来咬我呀！咬我呀！咬我呀！');
  }
});
```

好吧，我承认代码写的是有点逗逼（以前也经常这样逗同学）。但是需要注意的是，如果我们激活了Readline模块，并且使用了它，node程序是不会自动退出的（除非我们使用代码，比如上面的rl.close()或者直接关闭控制台）。

##### readline.createInterface(options)
创建一个readline的Interface实例，使用Interface实例，我们就可以与用户交互了。此方法有一个参数，此参数是一个配置项：

+ input 设置需要监听哪个可读流，例如：process.stdin

+ output 设置需要将可写流写入到哪里

+ completer 自动完成函数

+ terminal 如果设置为true则把输入输出看做一个虚拟终端

输入、输出就不多讲了，我们把输入输出分别设置成process.stdin和process.stdout。先来讲讲completer

```js
var readline = require('readline');

var completer = function(line){
  var completions = ['Hello', 'World', 'You', 'are', 'man'];

  var shots = completions.filter(function(s){
    return s.indexOf(line) == 0;
  });

  return [shots, line];
}

var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  completer: completer
});
```

运行以上代码，当我们输入'H'然后按TAB键时，就会自动补全'Hello'这个单词。

##### rl.setPrompt(prompt, length)
此方法包含两个参数，第一个参数表示命令行最前面的“前缀”，就像我们直接运行node安装路径下的node.exe程序，最前面会有一个'>'符号，我们也可以通过这个方法在Readline中设置我们自己的“前缀”：

```js
var readline = require('readline');

var completer = function(line){
  var completions = ['Hello', 'World', 'You', 'are', 'man'];

  var shots = completions.filter(function(s){
    return s.indexOf(line) == 0;
  });

  return [shots, line];
}

var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  completer: completer
});

rl.setPrompt('test>>', 6);
```

就拿我们刚刚的“自动补全代码”为例，在最后添加了一句代码。我们的在进行刚刚的操作：输入'H'然后按TAB键。就会出现'test>>'在'Hello'之前。此方法第二个参数官方API中也没有说明，并且在最新版本的API（V5.9.1）中，已经没有这个参数。但是以我打破砂锅问到底的精神，我去翻阅了老版本的node源码，发现这个length参数是用来设置光标位置的。其实上面的代码非常傻，只有我们在进行了一系列操作后才会出现“前缀”，还有一个看起来更舒服的处理方式，让我们的程序看起来更专业。

##### rl.prompt([preserveCursor]);
此方法效果就是在新的行前主动添加我们设置的“前缀”，参数为布尔型，我们如果传入true，可以防止光标位置被重置为0：

```js
var readline = require('readline');

var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.setPrompt('test>>', 6);
rl.on('line', function (cmd) {
  rl.prompt();
});
rl.prompt();
```

以上代码的效果如下：

![rl.prompt](../file/rl_prompt.png)

##### rl.question(query, callback)
此方法就是我在最开始使用的引入示例，此方法的功能就是先输出query，然后接收用户输入的内容，当检测到用户输入回车时，调用callback并且将用户刚刚输入的内容以回调参数的形式给出，过多的示例我就不举了，上面“2B”示例已经很2了，不想再2一次了。

##### rl.pause()、rl.resume()、rl.close()
暂停接收输入、恢复接收输入。

```js
var readline = require('readline');

var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

setTimeout(function(){
  rl.pause();
}, 1000);

setTimeout(function(){
  rl.resume();
}, 4000);

setTimeout(function(){
  rl.resume();
}, 6000);
```

运行以上代码，1秒之后会有3秒的空白（不可输入），到4秒之后又可以输入。然后6秒的时候程序自动退出了。

##### rl.write(data, [key])
想stdout输出data，key是一个object对象，可以模拟键盘按键。以下代码就是模拟了ctrl+u

```js
var readline = require('readline');

var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.write('这些文字回本显示出来！3秒后消失！');

setTimeout(function(){
  rl.write(null, {ctrl: true, name: 'u'});
}, 3000);
```

##### 事件：'line'、'pause'、'resume'、'close'
分别在换行、暂停、恢复、关闭时触发事件，都非常简单，这里就不多做介绍。

##### 事件'SIGINT'
表示终端信号（一般ctrl+c表示终端信号）

```js
var readline = require('readline');

var rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

rl.on('SIGINT', function(){
  rl.question('是否要退出程序?', function(answer) {
    if (answer.match(/^y(es)?$/i)) rl.close();
    else rl.prompt();
  });
});
```

##### 事件'SIGTSTP'、'SIGCONT'
分别表示接收到暂停信号、程序切换到后台运行时触发（这2个事件在Windows上是无法正常工作的）

##### 其他方法
+ readline.cursorTo(stream, x, y) 将光标移动到stream的指定位置

+ readline.moveCursor(stream, dx, dy) 将光标移动到stream的相对位置置
+ readline.clearLine(stream, dir) 清除当前行

+ readline.clearScreenDown(stream) 清除当屏幕


##### 总结
此模块不是一个非常常用的模块，但是我们可以利用此模块写出很好玩的程序，例如：一个简单的终端程序，或者可以整一下不懂程序的同学。
