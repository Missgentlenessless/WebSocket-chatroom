## WebSocket技术分享

### 第一部分 WebSocket简介

#### 1.WebSocket定义（都是套路（ー㉨ー|||））

WebSocket是一个新协议。RFC（The official Request For Comments）这样描述：
> WebSocket协议允许在受控环境中运行不受信任的代码的客户端与从该代码中选择通信的远程主机进行双向通信。
> 用于此安全模型是Web浏览器常用的基于源的安全模型。该协议包括一个开放的握手，然后是基本的消息框架，在TCP上分层。
>该技术的目标是为基于浏览器的应用程序提供一种机制，该机制需要与不依赖于打开多个HTTP连接的服务器进行双向通信。

#### 2. WebSocket 握手过程（专业性太强，了解一下~）

**要发起WebSocket通信，首先需要进行HTTP握手。**所以在Websocket通信开始之前，必须启动HTTP连接。

- 从HTTP协议切换到WebSocket协议时，浏览器会向服务器发送一个升级头，以通知他想要启动一个WebSocket连接。
```
	GET /chat HTTP/1.1
        Host: server.example.com
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
        Origin: http://example.com
        Sec-WebSocket-Protocol: chat, superchat
        Sec-WebSocket-Version: 13
```

  - 其中，
```
Upgrade: websocket
Connection: Upgrade
```
这个就是Websocket的核心了，告诉Apache、Nginx等服务器请求协议升级

  - 其次，
```
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```
这三行中，Sec-WebSocket-Key 是一个Base64 encode 的值，这个是浏览器随机生成的验证码。

然后，Sec_WebSocket-Protocol 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。

最后，Sec-WebSocket-Version 是告诉服务器所使用的Websocket Draft（协议版本）。

- 如果服务器支持WebSocket协议，它将发送下面的头响应。
```
	HTTP/1.1 101 Switching Protocols
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
        Sec-WebSocket-Protocol: chat
```
  - 其中，
```
Upgrade: websocket
Connection: Upgrade
```
依然是固定的，告诉客户端即将升级的是Websocket协议。

  - 然后，Sec-WebSocket-Accept 这个则是经过服务器确认，并且加密过后的 Sec-WebSocket-Key。
  
  后面的Sec-WebSocket-Protocol 则是表示最终使用的协议。

#### 3.WebSocket API （精华在此，不要错过↓↓↓）

WebSocket对象提供了用于创建和管理 WebSocket 连接，以及可以通过该连接发送和接收数据的 API。

**WebSocket API是 HTML5 标准的一部分， 但这并不代表 WebSocket 一定要用在 HTML 中，或者只能在基于浏览器的应用程序中使用。**
实际上，许多语言、框架和服务器都提供了 WebSocket 支持，例如：

- 基于 C 的 libwebsocket.org
- 基于 Node.js 的 Socket.io
- 基于 Python 的 ws4py
- 基于 C++ 的 WebSocket++
等等。。。

*本节以下内容来自：https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket

**构造器**

WebSocket构造器方法接受一个必须的参数和一个可选的参数：

`WebSocket WebSocket(in DOMString url, in optional DOMString protocols);`

`WebSocket WebSocket(in DOMString url, in optional DOMString[] protocols);`

- 参数① url
    表示要连接的URL。这个URL应该为响应WebSocket的地址。
- 参数② protocols 可选
    可以是一个单个的协议名字字符串或者包含多个协议名字字符串的数组。这些字符串用来表示子协议，这样做可以让一个服务器实现多种WebSocket子协议（例如你可能希望通过制定不同的协议来处理不同类型的交互）。如果没有制定这个参数，它会默认设为一个空字符串。
- 构造器方法可能抛出以下异常：
  - SECURITY_ERR
    试图连接的端口被屏蔽。

**方法**

`void close(in optional unsigned short code, in optional DOMString reason);`

- 参数① code 可选
    一个数字值表示关闭连接的状态号，表示连接被关闭的原因。如果这个参数没有被指定，默认的取值是1000 （表示正常连接关闭）。 请看CloseEvent页面的 list of status codes来看默认的取值。
- 参数② reason 可选
    一个可读的字符串，表示连接被关闭的原因。这个字符串必须是不长于123字节的UTF-8 文本（不是字符）。
