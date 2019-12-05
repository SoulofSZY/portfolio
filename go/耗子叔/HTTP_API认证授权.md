##HTTP API 认证授权
应对实际场景

1. 用户访问
2. 用户委托第三方应用
3. 企业与企业间调用

### 常用API认证技术

1. HTTP Basic
2. Digest Access
3. App Secret Key + HMAC
4. JWT - JSON Web Tokens
5. OAuth 1.0 - 3 legged & 2 legged
6. OAuth 2.0 - Authentication Code & Client Credential

#### HTTP Basic
> HTTP Basic 是一个非常传统的API认证技术，也是一个比较简单的技术。这个技术也就是使用 username和 password 来进行登录。整个过程被定义在了 RFC 2617 中

**技术原理**

1. 把 username和 password 做成  username:password 的样子（用冒号分隔）
2. 进行Base64编码。Base64("username:password") 得到一个字符串token
3. 把 2中token 放到HTTP头中 Authorization 字段中，形成 Authorization: Basic token，然后发送到服务端。
4. 服务端如果没有在头里看到认证字段，则返回401，以及一个WWW-Authenticate: Basic Realm='HelloWorld' 之类的头要求客户端进行认证。之后如果没有认证通过，则返回一个401。如果服务端认证通过，那么会返回200。

**总结**

这种把用户名和密码同时放在公网上传输的方式有点不太好，因为Base64（解决特殊字符的问题）不是加密协议，而是编码协议，所以就算是有HTTPS作为安全保护，给人的感觉还是不放心。

####Digest Access
> 中文称“HTTP 摘要认证”，最初被定义在了 RFC 2069 文档中（后来被 RFC 2617 引入了一系列安全增强的选项；“保护质量”(qop)、随机数计数器由客户端增加、以及客户生成的随机数）。

**技术原理**

请求方把用户名口令和域做一个MD5 –  MD5(username:realm:password) 然后传给服务器，这样就不会在网上传用户名和口令了，但是，因为用户名和口令基本不会变，所以，这个MD5的字符串也是比较固定的，因此，这个认证过程在其中加入了两个事，一个是 nonce 另一个是 qop

1. 调用方发起一个普通的HTTP请求
2. 服务端自然不能认证通过，服务端返回401错误，并且在HTTP头里的 WWW-Authenticate 包含如下信息：
WWW-Authenticate: Digest realm="testrealm@host.com",
qop="auth,auth-int",
nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
opaque="5ccc069c403ebaf9f0171e9517f40e41"
3. 其中的 nonce 为服务器端生成的随机数，然后，客户端做 HASH1=MD5(MD5(username:realm:password):nonce:cnonce) ，其中的 cnonce 为客户端生成的随机数，这样就可以使得整个MD5的结果是不一样的。
4. 如果 qop 中包含了 auth ，那么还得做  HASH2=MD5(method:digestURI) 其中的 method 就是HTTP的请求方法（GET/POST…），digestURI 是请求的URL。
5. 如果qop中包含了auth-init那么得做HASH2=MD5(method:digestURI:MD5(entityBody)) 其中的 entityBody 就是HTTP请求的整个数据体。
6. 然后，得到 response = MD5(HASH1:nonce:nonceCount:cnonce:qop:HASH2) 如果没有 qop则 response = MD5(HA1:nonce:HA2)
7. 最后，我们的客户端对服务端发起如下请求—— 注意HTTP头的 Authorization: Digest ...
    GET /dir/index.html HTTP/1.0
	Host: localhost
	Authorization: Digest username="Mufasa",
                   realm="testrealm@host.com",
                  nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
                     uri="%2Fcoolshell%2Fadmin",
                     qop=auth,
                     nc=00000001,
                     cnonce="0a4f113b",
                 response="6629fae49393a05397450978507c4ef1",
                    opaque="5ccc069c403ebaf9f0171e9517f40e41"

**总结**

