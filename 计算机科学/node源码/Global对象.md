# Global对象

全局对象：console、process

全局函数：Buffer、require、setTimeout、setInterval、clearTimeout

伪全局变量：_filename、_dirname

## module.exports vs exports

两点核心概念

- exports = module.exports = {};
- 而node导出的【也就是require加载的】，永远是module.exports指向的对象 

<img src="/Users/wangjin/Library/Application Support/typora-user-images/image-20200725025928267.png" alt="image-20200725025928267" style="zoom:50%;" />

案例：

```
// utils.js
exports.a = 200;
exports = 'test'; // 修改了exports的内存指向，此时module.exports依然为{a:200}
// b.js
var a = require('/utils'); // 导出的永远是module.exports
console.log(a) // 打印为 {a : 200};因为
```

