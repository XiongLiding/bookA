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
我们的第一步，必须是安装 node 啊！
尽管很多介绍 node 的文章或者书籍已经包含了相关内容，这里还是要简单提一下，并且指出一个和微博应用开发相关的编译细节。

### Node 的安装

目前，大多数主流的，面向桌面用户的 Linux 发行版，已经可以通过各自的包管理系统安装 node（以及 npm ）了。

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

到这里，node 的安装就完成了，我们可以简单检查一下：

    # node -v
    v0.6.18

### Npm 简介

简单的说，npm 是 node 的包管理器，在安装 node 的同时就已经安装了。它是一个类似 yum 或 apt 的工具，就连命令的格式也很相似，`npm install` 用来安装，`npm remove` 用来移除。
一般情况下，执行 `npm install` 会在当前目录下建立一个名为 node\_modules 的文件夹，并把你指定的模块安装到这个目录下。
当然，npm 的功能不止于此，但目前，我们知道这些就够了。

用 Express 搭建一个基础的网页服务器
-----------------------------------
占位

多个 Web 服务程序并存
---------------------
占位

Jade 和 Less
------------
占位

获得 App Key 和 App Secret
--------------------------
占位

用户授权
--------
既然是做一个微博应用，首先必须让我们的网络应用实现接入微博的能力。
通常，接入微博的第一步，就是实现通过微博帐号登录应用的功能。
这里，我们选择 oauth1.0 来实现这个功能，至于为什么选择 1.0 而不使用“更简单，更安全”的 2.0，我们会在后面讲到原因。

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
