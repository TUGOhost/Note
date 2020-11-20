目标：hook掉luckycoffee的风控关键类，让自己成为一个“新用户”，来薅“新用户”羊毛。

启动类：com.lucky.luckyclient.launcher.LuckyApplication



特征：

- 具有两个同名的进程，frida/objection都不太好hook住，可以使用frida的spawn方式来进行attach。

https://t.zsxq.com/rniY3NZ

- 