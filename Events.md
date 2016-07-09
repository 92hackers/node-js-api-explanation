Node.js的一大特点就是它是基于事件驱动的异步I/O, 这个特点的基础就是Events这个模块了，fs, http里面的异步I/O类都继承自这个基类。

### 什么时候用这个比较合适?
本质上这相当于一个构建良好的信号触发机制，如果你想要在某件事请完成，或者是遇到问题的时候，通知其他地方已经完成，或者是寻求帮助，就可以触发一个信号，然后其他地方都一直不停的盯着这个信号就可以了。

### 基本用法
你先要创建自己的EventEmitter对象。
```javascript
	const EventEmitter = require("events");
	const util = require("util");
	function MyEmitter() {
		EventEmitter.call(this);
	}
	util.inherits(MyEmitter, EventEmitter);
	const myEmitter = new MyEmitter();
```
在这之后，你可以随意修改MyEmitter, 而不至于修改到EventEmitter本身。
myEmitter就是我们新建的一个EventEmitter实例：
 - `myEmitter.emit("eventName")`用来触发事件信号，就是告诉那些监听这个事件的对象，这个事情发生了，你们可以干活了
 - `myEmitter.on("eventName", callback)`用来监听事件，在收到事件信号之后，就开始干活，执行callback.
 - `myEmitter.once("eventName", callback)`如果是用on来监听事件的话，每次收到信号之后，都会开始干活，而这个方法，收到一次之后，也会干活，但是在这之后，就罢工了，不干了，从监听者队列中移除了。
 - `myEmitter.addListener('eventName', callback)`跟`on`一样。

 上面这几个方法，囊括了Events这个模块的核心功能：触发事件信号，监听事件信号，执行回调。
 在 events 内部，每一个类型的事件信号都有一个数组，用来存放所有的监听者，`addListener`, `once`, `on`就是用来增加, 而`removeAllListeners`, `removeListener`就是用来执行清除监听者的任务。
 同时，node对于单个事件信号能够绑定的监听者的数量做了限制，因为当一个信号触发的时候，所有监听该事件的callback都会同步的一个接一个的被执行，如果数量太多，cpu 就会耗费很多时间去执行那些回调，这样系统性能就会得到比较大的损失，所以，node默认的最多只能绑定10个监听者在同一个信号事件上面，当然，如果你的机器足够强大，这个数量限制可以通过：`myEmitter.setMaxListeners(maxCount)`来进行设置。

### Notes:
 node提供了一个一般性的原则，那就是最好要监听 `error`事件信号，因为当你的EventEmitter实例内部发生错误的时候，默认触发的就是`error`事件信号，如果没有发现监听者，这样的错误会导致程序的崩溃，所以，类似: `myEmitter.on("error", callback)`这样的代码是很重要的。