摘要认证这个方式会比之前的方式要好一些，因为没有在网上传递用户的密码，而只是把密码的MD5传送过去，相对会比较安全，而且，其并不需要是否TLS/SSL的安全链接。但是，别看这个算法这么复杂，最后你可以发现，整个过程其实关键是用户的password，这个password如果不够复杂，其实是可以被暴力破解的，而且，整个过程是非常容易受到中间人攻击——比如一个中间人告诉客户端需要的 Basic 的认证方式 或是 老旧签名认证方式（RFC2069）

#### App Secret Key + HMAC
> 先说HMAC技术，这个东西来自于MAC – Message Authentication Code，是一种用于给消息签名的技术，也就是说，我们怕消息在传递的过程中被人修改，所以，我们需要用对消息进行一个MAC算法，得到一个摘要字串，然后，接收方得到消息后，进行同样的计算，然后比较这个MAC字符串，如果一致，则表明没有被修改过（整个过程参看下图）。而HMAC – Hash-based Authenticsation Code，指的是利用Hash技术完成这一工作，比如：SHA-256算法。  比如说u盾这类东东

App ID，这个东西跟验证没有关系，只是用来区分，是谁来调用API的，就像我们每个人的身份证一样，只是用来标注不同的人，不是用来做身份认证的。与前面的不同之处是，这里，我们需要用App ID 来映射一个用于加密的密钥，这样一来，我们就可以在服务器端进行相关的管理，我们可以生成若干个密钥对（AppID, AppSecret），并可以有更细粒度的操作权限管理

对称加密

**总结**
这种认证的方式好处在于，AppID和AppSecretKey，是由服务器的系统开出的，所以，是可以被管理的。但是不好的地方在于，这个东西没有标准 ，所以，各家的实现很不一致

#### JWT - JSON Web Tokens
JWT是一个比较标准的认证解决方案，这个技术在Java圈里应该用的是非常普遍的。JWT签名也是一种MAC（Message Authentication Code）的方法

**技术流程**

1. 用户使用用户名和口令到认证服务器上请求认证
2. 认证服务器验证用户名和口令后，以服务器端生成JWT Token，这个token的生成过程：
	1. 认证服务器还会生成一个 Secret Key
	2. 对JWT Header和 JWT Payload分别求Base64。在Payload可能包括了用户的抽象ID和的过期时间。
	3. 用密钥对JWT签名 HMAC-SHA256(SecertKey, Base64UrlEncode(JWT-Header)+'.'+Base64UrlEncode(JWT-Payload));
3. 然后把 base64(header).base64(payload).signature 作为 JWT token返回客户端
4. 客户端使用JWT Token向应用服务器发送相关的请求。这个JWT Token就像一个临时用户权证一样。
5. 应用服务器收到请求后，会检查 JWT  Token，确认签名是正确的
6. 因为只有认证服务器有这个用户的Secret Key（密钥），所以，应用服务器得把JWT Token传给认证服务器。
7. 认证服务器通过JWT Payload 解出用户的抽象ID，然后通过抽象ID查到登录时生成的Secret Key，然后再来检查一下签名
8. 认证服务器检查通过后，应用服务就可以认为这是合法请求了

**总结**

上面的这个过程，是在认证服务器上为用户动态生成 Secret Key的，应用服务在验签的时候，需要到认证服务器上去签，这个过程增加了一些网络调用，JWT除了支持HMAC-SHA256的算法外，还支持RSA的非对称加密的算法

使用RSA非对称算法，在认证服务器这边放一个私钥，在应用服务器那边放一个公钥，认证服务器使用私钥加密，应用服务器使用公钥解密，这样一来，就不需要应用服务器向认证服务器请求了，但是，RSA是一个很慢的算法，故，虽然省了网络调用，但是却费了CPU，尤其是Header和Payload比较长的时候。所以，一种比较好的玩法是，如果我们把header 和 payload简单地做SHA256，这会很快，然后，我们用RSA加密这个SHA256出来的字符串，这样一来，RSA算法就比较快了，而我们也做到了使用RSA签名的目的。


