用 Node 和 Express 开发微博应用
===============================

这是对我用 node 开发第一个微博应用时碰到的问题的总结。
介绍用 node 开发一个微博应用需要掌握的技能，开发过程中的难点，以及可能遇到的陷阱。

主要分成以下几块：
- node 的安装和 npm 的使用
- 用 express 搭建一个基础的网页服务器
- 多个 web 服务程序并存
- jade 和 less
- 获得 App Key 和 App Secret
- 用户授权
- 获得一个长期有效的 token 让服务器能从微博获取数据
- node 中的 http 请求模块
- 从微博获取数据

内容深度均以满足开发一个简单的微博应用的需求为准。

Node 的安装和 Npm 简介
----------------------
我们的第一步，是安装 node！
尽管很多介绍 node 的文章或者书籍已经包含了相关内容，这里还是要简单提一下，并且指出一个和微博应用开发相关的编译细节。

### Node 的安装

目前，大多数主流的，面向桌面用户的 Linux 发行版，已经可以通过各自的包管理系统安装 node 了。

以我使用的 Arch 为例，通过以下命令就能完成安装：

    # pacman -S nodejs

其他发行版，诸如 Fedora 和 Ubuntu 可以通过 yum 和 apt 命令实现。

不过，如果你希望使用最新版本，或者想在 CentOS 或 Debian 等官方源中尚未提供 node 的发行版中安装，就需要自行下载源代码进行编译了。

    $ wget http://nodejs.org/dist/v0.6.18/node-v0.6.18.tar.gz               下载最新版本的 node
    $ tar -zxf node-v0.6.18.tar.gz                                          解压
    $ cd node-v0.6.18                                                       切换工作路径

接下来就是安装了，我们以管理员身份进行（普通用户也是可以的）
    
    # ./configure

configure 脚本会对编译环境进行检查，在这一步，你很可能会得到如下的结果：

    # ./configure
    Checking for program g++ or c++          : /usr/bin/g++ 
    Checking for program cpp                 : /usr/bin/cpp 
    Checking for program ar                  : /usr/bin/ar 
    Checking for program ranlib              : /usr/bin/ranlib 
    Checking for g++                         : ok  
    Checking for program gcc or cc           : /usr/bin/gcc 
    Checking for gcc                         : ok  
    Checking for library dl                  : yes 
    Checking for openssl                     : not found 
    Checking for function SSL_library_init   : not found 
    Checking for header openssl/crypto.h     : not found 
    node-v0.6.18/wscript:386: error: Could not autodetect OpenSSL support. Make sure OpenSSL development packages are installed. Use configure --without-ssl to disable this message.

如果遇到这个错误，说明你的系统目前不能编译 OpenSSL 相关的功能。
你可以选择先安装 OpenSSL 的开发包或者用 `./configure --without-ssl` 编译不支持 ssl 功能的 node。
不支持 ssl，简单的说就是不支持 https 协议，考虑到微博的鉴权接口必须通过 https 进行，我们只能选择先安装 OpenSSL 的开发包。

在 CentOS 系统中，对应的包叫做 openssl-devel：

    # yum install openssl-devel

这样我们就能继续 node 的安装了：

    # ./configure
    # make && make install

到这里，node 的安装已经完成，我们可以简单验证一下：

    # node -v
    v0.6.18

### Npm 简介

简单的说，npm 是 node 的包管理器，在安装 node 的同时就已经安装了。它是一个类似 yum 或 apt 的工具，就连命令的格式也很相似，`npm install 模块名` 用来安装，`npm remove 模块名` 用来移除。
一般情况下，执行 `npm install 模块名` 会在当前目录下建立一个名为 node\_modules 的文件夹，并把你指定的模块安装到这个目录下。
当然，npm 的功能不止于此，但目前，我们知道这些就够了。

用 Express 搭建一个基础的网页服务器
-----------------------------------
不错，接下来就是介绍 express 了。
我知道学习一样东西应该循序渐进，先了解基本的原理，再来讲这些第三方的模块或库，但本文确实不打算介绍 hello world 和那些用于演示 node 基本功能或原理的示例。

实际上，我们将抛开 node 的原生模块，直接使用 express 来完成我们的第一步。也就是说，即使你没有看过其他介绍 node 的文章，也能顺着这里的路子做下去，并架设起一个网页服务器。
当然，我绝对支持那些认为要好好掌握 node 的基本原理和原生模块的看法。

### 安装 Express

