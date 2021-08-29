[[缓存]]是计算机科学的核心方案之一，大到文件，小到CPU寄存器，都采用了缓存策略；
http协议的缓存，可以分为两部分理解：服务器缓存、客户端缓存

## 服务器缓存

1、浏览器发现缓存无数据，于是发送请求，向服务器获取资源；
2.服务器响应请求，返回资源，同时标记资源的有效期；
3、浏览器缓存资源，等待下次重用

注意：Cache-Control”字段里的“max-age”字段是文件离开服务器的时间，而非客户端收到的时间，如果传输花了5秒，那么客户端缓存时间将扣除5秒的时间

## 客户端缓存
不止服务端，客户端也能发送：Cache-Control
- 强刷时Cache-Control：no-cache【等同于max-age=0】
- 浏览器后退时，只用基本的请求头，此时不含Cache-Control：在状态码旁边能看到：from disk cache  来自硬盘缓存；还有来自内存缓存
- 资源验证请求：验证资源是否被修改的条件有两个：“Last-modified”和“ETag”，需要服务器预先在响应报文里设置，搭配条件请求使用


强缓存：catch-contrl：max-age，【private（默认，仅最终client可缓存，中间代理不可）、public、no-cache、max-age，no-store】服务器通知浏览器一个缓存时间，在缓存时间内

协商缓存：时间到期了，再对比Etag（先）和Last-Modified（后）通过请求发送给服务器，由服务器校验，返回304

缓存来源：Service Worker（必须HTPPS环境，浏览器背后的独立线程）
Memory Cache （内存，小文件）
Disk Cache（硬盘）
Push Cache（http2）

> [[http]] 