# Using JSON Web Tokens with Node.js 在Nodejs中使用JSON WEB Tokens
原文：http://www.sitepoint.com/using-json-web-tokens-node-js/

翻译：连城究（LcjNiL）

诸如Ember，Angular，Backbone之类的前端框架类库正随着更加精细的Web应用而日益壮大。正因如此，服务器端的组建也正正在从传统的任务中解脱，转而变的更像API。API使得传统的前端和后端的概念解耦。开发者可以脱离前端，独立的开发后端，在测试上获得更大的便利。这种途径也使得一个移动应用和网页应用可以使用相同的后端。

当使用一个API时，其中一个挑战就是认证（authentication）。在传统的web应用中，服务端成功的返回一个响应（response）依赖于两件事。一是，他通过一种存储机制保存了会话信息（Session）。每一个会话都有它独特的信息（id），常常是一个长的，随机化的字符串，它被用来让未来的请求（Request）检索信息。其次，包含在响应头（Header）里面的信息使客户端保存了一个Cookie。服务器自动的在每个子请求里面加上了会话ID，这使得服务器可以通过检索Session中的信息来辨别用户。这就是传统的web应用逃避HTTP面向无连接的方法（This is how traditional web applications get around the fact that HTTP is stateless）。

API应该被设计成无状态的（Stateless）。这意味着没有登陆，注销的方法，也没有sessions，API的设计者同样也不能依赖Cookie，因为不能保证这些request是由浏览器所发出的。自然，我们需要一个新的机制。这篇文章关注于JSON Web Tokens，简写为JWTs，一个可能的解决这个问题的机制。这篇文章利用Node的Express框架作为后端，以及Backbone作为前端。

##背景
我们来简短的看一下几个通常的保护（secure）API的方法。

一个是使用在HTTP规范中所制定的Basic Auth， 它需要在在响应中设定一个验证身份的Header。客户端必须在每个子响应是附加它们的凭证（credenbtial），包括它的密码。如果这些凭证通过了，那么用户的信息就会被传递到服务端应用。

第二个方面有点类似，但是使用应用自己的验证机制。通常包括将发送的凭证与存储的凭证进行检查。和Basic Auth相比，这种需要在每次请求（call）中发送凭证。

第三种是OAuth（或者OAuth2）。为第三方的认证所设计，但是更难配置。至少在服务器端更难。

##使用Token的方法
不是在每一次请求时提供用户名和密码的凭证。我们可以让用户通过token交换凭证（we can allow the client to exchange valid credentials for a token)，这个token提供用户访问服务器的权限。Token通常比密码更加长而且复杂。比如说，JWTs通常会应对长达150个字符。一旦获得了token，在每次调用API的时候都要附加上它。然后，这仍然比直接发送账户和密码更加安全，哪怕是HTTPS。

把token想象成一个安全的护照。你在一个安全的前台验证你的身份（通过你的用户名和密码），如果你成功验证了自己，你就可以取得这个。当你走进大楼的时候（试图从调用API获取资源），你会被要求验证你的护照，而不是在前台重新验证。

##关于JWTs
JWTs是一份草案，尽管在本质上它是一个老生常谈的一种更加具体的认证个授权的机制。一个JWT被周期（period）分寸了三个部分。JWT是URL-safe的，意味着可以用来查询字符参数。（译者注：也就是可以脱离URL，不用考虑URL的信息）。

