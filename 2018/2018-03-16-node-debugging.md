# Node 调试入门教程

JavaScript 程序越来越复杂，调试工具的重要性日益凸显。客户端脚本有浏览器，Node 脚本怎么调试呢？

2016年，Node 决定将 Chrome 浏览器的“开发者工具”作为官方的调试工具，使得 Node 脚本也可以使用图形界面调试，这大大方便了开发者。

本文介绍如何使用 Node 脚本的调试工具。

## 一、示例程序

为了方便演示，下面是一个示例脚本。首先，新建一个工作目录，并进入该目录。

```bash
$ mkdir debug-demo
$ cd debug-demo
```

然后，生成`package.json`文件，并安装 [Koa](http://www.ruanyifeng.com/blog/2017/08/koa.html) 框架和 koa-route 模块。

```bash
$ npm init -y
$ npm install --save koa koa-route
```

接着，新建一个脚本`app.js`，并写入下面的内容。

```javascript
// app.js
const Koa = require('koa');
const router = require('koa-route');

const app = new Koa();

const main = ctx => {
  ctx.response.body = 'Hello World';
};

const welcome = (ctx, name) => {
  ctx.response.body = 'Hello ' + name;
};

app.use(router.get('/', main));
app.use(router.get('/:name', welcome));

app.listen(3000);
console.log('listening on port 3000');
```

上面代码是一个简单的 Web 应用，指定了两个路由，访问后会显示一行欢迎信息。如果想详细了解代码的详细含义，可以参考这篇 [Koa 教程](http://www.ruanyifeng.com/blog/2017/08/koa.html)。


## 二、启动开发者工具

第一步，运行上面的脚本。

```bash
$ node --inspect app.js

Debugger listening on ws://127.0.0.1:9229/70643120-f8a1-4012-aaae-bcc388de4ca0
For help see https://nodejs.org/en/docs/inspector
listening on port 3000
```

注意，上面命令包含`--inspect`参数，这是启动调试模式必需的。

访问 http://127.0.0.1//3000，就可以看到 Hello World 了。

这时有两种打开调试工具的方法。第一种是在 Chrome 浏览器的地址栏，键入 `chrome://inspect`或者`about:inspect`，回车后就可以下面的页面。

在 Target 部分，按下 inspect 链接，就能进入调试工具了。

第二种进入调试工具的方法，是在 http://127.0.0.1//3000 的窗口打开“开发者工具”，顶部左上角有一个 Node 的绿色标志，点击就可以进入。

## 三、调试工具窗口

调试工具其实就是“开发者工具”的定制版，只不过省去了那些对服务器脚本没用的部分。

主要有四个面板。

- Console：控制台
- Memory：内存
- Profiler：性能
- Sources：源码

这四个面板里面，Memory 、Profiler 和 Console 的用法与调试浏览器脚本差不多，这里就不多介绍了。源码面板主要用来查看源码和设置断点。

## 四、设置断点

进入 Sources 面板，找到正在运行的脚本`app.js`。

在第11行（也就是下面这一行）的行号上点一下，就设置了一个断点。

```javascript
ctx.response.body = 'Hello ' + name;
```

这时，浏览器访问 http://127.0.0.1:3000/alice ，页面会显示正在等待服务器返回。

切换到调试工具，可以看到 Node 主线程处于暂停（paused）阶段。

进入 Console 面板，输入 name，会返回 alice。这表明我们正处在断点处的上下文（context）。

再切回 Sources 面板，右侧可以看到 Watch、Call Stack、Scope、Breakpoints 等折叠项。打开 Scope 折叠项，可以看到 Local 作用域和 Global 作用域里面的所有变量。

Local 作用域里面，变量`name`的值是`alice`，双击进入编辑状态，把它改成`bob`。

然后，点击顶部工具栏的继续运行按钮。

页面上就可以看到 Hello bob 了。

命令行下，按下 ctrl + c，终止运行`app.js`。

## 五、调试非服务脚本

上面的例子是 Web 服务脚本，会一直在后台运行。但是，大部分脚本不是运行服务，而是处理某个任务，运行完就会终止。

这时，你可能根本没有时间打开调试工具。等你打开了，脚本早就运行完退出了。这种脚本应该怎么调试呢？

请看上面的命令。

```bash
$ node --inspect=9229 -e "setTimeout(function() { console.log('yes'); }, 30000)"
```

上面代码中，`--inspect=9229`指定调试端口为 9229，这是调试工具默认的通信端口。`-e`参数指定一个字符串，作为代码运行。

访问`chrome://inspect`，就可以进入调试工具，调试这段代码了。

上面的代码要运行 30 秒才会结束，对于那些运行时间较短的脚本，可能根本来不及打开调试工具。这时就要使用下面的方法。

```bash
$ node --inspect-brk=9229 app.js
```

上面代码中，`--inspect-brk`指定在第一行就设置断点。

## 六、忘了 --inspect 怎么办？

打开调试工具，需要启动 Node 脚本时就加上`--inspect`参数，但是如果忘了这个参数，还能不能调试呢？

回答是可以的。首先，正常启动脚本。

```bash
$ node app.js
```

然后，在另一个命令行窗口，查找上面脚本的进程号。

```bash
$ ps ax | grep app.js 

30464 pts/11   Sl+    0:00 node app.js
30541 pts/12   S+     0:00 grep app.js
```

上面命令中，`app.js`的进程号是`30464`。

接着，运行下面的命令。

```bash
$ node -e 'process._debugProcess(30464)'
```

上面命令会建立进程 30464 与调试工具的连接，然后就可以打开调试工具了。

除了上面的方法，还有一种方法，就是向脚本进程发送 SIGUSR1 信号，也可以建立调试连接。

```bash
$ kill -SIGUSR1 67827
```

## 参考链接

- [Debugging Node.js with Google Chrome](https://medium.com/the-node-js-collection/debugging-node-js-with-google-chrome-4965b5f910f4), by Jacopo Daeli
- [Debugging Node.js with Chrome DevTools](https://medium.com/@paul_irish/debugging-node-js-nightlies-with-chrome-devtools-7c4a1b95ae27), by Paul Irish
- [Last minute node debugging](https://remysharp.com/2018/03/03/last-minute-node-debugging), by Remy Sharp

（完）