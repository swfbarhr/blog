##### 便捷的object对象
在通常的开发过程中，我们很多时候都会用到object对象。例如：传递object参数、可选择的配置项、数据库输出对象，都可以使用object。这样做使得我们的程序简洁、易用、可扩展。需要使用时，就像如下代码：

```js
function getUer(name, age, country){
  var user;

  user = {
    name: name,
    age: age,
    country: country
  }

  return user;
}

console.log(getUer('peter', 28, 'America'));  // {
                                              //   name: 'peter',
                                              //   age: 28,
                                              //   country: 'America'
                                              // }
```
user = {...}是大家常用的一种定义对象的方式，这样非常的方便、直接。这样定义可以满足99%的业务需求，但是有的时候我们需要满足一些特殊的需求。

##### Object.create()
除了直接创建object对象外，我们还可以使用Object.create方法来创建对象，这个方法给我提供了更多的不同情况下的API。Object.create有1个必传参数和一个可选参数。

+ 第一个参数是必传参数，指定即将创建的对象的prototype（原型，这个我下一篇博客会详细介绍，这里就不赘述）。
+ 第二个参数一个可选参数，可以设置我们需要初始化给即将创建的对象的属性及对属性的控制。

```js
var newObj = Object.create({}, {
    'a': {
      value: 'this is a',
      configurable: false,
      enumerable: false,
      writable: false
    }
  });

console.log(newObj); // {a: 'this is a'}
```

上面的代码中我们把空对象“{}”作为了newObj的原型，并给newObj设置了一个值为“this is a”的属性“a”。我们看到，除了value属性，还有configurable、enumerable、writable三个配置属性。

+ configurable表示可否对当前对象进行配置。默认情况下使用Object.create创建的对象是不可配置的，也就是说configurable默认是false。

```js
var newObj = Object.create({}, {
    'a': {
      value: 'this is a',
      configurable: false,
      enumerable: false,
      writable: false
    }
  });

Object.defineProperty(newObj, 'a', {
    value: 'this is not a',
    enumerable: true
}); // Uncaught TypeError: Cannot redefine property
```
当我们试图对newObj的属性进行修改时，JavaScript引擎就会抛出错误“Cannot redefine property”，因为默认属性“a”是不可配置的。如果把configurable设置成true的话，错误就不会出现。

+ enumerable表示该属性不可枚举。如果需要枚举一个object对象，可以使用for...in方法，一旦我们把当前属性设置成false，for...in将访问不到此属性。

```js
var user = Object.create({}, {
    'name': {
      value: 'Lucy',
      enumerable: true
    },
    'age': {
      value: 24,
      enumerable: false
    },
    'country': {
      value: 'Australia',
      enumerable: true
    }
  });

for(p in user){
  console.log(user[p]); // Lucy
                        // Australia
}
```

这里age属性的enumerable设置成了false，然后我们对user进行属性枚举的时候，发现age并没有被输出。

+ writable表示是否可以被赋值运算符改变。

```js
(function(){
  "use strict";

  var song = Object.create({}, {
      'name': {
        value: 'Welcome to Beijing',
        configurable: true,
        writable: false
      }
  });

  Object.defineProperty(song, 'name', {
    value: 'WELCOME TO BEIJING'
  });

  console.log(song);

  song.name = '北京欢迎您'; // Uncaught TypeError: Cannot assign to read only property 'name' of object '#<Object>'

  console.log(song);
})();
```
为了更好的展示效果，这里我们使用了严格模式，在严格模式下，当writable设置成false，利用Object.defineProperty方法可以对“name”属性进行正常修改，但是使用赋值修改时就会抛出系统错误。说明在writable为false的情况下，赋值符号是不可以修改对应属性的。（在非严格模式下赋值会静默失败，不会抛出错误）

##### 引用传递
需要提到的是，如果需要在函数中传递object，请记住我们传递的永远是object的引用地址，如果在函数内部对该参数进行了修改，所有引用该参数的代码都会受到影响。

```js
A.js:
  var setting = {
    user: 'admin',
    group: 'vip'
  }

  function getSettings() {
    return setting;
  }

  function setSettings(k, v) {
    setting[k] = v;

    return setting;
  }

B.js（B.js引用了A.js）
  console.log(getSettings()); // {user: "admin", group: "vip"}

  console.log(setSettings('user', 'test')); // {user: "test", group: "vip"}

  console.log(getSettings()); // {user: "test", group: "vip"}
```

我们看到，当我们调用setSettings函数时，A文件的setting对象被修改了，然后后面一个getSettings就被影响了。这个习惯性的错误在我们编写JavaScript时很容易犯，并且有的时候这个错误是致命的。所以如果需要拷贝对象，请Object.assign或第三方库（lodash、underscore等）提供的拷贝函数。这里我们来介绍下ES2015中的新方法：Object.assign，将刚刚的代码小小修改下

```js
A.js:
  var setting = {
    user: 'admin',
    group: 'vip'
  }

  function getSettings() {
    return setting;
  }

  function setSettings(k, v) {
    var _setting = Object.assign({}, setting);

    _setting[k] = v;

    return _setting;
  }

B.js（B.js引用了A.js）
  console.log(getSettings()); // {user: "admin", group: "vip"}

  console.log(setSettings('user', 'test')); // {user: "test", group: "vip"}

  console.log(getSettings()); // {user: "admin", group: "vip"}
```
这样我们原来的setting对象就不会被改变了，需要注意的是Object.assign只对对象自身可枚举的属性进行拷贝，其他属性是不会拷贝的。

##### 很多其他实用方法
以上内容只介绍了Object.create和Object.defineProperty、Object.assign三个方法，object对象还有很多其他实用的方法。例如：Object.freeze、Object.seal、Object.prototype.hasOwnProperty等等，都是会在日常开发过程中用到的。这里就不一一介绍，有兴趣的话可以参考JavaScript规范。

##### 总结
可能我们在日常的coding业务时，不需要使用特别复杂的Object方法，但是对于一个框架与底层的开发人员来说，这些都是需要掌握的。另外本文也会在我的[博客](http://www.sunweifeng.cn/javascript-object/)上同步更新，欢迎访问。
