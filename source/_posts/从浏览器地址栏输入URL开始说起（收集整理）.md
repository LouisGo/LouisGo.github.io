---
title: 从浏览器地址栏输入URL开始说起（搬运整理）
date: 2018-04-25 13:45:05
tags: 
- HTTP
- 浏览器
- CSS
- JavaScript
- Web安全
categories: 
- 前端
keywords: HTTP,TCP/IP,浏览器渲染,网络请求,CSS可视化格式模型,前端知识体系
---

<div class="note info"><p>本文是基于 [从输入URL到页面加载的过程？如何由一道题完善自己的前端知识体系！](HTTP://www.dailichun.com/2018/03/12/whenyouenteraurl.html):100: 的搬运和整理，主要是为了进一步巩固知识，感谢大佬，侵删。</p></div>

# Step1. 从浏览器接收URL到开启网络请求线程

首先要了解浏览器的多进程，其中渲染进程里又有许多线程去处理各个任务。

## 多进程的浏览器

- 主控进程：浏览器的主进程（负责协调、主控），只有一个

- 第三方插件进程：每种类型的插件对应一个进程，仅当使用该插件时才创建

- GPU进程：最多一个，用于3D绘制

- **浏览器渲染进程（内核）**：默认每个Tab页面一个进程，互不影响，控制页面渲染，脚本执行，事件处理等（有时候会优化，如多个空白Tab会合并成一个进程）

## 多线程的浏览器渲染进程（内核）

- GUI线程

- JS引擎线程

- 事件触发线程

- 定时器线程

- 网络请求线程

<!--more-->

## 解析URL

输入URL后，会进行解析（**URL的本质就是统一资源定位符**），URL的组成分为：

- `protocol`：协议头，譬如有`HTTP`，`ftp`等

- `host`：主机域名或IP地址

- `port`：端口号

- `path`：目录路径

- `query`：即查询参数

- `fragment`：即#后的`hash`值，一般用来定位到某个位置

## 更多

[从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理](https://segmentfault.com/a/1190000012925872)

# Step2. 开启网络线程到发出一个完整的http请求

## DNS查询得到IP

- 如果浏览器有缓存，直接使用浏览器缓存，否则使用本机缓存，再没有的话就是用`host`

- 如果本地没有，就向`DNS`域名服务器查询（当然，中间可能还会经过路由，也有缓存等），查询到对应的IP

## TCP/IP请求

### 建立连接：三次握手

- 浏览器向服务器发送想建立连接的请求（你好，可以认识一下吗）

- 服务器向浏览器发送同意建立连接的响应（你好，当然可以啊）

- 浏览器向服务器发送确认收到响应的请求，客户端和服务器建立连接（非常高兴认识你）


### 相互通信

- 浏览器向服务器发起一个请求网页资源的请求

- 服务器返回对应网页资源

- 浏览器渲染、构建网页，在构建网页的过程中，可能会继续请求`css`、`JavaScript`等资源

### 断开连接：四次挥手（返回响应结果后，双方都没有请求时）

- 浏览器向服务器发送想断开连接的请求（我要走啦）

- 服务器向浏览器发送收到请求的响应（我知道啦）

- 服务器向浏览器发送断开连接的请求（可以了，我走啦）

- 浏览器断开连接并向服务器发送一个反馈请求，服务器收到后断开连接（好的，拜拜）

### 并发限制

浏览器对同一域名下并发的`TCP`连接是有限制的（2-10个不等）

而且在`HTTP1.0`中往往一个资源下载就需要对应一个`TCP/IP`请求

### get和post的区别

`get`和`post`虽然本质都是`TCP/IP`，但两者除了在`HTTP`层面外，在`TCP/IP`层面也有区别。

`get`会产生一个`TCP`数据包，post两个：

- `get`请求时，浏览器会把`headers`和`data`一起发送出去，服务器响应200（返回数据）

- `post`请求时，浏览器先发送`headers`，服务器响应`100 continue`， 浏览器再发送`data`，服务器响应`200`（返回数据）

这里的区别是specification（规范）层面，而不是implementation（对规范的实现）。

## 五层因特网协议

1. `应用层（DNS, http）`：DNS解析成IP并发送http请求

2. `传输层（tcp, udp）`：建立tcp连接（三次握手）

3. `网络层（IP, ARP）`：IP寻址

4. `数据链路层（PPP）`：封装成帧

5. `物理层（利用物理介质传输比特流）`：物理传输（然后传输的时候通过双绞线，电磁波等各种介质）

# Step3. 从服务器接收到请求到对应后台接收到请求

后台方面的知识，略。

# By 负载均衡

对于大型的项目，由于并发访问量很大，所以往往一台服务器是吃不消的，其中一种方式是一般会有若干台服务器组成一个集群，然后配合反向代理实现负载均衡。

简单的说：

**用户发起的请求都指向调度服务器（反向代理服务器，譬如安装了`nginx`控制负载均衡），然后调度服务器根据实际的调度算法，分配不同的请求给对应集群中的服务器执行，然后调度器等待实际服务器的`HTTP`响应，并将它反馈给用户。**

# By 后台处理

一般后台都是部署到容器中的，所以一般为：

1. 容器接受到请求（如`tomcat`容器）

2. 对应容器中的后台程序接收到请求（如`java`程序）

3. 后台会有自己的统一处理，处理完后响应响应结果

详细地说：

1. 一般有的后端是有统一的验证的，如安全拦截，跨域验证

2. 如果这一步不符合规则，就直接返回了相应的`HTTP`报文（如拒绝请求等）

3. 当验证通过后，才会进入实际的后台代码，此时是程序接收到请求，然后执行（譬如查询数据库，大量计算等等）

4. 等程序执行完毕后，就会返回一个`HTTP`响应包（一般这一步也会经过多层封装）

5. 将这个包从后端发送到前端，完成交互

# By 后台和前台的HTTP交互

前后端交互时，`HTTP`报文作为信息的载体。

## HTTP报文机构

### 通用头部

- `Request Url`：请求的web服务器地址

- `Request Method`：请求方式有多种

    - `HTTP1.0`定义了三种请求方法： `GET`, `POST`和 `HEAD`方法。
以及几种`Additional Request Methods`：`PUT`、`DELETE`、`LINK`、`UNLINK`

    - `HTTP1.1`定义了八种请求方法：`GET`、`POST`、`HEAD`、`OPTIONS`, `PUT`, `DELETE`, `TRACE`和 `CONNECT`方法。

- `Status Code`：请求的返回状态码，如`200`代表成功

- `Remote Address`：请求的远程服务器地址（会转为IP）

### 状态码

- {% label default@1XX %} 指示信息，表示请求已接收，继续处理

- {% label success@2XX %} 成功，表示请求已被成功接收、理解、接受

- {% label warning@3XX %} 重定向，要完成请求必须进行更进一步的操作

- {% label danger@4XX %} 客户端错误，请求有语法错误或请求无法实现

- {% label danger@5XX %} 服务器端错误，服务器未能实现合法的请求

### 常用请求头（Request Headers）

- `Accept`：接收类型，表示浏览器支持的MIME类型
（对标服务端返回的`Content-Type`）

- `Accept-Encoding`：浏览器支持的压缩类型,如`gzip`等,超出类型不能接收

- `Content-Type`：客户端发送出去实体内容的类型

- `Cache-Control`：指定请求和响应遵循的缓存机制，如`no-cache`

- `If-Modified-Since`：对应服务端的`Last-Modified`，用来匹配看文件是否变动，只能精确到1s之内，`HTTP1.0`中

- `Expires`：缓存控制，在这个时间内不会请求，直接使用缓存，`HTTP1.0`，而且是服务端时间

- `Max-age`：代表资源在本地缓存多少秒，有效时间内不会请求，而是使用缓存，`HTTP1.1`中

- `If-None-Match`：对应服务端的`ETag`，用来匹配文件内容是否改变（非常精确），`HTTP1.1`中

- `Cookie`：有`Cookie`并且同域访问时会自动带上

- `Connection`当浏览器与服务器通信时对于长连接如何进行处理,如`keep-alive`

- `Host`：请求的服务器`URL`

- `Origin`：最初的请求是从哪里发起的（只会精确到端口），`Origin`比`Referer`更尊重隐私

- `Referer`：该页面的来源`URL`(适用于所有类型的请求，会精确到详细页面地址，`csrf`拦截常用到这个字段)

- `User-Agent`：用户客户端的一些必要信息，如`UA头部`等

### 常用响应头（Response Headers）

- `Access-Control-Allow-Headers`：服务器端允许的请求`Headers`

- `Access-Control-Allow-Methods`：服务器端允许的请求方法

- `Access-Control-Allow-Origin`：服务器端允许的请求`Origin头部`（譬如为*）

- `Content-Type`：服务端返回的实体内容的类型

- `Date`：数据从服务器发送的时间

- `Cache-Control`：告诉浏览器或其他客户，什么环境可以安全的缓存文档

- `Last-Modified`：请求资源的最后修改时间

- `Expires`：应该在什么时候认为文档已经过期,从而不再缓存它

- `Max-age`：客户端的本地资源应该缓存多少秒，开启了`Cache-Control`后有效

- `ETag`：请求变量的实体标签的当前值

- `Set-Cookie`：设置和页面关联的`Cookie`，服务器通过这个头部把`Cookie`传给客户端

- `Keep-Alive`：如果客户端有`keep-alive`，服务端也会有响应（如`timeout=38`）

- `Server`：服务器的一些相关信息

### 请求/响应实体

`HTTP`请求时，除了头部，还有消息实体，一般来说

请求实体中会将一些需要的参数都放入进入（用于`post`请求）。

譬如实体中可以放参数的序列化形式（a=1&b=2这种），或者直接放表单对象（`Form Data`对象，上传时可以夹杂参数以及文件），等等

而一般响应实体中，就是放服务端需要传给客户端的内容

一般现在的接口请求时，实体中就是对于的信息的`json`格式，而像页面请求这种，里面就是直接放了一个`html`字符串，然后浏览器自己解析并渲染。

### CRLF

`CRLF（Carriage-Return Line-Feed）`，意思是回车换行，一般作为分隔符存在

请求头和实体消息之间有一个`CRLF`分隔，响应头部和响应实体之间用一个`CRLF`分隔

### 某http报文结构的简要分析

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/http_ajax_headers.png)

## Cookie以及优化

`Cookie`是浏览器的一种本地存储方式，一般用来帮助客户端和服务端通信的，常用来进行身份校验，结合服务端的`Session`使用。

常用使用场景简述：

1. 在登录页面，用户登录了

2. 此时，服务端会生成一个`Session`，`Session`中有对于用户的信息（如用户名、密码等）

3. 然后会有一个`sessionid`（相当于是服务端的这个`Session`对应的`key`）

4. 然后服务端在登录页面中写入`Cookie`，值就是:`jsessionid=xxx`

5. 然后浏览器本地就有这个`Cookie`了，以后访问同域名下的页面时，自动带上`Cookie`，自动检验，在有效时间内无需二次登录。

一般来说，**`Cookie`是不允许存放敏感信息的（千万不要明文存储用户名、密码）**，因为非常不安全，如果一定要强行存储，首先，一定要在`Cookie`中设置`httponly`（这样就无法通过js操作了），另外可以考虑`rsa`等非对称加密（因为实际上，浏览器本地也是容易被攻克的，并不安全）

另外，由于在同域名的资源请求时，浏览器会默认带上本地的`Cookie`，针对这种情况，在某些场景下是需要优化的。

譬如以下场景：

1. 客户端在域名A下有`Cookie`（这个可以是登录时由服务端写入的）

2. 然后在域名A下有一个页面，页面中有很多依赖的静态资源（都是域名A的，譬如有20个静态资源）

3. 此时就有一个问题，页面加载，请求这些静态资源时，浏览器会默认带上`Cookie`

4. 也就是说，这20个静态资源的`HTTP`请求，每一个都得带上`Cookie`，而实际上静态资源并不需要`Cookie`验证

5. 此时就造成了较为严重的浪费，而且也降低了访问速度（因为内容更多了）

针对这种场景，可以进行多域名拆分优化：

- 将静态资源分组，分别放到不同的域名下（如`static.base.com`）

- 而`page.base.com`（页面所在域名）下请求时，是不会带上`static.base.com`域名的`Cookie`的，所以就避免了浪费

但是进行了多域名拆分，在移动端，如果请求的域名数过多，会降低请求速度（因为域名整套解析流程是很耗费时间的，而且移动端一般带宽都比不上pc）。

此时就需要用到一种优化方案：**`DNS-prefetch`**（让浏览器空闲时提前解析`DNS`域名，不过也请合理使用，勿滥用）。

**cookie总结**

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/http_cookie_session.png)