#### OAuth 1.0 - 3 legged & 2 legged
委托授权协议

> OAuth也是一个API认证的协议，这个协议最初在2006年由Twitter的工程师在开发OpenID实现的时候和社交书签网站Ma.gnolia时发现，没有一种好的委托授权协议，后来在2007年成立了一个OAuth小组，知道这个消息后，Google员工也加入进来，并完善有善了这个协议，在2007年底发布草案，过一年后，在2008年将OAuth放进了IETF作进一步的标准化工作，最后在2010年4月，正式发布OAuth 1.0，即：RFC 5849 （这个RFC比起TCP的那些来说读起来还是很轻松的）

**案例**

根据RFC 5849，可以看到 OAuth 的出现，目的是，用户为了想使用一个第三方的网络打印服务来打印他在某网站上的照片，但是，用户不想把自己的用户名和口令交给那个第三方的网络打印服务，但又想让那个第三方的网络打印服务来访问自己的照片，为了解决这个授权的问题，OAuth这个协议就出来了。

协议有三个角色

1. User  用户
2. Consumer 第三方服务
3. Service Provider app服务

协议有三个阶段

1. Consumer获取Request Token
2. Service Provider 认证用户并授权Consumer
3. Consumer获取Access Token调用API访问Service Provider相关服务

**授权流程**

三脚流程

1. Consumer需要先上Service Provider获得开发的 Consumer Key 和 Consumer Secret
2. 当 User 访问 Consumer 时，Consumer 向 Service Provide 发起请求请求Request Token （需要对HTTP请求签名）
3. Service Provide 验明 Consumer 是注册过的第三方服务商后，返回 Request Token（oauth_token）和 Request Token Secret （oauth_token_secret）
4. Consumer 收到 Request Token 后，使用HTTP GET 请求把 User 切到 Service Provide 的认证页上（其中带上Request Token），让用户输入他的用户和口令。
5. Service Provider 认证 User 成功后，跳回 Consumer，并返回 Request Token （oauth_token）和 Verification Code（oauth_verifier）
6. 接下来就是签名请求，用Request Token 和 Verification Code 换取 Access Token （oauth_token）和 Access Token Secret (oauth_token_secret)
7. 最后使用Access Token 访问用户授权访问的资源。

两脚流程

1. Consumer需要先上Service Provider获得开发的 Consumer Key 和 Consumer Secret
2. Consumer（第三方照片打印服务）需要先上Service Provider获得开发的 Consumer Key 和 Consumer Secret
3. Service Provide 验明 Consumer 是注册过的第三方服务商后，返回 Request Token（oauth_token）和 Request Token Secret （oauth_token_secret）
4. Consumer 收到 Request Token 后，直接换取 Access Token （oauth_token）和 Access Token Secret (oauth_token_secret)
5. 最后使用Access Token 访问用户授权访问的资源。


**OAuth中的签名**

1. 有两个密钥，一个是Consumer注册Service Provider时由Provider颁发的 Consumer Secret，另一个是 Token Secret。
2. 签名密钥就是由这两具密钥拼接而成的，其中用 &作连接符。假设 Consumer Secret 为 j49sk3j29djd 而 Token Secret 为dh893hdasih9那个，签名密钥为：j49sk3j29djd&dh893hdasih9
3. 在请求Request/Access Token的时候需要对整个HTTP请求进行签名（使用HMAC-SHA1和HMAC-RSA1签名算法），请求头中需要包括一些OAuth需要的字段
	1. Consumer Key ： 也就是所谓的AppID
	2. Token： Request Token 或 Access Token
	3. Signature Method ：签名算法比如：HMAC-SHA1
	4. Timestamp：过期时间
	5. Nonce：随机字符串
	6. Call Back：回调URL


#### OAuth 2.0

