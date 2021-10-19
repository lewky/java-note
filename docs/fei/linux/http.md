# http协议

## http版本区别

* http1.0：客户端与web服务器建立连接后，一个链接只能获得一个web资源
* http1.1：客户端与web服务器建立连接后，一个链接能获得多个web资源

## http request

客户端与web服务器建立连接后，向服务器请求某个web资源，称为客户端向服务器发送了一个http request(请求行，请求头，实体内容)

* 请求行

  用于描述客户端的请求方式，请求的资源名称，使用的http协议版本号
* 请求头

  描述客户端的基本信息和请求数据的描述
  * `Accept`：客户端支持的数据类型
  * `Accept-Charset`：客户端采用的编码
  * `Accept-Encoding`：客户端支持的压缩格式
  * `Accept-Language`：客户端的语言环境
  * `Host`：客户端想访问的主机
  * `If-Modified-Since`：资源的缓冲时间
  * `Referer`：表明客户端是从哪个资源来访问服务器
  * `User-Agent`：表明客户端的软件环境
  * `cookie`：存储在客户端的用户信息
  * `Connection`：请求完了之后，保持连接还是断开连接
  * `Date`：请求的当前时间
* 实体内容

  http request body

## http response

服务器被客户端请求后，返回给客户端一个http response(状态行，响应头，一个空行，实体内容)

* 状态行

  描述服务器对请求的处理结果(http版本号 状态码 原因叙述, HTTP/1.1 200 ok)

  状态码

  * 100~199：表示成功接收请求，要求客户端继续提交下一次请求才能完成整个处理过程
  * 200~299：表示成功接收并已完成整个处理过程
  * 300~399：完成请求，客户端需要进一步细化请求
  * 400~499：客户端请求错误
  * 500~599：服务器处理错误
* 响应头

  描述服务器的基本信息和返回数据的描述
  * `Location`：原资源重定向后的新地址
  * `Server`：服务器的类型
  * `Content-Encoding`：数据的压缩格式
  * `Content-Length`：返回数据的长度
  * `Content-Language`：返回数据的语言
  * `Content-Type`：回返数据的类型
  * `Last-Modified`：当前资源的最后缓存时间
  * `Refresh`：需要刷新的间隔时间
  * `Content-Disposition`：数据处理的方式
  * `Transfer-Encoding`：数据的传送格式
  * `Set-Cookie`：设置cookie
  * `Expires`：返回资源应该缓存的时长
  * `Cache-Control`：通知客户端数据应不应该缓存
  * `Connection`：响应完成后，保持连接还是断开连接
  * `Date`：响应的当前时间
  * `range`：断点下载