## gzip压缩

首先，明确`gzip`是一种压缩格式，需要浏览器支持才有效（不过一般现在浏览器都支持）， 而且`gzip`压缩效率很好（高达70%左右）

然后`gzip`一般是由`apache`、`tomcat`等web服务器开启

当然服务器除了`gzip`外，也还会有其它压缩格式（如`deflate`，没有`gzip`高效，且不流行）

所以一般只需要在服务器上开启了`gzip`压缩，然后之后的请求就都是基于`gzip`压缩格式的， 非常方便。

## 长连接与短连接

长连接的定义：一个`TCP/IP`连接上可以连续发送多个数据包，在`TCP`连接保持期间，如果没有数据包发送，需要双方发检测包以维持此连接，一般需要自己做在线维持（类似于心跳包）。

短连接的定义：通信双方有数据交互时，就建立一个`TCP`连接，数据发送完成后，则断开此`TCP`连接。

`HTTP1.0`中，默认使用的是短连接，也就是说，浏览器没进行一次`HTTP`操作，就建立一次连接，任务结束就中断连接，譬如每一个静态资源请求时都是一个单独的连接

`HTTP1.1`起，默认使用长连接，使用长连接会有这一行`Connection: keep-alive`，在长连接的情况下，当一个网页打开完成后，客户端和服务端之间用于传输`HTTP`的`TCP`连接不会关闭，如果客户端再次访问这个服务器的页面，会继续使用这一条已经建立的连接

