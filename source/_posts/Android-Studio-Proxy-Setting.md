---
title: Android Studio proxy 设置
date: 2016-07-16
tags:
	- Proxy
	- Android Studio
---
广大程序员都知道，由于特殊的地理优势，在访问 jcenter 的时候，不科学上网是不行的。
<!-- more -->
## 正文
平时我使用科学上网的工具是 shadowsocks ，不知道 ss 的朋友可以去搜搜 ss 是什么。在我开启了全局模式之后，Android Studio 也是不会自动连接外网的。后来查到在 `Preferences/Appearance & Behavior/System Settings/HTTP Proxy` 里手动设置
![](https://ww2.sinaimg.cn/large/006tKfTcly1fd48qvkebzj31kw113jy3.jpg)
这样也是没有效果的！
最后终于找到了方法，在项目根目录下的 `gradle.properties` 里加入  `org.gradle.jvmargs=-DsocksProxyHost=127.0.0.1 -DsocksProxyPort=1080` ，然后进行 `sync`，完美解决问题！

## ps
顺便附上在 iterm2 里进行代理的命令行代码

``` shell
export http_proxy=socks5://127.0.0.1:1080
export https_proxy=$http_proxy
```
每次想设置 proxy 就复制一下就行了。如果不想那么麻烦，就写入 `.zshrc` 里好了。

