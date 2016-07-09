在node中，buffer是专门用来操作二进制数据的，它是一个类似数组的对象，里面存放的是16进制形式的字节序列。

### 特点
 - 一个buffer实例在初始化之后，它的大小就已经确定下来了，不能再更改。
 - buffer不占用v8的堆内存，而是在c++层面进行内存的申请, 所以这也算是一个突破v8内存限制的办法，所以，在文件模块操作大型文件的时候，传输的就是buffer类型的数据, stream默认的传输类型也是buffer类型。
 - Buffer类是暴露在全局的，所以你不需要require就可以直接使用。

### 应用场景
 - 处理图片
 - 接受上传文件
 - 处理网络协议
 - 处理音频, 视频流

### buffer和字符串的相互转换
 `const buf = new Buffer("hello, world")`, 只需要字符串作为参数传递给Buffer的构造函数，就会返回buffer类型的数据。`buf.toString(encoding)`就能够根据**encoding**将buf编码为相应的字符串。
 最常用的有：
  - 'utf8' :  普通的字符串
  - 'base64': base64编码过后的字符串
  - 'hex': 16进制字符的字符串，就是将buf里面的序列节点连接起来作为字符串。

### 关于Buffer更多
 - new Buffer(size) : 分配指定大小的buffer空间，之后这个空间的大小是不能够更改的，之后，你需要手动调用 buf.fill(0) 来将空间里面的数据初始化。
 - Buffer.byteLength(str): 返回 str 所占的字节数量，字符串本身的length计算的是字符的个数，比如：由于utf8编码的中文占三个字节，所以Buffer.byteLength("中国人")计算出来是**9**。
 - Buffer.concat(list, totalLength): 将list数组里面的小的buf组合起来，返回一个完整的buf, 在防止乱码，拼接buffer的时候尤其有用，
 - buf.keys() : 返回 buffer的index.
 - buf.values(): 遍历 一个buffer的值。
