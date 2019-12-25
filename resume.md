 <center>
     <h1>刘雨</h1>
     <div>
         <span>
             <img src="assets/phone-solid.svg" width="18px">
             15555733491
         </span>
         ·
         <span>
             <img src="assets/envelope-solid.svg" width="18px">
             tugohost@foxmail.com
         </span>
         ·
         <span>
             <img src="assets/github-brands.svg" width="18px">
             <a href="https://github.com/TUGOhost">https://github.com/TUGOhost</a>
         </span>
         ·</br>
         <span>
             <img src="assets/rss-solid.svg" width="18px">
             <a href="https://github.com/TUGOhost/Note">https://github.com/TUGOhost/Note</a>
         </span>
     </div>
 </center>

 ### <img src="assets/info-circle-solid.svg" width="30px"> 个人信息 

 - 求职意向：Java 研发工程师

### <img src="assets/graduation-cap-solid.svg" width="30px"> 教育经历

- 本科，宿州学院，软件工程，2016.9 ~ 2020.7
### <img src="assets/graduation-cap-solid.svg" width="30px"> 获奖经历
- 2017/05 2017年全国大学生安全与对抗技术个人挑战赛二等奖
- 2018/06 2018年全国大学生信息安全竞赛创新实践能力赛三等奖  

###  <img src="assets/tools-solid.svg" width="30px"> 技能清单

- 熟悉基础数据结构与经典算法，计算机网络等基础知识。
- 掌握 Java 基础，多线程编程，集合框架；熟悉 JVM 虚拟机平台规范，类加载机制，运行时数据区，GC垃圾回收机制。
- 掌握设计模式，面向对象编程，研读过相关书籍，希望能够学以致用。
- 热爱开源技术，熟悉 Spring MVC / MyBatis 等开源框架；深入研究过 Spring 技术栈，掌握 IOC / AOP 核心原理。
- 熟练掌握 MySQL ，深入研究过 InnoDB 存储引擎，研读过相关书籍，数据库设计规范，能够对常见的 SQL 语句进行优化。
- 熟练使用NoSQL数据库 Redis ，深入研究过Redis的集群，读写分离，持久化，数据类型以及应用场景。
- 熟悉项目构建工具 MAVEN ，熟悉项目管理工具 Git ，熟悉 Linux 的常用操作，熟悉使用 Docker 容器搭建相关集群等。

### <img src="assets/briefcase-solid.svg" width="30px"> 工作经历

- **中国平安，商圈研发团队，Java开发实习生，2019.9~2019.12**

   根据产品经理和项目主管的需求，开发、优化壹钱包和平安银行商城后端。主要参与了商城小程序砍价、商城敏感数据脱敏、重构了营销活动发布。

### <img src="assets/project-diagram-solid.svg" width="30px"> 项目经历

#### 个人项目

- **孤独星球**:
  - GitHub地址: https://github.com/TUGOhost/LonelyPlanet
  - 开发时间: 2019.04
  - 项目描述: 该项目是利用WebSocket协议通信的IM软件。使用Netty实现WebSocket以及HTTP网络协议；Spring Boot作为Web服务端；使用FastDFS作为图片文件存储系统，使用Google ZXing来实现通过二维码添加好友功能。
- **MVCFramework**:
  - GitHub地址: https://github.com/TUGOhost/MVCFramework
  - 开发时间: 2019-05
    - 项目描述:  该项目是一个仿照Spring MVC的api实现的简单的web框架。利用到了内嵌tomcat，以及反射。API注解只有一个value参数，指此类处理的的url。

#### 实习项目


- **小程序砍价**:

  - 开发时间: 2019.09 ~ 2019.10
  - 项目描述: 用户可以在壹钱包APP或者壹钱包商城小程序中分享砍价信息发给好友和群，请求好友帮砍。在这个项目中我主要负责用户砍价权益名称、帮砍次数、风控金额以及帮砍规则。后端使用dubbo配合zookeeper的分布式架构。
- **重构营销活动发布**: 
- 开发时间: 2019.11 ~ 2019.12
  - 项目描述: 运营需要在后台上架活动商品，由于需要添加的商品过多超过十万条数据，在json解析时和报日志时经常导致线上full gc。之前版本的商品活动鉴别需要大量的if...else，导致需要执行很多次才开始一个任务，我在这里做了优化，使用工厂+责任链模式来替代之前的大量if...else，并且，之前有个校验商品活动的方式过于繁琐，之前版本是直接查询所有商品，再校验，我这里直接做了分批次的校验商品，如果商品活动重复了，报错约50个左右商品的信息，再不执行。测试大量的数据时，未出现过full gc。
