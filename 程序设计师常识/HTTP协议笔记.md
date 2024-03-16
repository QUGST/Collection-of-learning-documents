# HTTP/HTTPS 简介

## HTTP/HTTPS 简介

HTTP 协议是 Hyper Text Transfer Protocol（超文本传输协议）的缩写，是用于从万维网（ WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。

HTTP 是一个基于 **TCP/IP** 通信协议来传递数据（HTML 文件、图片文件、查询结果等）。

HTTPS 协议是 HyperText Transfer Protocol Secure（超文本传输安全协议）的缩写，是一种通过计算机网络进行**安全**通信的传输协议。

**HTTPS 经由 HTTP 进行通信，但利用 SSL/TLS 来加密数据包**，HTTPS 开发的主要目的，是提供对网站服务器的身份认证，保护交换资料的隐私与完整性。

HTTP 的 URL 是由 **http:// **起始与默认使用端口** 80**，而 HTTPS 的 URL 则是由** https:// **起始与默认使用端口**443**。

## 工作原理

HTTP 协议工作于**客户端-服务端**架构上。

浏览器作为 HTTP 客户端通过 URL 向 HTTP 服务端即 WEB 服务器发送所有请求。

Web 服务器有：Apache 服务器，IIS 服务器（Internet Information Services）等。

Web 服务器根据接收到的请求后，向客户端发送响应信息。

HTTP 默认端口号为 80，但是你也可以改为 8080 或者其他端口。

### 特点

**HTTP 是无连接**：无连接的含义是限制每次连接只处理一个请求，服务器处理完客户的请求，并收到客户的应答后，即断开连接，采用这种方式可以节省传输时间。

**HTTP 是媒体独立的**：这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送，客户端以及服务器指定使用适合的 MIME-type 内容类型。

**HTTP 是无状态**：HTTP 协议是无状态协议，无状态是指协议对于事务处理没有记忆能力，缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大，另一方面，在服务器不需要先前信息时它的应答就较快。

## HTTPS 作用

HTTPS 的主要作用是在不安全的网络上创建一个安全信道，并可在使用适当的加密包和服务器证书可被验证且可被信任时，对窃听和中间人攻击提供合理的防护。

HTTPS 的信任基于预先安装在操作系统中的证书颁发机构（CA）

# HTTP消息结构

HTTP是基于客户端/服务端（C/S）的架构模型，通过一个可靠的链接来交换信息，是一个无状态的请求/响应协议。

一个HTTP"客户端"是一个应用程序（Web浏览器或其他任何客户端），通过连接到服务器达到向服务器发送一个或多个HTTP的请求的目的。

一个HTTP"服务器"同样也是一个应用程序（通常是一个Web服务，如Apache Web服务器或IIS服务器等），通过接收客户端的请求并向客户端发送HTTP响应数据。

HTTP使用统一资源标识符（Uniform Resource Identifiers,** URI**）来传输数据和建立连接。

一旦建立连接后，数据消息就通过类似Internet邮件所使用的格式[RFC5322]和多用途Internet邮件扩展（MIME）[RFC2045]来传送。

## 客户端请求消息

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：**请求行（request line）、请求头部（header）、空行和请求数据**四个部分组成，下图给出了请求报文的一般格式。

![截图](79548c72114f9f669d0f8c9672e4b7f3.png)

### 例子

客户端请求：

GET /hello.txt HTTP/1.1
User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
Host: www.example.com
Accept-Language: en, mi

## 服务器响应消息

   HTTP响应也由四个部分组成，分别是：**状态行、消息报头、空行和响应正文**。

![截图](ee2f596a2679e9aa6d13cf0f5764e094.png)

![截图](eea21179455809490d59a0b135726e05.png)

### 例子

服务端响应:

HTTP/1.1 200 OK
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
ETag: "34aa387-d-1568eb00"
Accept-Ranges: bytes
Content-Length: 51
Vary: Accept-Encoding
Content-Type: text/plain

# 请求方法

根据 HTTP 标准，HTTP 请求可以使用多种请求方法。

HTTP1.0 定义了三种请求方法： GET, POST 和 HEAD 方法。

HTTP1.1 新增了六种请求方法：OPTIONS、PUT、PATCH、DELETE、TRACE 和 CONNECT 方法。

1    **GET**    请求指定的页面信息，并返回实体主体。
2    **HEAD**    类似于 GET 请求，只不过返回的响应中没有具体的内容，用于获取报头
3    **POST**    向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立和/或已有资源的修改。
4    **PUT **   从客户端向服务器传送的数据取代指定的文档的内容。
5    **DELETE**    请求服务器删除指定的页面。
6    **CONNECT**    HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。
7    **OPTIONS**    允许客户端查看服务器的性能。
8    **TRACE**    回显服务器收到的请求，主要用于测试或诊断。
9    **PATCH**    是对 PUT 方法的补充，用来对已知资源进行局部更新 。

