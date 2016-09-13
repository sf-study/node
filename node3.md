## TTP服务器与客户端

Node.js 标准库提供了 http 模块，其中封装了一个高效的 HTTP 服务器和一个简易的HTTP客户端。

http.Server 是一个基于事件的 HTTP服务器，它的核心由 Node.js 下层 C++部分实现，而接口由 JavaScript 封装，兼顾了高性能与简易性。

http.request 则是一个HTTP 客户端工具，用于向 HTTP 服务器发起请求，例如实现 Pingback①或者内容抓取。

### HTTP服务器

http.Server 是 http 模块中的 HTTP 服务器对象，用 Node.js 做的所有基于 HTTP 协议的系统，如网站、社交应用甚至代理服务器，都是基于 http.Server 实现的。

它提供了一套封装级别很低的 API，仅仅是流控制和简单的消息解析，所有的高层功能都要通过它的接口来实现。

#### http.Server 的事件

http.Server 是一个基于事件的 HTTP 服务器，所有的请求都被封装为独立的事件，开发者只需要对它的事件编写响应函数即可实现 HTTP 服务器的所有功能。它继承自EventEmitter，提供了以下几个事件。

+ request，当客户端请求到来时，该事件被触发，提供两个参数 req 和res，分别是http.ServerRequest 和 http.ServerResponse 的实例，表示请求和响应信息。

+ connection，当 TCP 连接建立时，该事件被触发，提供一个参数 socket，为net.Socket 的实例。 connection 事件的粒度要大于 request，因为客户端在Keep-Alive 模式下可能会在同一个连接内发送多次请求。

+ close，当服务器关闭时，该事件被触发。注意不是在用户连接断开时

+ 除此之外还有 checkContinue、 upgrade、 clientError 事件，通常我们不需要关心，只有在实现复杂的 HTTP 服务器的时候才会用到。

#### http.ServerRequest

http.ServerRequest 是 HTTP 请求的信息

一般由http.Server 的 request 事件发送，作为第一个参数传递，通常简称 request 或 req。ServerRequest 提供一些属性：

|名称|含义|
|:--|---|
|complete|客户端请求是否已经发送完成|
|httpVersion|HTTP 协议版本，通常是 1.0 或 1.1|
|method|HTTP 请求方法，如 GET、 POST、 PUT、 DELETE 等|
|url|原始的请求路径，例如 /static/image/x.jpg 或 /user?name=byvoid|
|headers|HTTP 请求头|
|trailers|HTTP 请求尾（不常见）|
|connection|当前 HTTP 连接套接字，为 net.Socket 的实例|
|socket|connection 属性的别名|
|client|client 属性的别名|

HTTP 请求一般可以分为两部分： 请求头（Request Header）和请求体（Requset Body）。

以上内容由于长度较短都可以在请求头解析完成后立即读取。而请求体可能相对较长，需要一定的时间传输，因此 http.ServerRequest 提供了以下3个事件用于控制请求体传输。

+ data。当请求体数据到来时，该事件被触发。该事件提供一个参数 chunk，表示接收到的数据。如果该事件没有被监听，那么请求体将会被抛弃。该事件可能会被调用多次。

+ end ：当请求体数据传输完成时，该事件被触发，此后将不会再有数据到来。

+ close： 用户当前请求结束时，该事件被触发。不同于 end，如果用户强制终止了传输，也还是调用close。

#### 获取 GET 请求内容

由于 GET 请求直接被嵌入在路径中，URL是完整的请求路径，包括了 ? 后面的部分，因此你可以手动解析后面的内容作为 GET请求的参数。 Node.js 的 url 模块中的 parse 函数提供了这个功能

#### 获取 POST 请求内容

HTTP协议 1.1 版本提供了8种标准的请求方法，其中最常见的就是 GET 和 POST。

相比GET 请求把所有的内容编码到访问路径中， POST 请求的内容全部都在请求体中。

http.ServerRequest 并没有一个属性内容为请求体，原因是等待请求体传输可能是一件耗时的工作，譬如上传文件。而很多时候我们可能并不需要理会请求体的内容，恶意的 POST请求会大大消耗服务器的资源。所以 Node.js 默认是不会解析请求体的，当你需要的时候，需要手动来做。