好了，express 就是我们要安装的第一个模块，终于到了 npm 出马的时刻。
不过，express 是一个特殊的模块，除了和大多数模块一样，可以用 `npm install express` 安装到当前目录的 node\_models 下，作为一个模块被 node 脚本调用外，还可以安装为一个 shell 命令，也就是说，可以像 `ls`，`cd` 一样，直接在命令行中执行。由于安装 `express` 命令可以在后面帮我们省下很多功夫，所以我们以管理员身份用以下命令进行安装：
    
    # npm install express -g

命令中的 -g 表示全局安装，有这个标签时，模块不会被安装在当前目录的 node\_modules 下，而是被安装在系统级的 node\_modules 下，而对于 express 这样的模块，除了安装位置的变化外，还会安装相应的 shell 命令。一般，我们会在安装报告中看到这么一行：

    /usr/local/bin/express -> /usr/local/lib/node_modules/express/bin/express

表明已经通过软链的方式建立了一个新的 shell 命令 express，顺便提一下，上面那个 /usr/local/lib/node\_modules 就是你的系统级的 node\_modules，全局安装的模块都会被安装到这个目录下。

安装报告的最后通常是这个形式：

    express@2.5.9 /usr/local/lib/node_modules/express 
    ├── qs@0.4.2
    ├── mime@1.2.4
    ├── mkdirp@0.3.0
    └── connect@1.8.7

表明你已经安装了 `express`，版本是 2.5.9， 安装位置在 `/usr/local/lib/node_modules/express`，下面列出的 `qs` `mime` `mkdirp` `connect` 则是 `express` 所依赖的模块。

### 用 Express 初始化我们的项目

接下来，我们就要用刚刚安装的 `express` 命令来初始化这个项目了，我们先来看看 `express` 命令是如何使用的：

    $ express -h

      Usage: express [options] [path]

      Options:
        -s, --sessions           add session support
        -t, --template <engine>  add template <engine> support (jade|ejs). default=jade
        -c, --css <engine>       add stylesheet <engine> support (stylus). default=plain css
        -v, --version            output framework version
        -h, --help               output help information  

`express` 可以根据用户设置的参数来初始化一个项目，现在，看看我们的需求吧：
- 我们的应用需要用户登录，所以 session 支持是需要的；
- 模板引擎，我选择了 jade，由于这是默认选项，我们放弃使用 -t 标签；
- css 引擎，我想使用 less，但可惜的是当前版本已经不再提供这个选项，我会在后面手动添加对 less 的支持，所以 -c 标签也不需要；
- 最后，我们需要一个工作目录，就用 ~/weibo 吧

结合以上需求，最终的命令就是：
    
    $ express -s ~/weibo

      create : /home/bnlt/weibo
      create : /home/bnlt/weibo/package.json
      create : /home/bnlt/weibo/app.js
      create : /home/bnlt/weibo/public
      create : /home/bnlt/weibo/public/javascripts
      create : /home/bnlt/weibo/public/images
      create : /home/bnlt/weibo/public/stylesheets
      create : /home/bnlt/weibo/public/stylesheets/style.css
      create : /home/bnlt/weibo/routes
      create : /home/bnlt/weibo/routes/index.js
      create : /home/bnlt/weibo/views
      create : /home/bnlt/weibo/views/layout.jade
      create : /home/bnlt/weibo/views/index.jade

      dont forget to install dependencies:
      $ cd /home/bnlt/weibo && npm install

我们简单看一下安装结果，`express` 首先建立了 /home/bnlt/weibo 目录，就是我们指定的 ~/weibo，bnlt 是我在 Arch 系统中使用的用户名。
接着是 package.json 文件，这是 node 用来存放项目信息的文件，我们稍后就会用到，到时再来介绍。
然后是 app.js，这是应用的主文件，我们可以通过 `node app.js` 来启动这个应用。
之后是 public 文件夹，以及下面的 javascript，images，stylesheets，基本上都是用来存放静态资源的。
再跟着是 routes，用来存放路由规则。
最后是 views，里面存放的是模板，由于我们选择了 jade 作为模板引擎，所以里面有两个默认的文件 layout.jade 和 index.jade ，我们会在后面专门讲解 jade 相关的内容。

`express` 命令除了提示我们创建了这些文件外，还不忘提醒我们安装依赖的模块。我们注意到，这个命令最后执行了 `npm install`，但没有指定安装什么模块，这也是 npm 的一个特殊用法：当直接执行 `npm install` 时，npm 会在当前目录下寻找我们之前提到过的 package.json 文件，根据其中的配置来安装依赖的模块。