**历史**
在前面，我们可以看到，从Digest Access， 到AppID+HMAC，再到JWT，再到OAuth 1.0，这些个API认证都是要向Client发一个密钥（或是用密码）然后用HASH或是RSA来签HTTP的请求，这其中有个主要的原因是，以前的HTTP是明文传输，所以，在传输过程中很容易被篡改，于是才搞出来一套的安全签名机制，所以，这些个认证的玩法是可以在HTTP明文协议下玩的
这种使用签名方式大家可以看到是比较复杂的，所以，对于开发者来说，也是很不友好的，在组织签名的那些HTTP报文的时候，各种，URLEncode和Base64，还要对Query的参数进行排序，然后有的方法还要层层签名，非常容易出错，另外，这种认证的安全粒度比较粗，授权也比较单一，对于有终端用户参与的移动端来说也有点不够。所以，在2012年的时候，OAuth 2.0 的 RFC 6749 正式放出。

OAuth 2.0依赖于TLS/SSL的链路加密技术（HTTPS），完全放弃了签名的方式，认证服务器再也不返回什么 token secret 的密钥了，所以，OAuth 2.0是完全不同于1.0 的，也是不兼容的

**主要Flow**

1. 一个是Authorization Code Flow， 这个是 3 legged 的
2. 一个是Client Credential Flow，这个是 2 legged 的。


**Authorization Code Flow**

Authorization Code 是最常使用的OAuth 2.0的授权许可类型，它适用于用户给第三方应用授权访问自己信息的场景

基本流程

1. 当用户（Resource Owner）访问第三方应用（Client）的时候，第三方应用会把用户带到认证服务器（Authorization Server）上去，主要请求的是 /authorize API，其中的请求方式如下所示。
https://login.authorization-server.com/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Fexample-client.com%2Fcallback%2F
&scope=read
&state=xcoiv98CoolShell3kch
	1. client_id为第三方应用的App ID
	2. response_type=code为告诉认证服务器，我要走Authorization Code Flow。
	3. redirect_uri意思是我跳转回第三方应用的URL
	4. scope意是相关的权限
	5. state 是一个随机的字符串，主要用于防CSRF攻击。
2. 当Authorization Server收到这个URL请求后，其会通过 client_id来检查 redirect_uri和 scope是否合法，如果合法，则弹出一个页面，让用户授权（如果用户没有登录，则先让用户登录，登录完成后，出现授权访问页面）。
3. 当用户授权同意访问以后，Authorization Server 会跳转回 Client ，并以其中加入一个 Authorization Code。 如下所示：
https://example-client.com/callback?
code=Yzk5ZDczMzRlNDEwYlrEqdFSBzjqfTG
&state=xcoiv98CoolShell3kch
4. 接下来，Client 就可以使用 Authorization Code 获得 Access Token。其需要向 Authorization Server 发出如下请求。
POST /oauth/token HTTP/1.1
Host: authorization-server.com
code=Yzk5ZDczMzRlNDEwYlrEqdFSBzjqfTG
&grant_type=code
&redirect_uri=https%3A%2F%2Fexample-client.com%2Fcallback%2F
&client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&client_secret=JqQX2PNo9bpM0uEihUPzyrh
5. 如果没什么问题，Authorization 会返回如下信息。
{
  "access_token": "iJKV1QiLCJhbGciOiJSUzI1NiI",
  "refresh_token": "1KaPlrEqdFSBzjqfTGAMxZGU",
  "token_type": "bearer",
  "expires": 3600,
  "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciO.eyJhdWQiOiIyZDRkM..."
}
	1. access_token就是访问请求令牌了
	2. refresh_token用于刷新 access_token
	3. id_token 是JWT的token，其中一般会包含用户的OpenID
6. 接下来就是用 Access Token 请求用户的资源了。
GET /v1/user/pictures
Host: https://example.resource.com

Authorization: Bearer iJKV1QiLCJhbGciOiJSUzI1NiI

**Client Credential Flow**

Client Credential 是一个简化版的API认证，主要是用于认证服务器到服务器的调用，也就是没有用户参与的的认证流程
本质上就是Client用自己的 client_id和 client_secret向Authorization Server 要一个 Access Token，然后使用Access Token访问相关的资源