## HTTP 2.0

`HTTP2.0`不是`https`，它相当于是`HTTP`的下一代规范（譬如`https`的请求可以是`HTTP2.0`规范的）。

`HTTP2.0`和`HTTP1.1`的不同之处：

- `HTTP1.1`中，每请求一个资源，都是需要开启一个`TCP/IP`连接的，所以对应的结果是，每一个资源对应一个`TCP/IP`请求，由于`TCP/IP`本身有并发数限制，所以当资源一多，速度就显著慢下来

- `HTTP2.0`中，一个`TCP/IP`请求可以请求多个资源，也就是说，只要一次`TCP/IP`请求，就可以请求若干个资源，分割成更小的帧请求，速度明显提升。

所以，如果`HTTP2.0`全面应用，很多`HTTP1.1`中的优化方案就无需用到了（譬如打包成精灵图，静态资源多域名拆分等）

然后简述下`HTTP2.0`的一些特性：

- 多路复用（即一个`TCP/IP`连接可以请求多个资源）

- 首部压缩（`HTTP`头部压缩，减少体积）

- 二进制分帧（在应用层跟传送层之间增加了一个二进制分帧层，改进传输性能，实现低延迟和高吞吐量）

- 服务器端推送（服务端可以对客户端的一个请求发出多个响应，可以主动通知客户端）

