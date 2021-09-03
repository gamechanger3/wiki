# HTTP 协议的前世今生

---

## 0. 前言

你知道当我们在网页浏览器的地址栏中输入 URL 时，Web 页面是如何呈现的吗？

![](https://gitee.com/veal98/images/raw/master/img/20210126135155.png)

Web 界面当然不会凭空出来，根据 Web 浏览器地址栏中指定的 URL，Web 使用一种名为 HTTP 的协议作为规范，完成从客户端到服务端的一些流程。**可以说，Web 是建立在 HTTP 协议上进行通信的**。

## 1. HTTP 的诞生

其实，在 1983 年 3 月之前，互联网还只属于少数人，全世界的网民之间的信息是无法共享的。在这一互联网的黎明时期，HTTP 应运而生。

欧洲核子研究组织的 Tim Berners-Lee 博士提出了一种能够让远隔两地的网民共享知识的设想，最初的理念是：借助多文档之间相互关联的超文本（HyperTest），连成可相互参阅的 WWW（World Wide Web，万维网）。

现在已提出了 3 项 WWW 构建技术，分别是：

- 把 SGML（标准通用标记语言）作为页面的文本标记语言 HTML
- 作为文档传递协议的 HTTP
- 指定文档所在地址的 URL

WWW 这一名称，是 Web 浏览器当年用来浏览超文本的客户端应用程序的名称，现在用来表示这一系列的集合，也可简称为 Web。

## 2. 什么是 HTTP

说了这么多，大家只知道 HTTP 很牛逼，对 HTTP 是什么仍然没有很直观的概念。别急，在了解什么是 HTTP 之前，我们有必要知道超文本是什么。

HTTP 传输的内容就是**超文本**：

- 我们先来理解「文本」：在互联网早期的时候只是简单的字符文字，但随着技术的发展，现在「文本」的涵义已经可以扩展为图片、视频、压缩包等，在 HTTP 眼里这些都算做「文本」。

- 再来理解「超文本」：它就是超越了普通文本的文本，它是文字、图片、视频等的混合体。最关键有**超链接**，能从一个超文本跳转到另外一个超文本。

  HTML 就是最常见的超文本了，它本身只是纯文字文件，但内部用很多标签定义了图片、视频等的链接，在经过浏览器的解析，呈现给我们的就是一个文字、有画面的网页了。

OK，下面我们正式介绍什么是 HTTP？

**HTTP：超文本传输协议**（HyperText Transfer Protocol）是当今互联网上应用最为广泛的一种网络协议。所有的 WWW（万维网） 文件都必须遵守这个标准。<u>HTTP 和 TCP/IP 协议簇中的众多协议一样，用于客户端和服务器端之间的通信</u>。

![](https://gitee.com/veal98/images/raw/master/img/20210126211939.png)

## 3. 驻足不前的 HTTP

至今被世人广泛使用的 HTTP 协议，仍然是 20 多年前的版本。也就是说，作为 Web 文档传输协议的 HTTP，它的版本几乎没有更新，从另一方面来说，前人的智慧真的牛逼 👍

**HTTP/0.9**：HTTP 于 1990 年问世，<u>功能简陋，仅支持 GET 请求方式，并且仅能访问 HTML 格式的资源</u>。那时的 HTTP 并没有作为正式的标准被建立，因此被被称为 HTTP 0.9。

**HTTP/1.0**：1996 年 5 月 HTTP 正式作为标准被公布，版本号为 HTTP 1.0。<u>在 0.9 版本上做了进步，增加了请求方式 POST 和 HEAD；不再局限于 0.9 版本的 HTML 格式，根据 Content-Type 可以支持多种数据格式...... 需要注意的是：1.0 版本的工作方式是短连接</u>。虽说 HTTP/1.0 是初期标准，但该协议标准至今仍然在被广泛使用。

**HTTP/1.1**：1997 年公布的 HTTP 1.1 是目前主流的 HTTP 协议版本。当年的 HTTP 协议的出现主要是为了解决文本传输的难题，现在的 HTTP 早已超出了 Web 这个框架的局限，被运用到了各种场景里。当然，<u>1.1 版本的最大变化，就是引入了长连接以及流水线机制（管道机制）</u>。

这里面出现的各种专有名词大家留个印象就行，下文会逐渐讲解。

## 4. 区分 URL 和 URI

与 URI（**统一资源标识符**） 相比，大家应该更熟悉 URL（Uniform Resource Location，**统一资源定位符**），URL 就是我们使用 Web 浏览器访问 Web 页面时需要输入的网页地址。比如 `http://baidu.com`。

URI 是 Uniform Resource Identifier 的缩写，RFC 2386 分别对这三个单词进行如下定义：

- Uniform：统一规定的格式可方便处理多种不同类型的资源
- Resource：资源的定义是可标识的任何东西。不仅可以是单一的，也可以是一个集合
- Identifier：标识可标识的对象。也称为标识符

综上，URI 就是由某个协议方法表示的资源的定位标识符。比如说，采用 HTTP 协议时，协议方案就是 `http`，除此之外，还有 `ftp`、`telnet` 等，标准的 URI 协议方法有 30 种左右。

URI 有两种格式，相对 URI 和绝对 URI。

- **相对 URI**：指从浏览器中基本 URI 处指定的 URL，形如 `/user/logo.png`

- **绝对 URI**：使用涵盖全部必要信息

  ![](https://gitee.com/veal98/images/raw/master/img/20210126150648.png)

总结来说：**URI 用字符串标识某一处互联网资源，而 URL 标识资源的地点（互联网上所处的位置），可见 URL 是 URI 的子集**。

## 5. HTTP 请求和响应

HTTP 协议规定，在两台计算机之间使用 HTTP 协议进行通信时，在一条通信线路上必定有一端是客户端，另一端则是服务端。<u>当在浏览器中输入网址访问某个网站时， 你的浏览器（客户端）会将你的请求封装成一个 HTTP 请求发送给服务器站点，服务器接收到请求后会组织响应数据封装成一个 HTTP 响应返回给浏览器</u>。换句话说，肯定是先从客户端开始建立通信的，服务器端在没有接收到请求之前不会发送响应。

![](https://gitee.com/veal98/images/raw/master/img/20210126144333.png)

下面我们详细分析一下 HTTP 的请求报文和响应报文

### ① HTTP 请求报文

HTTP 请求报文由 3 大部分组成：

1）请求行（必须在 HTTP 请求报文的第一行）

2）请求头（从第二行开始，到第一个空行结束。请求头和请求体之间存在一个空行）

3）请求体（通常以键值对 `{key:value}`方式传递数据）

举个请求报文的例子：

![](https://gitee.com/veal98/images/raw/master/img/20210126145429.png)

请求行开头的 `POST` 表示请求访问服务器的类型，称为方法（method）。随后的字符串 `/form/login` 指明了请求访问的资源对象，也叫做请求 URI（request-URI）。最后的 `HTTP/1.1` 即 HTTP 的版本号，用来提示客户端使用的 HTTP  协议功能。

综上来看，这段请求的意思就是：请求访问某台 HTTP 服务器上的 `/form/login` 页面资源，并附带参数 name = veal、age = 37。

> 注意，无论是 HTTP 请求报文还是 HTTP 响应报文，请求头/响应头和请求体/响应体之间都会有一个**空行**，且请求体/响应体并不是必须的。

#### HTTP 请求方法

请求行中的方法的作用在于可以指定请求的资源按照期望产生某种行为，即**使用方法给服务器下命令**。

包括（HTTP 1.1）：`GET`、`POST`、`PUT`、`HEAD`、`DELETE`、`OPTIONS`、`CONNECT`、`TRACE`。当然，我们在开发中最常见也最常使用的就只有前面三个。

1）**GET 获取资源**

GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回响应内容

![](https://gitee.com/veal98/images/raw/master/img/20210126153110.png)

使用 GET 方法请求-响应的例子：

![](https://gitee.com/veal98/images/raw/master/img/20210126154500.png)

2）**POST 传输实体主体**

POST 主要用来传输数据，而 GET 主要用来获取资源。

![](https://gitee.com/veal98/images/raw/master/img/20210126153402.png)

使用 POST 方法请求-响应的例子：

![](https://gitee.com/veal98/images/raw/master/img/20210126154526.png)

3）**PUT 传输文件**

PUT 方法用来传输文件，由于自身**不带验证机制，任何人都可以上传文件**，因此存在安全性问题，一般不使用该方法。

![](https://gitee.com/veal98/images/raw/master/img/20210126154626.png)

使用 PUT 方法请求-响应的例子：

![](https://gitee.com/veal98/images/raw/master/img/20210126154655.png)

4）**HEAD 获取报文首部**

和 GET 方法类似，但是不返回报文实体主体部分。主要用于确认 URI 的有效性以及资源更新的日期时间等。

![](https://gitee.com/veal98/images/raw/master/img/20210126154727.png)

使用 HEAD 方法请求-响应的例子：

![](https://gitee.com/veal98/images/raw/master/img/20210126154752.png)

5）**DELETE 删除文件**

与 PUT 功能相反，用来删除文件，并且同样不带验证机制，按照请求 URI 删除指定的资源。

![](https://gitee.com/veal98/images/raw/master/img/20210126155025.png)

使用 DEELTE 方法请求-响应的例子：

![](https://gitee.com/veal98/images/raw/master/img/20210126155100.png)

6）**OPTIONS 查询支持的方法**

用于**获取当前 URI 所支持的方法**。若请求成功，会在 HTTP 响应头中包含一个名为 “`Allow`” 的字段，值是所支持的方法，如 “GET, POST”。

![](https://gitee.com/veal98/images/raw/master/img/20210126155245.png)

使用 OPTIONS 方法请求-响应的例子：

![](https://gitee.com/veal98/images/raw/master/img/20210126155317.png)

7）..........

#### HTTP 请求头

请求头用于补充请求的附加信息、客户端信息、对响应内容相关的优先级等内容。以下列出常见请求头：

1）**Referer**：表示这个请求是从哪个 URI 跳过来的。比如说通过百度来搜索淘宝网，那么在进入淘宝网的请求报文中，Referer 的值就是：www.baidu.com。如果是直接访问就不会有这个头。这个字段通常用于防盗链。

![](https://gitee.com/veal98/images/raw/master/img/20210126160853.png)

2）**Accept**：告诉服务端，该请求所能支持的响应数据类型。（对应的，HTTP 响应报文中也有这样一个类似的字段 `Content-Type`，用于表示服务端发送的数据类型，如果 `Accept` 指定的类型和服务端返回的类型不一致，就会报错）

![](https://gitee.com/veal98/images/raw/master/img/20210126162103.png)

上图中的 `text/plain;q = 0.3` 表示对于 `text/plain` 媒体类型的数据优先级/权重为 0.3（q 的范围 0 ~ 1）。不指定权重的，默认为 1.0。

数据格式类型如下图：

![](https://gitee.com/veal98/images/raw/master/img/20210126161552.png)

3）**Host**：告知服务器请求的资源所处的互联网主机名和端口号。该字段是 HTTP/1.1 规范中唯一一个必须被	包含在请求头中的字段。

4）**Cookie**：客户端的 Cookie 就是通过这个报文头属性传给服务端的！

```html
Cookie: JSESSIONID=15982C27F7507C7FDAF0F97161F634B5
```

5）**Connection**：表示客户端与服务连接类型；Keep-Alive 表示持久连接，close 已关闭

6）**Content-Length**：请求体的长度

7）**Accept-Language**：浏览器通知服务器，浏览器支持的语言

8）**Range**：对于只需获取部分资源的范围请求，包含首部字段 Range 即可告知服务器资源的指定范围

9）......

### ② HTTP响应报文

HTTP的响应报文也由三部分组成：

- 响应行（必须在 HTTP 响应报文的第一行）
- 响应头（从第二行开始，到第一个空行结束。响应头和响应体之间存在一个空行）
- 响应体

![](https://gitee.com/veal98/images/raw/master/img/20210126154224.png)

在响应行开头的 HTTP 1.1 表示服务器对应的 HTTP 版本。紧随的 200 OK 表示请求的处理结果的**状态码**和**原因短语**。

#### HTTP 状态码

**HTTP 状态码负责表示客户端 HTTP 请求的的返回结果、标记服务器端处理是否正常、通知出现的错误等工作**。（重中之重！！！，和我们日常开发息息相关）

![](https://gitee.com/veal98/images/raw/master/img/20210126205447.png)

状态码由 3 位数字组成，第一个数字定义了响应的类别：

|      |             类别              |          原因短语          |
| :--: | :---------------------------: | :------------------------: |
| 1xx  |  Informational 信息性状态码   |     接收的请求正在处理     |
| 2xx  |      Success 成功状态码       |      请求正常处理完毕      |
| 3xx  |   Redirection 重定向状态码    | 需要进行附加操作以完成请求 |
| 4xx  | Client Error 客户端错误状态码 |     服务器无法处理请求     |
| 5xx  | Server Error 服务器错误状态码 |     服务器处理请求出错     |

🔶 **2xx**：请求正常处理完毕

- `200 OK`：客户端请求成功

  ![](https://gitee.com/veal98/images/raw/master/img/20210126170721.png)

- `204 No Content`：无内容。服务器成功处理，但未返回内容。一般用在只是客户端向服务器发送信息，而服务器不用向客户端返回什么信息的情况。不会刷新页面。

  ![](https://gitee.com/veal98/images/raw/master/img/20210126171225.png)

- `206 Partial Content`：服务器已经完成了部分 GET 请求（客户端进行了范围请求）。响应报文中包含 Content-Range 指定范围的实体内容

  ![](https://gitee.com/veal98/images/raw/master/img/20210126170935.png)

🔶 **3xx**：需要进行附加操作以完成请求（重定向）

- `301 Moved Permanently`：永久重定向，表示请求的资源已经永久的搬到了其他位置。

- `302 Found`：临时重定向，表示请求的资源临时搬到了其他位置
- `303 See Other`：临时重定向，应使用GET定向获取请求资源。303功能与302一样，区别只是303明确客户端应该使用GET访问
- `304 Not Modified`：表示客户端发送附带条件的请求（GET方法请求报文中的IF…）时，条件不满足。返回304时，不包含任何响应主体。虽然304被划分在3XX，但和重定向一毛钱关系都没有
- `307 Temporary Redirect`：临时重定向，和302有着相同含义。POST不会变成GET

🔶 **4xx**：客户端错误

- `400 Bad Request`：客户端请求有语法错误，服务器无法理解。

- `401 Unauthorized`：请求未经授权，这个状态代码必须和 WWW-Authenticate 报头域一起使用。
- `403 Forbidden`：服务器收到请求，但是拒绝提供服务
- `404 Not Found`：请求资源不存在。比如，输入了错误的 URL
- `415 Unsupported media type`：不支持的媒体类型

🔶 **5xx**：服务器端错误，服务器未能实现合法的请求。

- `500 Internal Server Error`：服务器发生不可预期的错误。

- `503 Server Unavailable`：服务器当前处于超负载或正在停机维护，暂时不能处理客户端的请求，一段时间后可能恢复正常


#### HTTP 响应头

响应头也是用键值对 k：v，用于补充响应的附加信息、服务器信息，以及对客户端的附加要求等。

![](https://gitee.com/veal98/images/raw/master/img/20210126165341.png)

这里着重说明一下 `Location` 这个字段，可以将响应接收方引导至与某个 URI 位置不同的资源。通常来说，该字段会配合 `3xx:Redirection` 的响应，提供重定向的 URI。

![](https://gitee.com/veal98/images/raw/master/img/20210126165212.png)

## 6. HTTP 连接管理

### ① 短连接（非持久连接）

在 HTTP 协议的初始版本（**HTTP/1.0**）中，**客户端和服务器每进行一次 HTTP 会话，就建立一次连接，任务结束就中断连接**。当客户端浏览器访问的某个 HTML 或其他类型的 Web 页中包含有其他的 Web 资源（如JavaScript 文件、图像文件、CSS文件等），每遇到这样一个 Web 资源，浏览器就会重新建立一个 HTTP 会话。这种方式称为**短连接**（也称**非持久连接**）。

也就是说每次 HTTP 请求都要重新建立一次连接。由于 HTTP 是基于 TCP/IP 协议的，所以连接的每一次建立或者断开都需要 TCP 三次握手或者 TCP 四次挥手的开销。

![](https://gitee.com/veal98/images/raw/master/img/20210126210346.png)

显然，这种方式存在巨大的弊端。比如访问一个包含多张图片的 HTML 页面，每请求一张图片资源就会造成无谓的 TCP 连接的建立和断开，大大增加了通信量的开销

![](https://gitee.com/veal98/images/raw/master/img/20210126210903.png)

### ② 长连接（持久连接）

从 **HTTP/1.1** 起，默认使用**长连接**也称**持久连接 keep-alive**。使用长连接的 HTTP 协议，会在响应头加入这行代码：`Connection:keep-alive`

在使用长连接的情况下，当一个网页打开完成后，客户端和服务器之间用于传输 HTTP 数据的 TCP 连接不会关闭，客户端再次访问这个服务器时，会继续使用这一条已经建立的连接。**Keep-Alive 不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如 Apache）中设定这个时间**。实现长连接需要客户端和服务端都支持长连接。

![](https://gitee.com/veal98/images/raw/master/img/20210126211528.png)

> HTTP 协议的长连接和短连接，实质上是 TCP 协议的长连接和短连接。

### ③ 流水线（管线化）

默认情况下，HTTP 请求是按顺序发出的，下一个请求只有在当前请求收到响应之后才会被发出。由于受到网络延迟和带宽的限制，在下一个请求被发送到服务器之前，可能需要等待很长时间。

持久连接使得多数请求以**流水线**（管线化 pipeline）方式发送成为可能，即在**同一条持久连接上连续发出请求，而不用等待响应返回后再发送**，这样就可以做到同时**并行**发送多个请求，而不需要一个接一个地等待响应了。

![](https://gitee.com/veal98/images/raw/master/img/20210126211821.png)

## 7. 无状态的 HTTP

HTTP 协议是无状态协议。也就是说他不对之前发生过的请求和响应的状态进行管理，即无法根据之前的状态进行本次的请求处理。

这样就会带来一个明显的问题，如果 HTTP 无法记住用户登录的状态，那岂不是每次页面的跳转都会导致用户需要再次重新登录？

当然，不可否认，无状态的优点也很显著，由于不必保存状态，自然就减少了服务器的 CPU 及内存资源的消耗。另一方面，正式由于 HTTP 简单，所以才会被如此广泛应用。	

![](https://gitee.com/veal98/images/raw/master/img/20210126173806.png)

这样，在保留无状态协议这个特征的同时，又要解决无状态导致的问题。方案有很多种，其中比较简单的方式就是使用 **Cookie** 技术。

`Cookie` 通过在请求和响应报文中写入 Cookie 信息来控制客户端的状态。具体来说，Cookie 会根据从服务器端发送的响应报文中的一个叫作 `Set-Cookie` 的首部字段信息，通知客户端保存 Cookie。当下次客户端再往服务器发送请求时，客户端会自动在请求报文中加入 Cookie 值发送出去。服务器端收到客户端发来的 Cookie 后，会去检查究竟是哪一个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前的状态信息。

形象来说，在客户端第一次请求后，服务器会下发一个装有客户信息的身份证，后续客户端请求服务器的时候，带上身份证，服务器就能认得了。

下图展示了发生 Cookie 交互的情景：

1）**没有 Cookie 信息状态下的请求**：

![](https://gitee.com/veal98/images/raw/master/img/20210126184705.png)

对应的 HTTP 请求报文（没有 Cookie 信息的状态）

```
GET /reader/ HTTP/1.1
Host: baidu.com
* 首部字段没有 Cookie 的相关信息
```

对应的 HTTP 响应报文（服务端生成 Cookie 信息）

```
HTTP/1.1 200 OK
Date: Thu, 12 Jul 2020 15:12:20 GMT
Server: Apache
<Set-Cookie: sid=1342077140226; path=/; expires=Wed, 10-Oct-12 15:12:20 GMT>
Content-Type: text/plain; charset=UTF-8
```

2）**第 2 次以后的请求（存有 Cookie 信息状态）**

![](https://gitee.com/veal98/images/raw/master/img/20210126184936.png)

对应的 HTTP 请求报文（自动发送保存着的 Cookie 信息）

```
GET /image/ HTTP/1.1
Host: baidu.com
Cookie: sid=1342077140226
```

## 8. HTTP 断点续传

所谓断点续传指的是下载传输文件可以中断，之后重新下载时可以接着中断的地方开始下载，而不必从头开始下载。断点续传需要客户端和服务端都支持。

这是一个非常常见的功能，原理很简单，其实就是 HTTP 请求头中的字段 `Range` 和响应头中的字段 `Content-Range` 的简单使用。客户端一块一块的请求数据，最后将下载回来的数据块拼接成完整的数据。打个比方，浏览器请求服务器上的一个服务，所发出的请求如下： 

假设服务器域名为 www.baidu.com，文件名为 down.zip。 

```
GET /down.zip HTTP/1.1 
Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms- 
excel, application/msword, application/vnd.ms-powerpoint, */* 
Accept-Language: zh-cn 
Accept-Encoding: gzip, deflate 
User-Agent: Mozilla/4.0 (compatible; MSIE 5.01; Windows NT 5.0) 
Connection: Keep-Alive 
```

服务器收到请求后，按要求寻找请求的文件，提取文件的信息，然后返回给浏览器，返回信息如下： 

```
200 
Content-Length=106786028 
Accept-Ranges=bytes 
Date=Mon, 30 Apr 2001 12:56:11 GMT 
ETag=W/"02ca57e173c11:95b" 
Content-Type=application/octet-stream 
Server=Microsoft-IIS/5.0 
Last-Modified=Mon, 30 Apr 2001 12:56:11 GMT 
```

OK，那么既然要断点续传，客户端浏览器请求服务器的时候要多加一条信息 — 从哪里开始请求数据。 比如要求从 2000070 字节开始：

```
GET /down.zip HTTP/1.0 
User-Agent: NetFox 
RANGE: bytes=2000070- 
Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2 
```

仔细看一下就会发现多了一行 `RANGE: bytes=2000070- `。这一行的意思就是告诉服务器 down.zip 这个文件从 2000070 字节开始传，前面的字节不用传了。 

服务器收到这个请求以后，返回的信息如下： 

```
206
Content-Length=106786028
Content-Range=bytes 2000070-106786027/106786028
Date=Mon, 30 Apr 2001 12:55:20 GMT
ETag=W/"02ca57e173c11:95b"
Content-Type=application/octet-stream
Server=Microsoft-IIS/5.0
Last-Modified=Mon, 30 Apr 2001 12:55:20 GMT
```

和前面服务器返回的信息比较一下，就会发现增加了一行： `Content-Range=bytes 2000070-106786027/106786028 `。返回的代码也改为 206 了，而不再是 200 了。

## 9. HTTP 的缺点

到现在为止，我们已经了解到了 HTTP 具有相当优秀和方便的一面，然后，事务皆有两面性，他也是有不足之处的：

- 通信使用明文（不加密），内容可能被**窃听**
- 不验证通信对方的身份，因此有可能遭遇**伪装**
- 无法证明报文的完整性，所以有可能被**篡改**

这些问题不仅在 HTTP 上出现，其他未加密的协议中也存在类似问题，为了解决 HTTP 的痛点，HTTPS 应用而生，说白了 **HTTP + 加密 + 认证 + 完整性保护就是 HTTPS 协议**，关于 HTTPS 协议的内容也非常之多且重要，后续会单开一篇文章进行讲解。