##### 前言
由于Query Strings和URL模块的API都比较少和简单，所以我把这两个模块放在一起讲。

##### Query Strings
相信从字面上理解，大家就已经知道此模块是用于处理Query String的。

##### querystring.stringify(obj, [sep], [eq])
此方法可以将object转换成字符串，三个参数分别的意义是：

+ obj 待转换的对象

+ sep 此参数是一个可选参数，表示多个参数之间的分隔符（一般使用'&'作为分隔符，默认是'&'）

+ eq 此参数也是一个可选参数，表示参数与对应值之间的分隔符（一般使用'='作为分隔符，默认是'='）

```js
var querystring = require('querystring');

var obj = {
  method: 'add',
  user: 'admin',
  age: 31,
  score: [98, 91]
};

console.log(querystring.stringify(obj)); // method=add&user=admin&age=31&score=98&score=91
console.log(querystring.stringify(obj, ';', ':')); // method:add;user:admin;age:31;score:98;score:91
```

##### querystring.parse(str, [sep], [eq], [options])
querystring.parse是querystring.stringify方法的逆。参数sep和eq与querystring.stringify有着相同的意义，但是此方法有第四个可选参数，此参数是可以配置项内目前为止就一个配置参数：maxKeys，表示最大处理的keys数量，超过此数量就默认略过（maxKeys的默认值为1000）

```js
var querystring = require('querystring');

console.log(querystring.parse('method=add&user=admin&age=31&score=98&score=91')); // { method: 'add', user: 'admin', age: '31', score: [ '98', '91' ] }
console.log(querystring.parse('method:add;user:admin;age:31;score:98;score:91', ';', ':')); // { method: 'add', user: 'admin', age: '31', score: [ '98', '91' ] }
console.log(querystring.parse('method=add&user=admin&age=31&score=98&score=91', '&', '=', { maxKeys: 2 })); // { method: 'add', user: 'admin' }
```

这里需要注意的是，如果我们需要用到options参数，那么sep和eq是不可以省略的。不然的话，解析会出乎于预期：

```js
var querystring = require('querystring');

console.log(querystring.parse('method=add&user=admin&age=31&score=98&score=91', { maxKeys: 2 })); // { method: 'add&user=admin&age=31&score=98&score=91' }
```

##### querystring.escape、querystring.unescape
querystring.escape其实就是对字符串进行urlencode，而querystring.unescape则是对字符串进行urldecode

```js
var querystring = require('querystring');

console.log(querystring.escape('a=ss&b=111')); // a%3Dss%26b%3D111
console.log(querystring.unescape('a%3Dss%26b%3D111')); // a=ss&b=111
```

##### URL
此模块是处理url相关的问题。

##### url.parse(urlStr, [parseQueryString], [slashesDenoteHost])
将字符串转换成object对象，与querystring.parse功能类似。

+ urlStr 表示需要转换的字符串

+ parseQueryString 如果将此项设置为true，则表示使用querystring模块转换querystring

+ slashesDenoteHost 如果此项设置为true，则表示将双斜线后的当做为host

```js
var url = require('url');

console.log(url.parse('HTTP://admin:123@172.16.0.1:3000/a/b/c?test=true#list')); // { protocol: 'http:',
                                                                                 //   slashes: true,
                                                                                 //   auth: 'admin:123',
                                                                                 //   host: '172.16.0.1:3000',
                                                                                 //   port: '3000',
                                                                                 //   hostname: '172.16.0.1',
                                                                                 //   hash: '#list',
                                                                                 //   search: '?test=true',
                                                                                 //   query: 'test=true',
                                                                                 //   pathname: '/a/b/c',
                                                                                 //   path: '/a/b/c?test=true',
                                                                                 //   href: 'http://admin:123@172.16.0.1:3000/a/b/c?test=true#list' }

console.log(url.parse('HTTP://admin:123@172.16.0.1:3000/a/b/c?test=true#list', true)); // { protocol: 'http:',
                                                                                       //   slashes: true,
                                                                                       //   auth: 'admin:123',
                                                                                       //   host: '172.16.0.1:3000',
                                                                                       //   port: '3000',
                                                                                       //   hostname: '172.16.0.1',
                                                                                       //   hash: '#list',
                                                                                       //   search: '?test=true',
                                                                                       //   query: { test: 'true' },
                                                                                       //   pathname: '/a/b/c',
                                                                                       //   path: '/a/b/c?test=true',
                                                                                       //   href: 'http://admin:123@172.16.0.1:3000/a/b/c?test=true#list' }

console.log(url.parse('//admin:123@172.16.0.1:3000/a/b/c?test=true#list', true, true)); // { protocol: null,
                                                                                        //   slashes: true,
                                                                                        //   auth: 'admin:123',
                                                                                        //   host: '172.16.0.1:3000',
                                                                                        //   port: '3000',
                                                                                        //   hostname: '172.16.0.1',
                                                                                        //   hash: '#list',
                                                                                        //   search: '?test=true',
                                                                                        //   query: { test: 'true' },
                                                                                        //   pathname: '/a/b/c',
                                                                                        //   path: '/a/b/c?test=true',
                                                                                        //   href: 'http://admin:123@172.16.0.1:3000/a/b/c?test=true#list' }
```

##### url.format(urlObj)
此方法是url.parse的逆方法，就是将object转换成字符串

```js
var url = require('url');

var obj = {
  protocol: 'http:',
  slashes: true,
  auth: 'admin:123',
  host: '172.16.0.1:3000',
  port: '3000',
  hostname: '172.16.0.1',
  hash: '#list',
  search: '?test=true',
  query: {
    test: 'true'
  },
  pathname: '/a/b/c',
  path: '/a/b/c?test=true',
  href: 'http://admin:123@172.16.0.1:3000/a/b/c?test=true#list'
};

console.log(url.format(obj)); // http://admin:123@172.16.0.1:3000/a/b/c?test=true#list
```

##### url.resolve(from, to)
可以简单地理解成：用后面的对应的属性替换前面的属性。过程可以理解成（内部过程可能更加复杂，以下步骤仅供理解，不是官方步骤）：

+ 第一步：调用url.parse
+ 第二步：Object.assign
+ 第三部：url.stringify

```js
var url = require('url');

var url = require('url');

console.log(url.resolve('/one/two/three', '//host.com:300/four'))         // //host.com:300/four
console.log(url.resolve('http://example.com/', '/one'))    // 'http://example.com/one'
console.log(url.resolve('http://example.com/one', 'http://ssss.com/one/one/two')) // 'http://ssss.com/one/one/two'
```

##### 总结
Query Strings模块和URL模块其实都是非常实用的两个模块，但是由于我们做node一般都使用第三方框架，如：express、koa、restify等。里面都已经封装好了对应的方法，所以这两个模块可能在实际开发中用到的不多，但是在做node原生开发时（不使用第三方框架），会非常有用。
