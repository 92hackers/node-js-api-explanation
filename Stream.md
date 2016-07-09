在Node中的某些场景下, 我们会需要I/O像流水一样，从input端流向output那端，比如：由于v8内存大小的限制，传输几个G的视频不能直接用`fs.readFile`, `fs.writeFile`, 又或者，在服务器向客户端回复数据的过程中，由于数据是动态生成的，所以`Content-Length`在这个时候是不确定的，所以可以将数据编码为`Transfer-Encoding: chunked`, 分段传输数据，Node对于此类场景提出的解决方案就是:将input端, output端都持续打开，然后将数据一小块，一小块的移动到内存，然后写入output, 这样就可以将大型, 未知长度的文件分解，分段传输。stream模块就是用来做这件事的。

stream总共分为三种，readable stream，writable stream，duplex stream，所有的stream 都是`EventEmitter`的实例，预设了很多不同时间点，不同功能的事件信号，然后提供了很多触发这些事件信号的方法，用来操作stream对象。

前面说到，stream是一小段，一小段的将数据拷贝到内存中，然后写入可写流当中的，那这一小段是怎么设置的呢？这个是由一个参数: `highWaterMark`来确定的，默认值为: 可写流的默认为 16kb,  可读流的默认为 64kb, 你也可以在stream初始化的时候将这个参数传入自定义的值：
```javascript
	const readableStream = fs.createReadStream("./test.md", { highWaterMark: 1024 })
```
注意，这里传入的值**1024**的单位是byte。

node里面很多类都是继承自stream类型的，比如：
 - http request, response
 - fs read, write stream
 - zlib streams
 - tcp sockets

实际应用的时候，就是应用在这些类上面，当然，你也可以自己创建一个类，然后继承stream。 

### readable stream
由于物理世界当中的读取，写入之间存在速度差，读取的速度都会快于写入，最典型的比如：硬盘，所以，为了读取和写入之间能够平滑，稳定，readable stream 有两种模式：流动模式，静止模式，在初始化之后，就处于静止模式，有以下方法能够转变为流动模式，开始读取数据：
 - readableStream.on("data", callback(chunk))
 - readableStream.resume() 
 - readableStream.pipe()

有以下方法能够转回静止模式：
 - 去掉所有`data`事件的监听者
 - stream.pause()
 - stream.unpipe()

从上面可以看出，基本上都是一一对应的，同时也告诉我们，有哪些方法可以从一个readable stream当中读取数据。

#### Q: 哪些情况下需要转回静止模式？
A: 前面我们提到，在读取数据的时候，是先读入缓存，一次读取的大小默认是 64kb, 而写入的时候，也是先写入缓存，默认是 16kb, 这个时候问题就来了，如果写入流的 16kb 占满了，还在写入，这个时候缓存就不会再继续接受读取来的数据了，所以这个时候需要暂停读取，将readableStream转变为静止模式，等待 writableStream 的缓存清空之后，执行 readableStram.resume() 就可以重新读取数据了。

#### Q: 如果是 readableStream.pipe() 打开的读取流，用 readableStream.pause() 可以暂停吗？
A: 官方文档写的是不能保证，经过我自己测试，无法暂停，稍后会加上测试代码，所以，最好的读取原则是：按照上面所列出的转换模式的三条方法，一一对应的操作，比如：你用了 `readableStream.pipe()`来开始, 就用`readableStream.unpipe()`来暂停。

#### events:
 - close: 当 stream 关闭的时候，触发，可以手动用 stream.close() 来关闭。
 - data:  开始一段一段的读取数据，
 - end:  当数据读取完毕的时候触发.
 - error: 读取过程出错触发.

#### methods:
 - setEncoding: stream 默认读取的数据是`Buffer`类型的，如果你想要输出字符串，需要： `readableStream.setEncoding("utf-8")`
 - pause:  手动暂停读取，转变模式为静止模式
 - resume: 手动开启读取，转变模式为流动模式
 - pipe(destination): 将读取流用管道流到一个 writableStream.
 - unpipe(destination): 关闭该管道。

### writableStream
有了输出流，肯定要有一个输入流，在整个交互的过程当中，占有主动权的是输出流，输入流是被动的接受数据，这也体现在事件信号和方法的设计上面。

#### events:
 - drain: 当 writableStream的缓冲区占满了的时候，此时需要暂停readableStream的读取，等待writableStream将缓冲区清空，清空之后，将会触发这个事件，我们可以在这个事件信号的回调里面重新打开readableStream.
 - finish: 所有数据写入完毕的时候触发
 - pipe : 当有readableStream跟当前writableStream建立管道的时候触发。

#### methods:
 - end: 当你确定数据写入完成的时候，手动调用这个方法，会触发 `finish` 事件，一般情况下是将这个方法的调用放置在 readableStream监听的 `end` 事件当中。因为只有当数据读取完毕之后，你才能够确保写入完成, 不会再有数据写入。
 - write: 写入数据到 writableStream　当中，它的返回值很有意思，当写入的数据没有达到缓存的最大值的时候，返回true, 达到的时候，表示缓冲区已经占满，会返回false,所以可以通过它的返回值来得知缓冲区是否占满，是否需要暂停 readableStream 
 - cork: 缓存所有写入的数据在内存当中，不直接写入硬盘文件
 - uncork: 将所有缓存的数据一次性写入硬盘文件

### 普遍的使用方法：
默认传输的是buffer，如果你想传输字符串，可以 rs.setEncoding("utf8");
####1, 利用管道
```javascript
	var rs = fs.createReadStream("test.md");
	var ws = fs.createWriteStream("another.md");

	rs.pipe(ws);
	ws.on("finish", () => {
			// somethind to do.
	});
	rs.on("error", callback);
	ws.on("error", callback);
```
####2, 一段接一段的写入
```javascript
	var rs = fs.createReadStream("test.md");
	var ws = fs.createWriteStream("another.md");
	rs.on("data", (chunk) => {
		if (!ws.write(chunk)) {
			rs.pause();
		}
	});
	rs.on("end", () => {
		ws.end();
	});
	ws.on("drain", () => {
		rs.resume();
	});
	rs.on("error", callback);
	ws.on("error", callback);
```
