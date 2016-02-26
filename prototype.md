##### Object续
在[JavaScript之object](https://github.com/swfbarhr/blog/blob/master/object.md)一文中，我简单介绍了object几个常用的方法。之所有此篇博客第一个小标题叫做"Object续"，其实是因为我马上要讲的prototype是object的一个很重要的属性或者说是概念。可以说，如果JavaScript没有prototype这个思想的话，早就被世上的程序猿们所抛弃，还不知道在哪个犄角旮旯中哭泣呢。这就说明了prototype的对于JavaScript语言本身的重要性，很多东西需要靠prototype实现。

##### 什么是prototype
很多程序猿都知道prototype是什么概念，可能有很大部分人也是似懂非懂（包括几个月前的俺）。大家都会说，prototype不就是原型嘛，有什么难的。我之前也是这么认为的，但是如果真的要说出个所以然来的时候，却不知道从何说起。prototype其实就是一个对象对另一个对象的引用。

##### 原型链
原型链跟之前我在[JavaScript之作用域](https://github.com/swfbarhr/blog/blob/master/scope.md)一文中提到的作用域链一样，都是逐层查找属性的。

```js
var a = {
  group: 'administrator'
}

var b = Object.create(a, {
    'name': {
      value: 'Nicolas',
      writable: true,
      enumerable: true
    }
});

console.log(b.hasOwnProperty('group')); // false
console.log(Object.getPrototypeOf(b).hasOwnProperty('group')); // true
console.log(b.group); // administrator
```

上面的代码中，对象b是没有group属性的。但是我们在最后一句打印的时候却拿到了结果，group其实存在b的原型上。这里就有一条简单的原型链，当我们访问b的group属性时，在b上没有查找到group，然后会向b的原型（也就是a上）查找，发现a上有一个叫做group的属性，于是直接读取。（万一还是没有查找到，会向a的原型，也就是Object查找，如果最终还是没有找到，会返回一个undefined）。如果我们调用b.toString方法，最终会查找到Object.toString方法，然后调用，查找规则与我们查找group属性的情况一致。

##### 引用另一个对象的好处
很多人会问，好好的对象，为什么无缘无故地要引用其他对象呢。一个对象引用另一个对象后，可以使用目标对象上的属性和方法。

```js
function Vehicle(wheel){
  this.wheel = wheel;
}

Vehicle.prototype.getWheel = function(){
  return this.wheel;
}

function Car(wheel, fuel){
  Vehicle.call(this, wheel);

  this.fuel = fuel;
}

Car.prototype.getWheel = function(){
  return 'wheel is ok';
}

Car.prototype = Object.create(Vehicle.prototype);

Car.prototype.getWheel = function(){
  return this.wheel + 10;
}

var buickCar = new Car(4, 'gasoline');

console.log(buickCar); // {wheel: 4, fuel: "gasoline"}

console.log(buickCar.hasOwnProperty('getWheel')); // false

console.log(buickCar.getWheel()); // 14
```

我们可以看到，当我们的Car引用了Vehicle的对象后，虽然在Car中没有定义getWheel方法，但是我们仍然可以在Car的实例对象（buickCar）中访问到getWheel方法。这也是JavaScript中原型继承的基本思想。这种直接给prototype赋值的方法有点野蛮，这会把原来的prototype废弃掉，我们可以看到代码中原来就有一个getWheel方法，但是我们后来覆盖了prototype，所以之前的一切属性和方法都废弃了。在ES2015中提供了一种更温柔的方式Object.setPrototypeOf，大家可以试试。

##### 属性屏蔽
属性屏蔽也可是叫做属性遮蔽，如果A是B的原型，那么如果给B的属性赋值时，就有可能产生属性遮蔽。为什么是有可能呢，因为属性遮蔽只有在满足特定条件的情况下才会发生。在执行B.a = '0'时：

+ 如果A上有属性a，并且属性a的writable不为false时，这时会在B上新增一个属性a，并且B.a会屏蔽A.a

```js
var A = Object.create({}, {
  'a': {
    value: 'This is A',
    writable: true
  }
});

var B = Object.create(A);

console.log(B.a); // This is A
console.log(B.hasOwnProperty('a')); // false

B.a = 'this is B';

console.log(B.a); // This is B
console.log(B.hasOwnProperty('a')); // true
console.log(Object.getPrototypeOf(B)); // {a: "This is A"}
```

我们可以看到，当B上没有属性a的时候，B.a其实是B的原型上的属性即“this is A”。当我们给B.a赋值后，B.a就是我们所期望的“This is B”，也就是说此时出现了屏蔽现象。原型上的a属性被对象B的a属性屏蔽了。但是如果原型上的a属性的writable是false时，又会发生什么事情呢。

```js
(function(){
    "use strict";

    var A = Object.create({}, {
      'a': {
        value: 'This is A',
        writable: false
      }
    });

    var B = Object.create(A);

    console.log(B.a); // This is A
    console.log(B.hasOwnProperty('a')); // false

    B.a = 'this is B'; // Uncaught TypeError: Cannot assign to read only property 'a' of #<Object>(…)
    console.log(B.a);
})()
```

为了更好的看出效果，我使用了严格模式。这时JavaScript引擎抛出了一个错误，意思是属性a是只读的。这样就不会出现屏蔽的现象。

+ 如果原型上正好有一个setter，那么会直接设置这个setter。

```js
var val = 'This is A';

var A = Object.create({}, {
  'a': {
    get: function(){
        return val;
    },
    set: function(v) {
      val = v;
    }
  }
});

var B = Object.create(A);

console.log(B.a); // This is A
console.log(B.hasOwnProperty('a')); // false

B.a = 'this is B';

console.log(B.a); // This is B
console.log(B.hasOwnProperty('a')); // false
console.log(Object.getPrototypeOf(B)); // {}
```

A上存在一个setter，我们在执行“B.a = 'this is B';”语句的时候，没有出现屏蔽的情况，此行代码的结果是直接执行了A上的setter，根本没有在B上添加新的属性，所以没有出现屏蔽现象。以上两种情况说明，屏蔽不是很多人想象的那么容易就会发生的，一定要满足一定的条件。

##### 委托
我要说的委托，不是.NET中的事件委托。这里的委托是与JavaScript的“类”（其实JavaScript根本没有真正的类，一切都是障眼法，都是模仿的）有一致的功能，但是又号称更加的优雅。

```js
var computer = {
  screen: '26 inch',
  cpu: 'i7',
  setScreen: function(_s){
    this.screen = _s;
  },
  setCPU: function(_c){
    this.cpu = _c;
  }
}

var dell = Object.create(computer);

dell = Object.assign(dell, {
  brand: 'Dell',
  setDellScreen: function(_s){
    this.setScreen(_s);

    // 其他业务逻辑...
    console.log('Dell\'s screen is ' + this.screen);
  },
  setDellCPU: function(_c){
    this.setCPU(_c);

    // 其他业务逻辑...
    console.log('Dell\'s CPU is ' + this.cpu);
  }
});

dell.setDellScreen('21 inch'); // Dell's screen is 21 inch
```

我们看到，dell对象没有setScreen方法，运用原型链查找规则，dell实现了setScreen的效果。换句话说，dell对象把setScreen方法委托给了原型computer来做，这样做就是所谓的委托。这么做我们既保留了原来类的“继承”效果，又不会有原型类中复杂的关系（由于这个关系过于复杂，以后有时间会更新一篇相关主题的博客）。

##### 总结
到这里，prototype的简单介绍就完了。如果需要真正的深入了解，还需要我们在日常开发过程中积累，罗马也不是一天建成的，希望我的总结对大家学习JavaScript有帮助。JavaScript系列的博客到这就结束了，但是不代表完结，如果以后我对JavaScript有了新的理解，也会继续更新。预告下，下个系列是NODE.JS相关的博客，谈谈我个人对NODE.JS的理解。另外本文也会在我的[博客](http://www.sunweifeng.cn/javascript-prototype/)上同步更新，欢迎访问。