#### http.ServerResponse

http.ServerResponse 是返回给客户端的信息，决定了用户最终能看到的结果。它也是由 http.Server 的 request 事件发送的，作为第二个参数传递，一般简称为response 或 res。

http.ServerResponse 有三个重要的成员函数，用于返回响应头、响应内容以及结束请求。

+ response.writeHead(statusCode, [headers])：向请求的客户端发送响应头。statusCode 是 HTTP 状态码，如 200 （请求成功）、 404 （未找到）等。 headers是一个类似关联数组的对象，表示响应头的每个属性。该函数在一个请求内最多只能调用一次，如果不调用，则会自动生成一个响应头。

+ response.write(data, [encoding])：向请求的客户端发送响应内容。 data 是一个 Buffer 或字符串，表示要发送的内容。如果 data 是字符串，那么需要指定encoding 来说明它的编码方式，默认是 utf-8。在 response.end 调用之前，response.write 可以被多次调用。

+ response.end([data], [encoding])：结束响应，告知客户端所有发送已经完成。当所有要返回的内容发送完毕的时候，该函数 必须 被调用一次。它接受两个可选参数，意义和 response.write 相同。如果不调用该函数，客户端将永远处于等待状态。

### HTTP 客户端

http 模块提供了两个函数 http.request 和 http.get，功能是作为客户端向 HTTP服务器发起请求。

#### http.request(options, callback)

http.request(options, callback)发起 HTTP请求。接受两个参数， option 是一个类似关联数组的对象，表示请求的参数， callback 是请求的回调函数。

option常用的参数如下所示：

+ host:请求网站的域名或IP地址

+ port:请求网站的端口。默认是80

+ method:请求方法，默认是GET

+ path:请求的相对于根路径，默认是'/'，QueryString应该包含在其中。例如 /search?query=byvoid。

+ headers ：一个关联数组对象，为请求头的内容。

callback 传递一个参数，为 http.ClientResponse 的实例。

http.request 返回一个 http.ClientRequest 的实例。

#### http.get(options, callback)

http.get(options, callback) http 模块还提供了一个更加简便的方法用于处理GET请求： http.get。它是 http.request 的简化版，唯一的区别在于http.get自动将请求方法设为了 GET 请求，同时不需要手动调用 req.end()。

#### http.ClientRequest

http.ClientRequest 是由 http.request 或 http.get 返回产生的对象，表示一个已经产生而且正在进行中的 HTTP请求。

它提供一个 response 事件，即 http.request或 http.get 第二个参数指定的回调函数的绑定对象。我们也可以显式地绑定这个事件的监听函数

http.ClientRequest 像 http.ServerResponse 一样也提供了 write 和 end 函数，用于向服务器发送请求体，通常用于 POST、 PUT 等操作。所有写结束以后必须调用 end函数以通知服务器，否则请求无效。

http.ClientRequest 还提供了以下函数：

+ request.abort()：终止正在发送的请求。

+ request.setTimeout(timeout, [callback])：设置请求超时时间， timeout 为毫秒数。当请求超时以后， callback 将会被调用。

#### http.ClientResponse

http.ClientResponse 与 http.ServerRequest 相似，提供了三个事件 data、 end和 close，分别在数据到达、传输结束和连接结束时触发，其中 data 事件传递一个参数chunk，表示接收到的数据。

http.ClientResponse 也提供了一些属性，用于表示请求的结果状态：

|名称|含义|
|:--|---|
|statusCode|HTTP 状态码，如 200、 404、 500|
|httpVersion|HTTP 协议版本，通常是 1.0 或 1.1|
|headers|HTTP 请求头|
|trailers|HTTP 请求尾（不常见）|

http.ClientResponse 还提供了以下几个特殊的函数:

+ response.setEncoding([encoding])：设置默认的编码，当 data 事件被触发时，数据将会以 encoding 编码。默认值是 null，即不编码，以 Buffer 的形式存储。常用编码为 utf8。

+ response.pause()：暂停接收数据和发送事件，方便实现下载功能。

+ response.resume()：从暂停的状态中恢复。