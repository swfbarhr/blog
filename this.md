##### 问题
今天（2016-01-26）在逛[cnode](https://cnodejs.org)的时候，看到了这么一个[问题](https://cnodejs.org/topic/56a6075a073124894b190afd)，正好与我今晚要写的东西相关，所以借这个问题开始我们今天的探讨。下面是一段nodejs代码：

```js
var fs = require('fs');
var val1;

val1 = 'hello me';

function test() {
    var val1;

    val1 = 'hello he';

    fs.readFile('/test.js', function(err, data){
        var val1;

        val1 = 'hello you';

        console.log(this.val1);
        console.log(this === GLOBAL);
    });
}

test(); // undefined
        // true
```

好吧，我承认我看不惯原来的乱糟糟的代码（擅自修改，希望原提问者不要告我侵权）。但是效果是一样的，相信很多新手看到结果之后都很惊讶（我滴乖乖，怎么会出现这个结果？）。

##### this到底引用了什么
很多新手一上来就会问，this到底是什么？其实应该问：this到底引用的什么？因为在不同情况下，this的指向是不一样的。下面我们就来介绍各种不同情况下，this的指向。想要搞清楚this当前指向谁就要弄明白this的指向与变量的查找规则是有很大的不同。变量在其定义的时候就可以确定下来，而this是在调用时才能清楚。这就需要我们明白当前到底是哪些函数被调用了，也就是说要弄清楚调用栈。

```js
function test(){
  console.log('test is called');

  test1(); // test1被调用的位置
}

function test1(){
  console.log('test1 is called');

  test2(); // test2被调用的位置
}

function test2(){
  console.log('test2 is called');
}

test(); // test被调用的位置
```

以上代码很好的展示了调用栈：test--->test1--->test2。了解了基本概念，我们就可以开始介绍this的引用规则了。

##### 默认绑定
什么是默认规则，默认规则就是不能应用后面3种规则的规则。默认规则是最简单的规则，直接观察函数在哪里被调用也就是函数的调用位置。思考以下代码：

```js
var company;

company = 'BAT';

function getCompany(){
  var company = '全球无限皮包公司';

  console.log(this.company);
  console.log(this === window);
}

getCompany(); // BAT --->注意调用位置
              // true
```
我们看到getCompany的调用位置是全局作用域，那么this就是指向window（前端），后端指向global（nodejs）。所以this.cpmpany是“BAT”（俺也想进BAT，求推荐）。默认规则就是函数在哪调用，this就指向谁。

##### 隐式绑定
隐式绑定的意思就是没有显示的对this进行绑定操作。思考一下代码：

```js
var a = 0;

var obj = {
  a: -1,
  getVal: function(){
    return this.a;
  }
}

console.log(obj.getVal()); // -1
```

其实相信很多人都这么用过，但是不会有太多的人去探究到底为什么this会指向obj本身。其实这是一个典型的隐式绑定的效果，getVal函数的实际调用者是obj，所以this隐式的指向了obj对象。我们把这种绑定规则叫做"隐式绑定"。

##### 显示绑定
与隐式绑定相呼应，我们可以进行显示绑定或者说是硬绑定。就是明确的、手动指定this的指向。估计说到这里，大家都会想到2个函数：call、apply。是的，就是它们。我们来看下面的代码。

```js
function foo(iam){
  console.log(iam +　this.name);
}

foo.call({name: '小明'}, '我的名字叫'); // 我的名字叫小明
foo.apply({name: '小花'}, ['我的名字叫']); // 我的名字叫小花
```

call和apply的第一个参数是指定foo此次调用时的this引用（显示指定），关于call和apply的介绍与我们今天的主题无关，我这里就不多赘述了。然后foo中this.name就可以取到我们传入的值，说明此时this的指向就是我们传入的第一个参数。

##### new
在其他语言中，new是用来开辟空间，创建实例。在JavaScript中，new是用来创建一个新的对象实例（JavaScript内一切皆对象，函数也可以认为是对象，new后面只能跟函数），然后调用函数。其实一个new做了2个事情，但是有的时候我们需要分开做（这时候我们不应该用new，怎么做我之后会在prototype相关博客中详细介绍，这里就不多说了）。new之后返回的对象：如果函数没有返回值则返回一个新的对象实例（{}）；如果有返回值则是该返回值。new完之后this的引用是什么呢？思考以下代码：

```js
var style = '普通青年';

var Foo = function(){
  this.style = '文艺青年';
}

var obj = new Foo();
console.log(obj.style); // 文艺青年
```
按照上面我们提到的，在如果应用默认规则那么obj.style应该是“普通青年”，但是现在遇到“new”这个关键字就不一样了，this指向的是新创建的实例（通过new变成了一个文艺青年）。

##### 优先级
4种情况都介绍完了，那么问题又来了（挖掘机技术哪家强？）。如果几种情况不是单独出现的，是混合在一起的，那么我们该应用哪种情况？思考以下代码：

```js
function foo(v){
  this.v = v;
}

var obj = {
  v: 'obj_v',
  foo: foo
}

var obj2 = {}

console.log(obj.v); // obj_v
obj.foo('v');
console.log(obj.v); // v

obj.foo.call(obj2, 'argument_v');
console.log(obj.v); // v
console.log(obj2.v); // argument_v

var newFoo = new obj.foo('new_v');
console.log(newFoo.v); // new_v
console.log(obj.v); // v
```

通过上面的代码我们可以看出，我们直接打印obj.v是打印出默认值“obj_v”。当我们隐式绑定之后，obj.v就变成了“v”。后面我们有应用了显示绑定，打印的文本说明了foo内部的this指向了我们传入的obj2而不是obj。最后我们把new和隐式绑定比较，newFoo.v的值是“new_v”，obj.v的值还是“v”。一些列的测试说明，显示绑定的优先级高于隐式绑定，而new的优先级也高于隐式绑定。但是显示绑定和new是不可以同时使用的（var newFoo = new obj.foo.call('err_v'); // TypeError）。但是我们可以用另外一种方式来比较。

```js
function foo(v, v1){
  this.val = v + v1;
}

var baz = foo.bind(null, 'bind_v');
var newFoo = new baz('new_v');

console.log(newFoo.val); // bind_vnew_v
```

其实bind和apply或者call的内部实现机制是类似的。我们可以看到，调用了new之后，this绑定指向了新的实例对象。所以我们可以知道，new比显示绑定的优先级要高。至此，我们搞清楚了各种绑定方式的优先级：new绑定>显示绑定>隐式绑定>默认绑定。

##### 绑定丢失
其实绑定在某些情况下是会丢失的，特别是隐式绑定。思考以下代码：

```js
var a = '机器猫';

var obj = {
  a: '大熊',
  getA: function(){
    console.log(this.a);
  }
}

setTimeout(obj.getA, 1000); // 机器猫
```

按照上面的规则，这里应该应用隐式绑定，但是我们看到打印出来的a居然是“机器猫”。其实上面的代码可以有另外一种形式：

```js
var foo, a = '机器猫';

var obj = {
  a: '大熊',
  getA: function(){
    console.log(this.a);
  }
}

foo = obj.getA;

setTimeout(foo, 1000); // 机器猫
```

我们把obj.getA赋值给了foo，就是把obj.getA的引用赋值给了foo（function在JavaScript是对象）。而调用foo时，foo处在全局作用域中且应用的是默认规则。所以我们在等待漫长的1秒钟后，看到了“机器猫”而不是“大熊”。现在再回过来看我们一开始的问题，思路就很清晰了：test函数实在全局内调用，readFile函数第二个参数是一个隐式赋值绑定，所以丢失了绑定，最后导致this指向了global。

##### 总结
说了那么一大坨（希望我这一坨话能帮到你），我都已经口干舌燥了。现在相信大家应该知道this的引用规则了，不同的情况用不同的规则，混合情况使用优先级先后顺序，这样无论遇到哪个this都可以一招KO掉。另外本文也会在我的[博客](http://www.sunweifeng.cn/javascript-this/)上同步更新，欢迎访问。
