## nodejs核心模块

nodejs核心模块它由一些精简而高效的库组成，为nodejs提供了基本的API

## 全局对象

Node.js中的全局对象是global，所有全局变量除了global本身之外都是global的对象的属性

### process

process是一个全局变量，即global对象的属性，他用于描述当前nodejs进程状态的对象，提供一个与操作系统的简单接口

process对象的一些常用成员方法：

#### process.argv

process.argv是命令行参数数组。第一个元素是node，第二个元素是脚本文件名，从第三个元素开始，每一个元素是一个运行参数

#### process.stdout

标准输出流，process.stdout.write()函数提供了更底层的接口

#### process.stdin

标准输入流，初始时是被暂停的，想要从标准输入读取数据，你必须恢复流，并手动编写流的时间响应函数

#### process.nextTick(callback)

功能是为事件循环设置一项任务，nodejs会在下次事件循环调响应时调用callback

nodejs适合I/O密集型的应用，而不是计算密集型的应用，因为nodejs进程只有一个线程，因此在任何时候都只有一个事件在执行，如果这个事件占用大量的CPU事件，执行事件循环中的下一个事件就需要等很久，因此Nodejs的一个编程原则就是尽量缩短每个事件的执行事件，process.nextTick(callback)提供了一个这样的工具，可以把复杂的工作拆散，变成一个比较小的事件

### console

console用于提供控制台标准输出，Node.js提供了console对象，用于向标准输出流或标准错误流输出字符

#### console.log()

向标准输出流打印字符并以换行符结束

#### console.error()

向标准错误流输出

#### console.trace()

向标准错误流输出当前的调用栈

## 常用工具util

util是一个Node.js核心模块，提供常用的函数的集合，用于弥补核心javascript的功能过于精简的不足

### util.inherits

util.inherits(constructor,superConstructor)是一个实现对象间原型继承的函数

```javascript
var util = require('util');
function Base() {
    this.name = 'base';
    this.base = 1991;
    this.sayHello = function() {
        console.log('Hello ' + this.name);
    };
}
Base.prototype.showName = function() {
    console.log(this.name);
};
function Sub() {
    this.name = 'sub';
}
util.inherits(Sub, Base);
var objBase = new Base();
objBase.showName();
objBase.sayHello();
console.log(objBase);
var objSub = new Sub();
objSub.showName();
//objSub.sayHello();
console.log(objSub);
```

注意， Sub 仅仅继承了 Base 在原型中定义的函数，而构造函数内部创造的 base 属性和 sayHello 函数都没有被 Sub 继承。同时，在原型中定义的属性不会被 console.log 作为对象的属性输出。

###util.inspect

util.inspect(object,[showHidden],[depth],[colors])是一个将任意对象转换为字符串的方法，通常用于调试和错误输出。它至少接受一个参数 object，即要转换的对象。

showHidden 是一个可选参数，如果值为 true，将会输出更多隐藏信息。

depth 表示最大递归的层数，如果对象很复杂，你可以指定层数以控制输出信息的多少。如果不指定depth，默认会递归2层，指定为 null 表示将不限递归层数完整遍历对象。

如果color 值为 true，输出格式将会以 ANSI 颜色编码，通常用于在终端显示更漂亮的效果。

除了以上我们介绍的几个函数之外， util还提供了util.isArray()、util.isRegExp()、util.isDate()、 util.isError() 四个类型测试工具，以及 util.format()、 util.debug() 等工具。

## 事件驱动 events

events 是 Node.js 最重要的模块，没有“之一”，原因是 Node.js 本身架构就是事件式的，而它提供了唯一的接口，所以堪称 Node.js 事件编程的基石。 events 模块不仅用于用户代码与 Node.js 下层事件循环的交互，还几乎被所有的模块依赖。

### 事件发射器

events 模块只提供了一个对象： events.EventEmitter。 

EventEmitter 的核心就是事件发射与事件监听器功能的封装。EventEmitter 的每个事件由一个事件名和若干个参数组成，事件名是一个字符串，通常表达一定的语义。对于每个事件， EventEmitter 支持若干个事件监听器。当事件发射时，注册到这个事件的事件监听器被依次调用，事件参数作为回调函数参数传递。

EventEmitter常用API