# HTTP 请求头信息

## Accept:

浏览器（或者其他基于HTTP的客户端程序）可以接收的内容类型（Content-types）,例如 Accept: text/plain

## Accept-Charset：

浏览器能识别的字符集，例如 Accept-Charset: utf-8

## Accept-Encoding：

浏览器可以处理的编码方式，注意这里的编码方式有别于字符集，这里的编码方式通常指gzip,deflate等。例如 Accept-Encoding: gzip, deflate

## Accept-Language：

浏览器接收的语言，其实也就是用户在什么语言地区，例如简体中文的就是 Accept-Language: zh-CN

Accept-Datetime：（这个暂时没搞清楚什么意思）

## Authorization：

在HTTP中，服务器可以对一些资源进行认证保护，如果你要访问这些资源，就要提供用户名和密码，这个用户名和密码就是在Authorization头中附带的，格式是“username:password”字符串的base64编码，例如：Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ\==中，basic指使用basic认证方式，　QWxhZGRpbjpvcGVuIHNlc2FtZQ\==使用base64解码就是“Aladdin:open sesame”

## Cache-Control：

这个指令在request和response中都有，用来指示缓存系统（服务器上的，或者浏览器上的）应该怎样处理缓存，因为这个头域比较重要，特别是希望使用缓　存改善性能的时候，内容也较多，所以我想在下一篇博文中主要介绍一下。

## Connection：

告诉服务器这个user agent（通常就是浏览器）想要使用怎样的连接方式。值有keep-alive和close。http1.1默认是keep-alive。keep-alive就是浏览器和服务器　的通信连接会被持续保存，不会马上关闭，而close就会在response后马上关闭。但这里要注意一点，我们说HTTP是无状态的，跟这个是否keep-alive没有关系，不要认为keep-alive是对HTTP无状态的特性的改进。

## Cookie：

浏览器向服务器发送请求时发送cookie，或者服务器向浏览器附加cookie，就是将cookie附近在这里的。例如：Cookie:user=admin

## Content-Length：

一个请求的请求体的内存长度，单位为字节(byte)。请求体是指在HTTP头结束后，两个CR-LF字符组之后的内容，常见的有POST提交的表单数据，这个Content-Length并不包含请求行和HTTP头的数据长度。

## Content-MD5：

使用base64进行了编码的请求体的MD5校验和。例如：Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==

## Content-Type：

请求体中的内容的mime类型。通常只会用在POST和PUT方法的请求中。例如：Content-Type: application/x-www-form-urlencoded

## Date：

发送请求时的GMT时间。例如：Date: Tue, 15 Nov 1994 08:12:31 GMT

## Expect：

指示需要使用服务器某些特殊的功能。（这个我不是很清楚）

## From：

发送这个请求的用户的email地址。例如：From: user@example.com

## Host：

被服务器的域名或IP地址，如果不是通用端口，还包含该端口号，例如：Host: www.some.com:182

## If-Match:

通常用在使用PUT方法对服务器资源进行更新的请求中，意思就是，询问服务器，现在正在请求的资源的tag和这个If-Match的tag相不相同，如果相同，则证明服务器上的这个资源还是旧的，现在可以被更新，如果不相同，则证明该资源被更新过，现在就不用再更新了（否则有可能覆盖掉其他人所做的更改）。

## If-Modified-Since：

询问服务器现在正在请求的资源在某个时间以来有没有被修改过，如果没有，服务器则返回304状态来告诉浏览器使用浏览器自己本地的缓存，如果有修改过，则返回200，并发送新的资源（当然如果资源不存在，则返回404。）

## If-None-Match：

和If-Modified-Since用意差不多，不过不是根据时间来确定，而是根据一个叫ETag的东西来确定。关于etag我想在下一篇博客介绍一下。

## If-Range：

告诉服务器如果这个资源没有更改过(根据If-Range后面给出的Etag判断)，就发送这个资源中在浏览器缺少了的某些部分给浏览器，如果该资源以及被修改过，则将整个资源重新发送一份给浏览器。

## If-Unmodified-Since：

询问服务器现在正在请求的资源在某个时刻以来是否没有被修改过。

## Max-Forwards：

限制请求信息在代理服务器或网关中向前传递的次数。

## Pragma：