- 请求优先级（如果流被赋予了优先级，它就会基于这个优先级来处理，由服务器决定需要多少资源来处理该请求。）

## HTTPS

`https`就是安全版本的`HTTP`，譬如一些支付等操作基本都是基于`https`的，因为`HTTP`请求的安全系数太低了。

简单来看，`https`与`HTTP`的区别就是： 在请求前，会建立ssl链接，确保接下来的通信都是加密的，无法被轻易截取分析

一般来说，如果要将网站升级成`https`，需要后端支持（后端需要申请证书等），然后`https`的开销也比`HTTP`要大（因为需要额外建立安全链接以及加密等），所以一般来说http2.0配合`https`的体验更佳（因为`http2.0更`快了）

一般来说，主要关注的就是`SSL/TLS`的握手流程，如下：

1. 浏览器请求建立`SSL`链接，并向服务端发送一个随机数`–Client random`和客户端支持的加密方法，比如`RSA`加密，此时是明文传输。 

2. 服务端从中选出一组加密算法与`Hash`算法，回复一个随机数`–Server random`，并将自己的身份信息以证书的形式发回给浏览器
（证书里包含了网站地址，非对称加密的公钥，以及证书颁发机构等信息）

3. 浏览器收到服务端的证书后
    
    - 验证证书的合法性（颁发机构是否合法，证书中包含的网址是否和正在访问的一样），如果证书信任，则浏览器会显示一个小锁头，否则会有提示
    
    - 用户接收证书后（不管信不信任），浏览会生产新的随机数`–Premaster secret`，然后证书中的公钥以及指定的加密方法加密`Premaster secret`，发送给服务器。
    
    - 利用`Client random`、`Server random`和`Premaster secret`通过一定的算法生成`HTTP`链接数据传输的对称加密key-`Session key`
    
    - 使用约定好的`HASH`算法计算握手消息，并使用生成的`Session key`对消息进行加密，最后将之前生成的所有信息发送给服务端。 
    
