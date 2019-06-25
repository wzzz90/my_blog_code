---
title: github+hexo的部分配置
date: 2019-06-11 10:36:54
categories: "Hexo教程" 
tags: [github, hexo, 博客主题, 博客头像]
---

有关于hexo的配置
<!--more-->

#### 主题的配置
hexo的主题有很多种，无论是hexo官网主题<https://hexo.io/themes/>还是github上都有很多，供用户自己选择。

以我使用的主题为例，我使用的是yilia主题。
使用主题相当简单，只需要两步:
1.将主题代码文件克隆到themes目录下
` git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia `
2.修改根目录下_config.yml中的theme为yilia
` theme: yilia `

然后hexo d -g，等待部署成功之后，就可以看到博客的新主题了。

#### 配置头像

配置头像同样只需要两步：

1.将自己的头像放到themes\yilia\source目录下

![图片存放地址](./avatar_path.png)

2.在themes\yilia目录下，找到_config.yml文件，
完成下面的设置(下图中的avatar.jpg，是自己的头像的名称)

![图片存放地址](./avatar_config.png)


#### 如何在左侧显示总文章数？

修改themes\yilia\layout_partial\left-col.ejs

```
<nav class="header-menu">
    <ul>
    <% for (var i in theme.menu){ %>
        <li><a href="<%- url_for(theme.menu[i]) %>"><%= i %></a></li>
    <%}%>
    </ul>
</nav>
```
在上面的代码后加上

```
<nav>
    总文章数 <%=site.posts.length%>
</nav>
```
如下图

![文章数](./article_count.png)

之后就可以显示了。


#### 添加字数统计

1.首先需要安装hexo-wordcount
`npm i --save hexo-wordcount`

2.然后在themes\yilia\layout_partial\left-col.ejs中
```
	<nav>
        总文章数 <%=site.posts.length%>
    </nav>

```
在上面的代码后加上

`总字数 <span class="post-count"><%= totalcount(site, '0,0.0a') %></span>
`


在themes\yilia\layout_partial\article.ejs中的header下面加入
```
<div align="center" class="post-count">
    字数：<%= wordcount(post.content) %>字 | 预计阅读时长：<%= min2read(post.content) %>分钟
</div>
```
即可显示单篇字数和预计阅读时长。