好像只有一个值，就是:no-cache。Pragma:no-cache 与cache-control:no-cache相同，只不过cache-control:no-cache是http1.1专门指定的，而Pragma:no-cache可以在http1.0和1.1中使用

## Proxy-Authorization：

连接到某个代理时使用的身份认证信息，跟Authorization头差不多。例如：Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==

## Range：

在HTTP头中，"Range"字眼都表示“资源的byte形式数据的顺序排列，并且取其某一段数据”的意思。Range头就是表示请求资源的从某个数值到某个数值间的数据，例如：Range: bytes=500-999 就是表示请求资源从500到999byte的数据。数据的分段下载和多线程下载就是利用这个实现的。

## Referer：

指当前请求的URL是在什么地址引用的。例如在www.a.com/index.html页面中点击一个指向www.b.com的超链接，那么，这个www.b.com的请求中的Referer就是www.a.com/index.html。通常我们见到的图片防盗链就是用这个实现的。

## Upgrade：

请求服务器更新至另外一个协议，例如：Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11

## User-Agent：

通常就是用户的浏览器相关信息。例如：User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/12.0

## Via：

用来记录一个请求经过了哪些代理或网关才被送到目标服务器上。例如一个请求从浏览器出发(假设使用http/1.0)，发送给名为 SomeProxy的内部代理，然后被转发至www.somenet.com的公共代理（使用http/1.1），最后被转发至目标服务器www.someweb.com，那么在someweb.com中收到的via 头应该是：via:1.0 someProxy 1.1 www.someweb.com(apache 1.1)

## Warning：

记录一些警告信息。

## 通用但非标准的HTTP头（通常，非标准的头域都是用“X-”开头，例如"x-powered-by"）：

### X-Requested-With：

主要是用来识别ajax请求，很多javascript框架会发送这个头域（值为XMLHttpRequest）

### DNT:

DO NOT TRACK的缩写，要求服务器程序不要跟踪记录用户信息。DNT: 1 (开启DNT) DNT: 0 (关闭DNT)火狐，safari,IE9都支持这个头域，并且于2011年3月7日被提交至IETF组织实现标准化

### X-Forwarded-For:

记录一个请求从客户端出发到目标服务器过程中经历的代理，或者负载平衡设备的IP。

### X-Forwarded-Proto：

记录一个请求一个请求最初从浏览器发出时候，是使用什么协议。因为有可能当一个请求最初和反向代理通信时，是使用https，但反向代理和服务器通信时改变成http协议，这个时候，X-Forwarded-Proto的值应该是https

### Front-End-Https：

微软使用与其负载平衡的一个头域。

### X-ATT-DeviceId：

AT&A的产品中使用的头域，不过不是很清楚用途。

# HTTP 响应头信息

## Allow    

服务器支持哪些请求方法（如GET、POST等）。

## Content-Encoding    

文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即request.getHeader("Accept-Encoding")）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面。

## Content-Length    