那么让我们来看看 package.json 里面都有些什么：
    
    $ cat ~/weibo/package.json
    {
        "name": "application-name"
      , "version": "0.0.1"
      , "private": true
      , "dependencies": {
          "express": "2.5.8"
        , "jade": ">= 0.0.1"
      }
    }

其中 name 是项目的名称；version 是版本号；private 是用来防止你不小心把私有模块发布出去的；而 dependencies 就是对其他模块的依赖情况，`npm install` 正是根据这里的内容来决定要安装哪些模块，从这里看到，我们的项目需要 express 2.5.8 以及版本高于 0.0.1 的 jade，细心的朋友可能已经发现了，我们安装的 express 版本是 2.5.9，这里需要的则是 2.5.8。

让我们执行一下命令看看结果吧：

    $ cd ~/weibo && npm install

      npm http GET https://registry.npmjs.org/express/2.5.8
      npm http GET https://registry.npmjs.org/jade
      npm http 304 https://registry.npmjs.org/express/2.5.8
      npm http 304 https://registry.npmjs.org/jade
      npm http GET https://registry.npmjs.org/qs
      npm http GET https://registry.npmjs.org/connect
      npm http GET https://registry.npmjs.org/mime/1.2.4
      npm http GET https://registry.npmjs.org/mkdirp/0.3.0
      npm http GET https://registry.npmjs.org/commander/0.5.2
      npm http 304 https://registry.npmjs.org/qs
      npm http 304 https://registry.npmjs.org/connect
      npm http 304 https://registry.npmjs.org/mime/1.2.4
      npm http 304 https://registry.npmjs.org/mkdirp/0.3.0
      npm http 304 https://registry.npmjs.org/commander/0.5.2
      npm http GET https://registry.npmjs.org/formidable
      npm http 304 https://registry.npmjs.org/formidable
      jade@0.26.0 ./node_modules/jade
      ├── commander@0.5.2
      └── mkdirp@0.3.0
  
      express@2.5.8 ./node_modules/express
      ├── qs@0.4.2
      ├── mime@1.2.4
      ├── mkdirp@0.3.0
      └── connect@1.8.7 (formidable@1.0.9)

发现了么，尽管我们的系统中已经安装了更新版本的 express 2.5.9，npm 还是按照 package.json 的意思为当前项目安装了 2.5.8。
至于为什么用 express 2.5.9 初始化的项目想依赖 express 2.5.8，并不是我们现在要关心的问题（其实 2 天前已经有人报告了这个 issue），重要的是，我们要知道 npm 是按照 package.json 来决定安装哪个版本的模块的。

到目前为止，我们的 node 应用已经可以运行了：
    
    $ node app.js
    Express server listening on port 3000 in development mode

打开浏览器，访问 http://localhost:3000 就可以看到 express 的默认页面了：

