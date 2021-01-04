# node架构

Node.js主要分为四大部分：

- Node Standard Library：标准库：http、net、fs、events、buffer

- Node Bindings：沟通JS 和 C++的桥梁，封装V8和Libuv的细节，向上层提供基础API服务

- V8：JavaScript引擎，提供JavaScript运行环境，可以说它就是 Node.js 的发动机【解释、执行】

- Libuv：专门为Node.js开发的一个封装库，提供跨平台的异步I/O能力.

架构图如下:

<img src="/Users/wangjin/Library/Application Support/typora-user-images/image-20200727024653363.png" alt="image-20200727024653363" style="zoom:50%;" />

## Libuv



![img](https://yjhjstz.gitbooks.io/deep-into-node/content/chapter1/FuX1qcGJgwYtX9zNbBAOSaQeD8Qz.png)

从图中可以看出，对于Network I/O和以File I/O为代表的另一类请求，异步处理的底层支撑机制是完全不一样的。

- 对于Network I/O相关的请求， 根据OS平台不同，分别使用Linux上的epoll，OSX和BSD类OS上的kqueue，SunOS上的event ports以及Windows上的IOCP机制。

- 对于File I/O为代表的请求，则使用thread pool。利用thread pool的方式实现异步请求处理，在各类OS上都能获得很好的支持。

## V8

架构图：

<img src="https://yjhjstz.gitbooks.io/deep-into-node/content/chapter2/e09d7b330d9e754f7ff1282a1af55295.png" alt="img" style="zoom:100%;" />

JS 引擎的执行过程大致是：源代码 --->抽象语法树 --->字节码 --->JIT--->本地代码；



在使用 v8 引擎之前，先来了解一下几个基本概念：句柄（handle），作用域（scope），上下文环境（可以简单地理解为运行环境）。

## 先看：Isolate

一个 Isolate 是一个独立的虚拟机。对应一个或多个线程。但同一时刻 只能被一个线程进入。所有的 Isolate 彼此之间是完全隔离的, 它们不能够有任何共享的资源。如果不显式创建 Isolate, 会自动创建一个默认的 Isolate。

后面提到的 Context、Scope、Handle 的概念都是一个 Isolate 内部的，：

![img](https://yjhjstz.gitbooks.io/deep-into-node/content/chapter2/Context.png)



## Handle 概念

一句话：V8中对象都是用 Handle 方式引用的：

V8 为了对内存分配进行管理，GC 需要对 V8 中的 所有对象进行跟踪，而对象都是用 Handle 方式引用的，所以 GC 需要对 Handle 进行管理，这样 GC 就能知道 Heap 中一个对象的引用情况，当一个对象的 Handle 引用发生改变的时候，GC 即可对该对象进行回收或者移动。

Handle 分为 Local 和 Persistent 两种：

- Local 是局部的，它同时被 HandleScope 进行管理，作用域较小。
-  persistent，类似与全局的，不受 HandleScope 的管理，其作用域可以延伸到不同的函数

## Scope

作用域可以看成是一个句柄的容器，在一个作用域里面可以有很多很多个句柄

## Context 概念

上下文环境也可以理解为运行环境。在执行 javascript 脚本的时候，总要有一些环境变量或者全局函数。 

据说设计 Context 的最初目的是为了让浏览器在解析 HTML 的 iframe 时，让每个 iframe 都有独立的 javascript 执行环境，即一个 iframe 对应一个 Context。

Context可嵌套

## 垃圾回收

### 原始算法：引用计数垃圾收集

此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。



问题：无法处理循环引用：IE 6, 7 使用引用计数方式,循环引用时内存发生泄漏：

```js
function f(){
  var o = {};
  var o2 = {};
  o.a = o2; // o 引用 o2
  o2.a = o; // o2 引用 o

  return "azerty";
}

f();
```

###现在：标记-清除算法

这个算法把“对象是否不再需要”简化定义为“对象是否可以【从根全局对象中】获得”。

从2012年起，所有现代浏览器都使用了标记-清除垃圾回收算法

