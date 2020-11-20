包名：com.ss.android.article.news.ttt

0xd3ac2000.dex  0xd412b000.dex  

启动类：com.ss.android.article.ui.LaunchActivity



>  objection -g com.ss.android.article.news.ttt explore

目标：让自己的剩余观看次数无限

### 脱壳

使用frida-dexdump进行脱壳，发现壳是脱了，但是一堆混淆的东西，看不懂。

但是能够搜索到相关的字符，也就是对关键字并没有进行加密，也就是做了混淆。

### 利用frida修改设备id

```javascript
function changeAndroid_ID() {
    Java.perform(function () {
        var Secure = Java.use("android.provider.Settings$Secure");
        Secure.getString.implementation = function (p1, p2) {
            if (p2.indexOf("android_id") < 0) return this.getString(p1, p2);
            console.log("[*]Called - get android_ID, param is:" + p2);
            var temp = this.getString(p1, p2);
            console.log("real Android_ID: " + temp);
            return "844de23bfcf93801";
        }
    })
}

setImmediate(changeAndroid_ID);
```

> frida -UF -l 1.js

在输入自己的邀请码，即可无限的观看次数。