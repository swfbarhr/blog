# 前言

[上一篇](start.md)我们讲了如果简单的接收微信的消息，当然，公众号不止会推送消息，还提供了丰富的 API 供开发者调用。想要了解微信提供了哪些接口，可以访问微信[官方文档](https://mp.weixin.qq.com/wiki)。在公众号后台，“开发”--“接口权限”菜单中也可以看到所有 API，并且微信已经做好了分类。

# access_token

首先，所有的 API 都不是随意调用的。在调用任何 API 之前，都需要先获取令牌（即接口调用凭证：access_token）。不同公众号的 access_token 是不同的，微信会根据 access_token 来定位公众号，执行相应的操作。接下来我们就以最常用的“自定义菜单”接口为例，来介绍下如何调用微信公众号 API：

- 获取 access_token
  上面说到，在调用 API 之前，先要获取 access_token。根据官方[获取 access_token 文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183)，获取 access_token 是一个 3 个参数的 HTTP GET 请求：

  - grant_type 在说明中，我们看到，微信这个参数是定死的，就传 client_credential
  - appid 第三方用户唯一凭证
  - secret 第三方用户唯一凭证密钥

grant_type 没有问题，我们直接传 client_credential 就行了，那 appid 和 secret 哪来呢？我们可以登录公众号后台，在“开发”--“基本配置”中找到“开发者 ID(AppID)”和“开发者密码(AppSecret)”。开发者 ID 就是 appid，开发者密码就是 secret。好了，万事俱备，让我们来获取下 access_token：

```js
'use strict';

const rp = require('request-promise');

async function getAccessToken(appId, secret) {
  const result = await rp(
    `https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=${appId}&secret=${secret}`
  );

  if (result.access_token & result.expires_in) {
    // 打印结果
    return console.log(result);
  }
}

getAccessToken('myAppId', 'mySecret');
```

如果没有问题，我们会得到一个 json 的响应。json 里面有 2 个属性值：access_token 和 expires_in，access_token 就是我们需要的，而 expires_in 是表示 access_token 的有效实现，一般情况下是 7200 秒，也就是 2 小时有效期。所以，我们必须在获取到 access_token 后 2 小时内使用它，否则就会过期。如果 access_token 过期，我们必须重新获取一次。

# 调用创建菜单的 API

根据微信的创建菜单的[文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141013)，创建菜单是一个 HTTP POST 方法， 这个 API 有 2 个参数，其中一个就是刚刚获取的 access_token，另一个就是菜单的内容。注意，所有接口 access_token 都是作为 query 参数传过去的。


```js
'use strict';

const rp = require('request-promise');

async function setMenu(accessToken, menu) {
  const result = await rp({
    method: 'POST',
    uri: `https://api.weixin.qq.com/cgi-bin/menu/create?access_token=${accessToken}`,
    json: true,
    body: menu
  });

  if (result.errcode === 0) {
    // 设置成功
    return console.log('success');
  }

  // 设置失败
  console.log('failed');
}

const menuObj = {
  button: [
    {
      type: 'click',
      name: '今日歌曲',
      key: 'V1001_TODAY_MUSIC'
    },
    {
      name: '菜单',
      sub_button: [
        {
          type: 'view',
          name: '搜索',
          url: 'http://www.soso.com/'
        },
        {
          type: 'miniprogram',
          name: 'wxa',
          url: 'http://mp.weixin.qq.com',
          appid: 'wx286b93c14bbf93aa',
          pagepath: 'pages/lunar/index'
        },
        {
          type: 'click',
          name: '赞一下我们',
          key: 'V1001_GOOD'
        }
      ]
    }
  ]
};
setMenu('accessToken', menuObj);
```

大部分公众号的 API 调用之后，都会有一个错误码 errcode，如果 errcode 为 0，则表示成功；如果 errcode 为其他值，那表示 API 调用有问题。
如果返回了其他错误码，可以通过微信的[全局返回码说明](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1433747234)来查询具体的错误原因。

# 总结

只要记住，在调用微信 API 时，access_token 是必不可少且一定是放在 query 里面的。如果调用出了什么问题，可以根据微信返回的错误码去官方查询具体原因。不过，根据我的经验，有些错误码微信也没有给出解释，祝各位好运！