EventEmitter.on(event,listener)为指定事件注册一个监听器，接受一个字符串event和一个回调函数listener

EventEmitter.emit(event,[arg1],[arg2],[...])，发射event事件，传递若干可选参数到事件监听器的参数表

EventEmitter.once(event,listener)为指定事件注册一个单次监听器，即监听器最多只会触发一次，出发后立刻解除该监听器

EventEmitter.removeListener(event,listener)移除指定事件的某个监听器，listener必须是该事件已经注册过的监听器

EventEmitter.removeAllListeners([event])移除所有事件的所有监听器，如果指定event，则移除指定事件的所有监听器

### error 事件

EventEmitter 定义了一个特殊的事件 error，它包含了“错误”的语义，我们在遇到异常的时候通常会发射 error 事件。当 error 被发射时， EventEmitter 规定如果没有响应的监听器， Node.js 会把它当作异常，退出程序并打印调用栈。我们一般要为会发射 error事件的对象设置监听器，避免遇到错误后整个程序崩溃。

### 继承EventEmitter

大多数时候我们不会直接使用 EventEmitter，而是在对象中继承它。包括 fs、 net、http 在内的，只要是支持事件响应的核心模块都是 EventEmitter 的子类。

为什么要这样做呢？原因有两点。首先，具有某个实体功能的对象实现事件符合语义，事件的监听和发射应该是一个对象的方法。其次 JavaScript 的对象机制是基于原型的，支持部分多重继承，继承 EventEmitter 不会打乱对象原有的继承关系。

## 文件系统 fs

fs 模块是文件操作的封装，它提供了文件的读取、写入、更名、删除、遍历目录、链接等 POSIX 文件系统操作。与其他模块不同的是， fs 模块中所有的操作都提供了异步的和同步的两个版本，例如读取文件内容的函数有异步的 fs.readFile() 和同步的fs.readFileSync()。

### fs.readFile

	fs.readFile(filename,[encoding],[callback(err,data)])

它接受一个必选参数 filename，表示要读取的文件名。第二个参数 encoding是可选的，表示文件的字符编码。callback 是回调函数，用于接收文件的内容。如果不指定 encoding，则 callback 就是第二个参数。回调函数提供两个参数 err 和 data， err 表示有没有错误发生， data 是文件内容。如果指定了 encoding， data 是一个解析后的字符串，否则 data 将会是以 Buffer 形式表示的二进制数据。

Node.js 的异步编程接口习惯是以函数的最后一个参数为回调函数，通常一个函数只有一个回调函数。回调函数是实际参数中第一个是 err，其余的参数是其他返回的内容。如果没有发生错误， err 的值会是 null 或undefined。如果有错误发生， err 通常是 Error 对象的实例。

### fs.readFileSync

	fs.readFileSync(filename, [encoding])
	fs.readFile 同步的版本

它接受的参数和 fs.readFile 相同，而读取到的文件内容会以函数返回值的形式返回。如果有错误发生， fs 将会抛出异常，你需要使用 try 和 catch 捕捉并处理异常。

### fs.open

	fs.open(path, flags, [mode], [callback(err, fd)])
	POSIX open 函数的封装，与 C 语言标准库中的 fopen 函数类似
	接受两个必选参数， path 为文件的路径，flags 可以是以下值。

	r:以读取模式打开文件
	r+:以读写模式打开文件
	w:以写入模式打开文件，如果不存在则创建文件
	w+:以读写模式打开文件，如果不存在则创建文件
	a:以追加模式打开问价，如果文件不存在则创建
	a+:以读取追加模式打开文件，如果文件不存在则创建

mode 参数用于创建文件时给文件指定权限，默认是 0666。回调函数将会传递一个文件描述符 fd。

### fs.read

	fs.read(fd, buffer, offset, length, position, [callback(err, bytesRead,buffer)])

	POSIX read 函数的封装，相比 fs.readFile 提供了更底层的接口。fs.read的功能是从指定的文件描述符 fd 中读取数据并写入 buffer 指向的缓冲区对象。 

	offset 是buffer 的写入偏移量。
	length 是要从文件中读取的字节数。
	position 是文件读取的起始位置，如果 position 的值为。 null，则会从当前文件指针的位置读取。
	回调函数传递bytesRead 和 buffer，分别表示读取的字节数和缓冲区对象。

