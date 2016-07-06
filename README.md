# node-js-api-explanation
###node.js是什么?
node.js提供这样一个平台，让你能够通过javascript, 去调用已经封装好了的系统API, 从而写出能够跑在系统上的程序，而且由于架构里面已经包含了对**window**, **linux**跨平台的实现，写出来的代码是能够跨平台的。
###目前已经实现的模块
####1, 核心模块
这部分是首先要学习的，也是最重要的部分，基本上，其他的模块或多或少的都是基于这些核心模块来实现的。
	1. Events
	2. Stream
	3. Errors
	4. Buffer
	5. Timers

####2, 全局模块
下面这些模块里面包含了暴露在全局范围内的一些对象，以及node.js的模块机制
	1. Modules
	2. Process
	3. Globals
	
####3, 网络
包含了对底层Tcp, Udp, TLS/SSL, HTTP, HTTPS的包装实现
	1. Net
	2. UDP/Datagram
	3. TLS/SSL
	4. HTTP
	5. HTTPS
	6. DNS

####4, 文件处理
	1. File System
	2. Path

####5, 操作系统
	1. OS

####6, 多进程
由于javascript代码的执行是单线程的，导致无法充分利用服务器多核资源，node采用的解决方案是 Master-Workers, 用以下模块就可实现
	1. Child Processes
	2. Cluster
