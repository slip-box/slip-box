#Buffer

Buffer库为Node.js带来了一种存储原始数据的方法，可以让Nodejs处理二进制数据，内存分配不是在V8的堆内存中，在Node的C++层面实现内存的申请；

ps：如果没有Buffer，是不是总要encode和decode

在 lib/buffer.js 模块中，有个模块私有变量 pool， 它指向当前的一个8K 的slab：

```js
Buffer.poolSize = 8 * 1024;
var pool;

function allocPool() {
  pool = new SlowBuffer(Buffer.poolSize);
  pool.used = 0;
}
```

如果申请的空间大于8K，node 会直接调用SlowBuffer从新申请 ，如果小于8K ，新的Buffer 会建立在当前slab 之上。

## buffer的浅拷贝和深拷贝

浅拷贝不在赘述：buf2引用了buf1

```js
const buf1 = Buffer.allocUnsafe(26);

for (var i = 0 ; i < 26 ; i++) {
  buf1[i] = i + 97; // 97 is ASCII a
}

const buf2 = buf1.slice(0, 3);
buf2.toString('ascii', 0, buf2.length);
  // Returns: 'abc'
buf1[0] = 33;
buf2.toString('ascii', 0, buf2.length);
  // Returns : '!bc'
```

深拷贝：要用buf1.copy方法才行；

## 内存碎片

最小申请了8K的内存，但没有用完，剩余的则是内存碎片，无法使用【node总体约浪费1/2的空间】；如果减少内存碎片则要必须承受一定的管理代价，如tcmalloc，node并没有采用，只是选择了性能和内存的平衡；

## zero fill

命令行选项 `--zero-fill-buffers`, 强制在申请 `Buffer`时用0填充分配的内存；为什么？

- malloc：申请的空间可能存在历史脏数据
- calloc：申请的空间重置为0【node对buffer的零填充】
- realloc：申请后可扩充或裁剪内存空间

 `--zero-fill-buffers`相当于用calloc申请

## 为什么Buffer.allocUnsafe()和Buffer.allocUnsafeSlow()不安全？

同时，可能存在脏数据，有被攻击的风险



## 总结

- Buffer是一个典型的Javascript和C++结合的模块，性能相关部分用C++实现，非性能相关部分用javascript实现。

- Node在进程启动时Buffer就已经加装进入内存，并将其放入全局对象，因此无需require。

- Buffer内存分配，Buffer对象的内存分配不是在V8的堆内存中，在Node的C++层面实现内存的申请。

- 内存碎片

- 零填充

