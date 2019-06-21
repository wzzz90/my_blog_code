---
title: vuecli3.0开发全栈后台管理权限系统
date: 2019-06-21 17:18:15
tags: [vue, express, mongoose, mongoodb, 全栈, jwt, 权限, 后台管理]
---
该项目使用vue全家桶，express，mongooDB等进行开发
<!--more-->
### 项目地址

https://github.com/wzzz90/vue-manageSystem.git

### 技术栈
vue + vuex + axios + element ui + echarts + sass + webpack + express + vuex-localstorage + mlab（线上mongooDB库）等

### 前端项目目录（src）
```
    + components         
      - Dialog.vue       //弹窗组件
      - Echart.vue       //图表组件
      - HeadNav.vue      //顶部导航组件
      - Layout.vue       //内容组件
      - leftMenu.vue     //左边导航菜单
    + views
      - 404.vue          //404页面
      - Fundlist.vue     //资金流水页面
      - Home.vue         //首页
      - InfoShow.vue     //个人信息页面
      - Login.vue        //登录页面
      - MyEchart.vue     //图表页面
      - noPermission.vue //无权限页面
      - PriviManage.vue  //角色权限页面
      - Register.vue     //注册页面
    - App.vue            //App页面
    - directives.js      //项目所需的指令
    - filters.js         //项目所需的过滤器
    - http.js            //封装axios
    - main.js            //项目js入口
    - router.js          //路由文件
    - store.js           //vuex
```

### 后端项目目录(server)
```
    + config 
      - key.js           //全局配置文件，包括jwt验证用到的key和连接线上mongooDB的配置
      - passport.js      //jwt验证
    + models
      - Privilege.js     //权限models
      - Profile.js       //资金models
      - Users.js         //用户models
    + routes
      + apis
        - privileges.js  //权限api
        - profiles.js    //资金api
        - users.js       //用户api
    - server.js          //后端入口文件
```


### 都做了什么

* #### 前端：

  1. 选用echarts来形象的展现在各个年份各个月份资金资金收支情况

  2. 采用权限来控制页面中元素和左侧导航菜单某个模块的显示隐藏，例如资金流水页面添加按钮和权限管理模块因登录用户角色来显示或者隐藏

  3. 进一步完成权限的设置，在路由中检测角色权限，如无权限，跳往无权限页面

  4. 在资金流水页面，完成资金的添加编辑删除功能以及数据的分页功能。

  5. 个人信息页面，完成用户信息的设置，包括用户名，头像等

  6. 登录注册页面，完成对输入内容验证以及整个逻辑

  7. 权限管理页面，权限树充分显示该用户的所有权限，并可新增权限和删除权限。

  8. 对axios的进行封装拦截等逻辑

  9. 登录接口返回token存到本地，携带token与后台进行交互


* #### 后端

  1. 使用mongoose与线上mongooDB数据库mlab进行连接

  2. 完成jwt完整逻辑

  3. 规划权限、资金和用户等model和api.



* ### 开发项目中遇到的问题

 1. 没有在项目开始前规划好后端接口返回数据格式，前端项目结构，导致在开发过程中边开发边修改，浪费很多的时间与精力

 2. 没有完成整个系统逻辑的闭环，不知道该做些什么，往往是想起什么做什么



<br/>
<br/>
<p style="text-align:center">项目做得不够好，若是发现什么问题，欢迎指正。</p>



