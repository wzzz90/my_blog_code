---
title: hexo博客踩过的一些坑
date: 2019-06-11 10:53:05
categories: "Hexo教程" 
tags: [hexo问题, hexo, 博客]
---

使用github+hexo制作博客的过程中遇到了很多问题，在这里将它们记录下来以供参考
<!--more-->
### 1.在markdown文件中引入图片，发布成功之后显示不了。

hexo本质上是个静态页面生成、上传的工具。它不需要部署到我们的服务器上，我们的服务器上保存的，其实是基于在hexo通过markdown编写的文章，然后hexo帮我们生成静态的html页面，然后，将生成的html上传到我们的服务器。

图片目录必须跟该markdown文件同级，并同名，之后生成静态html文件时，才会生成该图片

![md文件与图片目录同级](./img_floder.png)

在markdown文件中，引入格式应如下所示

`![md文件与图片目录同级](./img_floder.png)`

可以通过手动来新建目录，将图片复制进去

或者可以通过安装插件更方便

1.安装一个图片路径转换的插件，这个插件名字是hexo-asset-image
`npm install https://github.com/CodeFalling/hexo-asset-image --save`

2.打开/node_modules/hexo-asset-image/index.js，将内容更换为下面的代码
```
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

var version = String(hexo.version).split('.');
hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
    	var link = data.permalink;
	if(version.length > 0 && Number(version[0]) == 3)
	   var beginPos = getPosition(link, '/', 1) + 1;
	else
	   var beginPos = getPosition(link, '/', 3) + 1;
	// In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
	var endPos = link.lastIndexOf('/') + 1;
    link = link.substring(beginPos, endPos);

    var toprocess = ['excerpt', 'more', 'content'];
    for(var i = 0; i < toprocess.length; i++){
      var key = toprocess[i];
 
      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function(){
		if ($(this).attr('src')){
			// For windows style path, we replace '\' to '/'.
			var src = $(this).attr('src').replace('\\', '/');
			if(!/http[s]*.*|\/\/.*/.test(src) &&
			   !/^\s*\//.test(src)) {
			  // For "about" page, the first part of "src" can't be removed.
			  // In addition, to support multi-level local directory.
			  var linkArray = link.split('/').filter(function(elem){
				return elem != '';
			  });
			  var srcArray = src.split('/').filter(function(elem){
				return elem != '' && elem != '.';
			  });
			  if(srcArray.length > 1)
				srcArray.shift();
			  src = srcArray.join('/');
			  $(this).attr('src', config.root + link + src);
			  console.info&&console.info("update link as:-->"+config.root + link + src);
			}
		}else{
			console.info&&console.info("no src attr, skipped...");
			console.info&&console.info($(this));
		}
      });
      data[key] = $.html();
    }
  }
});
```
3.打开根目录下的_config.yml文件，修改下述内容
`post_asset_folder: true`

4.之后使用hexo n '博客名称'的时候，可以同时生成与博客同名的图片目录。

5.部署成功后可以查看github中是否成功生成，根据新建博客时间找到该博客，如出现以下的情况，即可视为生成成功

![md文件与图片目录同级](./github_imgs.png)


### 2.部署成功之后md格式没有生效，例如将###显示出来了，这应是md文件格式符号，格式没有生效

那是因为md文件书写规则，###和标题间必须有空格。






