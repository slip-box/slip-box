# Stream

流，解决了处理大文件问题：

```js
var http = require('http');
	var fs = require('fs');

	var server = http.createServer(function (req, res) {
	    fs.readFile(__dirname + 'data.txt', function (err, data) {
	        res.end(data);
	    });
	});
	server.listen(8000);
```

如果data.txt很大，读入内存显然不合理；

改用流：因为(req,res)也是流对象，所以可用流读写：

```js
var http = require('http');
	var fs = require('fs');

	var server = http.createServer(function (req, res) {
	    var stream = fs.createReadStream(__dirname + '/data.txt');
	    stream.pipe(res);
	});
	server.listen(8000);
```

想要将数据进行压缩？我们可以使用相应的流模块完成这项工作!：

```js
var server = http.createServer(function (req, res) {
	    var stream = fs.createReadStream(__dirname + '/data.txt');
	    stream.pipe(oppressor(req)).pipe(res);
	});
```

node中的流的应用：

- (req,res)
- 文件读取

## 流的类型：

Readable：只读流

Writable：只写流

Duplex：全双工可读可写流

Transform：转换流，操作被写入数据，然后读出结果

## Duplex：可读可写，全双工流

原理：很简单，基于可读流，原型继承可写流的方法：

`Duplex` 首先继承了 `Readable`, 因为 javascript 没有 C++的多重继承的特性，所以 遍历 `Writable`的原型方法然后赋值到 `Duplex`的原型上。：

```js
const Readable = require('_stream_readable');
const Writable = require('_stream_writable');

util.inherits(Duplex, Readable);

var keys = Object.keys(Writable.prototype);
for (var v = 0; v < keys.length; v++) {
  var method = keys[v];
  if (!Duplex.prototype[method])
    Duplex.prototype[method] = Writable.prototype[method];
}
```

### transform流

转换流（Transform streams）是一种输出由输入计算所得的双工流。它同时实现了 Readable 和 Writable 接口。

Node中的转换流有：

- zlib streams
- crypto streams

你可以将transform流想象成一个流的中间部分，它可以读也可写，但是并不保存数据，它只负责处理流经它的数据。

### 总结

流式处理的优势: 将功能切分，并通过管道组合。

- 全双工流：基于可读流，继承可写流的方法