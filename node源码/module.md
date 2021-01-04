#module

node中包含以下4中模块

- （C++原生模块）builtin module: Node 中以 c++ 形式提供的模块，如：tcp_wrap、contextify 
- constants module: Node 中定义常量的模块，用来导出如 signal, openssl 库
- node原生模块，native module: Node 中以 JavaScript 形式提供的模块，如 http,https,fs，背后依赖原生C++模块，如：buffer 模块需要借助 builtin node_buffer.cc
- 第三方模块npm包

## require 是怎么来的？

[lib/module.js](https://github.com/nodejs/node/blob/v4.4.0/lib/module.js) 的中有如下代码：

```javascript
Module.prototype.require = function(path) {
  assert(path,'missing path');
  assert(typeof path ==='string','path must be a string');
  return Module._load(path, this);
};
```

_load函数：

- 先判断Module._cache中是否已加载【cache 解决无限循环引用的问题，通过以空间换时间，使得每次加载模块变得非常高效】
- 如果是原生的模块，通过调用 `NativeModule.require()` 返回结果
- 否则，创建一个新的模块，并保存到缓存中

```js
Module._load = function(request, parent, isMain) {
  if (parent) {
    debug('Module._load REQUEST %s parent: %s', request, parent.id);
  }

  var filename = Module._resolveFilename(request, parent);

  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    return cachedModule.exports;
  }

  if (NativeModule.nonInternalExists(filename)) {
    debug('load native module %s', request);
    return NativeModule.require(filename);
  }

  var module = new Module(filename, parent);

  if (isMain) {
    process.mainModule = module;
    module.id = '.';
  }

  Module._cache[filename] = module;

  var hadException = true;

  try {
    module.load(filename);
    hadException = false;
  } finally {
      if (hadException) {
        delete Module._cache[filename];
      }
  }

  return module.exports;
};
```

再看require：

- 先判断是否是原生模块，再判断缓存中有没有，最后push到moduleLoadList中
- 其中关键：nativeModule.compile();

```js
NativeModule.require = function(id) {
    if (id =='native_module') {
      return NativeModule;
    }

    var cached = NativeModule.getCached(id);
    if (cached) {
      return cached.exports;
    }

    if (!NativeModule.exists(id)) {
      throw new Error('No such native module '+ id);
    }

    process.moduleLoadList.push('NativeModule' + id);

    var nativeModule = new NativeModule(id);

    nativeModule.cache();
    nativeModule.compile();

    return nativeModule.exports;
  };
```

再来看nativeModule.compile();：

- `wrap` 函数将 http.js 包裹起来, 交由 `runInThisContext` 编译源码，返回 fn 函数, 依次将参数传入
- runInThisContext【类似于eval】来自于：require('vm')，是js代码的运行沙箱

```js
NativeModule.getSource = function(id) {
  return NativeModule._source[id];
};

NativeModule.wrap = function(script) {
  return NativeModule.wrapper[0] + script + NativeModule.wrapper[1];
};

NativeModule.wrapper = ['(function (exports, require, module, __filename, __dirname) {','\n});' ];

NativeModule.prototype.compile = function() {
  var source = NativeModule.getSource(this.id);
  source = NativeModule.wrap(source);

  var fn = runInThisContext(source, {
    filename: this.filename,
    lineOffset: 0
  });
  fn(this.exports, NativeModule.require, this, this.filename);

  this.loaded = true;
};
```

## process

process是底层 C++ 传递给 javascript 的一个变量，在一开始运行 node.js 时，把 process 作为参数去调用 js 主程序 src/node.js 返回的函数，这样 process 就传递到 javascript 里了

## 总结

#### 总结

- 模块分为C++原生模块，node模块，第三方模块npm包
- 模块通过require导入，中间先判断原生，在判断是否有缓存cache中，最后加载

注意：node模块哪怕只运行一次，加载后也常驻内存，增加了内存消耗，【空间换时间】

如果要降低内存：是否可考虑模块下载机制，降低node内存消耗？提升垃圾回收效率

