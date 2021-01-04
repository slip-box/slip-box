# HTTP

HTTP 协议属于应用层，在下层的传输层通常使用的是 TCP 协议，所以 net.Server 类正是 http.Server 类的父类：

```
// lib/_http_server.js
function Server(requestListener) {
  net.Server.call(this, { allowHalfOpen: true });

  if (requestListener) {
    this.addListener('request', requestListener);
  }
  this.addListener('connection', connectionListener);
  this.addListener('clientError', function(err, conn) {
    conn.destroy(err);
  });

  this.timeout = 2 * 60 * 1000;
}
util.inherits(Server, net.Server);
```

http继承net.Server，内部监听request、connection等事件，默认超时timeout为2分钟

## HTTP Parser

HTTP Parser解析通过 TCP 传输过来的数据；

parser 是从一个“池”中获取的，这个“池”使用了一种叫做 freelist的数据结构；

HTTPParser 的实现目前由 C++绑定实现，具体参见 deps/http_parser 目录

## 社区争议：http_parser 性能上 JS 实现的版本超越 C 的实现

原因：

- 去调了 C++ 绑定层。
- JS 实现，避免了 C 栈和 JS 堆栈的切换和参数拷贝。
- V8 JIT 对热点函数的优化。

js版http_parser已经提交但没合入，原因：

- 并发请求会导致 garbage collection 频繁，触发GC 停顿。
- 可以作为第三方模块存在。

## incoming和outgoing

两个队列：`incoming`和`outgoing`, 他们用于缓冲 IncomingMessage 实例和对应的 ServerResponse 实例

## req和res

req和res相互引用：

```js
req.res = res;
res.req = req;
```

## 总结：

- http继承net.server
- http_parser把tcp消息转为http：性能上通常js版由于c++版，但并未合入，调用c++有 C 栈和 JS 堆栈的切换消耗
- req和res相互引用

