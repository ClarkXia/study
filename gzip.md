### 前言

在前端性能优化手段中，脚本的大小一直是被重视的一个点，除了uglify的压缩手段之外，基于HTTP协议的网络传输优化中，GZip经常会被提及到。那GZip的压缩原理究竟是什么，能使文件在被压缩后在网络上的传输大小能够再次被大幅度缩减？本文是我从网上查找的一些文档资料，希望能够把这件事弄明白。

### 压缩规范RFC1952

[RFC1952](https://tools.ietf.org/html/rfc1952) 这个规范主要定义了GZip压缩在数据格式方面的规范，感兴趣的可以直接阅读RFC1952原文。

### gzip工作原理

![](/assets/gzip.png)

1. 浏览器请求资源，并在request header里设置accept-encoding: gzip
2. 服务器收到浏览器发送的请求，根据header里面的属性判断是否支持gzip，如果支持，则向浏览器传送压缩过的内容，不支持则发送原始内容。一般浏览器和服务器都支持gzip的话，response headers返回将包含content-encoding: zip
3. 浏览器接收到响应之后，判断是否是压缩内容，如果被压缩则先解压