表示内容长度。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入 ByteArrayOutputStream，完成后查看其大小，然后把该值放入Content-Length头，最后通过byteArrayStream.writeTo(response.getOutputStream()发送内容。

## Content-Type    

表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentType。

## Date    

当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦。

## Expires    

应该在什么时候认为文档已经过期，从而不再缓存它？

## Last-Modified    

文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置。

## Location    

表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。

## Refresh    

表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过setHeader("Refresh", "5; URL=http://host/path")让浏览器读取指定的页面。
注意这种功能通常是通过设置HTML页面HEAD区的＜META HTTP-EQUIV="Refresh" CONTENT="5;URL=http://host/path"＞实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。

注意Refresh的意义是"N秒之后刷新本页面或访问指定页面"，而不是"每隔N秒刷新本页面或访问指定页面"。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是＜META HTTP-EQUIV="Refresh" ...＞。

注意Refresh头不属于HTTP 1.1正式规范的一部分，而是一个扩展，但Netscape和IE都支持它。

## Server    

服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。

## Set-Cookie    

设置和页面关联的Cookie。Servlet不应使用response.setHeader("Set-Cookie", ...)，而是应使用HttpServletResponse提供的专用方法addCookie。参见下文有关Cookie设置的讨论。

## WWW-Authenticate    

客户应该在Authorization头中提供什么类型的授权信息？在包含401（Unauthorized）状态行的应答中这个头是必需的。例如，response.setHeader("WWW-Authenticate", "BASIC realm=＼"executives＼"")。
注意Servlet一般不进行这方面的处理，而是让Web服务器的专门机制来控制受密码保护页面的访问（例如.htaccess）。

# HTTP状态码

当浏览者访问一个网页时，浏览者的浏览器会向网页所在服务器发出请求。当浏览器接收并显示网页前，此网页所在的服务器会返回一个包含 HTTP 状态码的信息头（server header）用以响应浏览器的请求。

HTTP 状态码的英文为 HTTP Status Code。

## 状态码分类

HTTP 状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型。响应分为五类：信息响应(100–199)，成功响应(200–299)，重定向(300–399)，客户端错误(400–499)和服务器错误 (500–599)：

1xx 信息，服务器收到请求，需要请求者继续执行操作
2xx 成功，操作被成功接收并处理
3xx 重定向，需要进一步的操作以完成请求
4xx 客户端错误，请求包含语法错误或无法完成请求
5xx 服务器错误，服务器在处理请求的过程中发生了错误

## 状态码列表

#### 100    

Continue    继续。客户端应继续其请求

#### 101    

Switching Protocols    切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议

#### 200    

OK    请求成功。一般用于GET与POST请求

#### 201    

Created    已创建。成功请求并创建了新的资源

#### 202    

Accepted    已接受。已经接受请求，但未处理完成

#### 203    

Non-Authoritative Information    非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本

#### 204    

No Content    无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档

#### 205    

Reset Content    重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域

#### 206    

Partial Content    部分内容。服务器成功处理了部分GET请求

#### 300    

Multiple Choices    多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择

#### 301    

Moved Permanently    永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替

#### 302    

Found    临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI

#### 303    

See Other    查看其它地址。与301类似。使用GET和POST请求查看

#### 304    

Not Modified    未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源

#### 305    

Use Proxy    使用代理。所请求的资源必须通过代理访问

#### 306    

Unused    已经被废弃的HTTP状态码

#### 307    

Temporary Redirect    临时重定向。与302类似。使用GET请求重定向

#### 400    

Bad Request    客户端请求的语法错误，服务器无法理解

#### 401    

Unauthorized    请求要求用户的身份认证

#### 402    

Payment Required    保留，将来使用

#### 403    

Forbidden    服务器理解请求客户端的请求，但是拒绝执行此请求

#### 404    

Not Found    服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面

#### 405    

Method Not Allowed    客户端请求中的方法被禁止

#### 406    

Not Acceptable    服务器无法根据客户端请求的内容特性完成请求

#### 407    

Proxy Authentication Required    请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权

#### 408    

Request Time-out    服务器等待客户端发送的请求时间过长，超时

#### 409    

Conflict    服务器完成客户端的 PUT 请求时可能返回此代码，服务器处理请求时发生了冲突

#### 410    

Gone    客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置

#### 411    

Length Required    服务器无法处理客户端发送的不带Content-Length的请求信息

#### 412    

Precondition Failed    客户端请求信息的先决条件错误

#### 413    

Request Entity Too Large    由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息

#### 414    

Request-URI Too Large    请求的URI过长（URI通常为网址），服务器无法处理

#### 415    

Unsupported Media Type    服务器无法处理请求附带的媒体格式

#### 416    

Requested range not satisfiable    客户端请求的范围无效

#### 417    

Expectation Failed    服务器无法满足Expect的请求头信息

#### 500    

Internal Server Error    服务器内部错误，无法完成请求

#### 501    

Not Implemented    服务器不支持请求的功能，无法完成请求

#### 502    

Bad Gateway    作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应

#### 503    

Service Unavailable    由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中

#### 504    

Gateway Time-out    充当网关或代理的服务器，未及时从远端服务器获取请求

#### 505    

HTTP Version not supported    服务器不支持请求的HTTP协议的版本，无法完成处理

# HTTP content-type

Content-Type（内容类型），一般是指网页中存在的 Content-Type，用于定义网络文件的类型和网页的编码，决定浏览器将以什么形式、什么编码读取这个文件，这就是经常看到一些 PHP 网页点击的结果却是下载一个文件或一张图片的原因。

Content-Type 标头告诉客户端实际返回的内容的内容类型。

## 常见的媒体格式类型如下：

text/html ： HTML格式
text/plain ：纯文本格式
text/xml ： XML格式
image/gif ：gif图片格式
image/jpeg ：jpg图片格式
image/png：png图片格式

## 以application开头的媒体格式类型：

application/xhtml+xml ：XHTML格式
application/xml： XML数据格式
application/atom+xml ：Atom XML聚合格式
application/json： JSON数据格式
application/pdf：pdf格式
application/msword ： Word文档格式
application/octet-stream ： 二进制流数据（如常见的文件下载）
application/x-www-form-urlencoded ： <form encType=””>中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）

## 另外一种常见的媒体格式是上传文件之时使用的：

multipart/form-data ： 需要在表单中进行文件上传时，就需要使用该格式
