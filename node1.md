##HTTP服务器

supervisor可以帮助在开发过程中实现，修改后立即查看效果的功能，他会监视你对代码的改动，并自动重启node.js

###异步I/O与事件编程

线程在执行中如果遇到磁盘读写或网络通信（统称为 I/O 操作）

阻塞模式下，一个线程只能处理一项任务，要想提高吞吐量必须通过多线程。而非阻塞模式下，一个线程永远在执行计算操作，这个线程所使用的 CPU 核心利用率永远是 100%，I/O 以事件的方式通知

####事件

Node.js 所有的异步 I/O 操作在完成时都会发送一个事件到事件队列

####Node.js 的事件循环机制

Node.js 始终在事件循环中，程序入口就是事件循环第一个事件的回调函数。事件的回调函数在执行的过程中，可能会发出 I/O 请求或直接发射（emit）事件，执行完毕后再返回事件循环，事件循环会检查事件队列中有没有未处理的事件，直到程序结束。

事件循环会检查事件队列中有没有未处理的事件，如果有就去处理，处理完了，在返回事件循环检查有没有未处理的事件，知道程序结束

##模块和包

在浏览器 JavaScript 中，脚本模块的拆分和组合通常使用 HTML 的script 标签来实现。Node.js 提供了 require 函数来调用其他模块，而且模块都是基于文件的，机制十分简单。

###什么是模块

模块是 Node.js 应用程序的基本组成部分，文件和模块是一一对应的。换言之，一个Node.js 文件就是一个模块，这个文件可能是 JavaScript 代码、JSON 或者编译过的 C/C++ 扩展。

###创建及加载模块

在 Node.js 中，创建一个模块非常简单，因为一个文件就是一个模块

如何在其他文件中获取这个模块，Node.js 提供了 exports 和 require 两个对象，其中 exports 是模块公开的接口，require 用于从外部获取一个模块的接口，即所获取模块的 exports 对象。

创建一个 module.js 的文件，内容是：

```javascript
//module.js
var name;
exports.setName = function(thyName) {
name = thyName;
};
exports.sayHello = function() {
console.log('Hello ' + name);
};
```

在同一目录下创建 getmodule.js，内容是：

```javascript
//getmodule.js
var myModule = require('./module');
myModule.setName('BYVoid');
myModule.sayHello();
```

运行node getmodule.js，结果是：Hello BYVoid

在以上示例中，module.js 通过 exports 对象把 setName 和 sayHello 作为模块的访问接口，在 getmodule.js 中通过 require('./module') 加载这个模块，然后就可以直接访问 module.js 中 exports 对象的成员函数了。

上面这个例子有点类似于创建一个对象，但实际上和对象又有本质的区别，因为require 不会重复加载模块，也就是说无论调用多少次 require，获得的模块都是同一个。我们在 getmodule.js 的基础上稍作修改：

```javascript
//loadmodule.js
var hello1 = require('./module');
hello1.setName('BYVoid');
var hello2 = require('./module');
hello2.setName('BYVoid 2');
hello1.sayHello();
```

运行后发现输出结果是 Hello BYVoid 2，2，这是因为变量 hello1 和 hello2 指向的是同一个实例，因此 hello1.setName 的结果被 hello2.setName 覆盖，最终输出结果是由后者决定的。

####覆盖 exports

有时候我们只是想把一个对象封装到模块中，例如：

```javascript
//singleobject.js
function Hello() {
var name;
this.setName = function (thyName) {
name = thyName;
};
this.sayHello = function () {
console.log('Hello ' + name);
};
};
exports.Hello = Hello;
```

此时我们在其他文件中需要通过 require('./singleobject').Hello 来获取Hello 对象，这略显冗余，可以用下面方法稍微简化：

```javascript
//singleobject.js
function Hello() {
var name;
this.setName = function (thyName) {
name = thyName;
};
this.sayHello = function () {
console.log('Hello ' + name);
};
};
module.exports = Hello;
```

这样就可以直接获得这个对象了：

```javascript
//gethello.js
var Hello = require('./hello');
hello = new Hello();
hello.setName('BYVoid');
hello.sayHello();
```

###创建包

包是在模块基础上更深一步的抽象，Node.js 的包类似于 C/C++ 的函数库或者 Java/.Net的类库。它将某个独立的功能封装起来，用于发布、更新、依赖管理和版本控制。Node.js 根据 CommonJS 规范实现了包机制，开发了 npm来解决包的发布和获取需求。

Node.js 的包是一个目录，其中包含一个 JSON 格式的包说明文件 package.json。严格符合 CommonJS 规范的包应该具备以下特征：

+ package.json 必须在包的顶层目录下；

+ 二进制文件应该在 bin 目录下；

+ JavaScript 代码应该在 lib 目录下；

+ 文档应该在 doc 目录下；

+ 单元测试应该在 test 目录下。