4. 服务端收到浏览器的回复

    - 利用已知的加解密方式与自己的私钥进行解密，获取`Premaster secret`
    
    - 和浏览器相同规则生成`Session key`
    
    - 使用`Session key`解密浏览器发来的握手消息，并验证Hash是否与浏览器发来的一致
    
    - 使用`Session key`加密一段握手消息，发送给浏览器
    
5. 浏览器解密并计算握手消息的`HASH`，如果与服务端发来的`HASH`一致，此时握手过程结束，

**之后所有的`https`通信数据将由之前浏览器生成的`Session key`并利用对称加密算法进行加密**

# By HTTP缓存

## 强缓存与弱缓存

`强缓存（200 from cache）`时，浏览器如果判断本地缓存未过期，就直接使用，无需发起http请求

`协商缓存（304`）时，浏览器会向服务端发起`HTTP`请求，然后服务端告诉浏览器文件未改变，让浏览器使用本地缓存。

对于协商缓存，使用`Ctrl + F5`强制刷新可以使得缓存无效

但是对于强缓存，在未过期时，必须更新资源路径才能发起新的请求（更改了路径相当于是另一个资源了，这也是前端工程化中常用到的技巧）

**强缓存和协商缓存是通过不同的http头部控制。**

## 头部的区别

`HTTP1.0`中的缓存控制：

- `Pragma`：严格来说，它不属于专门的缓存控制头部，但是它设置`no-cache`时可以让本地强缓存失效（属于编译控制，来实现特定的指令，主要是因为兼容`HTTP1.0`，所以以前又被大量应用）

- `Expires`：服务端配置的，属于强缓存，用来控制在规定的时间之前，浏览器不会发出请求，而是直接使用本地缓存，注意，`Expires`一般对应服务器端时间，如`Expires：Fri, 30 Oct 1998 14:19:41`

- `If-Modified-Since/Last-Modified`：这两个是成对出现的，属于协商缓存的内容，其中浏览器的头部是`If-Modified-Since`，而服务端的是`Last-Modified`，它的作用是，在发起请求时，如果`If-Modified-Since`和`Last-Modified`匹配，那么代表服务器资源并未改变，因此服务端不会返回资源实体，而是只返回头部，通知浏览器可以使用本地缓存。`Last-Modified`，顾名思义，指的是文件最后的修改时间，而且只能精确到1s以内

`HTTP1.1`中的缓存控制：

- `Cache-Control`：缓存控制头部，有`no-cache`、`max-age`等多种取值

- `Max-Age`：服务端配置的，用来控制强缓存，在规定的时间之内，浏览器无需发出请求，直接使用本地缓存，注意，`Max-Age`是`Cache-Control`头部的值，不是独立的头部，譬如`Cache-Control: max-age=3600`，而且它值得是绝对时间，由浏览器自己计算

- `If-None-Match/E-tag`：这两个是成对出现的，属于协商缓存的内容，其中浏览器的头部是`If-None-Match`，而服务端的是E-tag，同样，发出请求后，如果`If-None-Match`和`E-tag`匹配，则代表内容未变，通知浏览器使用本地缓存，和`Last-Modified`不同，`E-tag`更精确，它是类似于指纹一样的东西，基于`FileEtag INode Mtime Size`生成，也就是说，只要文件变，指纹就会变，而且没有1s精确度的限制。

## Max-Age相比Expires？

`Expires`使用的是服务器端的时间

但是有时候会有这样一种情况：客户端时间和服务端不同步

那这样，可能就会出问题了，造成了浏览器本地的缓存无用或者一直无法过期

所以一般`HTTP1.1`后不推荐使用`Expires`

而`Max-Age`使用的是客户端本地时间的计算，因此不会有这个问题

因此推荐使用`Max-Age`。

注意，如果同时启用了`Cache-Control`与`Expires`，`Cache-Control`优先级高。

## E-tag相比Last-Modified？

`Last-Modified`：

- 表明服务端的文件最后何时改变的

- 它有一个缺陷就是只能精确到1s，

- 然后还有一个问题就是有的服务端的文件会周期性的改变，导致缓存失效

`E-tag`：

- 是一种指纹机制，代表文件相关指纹

