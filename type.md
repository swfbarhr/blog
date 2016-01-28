##### JavaScript基本类型
说到类型，在JavaScript中有6中基本类型：boolean、number、string、object、null、undefined，所有的JavaScript代码都不会离开这6个类型。

##### 隐式类型转换
在JavaScript存在着很多隐式的类型转换，很多我们都用到过。思考以下代码：

```
function foo(){
  var isSuperMan = 'no';

  if(isSuperMan) {
    console.log('you are superman');
  }
  else {
    console.log('you are human');
  }
}

foo(); // you are superman
```

我们知道在其他语言中，if后面的判断语句一定是布尔型变量或者结果是布尔型的表达式。但是在JavaScript中，if后面可以跟任意类型。这里就涉及到了隐式转换，JavaScript把不在以下值中的所有非布尔值都转换成true：null、undefined、0、-0、+0、''。还有一种更常见的隐式类型转换出现在“+”的两侧，例如：

```
function stringConcat(val1, val2){
  return val1 +　val2;
}

stringConcat(1, 'abc'); // 1abc
```

在stringConcat中，数字1被隐式转换成了字符串之后被输出。因为隐式转换，数学中的结合律在这里也是不成立的：

```
console.log(1 + 2 + '3' === 1 + (2 + '3')); // false
```
前一个表达式的结果是字符串“33”，而后面一个表达式的结果是字符串“123”，是2个完全不同的结果。这一切都是拜隐式转换所赐。现在大家可以去自己的代码中去发现隐式转换了。

##### 坑爹的浮点数
要知道在JavaScript中所有的数字都是双精度浮点数，然后浮点数是出了名的不精确（即使它表示的范围已经足够大）。在某些计算时要特别小心掉入此坑中，来看下面的代码：

```
function plus(p1, p2){
  return p1 + p2;
}

plus(0.0001, 0.0002); // 0.00030000000000000003
```

（妈呀，都说电脑很强大，但是这个时候还不如俺的“猪脑子”）我们看到0.0001与0.0002的和居然不是0.0003，后面多了尾巴。这是因为浮点数本来就不是精确表示的，计算结果都会进行舍入。那么我们如果真的要计算的话，该怎么办呢？其实也不是什么难事：

```
function plus(p1, p2){
  return (p1*10000 + p2*10000)/1000;
}

plus(0.0001, 0.0002); // 0.0003
```

方法就是讲小数转换成整数，因为整数是没有小数部分的，不需要舍入，是精确的。

##### 既熟悉又陌生的string
string对于任何一个写程序的程序猿都是非常熟悉的，我们几乎每天都会用到。但是问题来了，像"abcd"这种，只是字面量，根本不是对象。但是，我们又可以像这样

```
console.log("123abc".length); // 6
```

调用length属性或者方法（charAt等）。并且：

```
console.log("123abc" instanceof String); // false
```

"123abc"也不是内置对象String的一个实例，却还可以使用String的length属性和其它所有的方法。"123abc"是字面量没错，每当其获取属性length或者调用其它String的方法时，JavaScript都会自动用字面量生成一个String的实例，这样看上去就是字面量在操作一样。这个对象在使用完成之后会被立即销毁。

##### 小心typeof掉坑
首先大家要记住：typeof是一个操作符，不是一个函数，虽然现在很多JavaScript引擎可以让我们把typeof当做函数来调用，但我还是强烈建议去掉括号，就想这样：

```
console.log(typeof {a: 1}); // object
```

大多数时候，我们用typeof是没有问题的。但是当我们对null做typeof的时候，结果会令人很惊奇：

```
console.log(typeof null); // object
```

按照道理应该显示null才对，现在却打印了object。这其实是JavaScript的一个隐形的BUG，在JavaScript设计之初，JavaScript中的值是由两部分组成：一部分是表示类型的标签，另一部分表示值。然后null表示的空指针在大多数平台下都是0x00，与JavaScript中object标签值0正好一样。所以typeof null的结果是“object”（有人提出在ES2015中修复这个BUG，但是后来被否决了）。下图展示了type各种类型的结果：

![typeof](http://120.27.119.47/content/images/manual/typeof.png)

所以我们在用typeof的时候，最好加入其它判断条件一起判断。

##### 总结
好了，JavaScript的基本类型其实就6种，只要搞清楚这其中哪里有坑，下次再coding的时候就不会莫名其妙的掉进去了。由于object我会在以后专门写一篇博客来讲（因为object实在太强大了，作为一部分讲会贬低了它）。另外本文会同步更新到我的[博客](http://sunweifeng.cn/javascript-type/)上，欢迎访问。
