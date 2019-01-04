[How JavaScript works: an overview of the engine, the runtime, and the call stack](https://blog.sessionstack.com/how-does-javascript-actually-work-part-1-b0bacc073cf)

[How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e)

[How JavaScript works: memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)

[How JavaScript works: Event loop and the rise of Async programming + 5 ways to better coding with async/await](https://blog.sessionstack.com/how-javascript-works-event-loop-and-the-rise-of-async-programming-5-ways-to-better-coding-with-2f077c4438b5)
# JavaScript Engine

 > is a program or an interpreter which executes JavaScript code

## 组成
 - 内存堆(Memory Heap)
 - 调用栈(Call Stack)：记录程序执行位置的数据结构

## 各类引擎简表

|Engine|Developer|Develop Language|Open Source|
|:----:|:-------:|:--------------:|:---------:|
|V8|Google|C++|Y|
|Rhino|Mozilla|Java|Y|
|SpiderMonkey|Firefox|-|-|
|JavaScriptCore|Apple|-|Y|
|KJS|Harri Porten|-|-|
|Chakra(JScript9)|Internet Explorer|-|-|
|Chakra(JavaScript)|Microsoft Edge|-|-|
|JerryScript|-|-|-|

### V8 引擎

JIT(Just-In-Time) compiler

>V8 doesn’t produce bytecode or any intermediate code.

#### version < 5.9

##### 两套编译器：

full-codegen：正常编译
Crankshaft：JIT优化编译

##### 多线程

主线程：获取，编译，执行代码
额外编译线程：优化代码
分析进程：分析需要优化的代码
垃圾回收线程：垃圾回收

##### 优化步骤

抽象树(abstract syntax tree AST)

静态单赋值(static single-assignment SSA)

>哈希函数(Hash Function)：A hash function is any function that can be used to map data of arbitrary size to data of a fixed size. The values returned by a hash function are called hash values, hash codes, digests, or simply hashes. Hash functions are often used in combination with a hash table, a common data structure used in computer software for rapid data lookup

1. AST->SSA
2. Inlining，根据调用顺序使用函数体替换调用入口
3. hidden classes：由于JavaScript是弱类型语言，在执行过程中，变量类型随时可变，因此相对强类型语言而言，对变量或属性的寻址过于消耗性能

In such cases, it’s much better to initialize dynamic properties in the same order so that the hidden classes can be reused.

Web APIs是由浏览器提供，而非JS引擎。包括：DOM，AJAX，setTimeout等等。

事件循环(event loop)
回调队列(callback queue)
堆栈帧(stack frame)：调用栈中的每一个入口

单线程(single thread)
 - 无须操心多线程环境的问题，如死锁
 - 单线程固有的限制

异步回调(asynchronous callbacks)

Inline caching

内存生命周期

1. 分配->使用->释放

## 垃圾回收器（GC/garbage collector）

### 引用（reference）
 - 显性引用（explicit reference）
 - 隐性引用（implicit reference）

### 引用计数机制（reference-counting garbage collection）

1. 存在循环引用的问题

### 标记和清除机制（mark-and-sweep）: 判断对象是否能从根节点递进的访问到

1. 根节点->字节点->清除未标记为active的内存空间

非确定性的（non-determinism）：无法预知垃圾回收机制运行的时间

造成内存泄漏的四种情况

1. 全局变量（global variables）
2. 计时器或回调
3. 闭包（Closures）
4. DOM引用（DOM references）

# Todo

[Inline caching](https://github.com/sq/JSIL/wiki/Optimizing-dynamic-JavaScript-with-inline-caches)

[Tracing garbage collection](https://en.wikipedia.org/wiki/Tracing_garbage_collection)