- 可能抛出的异常：
  - INVALID_ACCESS_ERR
    选定了无效的code。
  - SYNTAX_ERR
    reason 字符串太长或者含有unpaired surrogates。

`void send(in USVString data);`

`void send(in ArrayBuffer data);`

`void send(in Blob data);`

`void send(in ArrayBufferView data);`

- 参数① data
    要发送到服务器的数据。
- 可能抛出的异常
  - INVALID_STATE_ERR
    当前连接的状态不是OPEN。
  - SYNTAX_ERR
    数据是一个包含unpaired surrogates的字符串。

**注意:** Gecko 6.0实现的send()方法与规范的要求有一些不同。
Gecko会返回一个 boolean表示连接是否依然处于开启状态 （并且这个数据被成功放入的发送队列或者被发送）。
在 Gecko 8.0中这个问题被修正了。到了 Gecko 11.0，实现了接受 ArrayBuffer的参数的方法，但接受 Blob数据类型的方法没有被实现。

**属性**

 --- 属性名 ---------- 类型 -------------------- 描述--------------------------------
- binaryType ------	DOMString ------- 一个字符串表示被传输二进制的内容的类型。取值应当是"blob"或者"arraybuffer"。"blob"表示使用DOMBlob 对象，而"arraybuffer"表示使用 ArrayBuffer 对象。
- bufferedAmount --	unsigned long	--- 调用 send() 方法将多字节数据加入到队列中等待传输，但是还未发出。该值会在所有队列数据被发送后重置为 0。而当连接关闭时不会设为0。如果持续调用send()，这个值会持续增长。只读。
- extensions ------	DOMString	------- 服务器选定的扩展。目前这个属性只是一个空字符串，或者是一个包含所有扩展的列表。
- onclose -------	EventListener	----- 用于监听连接关闭事件监听器。当 WebSocket 对象的readyState 状态变为 CLOSED 时会触发该事件。这个监听器会接收一个叫close的 CloseEvent 对象。
- onerror	--------EventListener -----	当错误发生时用于监听error事件的事件监听器。会接受一个名为“error”的event对象。
- onmessage -----	EventListener	----- 一个用于消息事件的事件监听器，这一事件当有消息到达的时候该事件会触发。这个Listener会被传入一个名为"message"的 MessageEvent 对象。
- onopen --------	EventListener	----- 一个用于连接打开事件的事件监听器。当readyState的值变为 OPEN 的时候会触发该事件。该事件表明这个连接已经准备好接受和发送数据。这个监听器会接受一个名为"open"的事件对象。
- protocol ------- DOMString	------- 一个表明服务器选定的子协议名字的字符串。这个属性的取值会被取值为构造器传入的protocols参数。
- readyState-----	unsigned short	--- 连接的当前状态。取值是 Ready state constants之一。 只读。
- url	------------ DOMString	--------传入构造器的URL。它必须是一个绝对地址的URL。只读。

**常量**

这些常量是 readyState 属性的取值，可以用来描述 WebSocket 连接的状态。

--- 常量 --------	值	------- 描述 -----------------
- CONNECTING ---	0 -----	连接还没开启。
- OPEN	--------  1 ------ 连接已开启并准备好进行通信。
- CLOSING	------  2 ------ 连接正在关闭的过程中。
- CLOSED -------  3	----- 连接已经关闭，或者连接无法建立。

**浏览器兼容性**
 
从Gecko 6.0开始，构造器含有前缀，你需要调用MozWebSocket(): var mySocket = new MozWebSocket("http://www.example.com/socketserver");

extensions 属性直到Gecko 8.0才被支持。

在 Gecko 11.0之前，用send()方法发送的数据被限制在16MB以内。现在数据大小可以达到2 GB。

 
### 第二部分 协议比较（我们来找茬儿~~o(￣ヘ￣o＃)）

#### 1.与HTTP协议的比较
HTTP协议是一种被动的无状态的单向协议。被动，就是不管有没有要发给客户端的消息，只要客户端不发送请求，服务器就不会主动推送消息。无状态，就是这一次发送请求头部里包含了所有的信息服务器并不会保存记录，在下一次发送请求还是要再发一遍所有信息，而HTTP的头部可以达到800甚至2000字节。单向，就是说建立一次HTTP连接，要么是客户端向服务器发消息，要么是服务器向客户端发消息，只能有一个方向。
而相对与HTTP协议，WebSocket协议就是一种能够实现服务器推的全双工协议，让服务器不仅能够主动向客户端推送消息，而不依赖于客户端的请求。而且WebSocket连接一旦建立，客户端和服务器可以互相发送数据。数据包含在帧中，每个帧以4-12字节为前缀。

