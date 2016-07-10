Modules模块为node提供了将代码模块化的解决方案，让我们能够更好的组织代码，抽象代码，以便构建大型程序。

### 这个机制是如何实现的？
在一个模块内部，我们可以自由的像这样：`require("http")`包含其他模块、用`module.exports = {}`导出当前模块提供的各种接口、用`__dirname`显示当前文件所在目录的绝对路径等等。
但是，这是如何实现的呢？我们并没有定义这些变量啊，其实，node里面的每个模块，node都会将其包装一番：
```javascript
	(function (exports, require, module,  __dirname, __filename) {
	 // real module code.
	 });
```
这样做有两个好处：
 - 保持当前模块内部的全局变量为该模块的私有变量
 - 注入exports, require等这些变量

在第一次 `require(modulename)`一个模块之后，系统就将该模块缓存，当第二次再次 `require`这个模块时，就直接将缓存中的加载过去，这样免去了再次加载的开销。
在node中，有两种模块，一种是文件模块，也就是单个文件为一个模块，还有一种是类似于包的概念，将一个目录下面的所有文件打包为一个模块。对于文件模块，如果你不写文件类型后缀，系统会根据 `.js`、`.json`、`.node`的顺序为名字加上后缀，然后再在给定的目录下面寻找看是否能找到。对于目录模块，需要在该目录下面增加一个出口模块，在里面包括该目录下面所有要导出的文件，node规定了有两种办法可以找到该出口模块，一种是提供 package.json 文件，在里面用 `main` 字段指明出口模块，另外就是如果没有找到 package.json 文件，就默认依次查找 index.js, index.json, index.node文件。
除了这种分类方法，还可以将模块分为自己写的和非自己写的，如果是自己写的，在 `require(modulename)`的时候，必须传入包含模块相对于当前模块的相对地址，以表明这个模块使用自己写的，而不是内建的或者是其他的，如果是非自己写的，除了node本身提供的诸多模块以外，还可以通过 `npm install module` 的方式安装社区提供的模块，会安装到命令运行时候的当前目录当中的node_modules目录当中，如果要引入的模块既不是以 `/`, `../`, `./`这些开头，又不是内建模块，那就只能是安装的社区模块，这个时候，查找的路径是从当前目录开始，一直到根目录，依次查找node_modules目录，都没有找到的话，就抛出找不到当前模块错误。

####Notes:
 - 使用 `require.main`可以访问到整个程序的入口模块，类似于c语言当中的main函数
 - 最好将一个模块要导出的内容放在 `module.exports`上面，而不是 `exports` 上面
 - require加载模块的方式是同步加载的，这意味着不要提供比较深层级的模块，避免在查找的时候耗费太多时间。 