```javascript
var fs = require('fs');
fs.open('test.txt', 'r', function(err, fd) {
    if (err) {
        console.error(err);
        return;
    }
    var buf = new Buffer(8);
    fs.read(fd, buf, 0, 8, null, function(err, bytesRead, buffer) {
        if (err) {
            console.error(err);
            return;
        }
        console.log('bytesRead: ' + bytesRead);
        console.log(buffer);
    })
});
```

一般来说，除非必要，否则不要使用这种方式读取文件，因为它要求你手动管理缓冲区和文件指针，尤其是在你不知道文件大小的时候，这将会是一件很麻烦的事情。

### fs 模块函数表

|功能|异步函数|同步函数|
|:--|---|---|
|打开文件|fs.open(path,flags, [mode], [callback(err,fd)])|fs.openSync(path, flags, [mode])|
|关闭文件|fs.close(fd, [callback(err)])|fs.closeSync(fd)|
|读取文件（文件描述符）|fs.read(fd,buffer,offset,length,position,[callback(err, bytesRead, buffer)])|fs.readSync(fd, buffer, offset,
length, position)|
|写入文件（文件描述符）|fs.write(fd,buffer,offset,length,position,
[callback(err, bytesWritten, buffer)])|fs.writeSync(fd, buffer, offset,
length, position)|
|读取文件内容|fs.readFile(filename,[encoding],[callback(err, data)])|fs.readFileSync(filename,[encoding])|
|写入文件内容|fs.writeFile(filename, data,[encoding],[callback(err)])|fs.writeFileSync(filename, data,[encoding])|
|删除文件|fs.unlink(path, [callback(err)])|fs.unlinkSync(path)|
|创建目录|fs.mkdir(path, [mode], [callback(err)])|fs.mkdirSync(path, [mode])|
|删除目录|fs.rmdir(path, [callback(err)])|fs.rmdirSync(path)|
|读取目录|fs.readdir(path, [callback(err, files)])|fs.readdirSync(path)|
|获取真实路径|fs.realpath(path, [callback(err,resolvedPath)])|fs.realpathSync(path)|
|更名|fs.rename(path1, path2, [callback(err)])|fs.renameSync(path1, path2)|
|截断|fs.truncate(fd, len, [callback(err)]) |fs.truncateSync(fd, len)|
|更改所有权|fs.chown(path, uid, gid, [callback(err)])|fs.chownSync(path, uid, gid)|
|更改所有权（文件描述符）|fs.fchown(fd, uid, gid, [callback(err)])|fs.fchownSync(fd, uid, gid)|
|更改所有权（不解析符号链接）|fs.lchown(path, uid, gid, [callback(err)])|fs.lchownSync(path, uid, gid)|
|更改权限|fs.chmod(path, mode, [callback(err)])|fs.chmodSync(path, mode)|
|更改权限（文件描述符）|fs.fchmod(fd, mode, [callback(err)])|fs.fchmodSync(fd, mode)|
|更改权限（不解析符号链接）|fs.lchmod(path, mode, [callback(err)])|fs.lchmodSync(path, mode)|
|获取文件信息|fs.stat(path, [callback(err, stats)])|fs.statSync(path)|
|获取文件信息 （文件描述符）|fs.fstat(fd, [callback(err, stats)])|fs.fstatSync(fd)|
|获取文件信息 （不解析符号链接）|fs.lstat(path, [callback(err, stats)])|fs.lstatSync(path)|
|创建硬链接|fs.link(srcpath, dstpath, [callback(err)])|fs.linkSync(srcpath, dstpath)|
|创建符号链接|fs.symlink(linkdata, path, [type],[callback(err)])|fs.symlinkSync(linkdata, path,[type])|
|读取链接|fs.readlink(path, [callback(err,linkString)])|fs.readlinkSync(path)|
|修改文件时间戳|fs.utimes(path, atime, mtime, [callback(err)])|fs.utimesSync(path, atime, mtime)|
|修改文件时间戳（文件描述符）|fs.futimes(fd, atime, mtime, [callback(err)])|fs.futimesSync(fd, atime, mtime)|
|同步磁盘缓存|fs.fsync(fd, [callback(err)])|fs.fsyncSync(fd)|