#### 2.与Socket的比较
套接字（socket）是通信的基石，是支持TCP/IP协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：连接使用的协议，本地主机的IP地址，本地进程的协议端口，远地主机的IP地址，远地进程的协议端口。
应用层通过传输层进行数据通信时，TCP会遇到同时为多个应用程序进程提供并发服务的问题。多个TCP连接或多个应用程序进程可能需要通过同一个 TCP协议端口传输数据。为了区别不同的应用程序进程和连接，许多计算机操作系统为应用程序与TCP／IP协议交互提供了套接字(Socket)接口。应用层可以和传输层通过Socket接口，区分来自不同应用程序进程或网络连接的通信，实现数据传输的并发服务。
创建Socket连接时，可以指定使用的传输层协议，Socket可以支持不同的传输层协议（TCP或UDP），当使用TCP协议进行连接时，该Socket连接就是一个TCP连接。
所以说Socket和WebSocket就像Java和JavaScript，并没有什么太大的关系，但又不能说完全没关系。可以这么说：命名方面，Socket是一个深入人心的概念，WebSocket借用了这一概念；使用方面，完全两个东西。WebSocket工作在应用层，是协议，Socket工作在应用层和传输层的中间层，并不是协议。

*更多精彩请移步：https://www.cnblogs.com/merray/p/7918977.html
 
### 第三部分 新旧技术对比（拓展一下，涨姿势(☆_☆)/~~）

#### 第一类：最简单的基于HTTP的“服务器推”

##### 1.轮询（polling）

客户端定时向服务器发送Ajax请求，服务器接到请求后马上返回响应信息并关闭连接。 
- 优点：后端程序编写比较容易。 
- 缺点：请求中有大半是无用，浪费带宽和服务器资源。 
- 实例：适于小型应用。
 
*本节以下内容详见：https://www.ibm.com/developerworks/cn/web/wa-lo-comet/

#### 第二类：基于客户端套接口的“服务器推”技术

- 特点：需要在浏览器端安装插件，基于套接口传送信息，或是使用 RMI、CORBA 进行远程调用。

##### 2.Flash Socket

在页面中内嵌入一个使用了Socket类的 Flash 程序JavaScript通过调此Flash程序提供的Socket接口与服务器端的Socket接口进行通信，JavaScript在收服务器端传送的信息后控制页面的显示。 
- 优点：实现真正的即时通信，而不是伪即时。 
- 缺点：客户端必须安装Flash插件；非HTTP协议，无法自动穿越防火墙。 
- 实例：网络聊天室，网络互动游戏。

##### 3．Java Applet 套接口

在客户端使用 Java Applet，通过 java.net.Socket 或 java.net.DatagramSocket 或 java.net.MulticastSocket 建立与服务器端的套接口连接，从而实现“服务器推”。
这种方案最大的不足在于 Java applet 在收到服务器端返回的信息后，无法通过 JavaScript 去更新 HTML 页面的内容。 

#### 第三类：基于 HTTP 长连接的“服务器推”技术（comet）

- 特点：无须浏览器安装任何插件、基于 HTTP 长连接

##### 4.基于 AJAX 的长轮询（long-polling）

客户端向服务器发送Ajax请求，服务器接到请求后hold住连接，直到有新消息才返回响应信息并关闭连接，客户端处理完响应信息后再向服务器发送新的请求。 
- 优点：在无消息的情况下不会频繁的请求，耗费资源小。 
- 缺点：服务器hold连接会消耗资源，返回数据顺序无保证，难于管理维护。 
- 实例：WebQQ、Hi网页版、Facebook IM。 
 

##### 5.基于 Iframe 及 htmlfile 的流（streaming）

在页面里嵌入一个隐蔵iframe，将这个隐蔵iframe的src属性设为对一个长连接的请求或是采用xhr请求，服务器端就能源源不断地往客户端输入数据。 
- 优点：消息即时到达，不发无用请求；管理起来也相对方便。 
- 缺点：服务器维护一个长连接会增加开销。 
- 实例：Gmail聊天
 

感谢阅读，欢迎指正o(*^_^*)o