JWT的第一部分是一个js对象，表面JWT的加密方法。实例使用了HMAC SHA-266
```
{
  "typ" : "JWT",
  "alg" : "HS256"
}
```
在加密之后，这个对象变成了一个字符串：
```
eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9
```
JWT的第二部分是token的核心，他也是一个JS兑现，包含了一些信息。有一些是必须的，有一些是选择性的。一个实例如下：
```
{
  "iss": "joe",
  "exp": 1300819380,
  "http://example.com/is_root": true
}
```
这被称为JWT Claims Set。因为这篇文章的目的，我们将忽视第三个参数。但是你可以阅读这篇[文章](http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-19).这个```iss```是```issuer```的简写，表明请求的实体。通常意味着请求API的用户。```exp```是```expires```的简写，是用来限制token的生命周期。一旦加密，JSON token就像这样：
```
eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ
```
第三个也是最后一个部分，是JWT根据第一部分和第二部分的签名（Signature）。像这个样子：
```
dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```
整个的JWT是这样的
```
eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
```
在规范中，有一些选择性的附加属性。有```iat```表明什么时候token被半吧，```nbf```去验证在什么时间之前token无效，和```aud```去指明这个token的收件人是谁。

##处理Tokens
我们将用[JWT simple](https://github.com/hokaccha/node-jwt-simple)模块去处理token，它将使我们从钻研如何加密解密中解脱出来。如果你有兴趣，可以阅读这篇[说明](http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-19)，或者读这个仓库的源码。
首先我们将使用下面的命令安装这个库。记住你可以在命令中加入```--save```，让其自动的让其加入到你的```package.json```文件里面。
```
npm install jwt-simple
```
在你应用的初始环节，加入以下代码。这个代码引入了Express和JWT simple，而且创建了一个新的Express应用。最后一行设定了app的一个名为```jwtTokenSecret```的变量，其值为‘YOUR_SECRET_STRING’（记得把它换成别的）。

```
var express = require('express');
var jwt = require('jwt-simple');
var app = express();

app.set('jwtTokenSecret', 'YOUR_SECRET_STRING');
```

##获取一个Token
我们需要做的第一件事就是让客户端通过他们的账号密码交换token。这里有2种可能的方法在RESTful API里面。第一种是使用**POST**请求来通过验证，使服务端发送带有token的响应。除此之外，你可以使用**GET**请求，这需要他们使用参数提供凭证（指URL），或者更好的使用请求头。

这篇文章的目的是为了解释token验证的方法而不是基本的用户名/密码验证机制。所以我们假设我们已经通过请求得到了用户名和密码：
```
User.findOne({ username: username }, function(err, user) {
  if (err) {
    // user not found
    return res.send(401);
  }

  if (!user) {
    // incorrect username
    return res.send(401);
  }

  if (!user.validPassword(password)) {
    // incorrect password
    return res.send(401);
  }

  // User has authenticated OK
  res.send(200);
});
```
下一步，我们就需要返回JWT token通过一个验证成功的响应。
```
var expires = moment().add('days', 7).valueOf();
var token = jwt.encode({
  iss: user.id,
  exp: expires
}, app.get('jwtTokenSecret'));

res.json({
  token : token,
  expires: expires,
  user: user.toJSON()
});
```
注意到```jwt.encode()```函数有2个参数。第一个就是一个需要加密的对象，第二个是一个加密的密钥。这个token是由我们之前提到的```iss```和```exp```组成的。注意到[Moment.js](http://momentjs.com/)被用来设置token将在7天之后失效。而```res.json()```方法用来传递这个JSON对象给客户端。

##验证Token
为了验证JWT，我们需要写出一些可以完成这些功能的中间件（Middleware）：

- 检查附上的token
- 试图解密
- 验证token的可用性
- 如果token是合法的，检索里面用户的信息，以及附加到请求的对象上

我们来写一个中间件的框架
```
// @file jwtauth.js

var UserModel = require('../models/user');
var jwt = require('jwt-simple');

module.exports = function(req, res, next) {
  // code goes here
};
```
为了获得最大的可扩展性，我们允许客户端使用一下3个方法附加我们的token：作为请求链接（query)的参数，作为主体的参数（body），和作为请求头（Header）的参数。对于最后一个，我们将使用Header ```x-access-token```。

下面是我们的允许在中间件的代码，试图去检索token：
```
var token = (req.body && req.body.access_token) || (req.query && req.query.access_token) || req.headers['x-access-token'];
```
注意到他为了访问```req.body```，我们需要首先使用```express.bodyParser()```中间件（译者注，这个是Express 3.x的中间件）。

下一步，我们讲解析JWT:
```
if (token) {
  try {
    var decoded = jwt.decode(token, app.get('jwtTokenSecret'));

    // handle token here

  } catch (err) {
    return next();
  }
} else {
  next();
}
```
如果解析的过程失败，那么JWT Simple组件将会抛出一段异常。如果异常发生了，或者没有token，我们将会调用```next()```来继续处理请求。这代表喆我们无法确定用户。如果一个合格的token合法并且被解码，我们应该得到2个属性，```iss```包含着用户ID以及```exp```包含token过期的时间戳。我们将首先处理后者，如果它过期了，我们就拒绝它：
```
if (decoded.exp <= Date.now()) {
  res.end('Access token has expired', 400);
}
```
如果token依旧合法，我们可以从中检索出用户信息，并且附加到请求对象里面去：
```
User.findOne({ _id: decoded.iss }, function(err, user) {
  req.user = user;
});
```
最后，将这个中间件附加到路由里面：
```
var jwtauth = require('./jwtauth.js');

app.get('/something', [express.bodyParser(), jwtauth], function(req, res){
  // do something
});
```
或者匹配一些路由
```
app.all('/api/*', [express.bodyParser(), jwtauth]);
```

##客户端
我们提供了一个简单的```get```端去获得一个远端的token。这非常直接了，所以我们不用纠结细节，就是发起一个请求，传递用户名和密码，如果请求成功了，我们就会得到一个包含着token的响应。

我们现在研究的是后续的请求。一个方法是通过JQuery的```ajaxSetup()```方法。这可以直接用来做Ajax请求，或者通过前端框架使用包装过的Ajax方法。比如，假设我们将我们的请求使用```window.localStorage.setItem('token', 'the-long-access-token'); ```放在本地存储（Local Storage）里面，我们可以通过这种方法将token附加到请求头里面：

```
var token = window.localStorage.getItem('token');

if (token) {
  $.ajaxSetup({
    headers: {
      'x-access-token': token
    }
  });
}
```
很简单，但是这会劫持所有Ajax请求，如果这里有一个token在本地存储里面。它将会附加到一个名为```x-access-token```的Header里面。

##使用Backbone
我们将前一个方法换成Backbone应用。最简单的方法就是使用全局的```Backbone.sync()```，如下面所示
```
// Store "old" sync function
var backboneSync = Backbone.sync

// Now override
Backbone.sync = function (method, model, options) {

  /*
   * "options" represents the options passed to the underlying $.ajax call
   */
  var token = window.localStorage.getItem('token');

  if (token) {
    options.headers = {
      'x-access-token': token
    }
  }

  // call the original function
  backboneSync(method, model, options);
};
```

##更多的安全
你可以保存签证过的token记录在服务器上，来添加一个附加的安全层，，然后在每一步验证token的时候验证这个记录。这将会组织第三方伪装一个token，也将会使得服务器可以失效一个token。我不会提到这个方面，但是它应当被直接的实现。

##总结
在这篇文章，我们探究了一个API验证的方法，仔细的查看了JSON WEB Tokens。我们使用了Node和Express来写一个简单的实现，而且也在客户端使用了Backbone作为一个示例。代码可以在[GitHub](https://github.com/lukaswhite/jwt-node-express)上找到。

这里还有很多我们都没有完全实现到，比如资源的请求(claim），但是我们已经做了如何使用简单的方法来构建一个获取token的机制。在这个例子里面，客户端和服务端都是JS应用。

当然你可以通过其他技术使用这个方法，比如Ruby或者PHP作为后端，或者Ember或者AngularJS。除此之外，你还可以在移动应用中使用。比如，使用web技术和PhoneGap之类的结合，使用类似于Sencha之类的工具，或者完全的本地应用。
