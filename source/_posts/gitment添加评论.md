---
title: gitment添加评论
date: 2019-06-12 17:05:02
categories: "Hexo教程" 
tags: [gitment, hexo, 添加评论, gitment问题]
---

Hexo博客的Yilia主题中评论系统提供了畅言、网易云跟帖、多说、Disqus和gitment。
由于多说评论、网易云跟帖已经关闭系统，畅言需要域名备案，disqus是国外的，经常被墙，你懂得。
所以斟酌再三选择gitment。
<!--more-->

### 正文

gitment评论系统操作只需要三步：

1.注册OAuth Application, 注册网址为<https://github.com/settings/applications/new>

![注册OAuth Application](./register_oath.png)

Application name:博客名字
Homepage URL; 网站主页地址(这里我填写的是域名)
Application description:描述，选填
Authorization callback URL: 填写域名就行了

填写完成后，会获得gitment_oauth的client_id和client_secret

2.在themes\yilia\_config.yml中，配置gitment

![注册OAuth Application](./oath_config.png)

3.配置成功后，hexo s，在文章底部查看是否存在评论框

![查看是否配置成功](./comment.png)



### 配置gitment过程中出现的问题

1.配置完成后，hexo s查看文章底部并没有出现评论框

打开F12之后发现出现报错，Gitment is undefined，并且有关于gitment的js，css资源加载失败

在themes\yilia\layout\_partial\post\gitment.ejs中将
```
<link rel="stylesheet" href="//imsun.github.io/gitment/style/default.css">
<script src="//imsun.github.io/gitment/dist/gitment.browser.js"></script>
```
修改为
```
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
```
部署成功后在文章底部就会出现评论框

2.出现评论框后，点击登录github账号，弹出提示（Object ProgressEvent），一直在登录。

最近gitment作者那个服务器过期了好像，所以登陆时一直报Object ProgressEvent。

可以将themes\yilia\layout\_partial\post\gitment.ejs中的
```
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
```
修改为
```
<link rel="stylesheet" href="https://jjeejj.github.io/css/gitment.css">
<script src="https://jjeejj.github.io/js/gitment.js"></script>
```

3.出现Error：Comments Not Initialized

![Error：Comments Not Initialized](./init.png)


文章作者需要登陆GitHub授权后，会显示初始化按钮(注意，不要多点按钮，否则issues出现多条一样的)

![init](./init_success.png)


4.点击初始化评论之后，出现弹框提示Error：validation failed。

issue的标签label有长度限制！labels的最大长度限制是50个字符。

在issues里面，可以发现是根据网页标题来新建issues的，然后每个issues有两个labels（标签），一个是gitment，另一个就是id。这个id的作用，就是针对一个文章有唯一的标识来判断这篇本章。

所以明白了原理后，就是因为id太长，导致初始化失败，现在就是要让id保证在50个字符内。

还是在themes\yilia\layout\_partial\post\gitment.ejs中，将id保证在50个字符以内
`id: '<%= page.date %>'`

![id](./id.png)


参考：<https://www.jianshu.com/p/57afa4844aaa>




