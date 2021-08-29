# Yield

Generator之所以可用来控制代码流程，就是通过yield来将两个或者多个Generator的执行路径互相切换。这种切换是语句级别的，而不是函数调用级别的。其本质是CPS变换。

## ES6 系列之 Babel 将 Generator 编译成了什么样子

https://segmentfault.com/a/1190000016922141 



## CPS 变换

说白了就是把程序内部原本隐式的控制流跳转，用某种方法抽象出来暴露给程序员。



CPS是将控制流显式表示为continuation的一种编程风格. 简单来理解就是显式使用函数表示函数返回的后续操作.



[co](https://github.com/tj/co) 正是一个解决JavaScript异步流程难题的库。它利用ES2015标准中的 `yield`，实现了与（本来被期望加入ES2016标准却因为大厂跳票没赶上deadline所以只能明年再说的）`async/await` 相似的功能而红透半边天。其原理就是利用 `yield` 的状态机特性，将（看起来是）同步的代码转换为CPS的形式。

co的源文件有两百多行，而其核心思想其实非常简洁——不断的将 `yield` 的 `next` 值追加到 `promise` 链中，达到“一步接一步”执行的效果。



协程，遇到异步调用就 "yield" 放弃 CPU， 交由协程调度



yield 后，实际上就是保存当前的执行环境，保存当前的操作数栈，并保存到JSGeneratorObject对象中。



yield之后，实际上本次调用就结束了，控制权实际上已经转到了外部调用了generator的next方法的函数，调用的过程中伴随着状态的改变。那么如果外部函数不继续调用next方法，那么yield所在函数就相当于停在yield那里了。所以把异步的东西做完，要函数继续执行，只要在合适的地方再次调用generator 的next就行，就好像函数在暂停后，继续执行。