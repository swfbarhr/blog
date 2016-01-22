##### 琢磨不透的变量
我们在使用JavaScript写业务代码时，经常会得到意料之外的变量值。我们来看下以下代码：
```
for(var i = 0; i < 3; i++){
    // 业务...
}

console.log(i); // 3(纳尼！怎么会是3！不是undefined！)
```
80%的.NET程序猿都会惊讶这个打印结果（包括我一开始也是），我们都把.NET的作用域思想强加到了JavaScript上了。在.NET中是有局部变量的概念，但是在JavaScript中（ES2015）之前是没有块作用域的概念的。

##### JavaScript中的作用域
那么在JavaScript中的作用域到底是怎么运作的？其实JavaScript的作用域一点都不难。只要掌握了规律，在写代码的时候就可以避免不必要的错误。

我们知道，JavaScript中函数是一等公民。JavaScript中的作用域也是围绕函数来的，每个函数（同一层次）内部的作用域是独立的、互不影响的。
```
function foo(){
	var i = 0;
	i++;

	console.log(i) // 1
}

function baz(){
	var i = 1;
	i--;

	console.log(i) // 0
}
```
函数foo中的变量i和函数baz中的变量i存在于2个完全不同的作用域中,是2个完全不同的变量,foo的调用并不会影响baz中i的值。

我们在回过头来看我们一开始提出的问题：为什么在for循环结束之后，我们仍然可以访问到“其内部变量i”呢？其实对于JavaScript引擎来说，for循环只是一个代码块而已，根本不能算一个函数。所以，for循环中定义个变量i，其实是一个全局变量，其实之前的代码等价于：
```
var i;
for(i = 0; i < 3; i++){
    // 业务...
}

console.log(i); // 3
```
这里就会涉及到JavaScript的编译规则了（您没看错，是编译）。很多人会觉得奇怪，JavaScript不是解释型动态语言吗，也需要编译？是的，JavaScript是需要编译的，这个我以后会专门写一篇相关的博客。

#####作用域链
我们知道JavaScript函数的内部可以定义变量，也可以定义函数。就像这样：
```
var arg2 = 3;

function foo(arg1){
	var arg2 = arg1 + 1;

	function baz(arg3){
		console.log(arg2 + arg3);
	}

	baz(4);
}

foo(1); // 6
```
看到这段代码的时候，肯定有很多初学者会疑惑：到底baz中打印的arg2参数，是foo中定义的arg2还是在全局定义的arg2，或者都不是。这里就涉及到我们作用域链的问题。

当程序执行到baz内部时，引擎首先会检查当前的作用域中有没有定义arg2，如果没有就向上一级作用域查询，如果上一级作用域有定义arg2，那么就直接使用这个定义；如果上一级作用域也没有arg2的定义，再往上一级作用域查找，依次类推至全局作用域（浏览器中的window或Node.js的global），万一全局作用域也没有，那么JavaScript就会抛出异常（"arg2 is not defined"）。根据这个规则，我们可以轻松解析以上代码（图1可以帮助我们理解这一过程）。
![scope chain](http://120.27.119.47/content/images/manual/scope.png)
所有的变量查找都是从最内部开始，到全局作用域结束。所以很多时候我们会一部小心就访问到全局作用域，并且JavaScript中所有的对象都是可以任意改变的（除非Object.freeze之后），这样一来全局作用域很容易被修改，也就是我们常说的“全局作用域污染”。

#####避免全局污染
全局污染是个很头疼的问题，我们经常拿到不是自己想要的变量值。为了更直观的了解全局污染，我们先来看段代码：
```
var arg = 'start';

function foo(){
	console.log(arg);

	arg = 'foo_start';

	console.log(arg);

	console.log(window.arg)
}

function baz(){
	// 使用arg...
}

foo(); // start
	   // foo_start
	   // foo_start
baz();
```
foo被调用后，全局变量arg已经被更改，当baz被调用的时候，已经不是原来的arg了（偶漏！怎么会这样！）！如果foo和baz一定要公用一个变量，而又不能污染全局变量，我们可以这么做：
```
var arg = 'start';

(function(obj){
	var arg;

	function foo(){
		arg = 'foo_start';

		console.log(arg);
	}

	function baz(){
		console.log(arg);
		console.log(obj.arg);
	}

	foo(); // foo_start
	baz(); // foo_start
		   // start
})(window);

function foo1(){
	// 使用arg...
}

foo1();
```
由于自执行函数的作用域是独立的，所有并且我们将window当做参数传入。由于我们没有改变全局变量，所以我们在foo1函数中使用的arg还是原来的'start'（保持初心，吼吼！）。其实这个应用很常见，比如我们熟知的JQuery就是这么干的。

#####总结
JavaScript的作用域其实不难，只要掌握了规则就可以很好的驾驭。关于文中提到的JavaScript编译，我以后会更新一篇新的相关博客。其实讲到JavaScript的作用域，就不得不讲到一个概念：闭包。我们下一篇博客就来讲讲“闭包”。本文会同步更新到我的[博客](http://120.27.119.47/javascript-scope/)。
