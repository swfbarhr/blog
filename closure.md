##### 一句话概括闭包
相信很多写了很多JavaScript代码的老程序猿都知道有闭包这个概念，但是如果真的要大家用语言来描述闭包的话，很多程序猿都吱吱呜呜说不清楚（包括很大一部分前端攻城狮）。闭包简单来讲就是：当函数记住并访问所在作用域时，就产生了闭包。

##### 示例
```
function foo(){
  var a = 123;

  return function(){
    return a;
  }
}

var baz = foo();
console.log(baz()); // 123
```
上面是一个典型的闭包。我们来分析下代码：
首先我们定义了一个函数foo，然后在其作用域内部又定义了一个匿名函数并且将此匿名函数（在实际编码过程中尽量少用匿名函数，原因我会在后面的相关博客中说明）当做返回值return了出来。当我们调用foo并把结果赋值给baz变量（函数表达式），我们在foo外调用baz时，竟然能访问到foo的内部变量“a”（妈呀！我穿越了吗？）。在词法上，foo和baz是在同一层次的不同函数，应该互不影响才对，那现在这种情况又是怎么回事呢？

##### JavaScript中闭包的实现原理
要知道闭包不是JavaScript独有的，其他语言像C#、JAVA都有闭包的概念。但是各自的实现原理是不一样的，其他语言暂且不论，JavaScript中闭包的实现其实就是靠的作用域链（就是ES5中的词法环境）。在JavaScript作用域一文中我简单地提到过作用域链的概念，现在我就来详细的解读下什么是作用域链，它的形成过程和实现闭包的方法。先思考一下代码：
```
var arg1, foob;

arg1 = 'world';

function fooa(){
  var v1;

  v1 = 'hello'

  return function(arg1){

    if(arg1){
      return v1 + ' ' + arg1;
    }

    return v1 + ' everyone';
  }
}

foob = fooa();
foob('Bob'); // hello Bob
```
代码在执行过程中，fooa会把全局作用域的变量（可以认为fooa把全局作用的所有变量放入了一个堆栈里面以备用）引用至其作用域。同样的，fooa中return的匿名函数也会把fooa中的变量加入到这个“堆栈”中并且在其作用域中引用（如图1所示）。

![scope chain](http://120.27.119.47/content/images/manual/scope_chain.png)

其实在执行foob时，fooa的运行状态已经不存在了，之所以foob中能访问fooa的变量就是因为foob保存了对fooa作用域变量的引用，这样foob可以在需要时取出fooa中的变量来使用，也就是我们常说的：产生了闭包。（哎妈呀！原来如此！）闭包就是这么简单！

##### 闭包的应用
前面已经介绍了什么是闭包以及闭包的实现原理，那闭包在我们实际编码过程中到底有怎样的用途呢？
示例1：
```
var firFunc, secFunc;

function func1(arg1){
  return function(arg2){
    return arg1 +' ' + arg2;
  }
}

firFunc = func1('hello');
secFunc = func1('hello');

console.log(firFunc('Lily')); // hello Lily
console.log(secFunc('Lucy')); // hello Lucy
```
我们可以利用闭包来暂时记录参数（例如：配置项、数据库连接等）。就是这么的优雅！就是这么酸爽！
示例2：
```
function foo(logInfo){
  setTimeout(function(){
    console.log(logInfo);
  }, 3000);
}

foo('pizza'); // pizza
```
setTimeout大家都很熟悉：延迟指定的时间执行代码。这里也涉及到了一个闭包，setTimeout的第一个参数（function）记住了foo的作用域内的变量logInfo。
示例3：
```
var isIE = true;

$('#dvTest').click(function(){
  if(isIE){
    console.log('IE');
  }
  else{
    console.log('other');
  }
});
```
这是一段大家都很熟悉的JQuery事件绑定代码，里面也有一个非常明显的闭包。还有很多的例子，可能大家在编写代码的过程中已经很熟练地使用了闭包，自己却没有发现。现在去回顾下自己的代码，从中找出闭包！

##### 总结
讲到这里，我们就把闭包的概念叙述完了。其实也不复杂，很多人一直搞不清楚闭包，并不是因为闭包有多难，只是没有用心去理解。希望我上面的描述能帮助大家理解闭包，拥抱JavaScript。下一篇我们就来谈谈我在[JavaScript之作用域](https://github.com/swfbarhr/blog/blob/master/scope.md)一文中提到的JavaScript编译。另外本文也会在我的[博客](http://www.sunweifeng.cn/javascript-closure/)上同步更新，欢迎访问。