![express.png](https://github.com/surmind/bookA/blob/master/images/Express.png?raw=true)

监听端口以及多 Web 服务程序并存
-------------------------------

这一节，我们会讨论如何设置应用监听的端口，以及如何通过设置反向代理来和其他 Web 服务器共存。

### 监听端口

你应该已经注意到，我们之前访问的地址是 http://localhost:3000，也就是说，用 express 初始化的项目，默认使用了 3000 端口。
这个设置可以在 app.js 文件中修改，我们看到， app.js 的最后三行是这样的：

    app.listen(3000, function(){
      console.log("Express server listening on port %d in %s mode", app.address().port, app.settings.env);
    });

这里通过 listen 方法开始监听请求，第一参数 3000 就是指定的监听端口。
紧跟其后的匿名函数会在成功开启监听后回调，我们之前执行 `node app.js` 时看到的提示信息正是通过这个函数输出的，但这里不对它做更深入的分析。

通过修改 listen 函数的第一个参数，我们就能改变应用监听的端口，事实上除了用数字表示的端口外，这个参数也可以设为一个指向系统中的 unix domain socket 文件的字符串，通过 socket 通讯。

好了，那让我们把这个端口改成 80 吧，这样通过浏览器访问时就不用再指定端口了，你和你的用户都会很满意。改完之后，我们再次执行：

    $ node app.js

    events.js:48
            throw arguments[1]; // Unhandled 'error' event
                           ^
    Error: listen EACCES
        at errnoException (net.js:670:11)
        at Array.0 (net.js:756:28)
        at EventEmitter._tickCallback (node.js:190:38)

哦，看样子我们遇到了一个错误：listen EACCES ，这是由于我们在以普通用户的身份执行 node ，而在 linux 下，普通用户是无权占用 0 - 1023 的，也就是通常所说的周知端口。

好吧，让我们试试用管理员权限运行，你成功了吗？也许吧。
但是，对于部分用户，仍有可能遇到错误：

    # node app.js

    events.js:48
            throw arguments[1]; // Unhandled 'error' event
                       ^
    Error: listen EADDRINUSE
        at errnoException (net.js:670:11)
        at Array.0 (net.js:771:26)
        at EventEmitter._tickCallback (node.js:190:38)

这里的 listen EADDRINUSE 表示 80 端口已经被占用了。
一般情况下，这是由于已经开启了其他 Web 服务程序引起的，可以看看你的服务器上是不是启动了 apache nginx lighttpd 等其他 Web 服务程序。
还有另一种比较常见的情况就是你重复执行了 `node app.js`，比如你同时打开了两个终端，然后各自去启动了相同的应用，后启动的那个就会报告上面这个错误。

针对 80 端口被占用的情况，如果是在你自己开发用的机器上，你可以先关闭那个占用端口的软件。
但如果这是一台运行中的生产服，并且你已经通过那个软件部署了公开的站点，就要按照下面的方法来处理了。

### 多 Web 服务器并存

我们刚刚提到了 80 端口被其他软件占用的情况，其实这是很常见的，比如我就和一位朋友合租了一台 vps ，我们通过 lighttpd 和 php 在上面托管了各自的博客，还有一些试验性质的小东西。

如果我想让 node 再去使用 80 端口，那是不可能的，但我可以通过一些手段，让它看起来像使用了 80 端口，那就是使用 Web 服务器的代理功能。

我们先把 app.js 中的端口设置改回 3000，这样，我们就能以普通用户的身份启动它了。

然后我们对 lighttpd 的配置文件进行修改，添加以下内容：

    $HTTP["host"] =~ "nb.gl" {
        proxy.server = ( "" =>
             ((
                 "host" => "127.0.0.1",
                 "port" => 3000
             ))
        )
    }

这里的 nb.gl 是我给我的应用准备的域名，你可以用自己的域名代替。
这段配置的意思就是把所有对 nb.gl 域下的内容的访问，转发到 3000 端口。
另外，我们要确保加载了 mod\_proxy 模块，然后通过 lighttpd 的 reload 机制让新配置生效。

这样一来，当我们访问 http://nb.gl 时，其实就是访问了那台服务器的 3000 端口，也就是我们的 node 应用了。

apache 和 nginx 也都支持类似的代理功能，具体实现，就不在这里讲述了。

视图和 Jade
-----------

在用 `express` 命令初始化项目时，我们已经提到过，我们选择的模板引擎是默认的 jade，也知道模板文件是被存放在 views 目录下的。
我们就从这里开始：

    $ ls ~/weibo/views
    index.jade  layout.jade

我们先来看一下 index.jade 的内容
    
    h1= title
    p Welcome to #{title}

你可能对 jade 的语法不了解，这里简单解释一下。
jade 使用类似 css 样式选择器的语法来构建标签，比如：

    p#i.c

会被编译成：

    <p id="i" class="c"></p>

在“选择器”后面跟上一个字符串，就会被视作标签里的文本，比如：

    p#i.c content 

会被编译成：

    <p id="i" class="c">content</p>

如果像 index.jade 中那样在标签后面加上 `=` 符号，后面的字符串就会被视为变量名而非文本；如果要在文本串中嵌入变量，就要采取 index.jade 中第二行所采用的语法，使用 `#{变量名}` 来引入变量。

目前，`title` 的值是 `Express`——当我们讲到 routes 时会说明如何设定模板中的变量——因此，index.jade 被渲染后应该是：

    <h1>Express</h1>
    <p>Welcome to Express</p>

正是我们访问 http://localhost:3000 所见的内容。

但是，如果你查看源文件的话，你会发现它是这样的：

    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"><html><head><title>Express</title><link rel="stylesheet" href="/stylesheets/style.css"/></head><body><h1>Express</h1><p>Welcome to Express</p></body></html>

经过整理后，就是这样：

    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    <html>
    <head>
      <title>Express</title>
      <link rel="stylesheet" href="/stylesheets/style.css"/>
    </head>
    <body>
      <h1>Express</h1>
      <p>Welcome to Express</p>
    </body>
    </html>

也就是说，index.jade 渲染出来的只是这个页面的 `body` 标签里的部分。

那么其他内容是哪来的呢？
这里，我们就要介绍下 express 的 `view options` 参数了。
在默认情况下，这个参数被设置为 true，此时，如果我们去渲染 index.jade ，得到的内容并不会被作为最终结果，而是被存在变量 body 中，之后 express 会带着这个变量再去渲染 layout.jade，这次解析得到的结果才会作为最终内容被发送到访问者的浏览器中。

我们可以通过在 app.js 中加入以下代码来禁用这个机制：
    
    app.set('view options', {
      layout: false
    });

我一般会把这段代码跟在

    app.set('view engine', 'jade');

后面。

我们也可以通过类似的方法指定自定义的模板文件：

    app.set('view options', {
      layout: 'mylayout.jade'
    });

除了指定全局的 layout 外，我们还可以针对不同文件，分别指定，但这里不再详述。

现在，我们回到项目中，看看 layout.jade 里有些什么：

    !!!
    html
      head
        title= title
        link(rel='stylesheet', href='/stylesheets/style.css')
      body!= body

不难看出， jade 通过缩进来表示 html 标签间的嵌套关系。
这里需要特别说明一下的是 `!!!` 是用来生成 `DOCTYPE` 的，目前我们不需要动它。

另外需要引起注意的就是 `body!= body` 了，我们已经知道，`=` 是用来引入一个变量的，但实际上它会先对那个变量进行所谓的 escape 操作，也就是把变量中包含的 html 特殊符号进行转义，从而防止变量对 html 的结构产生破坏并达到预防跨站脚本攻击的目的。
由于 body 变量是通过渲染 index.jade 得到的，其中包含了 html 标签，而我们又是希望这些标签能按原样输出的，因此就需要使用直接输出变量而不进行 escape 操作的 `!=` 了 。

获得 App Key 和 App Secret
--------------------------

接下来的内容都是和微博相关的，因此，在继续之前，我们需要准备好一对 APP KEY 和 APP SECRET 。
你可以选择自己喜欢的微博平台进行申请，成功创建一个应用后，平台就会为你分配一对 APP KEY 和 APP SECRET。

不同的微博平台申请流程略有不同，但都有比较完善的引导机制，你只要按着你所选择的平台上的说明，提交一系列的表单就能完成申请了。

不过为了演示的方便，这里建议你使用腾讯微博。

- 腾讯微博：[http://open.t.qq.com](http://open.t.qq.com)

现在，去创建你的应用并把 APP KEY 和 APP SECRET 带来，我会在这里等你。

用户授权
--------

既然是做一个微博应用，首先必须解决微博用户的接入问题。
直接点讲，就是允许用户使用他们的微博帐号登录我们的网络应用，也就是让用户授权。

只有完成了用户授权，我们才能进行进一步的开发，实现转发，评论等交互功能。

### 登录按钮

我们先在自己的网页上添加一个登录按钮，其链接的地址是微博的用户授权页面。
以我的应用为例，授权页面链接是：
    
    https://open.t.qq.com/cgi-bin/oauth2/authorize?client_id=801145472&response_type=code&redirect_uri=http://nb.gl/login 

其中 `https://open.t.qq.com/cgi-bin/oauth2/authorize` 是腾讯微博用于用户授权的页面地址，`801145472` 是我的应用的 APP KEY，你要使用自己的 KEY，最后 `http://nb.gl/login` 是用于接收用户授权后产生的 code 的页面地址，并且我们会在这里完成把 code 换成 accesstoken 的步骤。

在本地测试时，你可以把 `redirect_uri` 设置为 `http://127.0.0.1:3000/login` ，注意，这里只能使用 `127.0.0.1` 而不能使用 `localhost`，否则授权页面会提示你的 `redirect_uri` 是非法的。

接下来我们通过修改 index.jade 来生成这个链接：

    a(href='https://open.t.qq.com/cgi-bin/oauth2/authorize?client_id=801145472&response_type=code&redirect_uri=http://127.0.0.1/login') 登录

为了让演示更直观，我们把地址直接写在模板文件中，在实际开发中，这可不是好习惯。
不管怎样，我们的链接做好了。
需要注意：修改 views 下的模板文件，不需要重新启动应用，它们是立即生效的。

访问 http://127.0.0.1:3000/ 我们看到如下界面：

![express.png](https://github.com/surmind/bookA/blob/master/images/AuthorizeLink.png?raw=true)

### 授权

点击上一步生成的链接，就会进入微博的授权页面，这个页面将向你确认是否授权给这个应用，登录微薄并完成授权后，页面会跳转到 `redirect_uri` 对应的页面。

### 接收 code

用户完成授权后，就会跳转到类似下面的地址：

    http://127.0.0.1:3000/login?code=5e742a02447f10d0e87d8a89dcdd6483&openid=F2CFxxxxxxxxxxxxxxxxxxxxxxxx21E6&openkey=680AxxxxxxxxxxxxxxxxxxxxxxxxDCA1

其中的 openid 和 openkey 参数是腾讯微博特有的，这里我们不去管它们。

永远在线的用户
--------------
到目前为止，用户只要通过授权操作，就可以在我们的应用中访问到微博的内容了。
我们也知道，要从微博获取信息，必须以某个用户的身份发起请求。
那么，对于没有经过授权或登录操作的游客用户，我们的应用该如何从微博获取信息并展示给他们呢？

看来，我们需要一个用户随时待命，永远在线，专门为没有登录的用户获取数据。

这样的任务显然不能交给人类来做。

但是我们知道，授权过程中有一步是在一个弹出的页面中，人工输入帐号密码进行登录的，这个步骤不是接口的一部分，也就是说我们无法通过简单的发送帐号密码来实现登录（注：新浪的 oauth2.0 中，有通过帐号密码登录的接口，但是应用必须达到一定规模，并主动向新浪申请开通）。
那么我们可以模拟表单的提交么？理论上这么做是没问题的，但是会比较复杂，也不够可靠，毕竟这不是接口的一部分。

实际上，我们将使用一个更加可靠的方法来实现这个永远在线的用户。

我们将手动授权一次，然后记录 Access Token，对微博的操作都可以通过这个 Access Token 来进行。

不幸的是，目前无论是新浪还是腾讯，通过 oauth2.0 鉴权得到的 Access Token 都是有有效期的，不适用于我们的情况。

而通过 oauth1.0 鉴权得到的 Access Token 和 Access Token Secret 则没有有效期限制，除非用户主动取消授权。
这一点在腾讯微博的 oauth1.0 相关文档中有明确说明。

oauth1.0 鉴权需要通过三个接口实现，分别是：request\_token，authorize 和 access\_token。
其中 authorize 步骤需要用户手动输入帐号密码。

要完成这个过程，并不需要我们亲自去编写和发送这几个请求，腾讯微博提供了一个在线的授权及接口调试工具。
通过这个工具我们就能方便地获取到 Access Token 和 Access Token Secret 。

首先在授权及接口调试工具中，把授权方式设为 OAuth1.0，然后在接口列表中选择 request\_token，按照要求填写参数，其中 oauth\_signature 的算法可以在 Oauth1.0 的文档中找到，但如果你想偷懒的话，可以随便填些内容，然后点击下方的“检查问题”，系统会通过提示告诉你正确的签名值。
把正确的签名值填写到 oauth\_signature 后，再次点击“检查问题”，就能通过并得到未授权的 oauth\_token 和 oauth\_token\_secret，把它们记下来备用。

接下来在接口列表选择 authorize，这次只需要填写一个参数，就是前面那步里得到的那个 oauth\_token，点击“检查问题”，会给出一个“授权”按钮，点击授权按钮就进入授权页面了。
在授权页面中输入正确的用户名和密码，完成对应用的授权。然后就会跳转到你在第一步中填写的网址了。

这个网址后面跟了很多参数，我们需要的是其中的 verifier。

再次回到授权及接口调试工具，在接口列表中选择 access\_token ，然后按要求填入参数，oauth\_signature 仍然可以用之前的方法，随便乱填一个，让系统把正确的告诉你。

所有参数填写正确后，点击“检查问题”给出的那个字符串中的 oauth\_token 和 oauth\_token\_secret 就是我们需要的已经经过授权并且长期有效的 Access Token 和 Access Token Secret 了。

通过以上步骤，我们不用写一行代码，就完成了一个永远在线的用户的准备工作。

Node 中的 http 请求模块
-----------------------
http.request
http.get


    var http = require('http');
    
    var option = {
      host: 'nb.gl',
    }
    
    http.get(option, function(res){
      var data = '';
      console.log(res.headers);
      res.on('data', function(trunk){
        data += trunk;
      });
      res.on('end', function(trunk){
        if (trunk) {
          data += trunk;
        }
        console.log(data);
      });
    });

从微博获取数据
--------------
占位