Node.js 对包的要求并没有这么严格，只要顶层目录下有 package.json，并符合一些规范即可。当然为了提高兼容性，我们还是建议你在制作包的时候，严格遵守 CommonJS 规范。

####作为文件夹的模块

模块与文件是一一对应的。文件不仅可以是 JavaScript 代码或二进制代码，还可以是一个文件夹。最简单的包，就是一个作为文件夹的模块。下面我们来看一个例子，建立一个叫做 somepackage 的文件夹，在其中创建 index.js，内容如下：

```javascript
//somepackage/index.js
exports.hello = function() {
    console.log('Hello.');
};
```

然后在 somepackage 之外建立 getpackage.js，内容如下：

```javascript
//getpackage.js
var somePackage = require('./somepackage');
somePackage.hello();
```

运行 node getpackage.js，控制台将输出结果 Hello.。

我们使用这种方法可以把文件夹封装为一个模块，即所谓的包。包通常是一些模块的集合，在模块的基础上提供了更高层的抽象，相当于提供了一些固定接口的函数库。通过定制package.json，我们可以创建更复杂、更完善、更符合规范的包用于发布。

####package.json

Node.js 在调用某个包时，会首先检查包中 package.json 文件的 main 字段，将其作为包的接口模块，如果 package.json 或 main 字段不存在，会尝试寻找 index.js 或 index.node 作为包的接口。

package.json 是 CommonJS 规定的用来描述包的文件，完全符合规范的 package.json 文件应该含有以下字段:

+ name：包的名称，必须是唯一的，由小写英文字母、数字和下划线组成，不能包含空格。

+ description：包的简要说明。

+ version：符合语义化版本识别①规范的版本字符串。

+ keywords：关键字数组，通常用于搜索。

+ maintainers：维护者数组，每个元素要包含 name、email （可选）、web （可选）字段。

+ contributors：贡献者数组，格式与maintainers相同。包的作者应该是贡献者数组的第一个元素。

+ bugs：提交bug的地址，可以是网址或者电子邮件地址。

+ licenses：许可证数组，每个元素要包含 type （许可证的名称）和 url （链接到许可证文本的地址）字段。

+ repositories：仓库托管地址数组，每个元素要包含 type （仓库的类型，如 git ）、url （仓库的地址）和 path （相对于仓库的路径，可选）字段。

+ dependencies：包的依赖，一个关联数组，由包名称和版本号组成。

下面是一个完全符合 CommonJS 规范的 package.json 示例：

```json
{
	"name": "mypackage",
	"description": "Sample package for CommonJS. This package demonstrates the required
	elements of a CommonJS package.",
	"version": "0.7.0",
	"keywords": [
	"package",
	"example"
	],
	"maintainers": [
		{
			"name": "Bill Smith",
			"email": "bills@example.com",
		}
	],
	"contributors": [
		{
			"name": "BYVoid",
			"web": "http://www.byvoid.com/"
		}
	],
	"bugs": {
		"mail": "dev@example.com",
		"web": "http://www.example.com/bugs"
	},
	"licenses": [
		{
			"type": "GPLv2",
			"url": "http://www.example.org/licenses/gpl.html"
		}
	],
	"repositories": [
		{
			"type": "git",
			"url": "http://github.com/BYVoid/mypackage.git"
		}
	],
	"dependencies": {
		"webkit": "1.2",
		"ssl": {
			"gnutls": ["1.0", "2.0"],
			"openssl": "0.9.8"
		}
	}
}
```

###Node.js 包管理器

Node.js包管理器，即npm是 Node.js 官方提供的包管理工具①，它已经成了 Node.js 包的标准发布平台，用于 Node.js 包的发布、传播、依赖控制。npm 提供了命令行工具，使你可以方便地下载、安装、升级、删除包，也可以让你作为开发者发布并维护包。

####获取一个包

使用 npm 安装包的命令格式为：npm [install/i] [package_name]

####本地模式和全局模式

npm在默认情况下会从http://npmjs.org搜索或下载包，将包安装到当前目录的node_modules子目录下。

在使用 npm 安装包的时候，有两种模式：本地模式和全局模式。默认情况下我们使用 npm install命令就是采用本地模式，即把包安装到当前目录的 node_modules 子目录下。Node.js的 require 在加载模块时会尝试搜寻 node_modules 子目录，因此使用 npm 本地模式安装的包可以直接被引用。

npm 还有另一种不同的安装模式被成为全局模式，使用方法为：npm [install/i] -g [package_name]

为什么要使用全局模式呢？多数时候并不是因为许多程序都有可能用到它，为了减少多重副本而使用全局模式，而是因为本地模式不会注册 PATH 环境变量。

本地模式与全局模式

|模式|可通过 require 使用|注册PATH|
|:--|---|---|
|本地模式|是|否|
|全局模式|否|是|

当我们要把某个包作为工程运行时的一部分时，通过本地模式获取，如果要在命令行下使用，则使用全局模式安装。

####创建全局链接

52