---
title: github+hexo制作博客
date: 2019-06-10 10:00:10
categories: "Hexo教程" 
tags: [github, hexo, 博客]
---

hexo 是基于node.js制作的一个博客工具，不是我们理解的一个开源的博客系统。

hexo 正常来说，不需要部署到我们的服务器上，我们的服务器上保存的，其实是基于在hexo通过markdown编写的文章，然后hexo帮我们生成静态的html页面，然后，将生成的html上传到我们的服务器。简而言之：hexo是个静态页面生成、上传的工具。
<!--more-->
## 准备阶段
1.安装node（如果安装过node，直接跳过这一步骤），前往node官网<https://nodejs.org/zh-cn/>下载并安装。安装成功之后,win+r快捷键出现面板输入cmd，打开cmd命令行，成功界面如下：

![node安装成功](./node_success.png)

2.Github账户注册和新建项目，项目必须要遵守格式：账户名.github.io，不然接下来会有很多麻烦。并且需要勾选Initialize this repository with a README，如下图所示：

![github注册账号并创建项目](./github_register.png)

(不要在意红色提示，那是因为我之前已经生成了博客项目)

3.安装hexo，在自己的磁盘中新建一个项目，我是在桌面也就是C盘（在哪个盘新建看自己）

![新建项目目录](./mkdir.png)

输入npm install hexo -g，开始全局安装Hexo
（如果网速不好，可以选择使用淘宝镜像cnpm install hexo -g）

![安装hexo](./install_hexo.png)

输入hexo -v，检查hexo是否安装成功

![检测hexo是否安装成功](./check_hexo.png)


## 正式开始
1.输入hexo init，初始化该文件夹，时间可能会稍微有点长

![检测hexo是否安装成功](./hexo_init.png)

 输入npm install，安装项目所需的组件

![安装组件](./install_modules.png)

2.在本地生成博客
输入hexo g，生成hexo配置文件
输入hexo s（若端口被占用，也可使用hexo s -p 5000，切换端口号），在本地生成博客

![生成博客](./hexo_order.png)

![切换端口](./switch_port.png)

在浏览器地址栏中输入localhost:4000（默认是4000，如之前切换了端口号，就在浏览器中输入localhost:<修改的端口号>）,出现以下页面说明生成成功。

![本地查看博客](./hexo_success.png)

3.将博客项目和github链接
设置Git的user name和email（如果是第一次的话）

git config --global user.name "<用户名>"
例如：git config --global user.name "terd"

git config --global user.email "<用户名>"
例如：git config --global user.email "xxxx@qq.com"


登录github<https://github.com>，在项目中点击下图所示按钮，就将网址复制下来了

![github项目](./my_github.png)

根据下图所示找到_config.yml文件，在末尾找到deploy

![github项目](./yml_path.png)

修改成下图所示，repo就是上面复制的网址

![github项目](./deploy.png)

在生成以及部署文章之前，需要安装一个扩展：npm install hexo-deployer-git --save

4.生成项目配置文件并发布
hexo g  生成配置文件
hexo d  发布
也可以简写hexo d -g，这是以上两个命令简写。
部署成功后访问你的地址：http://用户名.github.io
例如我的博客地址为：https://wzzz90.github.io/


## 小提示：
部署成功之后，访问地址会有延迟，稍等一会儿就好或者使用hexo clean删除之前的配置，重新hexo d -g部署

最后说一句，我这里使用的是vscode，安装了一个markdown插件（Markdown Preview Enhanced），在markdown文件中右键选择下图选项，可以预览文件

![预览markdown文件](./preview.png)