- 只有文件变才会变，也只要文件变就会变，

- 也没有精确时间的限制，只要文件一遍，立马`E-tag`就不一样了

如果同时带有`E-tag`和`Last-Modified`，服务端会优先检查`E-tag`。

## 各缓存头部的关系

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/http_cache.png)

# Step4. 客户端解析页面

## 流程简述

浏览器内核拿到内容后，渲染步骤大致可以分为以下几步：

1. 解析`HTML`，构建`DOM`树

2. 解析`CSS`，生成`CSS`规则树

3. 合并`DOM`树和`CSS`规则，生成`render`树

4. 布局`render`树（`Layout/reflow`），负责各元素尺寸、位置的计算

5. 绘制`render`树（`paint`），绘制页面像素信息

6. 浏览器会将各层的信息发送给`GPU`，`GPU`会将各层合成（`composite`），显示在屏幕上

如下图：

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/browser_rending.png)

## 1.解析HTML，构建DOM树

浏览器处理流程如下图：

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/browser_parse_html.png)

其中的一些重点过程：

1. `Conversion`转换：浏览器将获得的`HTML`内容（`Bytes`）基于他的编码转换为单个字符

2. `Tokenizing`分词：浏览器按照`HTML`规范标准将这些字符转换为不同的标记`token`。每个`token`都有自己独特的含义以及规则集

3. `Lexing`词法分析：分词的结果是得到一堆的`token`，此时把他们转换为对象，这些对象分别定义他们的属性和规则

4. `DOM`构建：因为`HTML`标记定义的就是不同标签之间的关系，这个关系就像是一个树形结构一样
例如：`body`对象的父节点就是`HTML`对象，然后段略p对象的父节点就是`body`对象

最后的DOM树如下：

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/browser_parse_dom.png)

## 2.构建CSSOM树

`Bytes → characters → tokens → nodes → CSSOM`

假如`style.css`内容如下：

```css
body { 
    font-size: 16px 
}
p { 
    font-weight: bold 
}
span { 
    color: red 
}
p span { 
    display: none 
}
img { 
    float: right 
}
```

那么最终的CSSOM树就是：

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/browser_parse_cssom.png)

## 3.构建render树

当`DOM树`和`CSSOM`都有了后，就要开始构建`render树`了

一般来说，`render树`和`DOM树`相对应的，但不是严格意义上的一一对应

因为有一些不可见的`DOM`元素不会插入到`render树`中，如`head`这种不可见的标签或者`display: none`等

整体来说可以看图：

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/browser_parse_rendertree.png)

## 4.布局和绘制render树（渲染）

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/browser_rendingprocess.jpg)

图中重要的四个步骤就是：

1. 计算`CSS`样式

2. 构建渲染树

3. 布局，主要定位坐标和大小，是否换行，各种`position` `overflow` `z-index`属性

4. 绘制，将图像绘制出来

然后，图中的线与箭头代表通过`js`动态修改了`DOM`或`CSS`，导致了重新布局（`Layout`）或渲染（`Repaint`）

### Layout和Repaint的区别：

- `Layout`，也称为`Reflow`，即回流。一般意味着元素的内容、结构、位置或尺寸发生了变化，需要**重新计算样式和渲染树**

- `Repaint`，即重绘。意味着元素发生的改变只是影响了元素的一些外观之类的时候（例如，背景色，边框颜色，文字颜色等），此时只需要**应用新样式绘制这个元素**就可以了

回流的成本开销要高于重绘，而且一个节点的回流往往回导致子节点以及同级节点的回流， 所以优化方案中一般都包括，**尽量避免回流。**

### 引起回流的几种方式

1. 页面渲染初始化

2. `DOM`结构改变，比如删除了某个节点

3. `render树`变化，比如减少了`padding`

4. 窗口`resize`

5. 最复杂的一种：获取某些属性，引发回流，很多浏览器会对回流做优化，会等到数量足够时做一次批处理回流，但是除了`render树`的直接变化，当获取一些属性时，浏览器为了获得正确的值也会触发回流，这样使得浏览器优化无效，包括:
    - `offset(Top/Left/Width/Height)`
    - `scroll(Top/Left/Width/Height)`
    - `cilent(Top/Left/Width/Height)`
    - `width`, `height`
    - 调用了`getComputedStyle()`或者`IE`的`currentStyle`

6. 改变字体大小

