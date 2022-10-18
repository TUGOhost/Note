# 关于wordpress自动更新详解

### 前言

web自动更新，这样方便使用者能够可以较安全的使用web程序，并且开发者及时更新也是对使用者的负责，修复bug和优化使用。可以让web程序更符合“我”为“人人”这一理念。所以，我自己花了段时间仔细的研究了下wordpress自动更新的背后故事。

---

贴出来官网文档关于wordpress更新的相关说明

https://codex.wordpress.org/Upgrading_WordPress_Extended

https://codex.wordpress.org/zh-cn:%E5%8D%87%E7%BA%A7_WordPress

----

今天成功把模仿天猫官网的项目放到了vps中，说明只要tomcat启动了，项目源码再webroot目录下，即可使用了。

然后就是，wordpress更新设置，根据官网的说明，是先对数据库进行备份，并且提醒用户要备份好网站源码，也就是说，wordpress更新不一定能够成功，他更新也只是把所要替换的关键文件给删除，再从服务器重新down一份最新的下来。

