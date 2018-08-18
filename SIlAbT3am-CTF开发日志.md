# SIlAbT3am-CTF开发日志

## 第一天

SIlAbT3am-CTF是一个CTF平台，做给学弟们用来校内选拔人才和进行组内出题交流的。做为一个退役的二流CTFer希望做最后一件组里的事。也希望热爱信安的学弟们不要放弃。

该平台根据Spring+SpringMVC+Mybatis，所以直接采用spring boot开发框架。

前部分时间全是仿照[PhrackCTF](https://github.com/zjlywjh001/PhrackCTF-Platform-Team)开发，然后很多东西都不太清楚。今天刚一上手，虽然tomcat运行成功了，但是项目的URL格式却不知道，一直看不到真面目，虽然把几个关键的xml都看了也没看到。另外还有postgresql和mail邮箱系统，这两块我都没接触过，当然还有阿里巴巴的druid数据连接池 Shrio 安全框架。自己目前的开发水平只能做个hello jsp。这些开发技能都还没有掌握，也借此机会学习一波。安装SSM框架的思路，最先要做的就是数据库了，但是postgresql自己实在太笨了，学了一天（网上搜索）也没搞明白，自己也就在Centos7中搭建和创建第一个DB，其次如何连接其service都不太明白，只能说前路漫漫其修远兮。

然后，这个当作一个开发系列博客，每次学了点都会写进去，加油，我一定能够成功写出来的。

#### 技术栈

**后端框架**

spring+springMVC

**数据库连接池**

阿里巴巴Druid

**SQL映射框架**

Mybatis

**安全框架**

Apache Shrio

**数据库**

postgresql

**前端**

Bootstrap+jQuery

### PhrackCTF的功能

开发思路肯定是先知道自己的程序要实现那些功能，然后逐个开发、优化。

根据PhrackCTF的源码，我先来看看这个平台具有那些功能，原作者也称他为功能最全的CTF平台。

1. 缓存管理器（SpringCacheManagerWrapper）
2. 对日期排序（CompareDate）用户按得分排序（CompareScore）队伍得分排序（CompareTeamScore）
3. 验证码生成器（CaptchaImageCreateController）
4. 答题页面的Controller（ChallengeController）
5. 国家页面的控制器 （CountryController）
6. 个人面板控制器 （DashboardController ）
7. 后台文件上传的控制器 （FileUploadController ）用于放MISC或者PWN/RE题目文件
8. Home控制器 （HomeController）
9. 登陆（LoginController）
10. 后台邮件功能控制器 （MailController ）
11. 后台主页 （ManageController ）
12. 用户资料显示页面 （ProfileController ）
13. 注册页面控制器 （RegisterController ）
14. 重置密码 （ResetPassController ）
15. 规则页面 （RuleController ）
16. 排名页面的控制器 （ScoreBoardController ）
17. 提示信息的页面 （ShowInfoController ）
18. 赛题的解题状态 （SolveStatusController ）
19. My Team页面Controller （TeamController ）
20. 队伍公共信息页面 （TeamInfoController ）
21. CSRF安全处理
    1. Request预处理器，先过滤csrf然后才允许访问对应controller （CSRFHandlerInterceptor ）
    2. 过滤CSRF请求 （CSRFRequestDataValueProcessor ）
    3. CSRF Token管理 （CSRFTokenManager ）
22. 禁止登录IP查询服务类的实现 （BannedIpServicesImpl ）
23. 类别服务实现类 （CategoryServicesImpl ）
24. 赛题的服务实现类 （ChallengeServicesImpl ）
25. 系统设置服务实现类 （ConfigServicesImpl ）
26. 国家接口的实现 （CountryServicesImpl ）
27. Files字段实现类 （FileServicesImpl ）
28. Hints表段服务实现类 （HintServicesImpl ）
29. 用户操作日志服务实现类（IPlogServicesImpl ）
30. 新闻服务实现类（NewsServiceImpl ）
31. 后台操作日志服务的实现类 （OperateLogServicesImpl ）
32. 密码重置服务的实现类 （PassResetServicesImpl ）
33. 规则页面实现方法 （RuleServicesImpl ）
34. 对应表submissions服务实现类 （SubmissionServicesImpl ）
35. 队伍查询服务的实现类 （TeamServicesImpl ）
36. 被禁止登录的IP查询服务 （BannedIpServices ）
37. 题目类型的Services （CategoryServices ）
38. 所有赛题的服务接口 （ChallengeServices ）
39. 系统设置信息 （ConfigServices ）
40. 国家接口 （CountryServices ）
41. 对应数据库Files字段 （FileServices ）
42. 服务类接口，对应字段hints （HintServices ）
43. 用户操作日志 （IPlogServices ）
44. 新闻的数据库访问操作 （NewsServices ）
45. 后台操作日志的接口 （OperateLogServices ）
46. 密码重置服务 （PassResetServices ）
47. 规则页面数据接口 （RuleServices ）
48. 用户提交答案记录的Services （SubmissionServices ）
49. 管理参赛队伍 （TeamServices ）
50. 用户数据库访问接口 （UserServices ）
51. 每道题最后的附件对象 （Attaches ）
52. 按分类统计的进度 （CateProcess ）
53. 题目管理（ChallengeInManager ）
54. 国家页面的排行榜 （CountryRank ）
55. 用于在后台展示用的Hint呈现类 （HintDisp ）
56. team后台显示的ip地址列表 （IpLogsList ）
57. 后台操作界面用于显示的对象 （OpLogDisp ）
58. 用户排名对象 （RanklistObj ）
59. 分数页面展示用的对象 （ScoreBoardObj ）
60. 趋势图的POJO类 （ScoreTrend ）
61. 每个用户的每个题目状态 （SolveList ）
62. 赛题通过列表里的对象 （SolveStat ）
63. 用户信息页面通过的赛题列表对象 （SolveTable ）
64. 提交列表的POJO类 （SubmissionList ）
65. 用于显示的Hint POJO类 （TaskHint ）
66. 后台team列表 （TeamList ）
67. 团队成员的类 （TeamMember ）
68. 后台用户列表的对象 （UserList ）
69. 通用操作（CommonUtils ）
    1. 获取当前控制器名称 
    2. 对用户输入内容进行过滤  
    3. 统计用户使用信息 
    4. xss过滤器 
70. 后台日志记录工具类 （后台日志记录工具类 ）
71. 邮件发送工具类 （邮件发送工具类 ）
72. Shiro（shiroDbRealm ）
    1. 清除所有用户授权信息缓存 

---

**有这些功能**

缓存管理器
排序，日期、个人得分、团队得分
验证码生成器
答题页面的控制器
国家页面的控制器
个人面板控制器
后台文件上传
Home
登陆
后台邮件功能
后台主页
用户资料显示页面
注册页面
重置密码
规则页面
排名页面
提示信息
赛题的解题状态
我的团队页面
队伍公共信息页面
CSRF
禁止登陆IP查询
用户操作日志
新闻服务
后台操作日志
队伍查询
系统设置信息
管理参赛队伍
题目管理
按分类统计的进度
团队后台显示的ip地址列表
趋势图
xss过滤器
Shiro

---