**回流一定伴随着重绘，重绘却可以单独出现**

避免回流的一些优化方案，如：

- 减少逐项更改样式，最好一次性更改`style`，或者将样式定义为`class`并一次性更新

- 避免循环操作`dom`，创建一个`documentFragment`或`div`，在它上面应用所有`DOM`操作，最后再把它添加到`window.document`

- 避免多次读取`offset`等属性。无法避免则将它们缓存到变量

- 将复杂的元素绝对定位或固定定位，使得它脱离文档流，否则回流代价会很高

## 5.将渲染的DOM显示到屏幕上

上述中的渲染中止步于绘制，但实际上绘制这一步也没有这么简单，它可以结合复合层和简单层的概念来讲。

简单介绍下：

- 可以认为默认只有一个复合图层，所有的`DOM`节点都是在这个复合图层下的

- 如果开启了硬件加速功能，可以将某个节点变成复合图层

- 复合图层之间的绘制互不干扰，由`GPU`直接控制

- 而简单图层中，就算是`absolute`等布局，变化时不影响整体的回流，但是由于在同一个图层中，仍然是会影响绘制的，因此做动画时性能仍然很低。而复合层是独立的，所以一般做动画推荐使用硬件加速

### 更多

[关于普通图层和复合图层更详细的介绍](https://segmentfault.com/a/1190000012925872#articleHeader16)

## Chrome中的调试

Chrome开发者工具-`Perfomance`查看详细渲染过程：

![](https://dailc.github.io/staticResource/blog/basicKnowledge/whenyouenteraurl/browser_chrome_debug_1.png)

## 资源外链的下载（JS、CSS、IMG等）

上面介绍了html解析，渲染流程。但实际上，在解析html时，会遇到一些资源连接，此时就需要进行单独处理了

简单起见，这里将遇到的静态资源分为一下几大类（未列举所有）：

- `CSS`

- `JavaScript`

- `img`

### 遇到外链时的处理

当遇到上述的外链时，会单独开启一个下载线程去下载资源（`HTTP1.1`中是每一个资源的下载都要开启一个`HTTP`请求，对应一个`TCP/IP`链接）

### 遇到CSS样式资源

- `css`下载时异步，不会阻塞浏览器构建`DOM`树

- 但是会阻塞渲染，也就是在构建`render`时，会等到`css`下载解析完毕后才进行（这点与浏览器优化有关，防止`css`规则不断改变，避免了重复的构建）

- 有例外，`media query`声明的`CSS`是不会阻塞渲染的

### 遇到JS脚本资源

- 阻塞浏览器的解析，也就是说发现一个外链脚本时，需等待脚本下载完成并执行后才会继续解析`HTML`

- 浏览器的优化，一般现代浏览器有优化，在脚本阻塞时，也会继续下载其它资源（当然有并发上限），但是虽然脚本可以并行下载，解析过程仍然是阻塞的，也就是说必须这个脚本执行完毕后才会接下来的解析，并行下载只是一种优化而已

- `defer`与`async`，普通的脚本是会阻塞浏览器解析的，但是可以加上`defer`或`async`属性，这样脚本就变成异步了，可以等到解析完毕后再执行

注意，`defer`和`async`是有区别的： `defer`是延迟执行，而`async`是异步执行。

- `async`是异步执行，异步下载完毕后就会执行，不确保执行顺序，一定在`onload`前，但不确定在`DOMContentLoaded`事件的前或后

- `defer`是延迟执行，在浏览器看起来的效果像是将脚本放在了`body`后面一样（虽然按规范应该是在`DOMContentLoaded`事件前，但实际上不同浏览器的优化效果不一样，也有可能在它后面）

### 遇到img图片类资源

遇到图片等资源时，直接就是异步下载，不会阻塞解析，下载完毕后直接用图片替换原有src的地方

## loaded和domcontentloaded

区别：

- `DOMContentLoaded` 事件触发时，仅当`DOM`加载完成，不包括样式表，图片(譬如如果有`async`加载的脚本就不一定完成)

- `load`事件触发时，页面上所有的`DOM`，样式表，脚本，图片都已经加载完成了

# 其它

## 跨域

[ajax跨域，这应该是最全的解决方案了](https://segmentfault.com/a/1190000012469713)

## Web安全

[AJAX请求真的不安全么？谈谈Web安全与AJAX的关系](https://segmentfault.com/a/1190000012693772)
