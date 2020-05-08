---
header:
  image:/assets/images/IMG_0098.jpeg
layout: single
toc: true
toc_label: "目录"
toc_icon: "bars"
category: 博客搭建
title:	"Docker + Jekyll + Nginx 静态博客自动化部署过程总结"
date:   2020-03-15 07:15:00 +0000
typora-root-url: ../assets/images
---

在学习专业课和服务器运行环境搭建的过程中，从网络上各种博文中学到了很多东西，也慢慢意识到写博客的重要性，不仅是为了方便日后的查询和回顾，在写博客的过程中也是一种对所学知识的复现和巩固，某种程度上也是费曼学习法的一种体现。

网络上各种博客站点很多，但我觉得一方面是管理文章不够自由，另一方面是界面也不够美观，于是就开始着手建立属于自己的博客



### 选择博客形式

首先就是选择博客的形式，博客分静态博客和动态博客，动态博客就是网络上较为流行的那些网站的形式，写文章是直接在网站上写的，比如CSDN，简书等等，搭建这种博客比较流行的就是[Wordpress](https://cn.wordpress.org/)了，而Wordpress基本上是傻瓜式建站，虽然有很多插件，但可定制还是感觉有点低，而且我也比较倾向于在本地的环境中进行沉浸式写作，所以最后选择了静态博客站点。

静态博客大多数都是直接部署在github pages上的，国内访问速度难以接受，所以我就想把博客搭建在自己的云服务器上。

要想在自己的服务器上搭建静态博客，就需要借助静态站点生成器，我选择的是[Jekyll](http://jekyllcn.com/)，非常简单而又强大的生成器，github pages就是内置了jekyll，我比较喜欢的一点就是，只要源文件有了变化，jekyll就可以自动重建网页，这就为自动化部署网站带来了方便。类似的还有[Hexo](https://hexo.io/zh-cn/index.html)等等，具体介绍可以参见官网。

这里的自动化流程如下：

> - 本地写好网站文章的文本文件，可以是markdown或者html等等
> - 将文本推送github
> - github检测到push操作，触发webhooks，向远程服务器发送http请求
> - 服务器的node js程序监听到http请求，执行bash脚本从github拉取更新后的源文件
> - 只要源文件更新了，jekyll自动重构
> - 网页则通过nginx进行代理转发 



### 开始部署环境



刚开始搭建的过程中，参考了方志朋的[这篇博文](https://cloud.tencent.com/developer/article/1434577)，从零开始搭建服务器环境，较为繁琐，容易遇到太多服务器环境问题，解决起来还比较麻烦，不太推荐这种方法了，我搭过之后深有体会。



后面通过其他的博文了解到可以使用docker进行一键部署。对于docker其实早有了解，只是知道可以通过容器把应用程序和其运行环境绑定起来，一次搭建，后面就可以在任何机器环境中运行。但直到真正用docker搭建一个了web应用之后我才深刻体会到docker的强大之处。



关于docker的学习和介绍，后续会再写一些博文补充。上面的自动化过程看起来很复杂，但其实用到docker之后一行命令就能搞定！



#### 准备工作

docker的安装过程比较简单，官方文档很详细了，这里不再赘述。

主要介绍一下docker-compose程序，[docker-compose](https://docs.docker.com/compose/)是基于docker的一款应用程序，它可以把多个docker容器关联起来，从而使分散的容器联合起来运作，既能保证各容器的独立性，又能使其相互通信变得很简单，如果是桌面版（Windows 和 Mac版）的话应该集成在docker中了，在linux系统中需另外安装，安装方法如下

##### 安装docker-compose

执行如下命令下载二进制文件：

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

然后再更改文件权限：

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

如果没有安装curl或者下载速度很慢，也可以直接去[github](https://github.com/docker/compose/releases)下载源文件放到`/usr/local/bin/`下，再更改文件权限。

安装完就可以用`docker-compose -v`查看版本信息

再安装一些必备工具如git，vim之类的，准备工作就差不多了，下面开始正式搭建博客。



#### 搭建docker应用群

新建工作目录，并进入

```shell
mkdir blog
cd blog
```

这个工作目录就相当于整个博客项目的主目录了，所有的配置过程都在这个目录下进行，完全不需要在目录之外再进行任何配置，这就为打包迁移项目提供了很大的方便，这就是docker的魅力所在。

创建网站文件夹存放网站源码和编译后的文件

```shell
mkdir site
cd site
mkdir src
mkdir data
```

`src` 用于存放源码，`data` 存放jekyll构建后的网页

用docker构建的**容器**群组，主要由三部分组成：

> 1. Nginx 服务，提供网页的代理和端口转发
> 2. Jekyll ，构建静态网页
> 3. node js 服务，用于监听端口，实现和webhook的联动

下面就分别开始建立这三个容器



#### Nginx

在主目录创建nginx目录并进入

```shell
mkdir nginx
cd nginx
```

创建`conf.d`文件夹用于存放nginx配置文件

```shell
mkdir conf.d
```

并在其中创建一个配置文件`default.conf`

```shell
cd conf.d
vim default.conf
```

输入下面的内容：

```nginx
server {
    listen       80;
    server_name  localhost;
    root /usr/share/nginx/html;
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    location ~* /(images|css|assets) {
        index index.html;
    }	
}
```

回到`nginx`目录，创建nginx的`Dockerfile`

```shell
vim Dockerfile
```

并输入一下内容：

```dockerfile
FROM nginx
COPY nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
```

然后回到主工作目录，创建`docker-compose.yml` 

```shell
vim docker-compose.yml
```

输入下面的内容：

```yml
version: '3.7'

services:
  nginx:
    build:
      context: .
      dockerfile:  ./nginx/Dockerfile
    volumes:
      - ./site/data:/usr/share/nginx/html
      - ./nginx/conf.d:/etc/nginx/conf.d
    ports:
      - 25564:80
```

主要规定了`docker-compose` 创建镜像的路径和`nginx`容器挂载的目录`./site/data` 以及端口映射，这里我的域名还没备案，暂时先用其他端口代替

其他细节，可参见`docker-compose` [官方文档](https://docs.docker.com/compose/compose-file/)

现在执行以下命令就可以让`nginx`跑起来了

```shell
docker-compose up --build 
```

不过现在`nginx`挂载的目录下还没有文件，可以试着写一个`index.html` ，然后访问`ip`应该就能看到了。

要停止容器，运行：

```shell
docker-compose down
```

查看容器运行状态：

```shell
docker-compose ps
```



#### jekyll

下面开始安装`jekyll`容器，创建`jekyll`目录并进入

```shell
mkdir jekyll
cd jekyll
```

创建`Dockerfile` :

```shell
vim Dockerfile
```

输入下面的内容：

```dockerfile
FROM phlens/jekyll:latest
CMD bundle exec jekyll build --watch
```

这里注意每个主题的镜像不一样，不能照搬，根据主题的官方文档构建好自己的镜像

主题可以在Jekyll官网或者GitHub上查看挑选

接下来回到主目录，修改`docker-compose.yml` ,添加`jekyll`的部分

```yaml
version: '3.7'

services:
  nginx:
    build:
      context: .
      dockerfile:  ./nginx/Dockerfile
    volumes:
      - ./site/data:/usr/share/nginx/html
      - ./nginx/conf.d:/etc/nginx/conf.d
    ports:
      - 25564:80
  jekyll:
    build:
      context: .
      dockerfile:  ./jekyll/Dockerfile
    volumes:
      - ./site/src:/srv/jekyll
      - ./site/data:/srv/jekyll/_site
```

这里给`jekyll`指定了工作的目录和构建好的文件目录，`./site/data`与`nginx`转发的目录正好是对应的

之后我们只需要在`src`文件夹里从github拉取代码即可

```shell
cd /site/src
git clone 你的代码仓库 .
```

注意最后的`.` 不要忘了

然后现在直接运行`docker-compose` 就可以了

```shell
docker-compose up --build
```

运行成功大概是这样

![image-20200315221706754](/image-20200315221706754.png)

现在访问ip应该就可以看到自己的网站了

最后只需要完成自动部署就大功告成



#### 自动部署

关于自动部署，`jekyll`的官方文档有不少详细的描述，我没有细看，只看到有一个利用本地的`githooks`脚本的，但我尝试之后发现，那种方法貌似是基于宿主机的，对`docker` 貌似无效，而且依赖宿主机也就违背了最开始建站的初衷了，所以最后是利用了`github`自带的`webhooks` ，详细描述可以查看github文档

这里参考了之前方志朋的那篇博文，利用`node js` 自带的`http`组件，构建一个`http` 响应程序

先创建一个`node js` 的`docker`容器

```shell
mkdir autopull
cd autopull
vim Dockerfile
```

`Dockerfile`的内容如下：

```dockerfile
FROM node:6-alpine
ENV NODE_ENV production
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories && apk update && apk upgrade && \
    apk add --no-cache bash git openssh 
RUN npm config set unsafe-perm true 
WORKDIR /app
COPY ./autopull/* /app/
RUN npm install github-webhook-handler
EXPOSE 9091
CMD node deploy.js

```

这里的`dockerfile` 我改了很多次才最后能运行起来，遇到各种问题，先是源更新过慢，我就把镜像源换成了[清华的镜像源](https://mirrors.tuna.tsinghua.edu.cn/) ，然后利用npm安装 `github-webhook-handler`组件，用来拆分github送来的数据，以便于分析并作出回应，这里要注意

> 直接把	`github-webhook-handler`	安装在node容器的工作目录下，就是不使用-g全局参数，不然后面调用require包含这个模块时会出错

可以看到容器最后就是运行一个`deploy.js`程序，这个就是用来响应http请求的小程序。

因为`dockerfile`中已经把`/autopull`目录下的文件都拷贝到了node的工作目录`/app`下，所以直接在`autopull`目录下创建`deploy.js`

```shell
vim deploy.js
```

代码如下：

```js
var http = require('http')
var createHandler = require('github-webhook-handler')
var handler = createHandler({ path: '/autopull', secret: '123456' });//这里根据你的情况自己改
 
function run_cmd(cmd, args, callback) {
  var spawn = require('child_process').spawn;
  var child = spawn(cmd, args);
  var resp = "";
 
  child.stdout.on('data', function(buffer) { resp += buffer.toString(); });
  child.stdout.on('end', function() { callback (resp) });
}
 
http.createServer(function (req, res) {
  handler(req, res, function (err) {
    res.statusCode = 404
    res.end('no such location')
  })
}).listen(9091)//端口根据自己的情况改，与nginx配置要一致
 
handler.on('error', function (err) {
  console.error('Error:', err.message)
})
 
handler.on('push', function (event) {
  console.log('Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref);
  run_cmd('sh', ['./start_blog.sh'], function(text){ console.log(text) });
})
```

这里最后检测到`http`请求后会调用一个`shell`脚本,`start_blog.sh` ,该脚本和`deploy.js`放在同一目录下，即`autopull`，内容如下：

```
echo `date`
cd /usr/local/lib/site
echo start pull from github 
git pull 你自己的仓库名
echo start build..
```

脚本会去到`/usr/local/lib/site` 目录下进行代码拉取，这是`node`容器里的目录，其应挂载到宿主机的`/site/src`目录下，注意不要直接在`/app`目录下拉取代码，会报错找不到`github-webhook-handler`模块，具体原因暂时还不清楚。

所以修改`docker-compose.yml`:

```yaml
version: '3.7'

services:
  nginx:
    build:
      context: .
      dockerfile:  ./nginx/Dockerfile
    volumes:
      - ./site/data:/usr/share/nginx/html
      - ./nginx/conf.d:/etc/nginx/conf.d
    ports:
      - 25564:80
  jekyll:
    build:
      context: .
      dockerfile:  ./jekyll/Dockerfile
    volumes:
      - ./site/source:/srv/jekyll
      - ./site/data:/srv/jekyll/_site
  autopull:
    build:
      context: .
      dockerfile: ./autopull/Dockerfile
    ports:
      - 9091:9091
    volumes:
      - ./site/src:/usr/local/lib/site
      - ./autopull:/usr/local/lib
```

最后将`/usr/local/lib`挂载到`/autopull`是为了后面接受来自`github`的请求

现在，到`github`上设置`webhooks`参数

![image-20200315231752150](/image-20200315231752150.png)

注意这里虽然填了一个`/autopul`作为`URL`，但实际上只作标识用，`github`会在`http`请求中带上`/autopull` ,这样deploy.js程序就能识别到，进而触发后续操作

点击更新，就大功告成了，现在，只需要在服务器开启docker-compose,本地写文本就能自动构建博客了。

