---
title: TinkerPatch 最简单的接入方案
tags: 
	- Hotfix
	- Tinker
	- Android
---
天天埋头做业务的你，是不是会经常听到下面一些话：“完了，xx页面的文案忘了改了”。由于初创公司的开发流程不完善，可能会导致各种各样的问题，甚至在 app 上线之后，立马就会找到一个 bug。如何避免这种情况呢？Hotfix 可能是目前比较成熟的一个方案。
<!-- more -->
## 什么是 TinkerPatch
你可能听过 [Tinker](https://github.com/Tencent/tinker) ， 以 Dota 中的[修补匠](http://www.dota2.com.cn/hero/tinker/)命名，是腾讯团队出的一个热修补框架，能够在不更新新的安装包的情况下对 app 进行修复。但是 Tinker 需要你自己有一个服务器，并且有相应的后台机制，这对初创公司来说可能是一个非常难以实现的要求。
所以， TinkerPatch 就出现了。它使用七牛 CDN 对开发者上传的补丁进行加速，开发者无需自己搭建后台，就可以享受热修补带来的方便。当然， TinkerPatch 目前是免费的，以后是否会收费也从来没有说明过，但我想说的是，如此好用的平台，就算以后收费了，也是值得继续使用的。更何况腾讯将 [server 端](https://github.com/baidao/tinker-manager)也开源了，不过据说没有 TinkerPatch 的一键接入好用。
## SDK 接入
如果你想快速上手热修补，想快速做出一个 Demo 查看成果，是无须知道 Tinker 相关事宜的。当然，在入门之后深入了解 Tinker 还是非常有必要的。官方的开发文档
[TinkerPatch SDK 接入](http://tinkerpatch.com/Docs/SDK)比较详细，但是还是有一些让人觉得不太清楚，我会详细的将每一步都说明出来。
### 注册 Tinker Platform
首先得[注册](http://tinkerpatch.com/Index/reg)一个账号，在验证邮箱之后，会进入 App 管理页面，点击新增 APP ![新增 APP](https://ww2.sinaimg.cn/large/006tNbRwly1fd2h4myjsaj30g80msdg5.jpg) ，输入 Demo 的名字，叫 `TinkerPatchDemo` 好了。之后进入此 app 的管理页面，左边的 appKey 是之后要放入 `tinkerpatch.gradle` 里的参数，右边可以为某一版本添加补丁。左下角还有一些其他功能，我们暂时不需要用到它们。
![](https://ww4.sinaimg.cn/large/006tNbRwly1fd2jd6yrlyj30yy0ywgnn.jpg)
### gradle
首先告诉大家，我已经将该 demo 的源码上传至[TinkerPatchDemo](https://github.com/abcghy/TinkerPatchDemo) ，有的地方不清楚的可以看源码，实在不清楚的可以在评论区提交问题。
打开 Android Studio ，新建一个名为 `TinkerPatchDemo` 的项目。进入之后先将项目架构由 Android 改为 Project 以便于我们的操作。![](https://ww1.sinaimg.cn/large/006tNbRwly1fd2hcezy5uj30ds0h075d.jpg)
在项目根目录中的 [`gradle.properties`](https://ww3.sinaimg.cn/large/006tNbRwly1fd2hh7hqnzj30gg0is0u9.jpg) 文件中设置我们 Tinker, TinkerPatch 的版本号。

``` properties
TINKER_VERSION=1.7.7
TINKERPATCH_VERSION=1.1.3
```
接着，在根目录的 [`build.gradle`](https://ww3.sinaimg.cn/large/006tNbRwly1fd2hksvjojj30ki0lqgnc.jpg) 添加

``` gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        //无需再单独引用tinker的其他库
        classpath "com.tinkerpatch.sdk:tinkerpatch-gradle-plugin:${TINKERPATCH_VERSION}"
    }
}
```
接着，进入 app 的目录，找到 `build.gradle` ，添加几个 dependencies，并在最后添加 `apply from: 'tinkerpatch.gradle'`。

``` gradle
dependencies {
    //********
    compile "com.android.support:multidex:1.0.1"
    //若使用annotation需要单独引用,对于tinker的其他库都无需再引用
    provided("com.tencent.tinker:tinker-android-anno:${TINKER_VERSION}") { changing = true }
    compile("com.tinkerpatch.sdk:tinkerpatch-android-sdk:${TINKERPATCH_VERSION}") { changing = true }
}

apply from: 'tinkerpatch.gradle'
```
这个时候就遇到刚才的提到的 `tinkerpatch.gradle` 了，这是什么呢？它很显然是一个 gradle 文件，里面包含了 tinkerpatch 的各个参数，对于里面的各项参数是什么意思，官网都给出了[答案](http://tinkerpatch.com/Docs/SDK)，我就不费口舌了，不过在咱们的 demo 里面是不需要仔细知道各个参数的意义的，你可以直接从 [我的源码](https://github.com/abcghy/TinkerPatchDemo/blob/master/app/tinkerpatch.gradle)里下载该文件。下载完之后，放入 app 目录里，并将刚刚得到的 appKey 填入。
![](https://ww2.sinaimg.cn/large/006tNbRwly1fd2hu0cabqj30tw0cqjtb.jpg)
这里会看到一个变量叫 `appVersion` ，这个东西比较重要，你需要知道的是，这个变量和你的 app 本身的那个 `versionCode` 是没有什么关系的，它只是表示了和补丁相关的版本，你在热修补的时候，可以不升级 `versionCode` 但是一定要升级 `appVersion` ，否则在升级的时候会出现一些 bug ，比如升级到老版本补丁这种情况。
当然，我们现在不需要改变 `appVersion` 就让他为 `1.0.0` 好了。以上弄完了就可以点击右上角的 `Sync Now`了。
### Application
首先新建一个 Application 类作为该项目的 Application ，我取名为 `App` ,之后在 AndroidManifest 里修改。 

``` xml
<application
        android:name=".App"
        ... />
```
你可以直接从[我的源码](https://github.com/abcghy/TinkerPatchDemo/tree/master/app/src/main/java/com/gaohuiyu/tinkerpatchdemo)中，复制该类和 `FetchPatchHandler` 类，具体的用法可以直接参考官方文档，由于我们是快速实现demo就不解释 API 了。那么到这里为止，基本的 sdk 就已经接入完成，接下来就是如何构建补丁包和上传热修补补丁了。

## 打包
不得不说这一步骤官方文档说的很不清晰，我摸索了一下才摸索出来，希望我的配图可以使你了解该过程。
### 老版本
为了分辨老版本和新版本，我们将老版本的 `activity_main.xml` 中的 `Hello World` 改为 `1.0.0老版本`。点击调试按钮，在真机上装上这第一个版本。
![](https://ww2.sinaimg.cn/large/006tNbRwly1fd2jpe5wimj30do05w3yz.jpg)
等待安装完毕，你的手机上应该只有一个空荡荡的 `TextView` 上面写着 `1.0.0老版本`，那么就对了。这时候看 as 中的项目架构，打开 `app/build/bakApk` 里面会有一个文件夹，打开它会看到 debug 文件夹，再里面就是debug的apk和R.txt文件了。
![](https://ww1.sinaimg.cn/large/006tNbRwly1fd2ju7yhtwj30mw0l4gnp.jpg)
接下来，打开之前所说的 `tinkerpatch.gradle`文件，找到

``` gradle
def baseInfo = "app-0115-23-11-20"
def variantName = "debug"
```
这两行，复制 baseinfo 里的字符串，在 bakApk 里新建一个文件夹，名字就叫 `app-0115-23-11-20`，然后再在该文件夹里新建一个文件夹名字叫 `debug`，之后再将刚才得到的两个文件复制进去。如图：
![](https://ww3.sinaimg.cn/large/006tNbRwly1fd2jy7d9aqj30nq0iy40h.jpg)
### 新版本
复制好了之后，就是开始新版本的构建工作了，我们先将 `activity_main.xml` 中的文字改成 `1.0.1 新版本` ，你也可以增加一些其他的特性，在这里我们只是突显出区别。然后将 `tinkerpatch.gradle` 中的 `appVersion` 改为 `1.0.1` 。
打开右边的 gradle 页面，找到 app 里 tinker 中的 `tinkerPatchDebug`，双击运行。
![](https://ww1.sinaimg.cn/large/006tNbRwly1fd2ktp6xe4j30n20madi2.jpg)
我们可以看到 bakApk 里又多了一个 1.0.1的文件夹。tinkerpatch 根据我们提供的旧版本和当时编译的新版本进行判断，然在 `app/build/outputs/tinkerPatch/debug/` 后生成一些补丁包。
![](https://ww3.sinaimg.cn/large/006tNbRwly1fd2lkkogi6j30lk0qkgnt.jpg)
这个补丁包就是我们要上传的补丁包。
### 上传
打开 [我的app](http://tinkerpatch.com/Apps/index) 进入你刚才生成的 app，点击添加app版本，输入老版本的 `appVersion` 也就是 `1.0.0` 。点击生成的版本号，会进入发布补丁的页面，这时候找到 `patch_signed_7zip.apk` 提交就可以了。需要注意的是，有的运营商可能会对 `*.apk` 文件进行一个拦截，所以官方建议修改后缀名，我的运营商并没有干这种事情，所以我就没有改后缀名了。
### 测试
既然补丁包都上传了，我们总得看看成不成功了吧？值得一提的是，tinkerpatch 并不是直接就会生效的，根据 `App` 里的设定，是每隔三个小时查看一次后台，这样比较省流量也不会消耗电量。
为了快速的查看是否成功，我们将该 app 从后台杀掉，再次进入，这个时候他就会访问七牛 CDN 看看有没有补丁包了。获取到了之后并不会立即生效，我们可以关闭屏幕，或者退出 app 再进，就可以看到新的 app `1.0.1新版本`了。
![](https://ww1.sinaimg.cn/large/006tNbRwly1fd2lvz7b4ej30u01hct9g.jpg)

## 总结
这篇文章写了一些简单的接入，目的是为了让广大没有接触过 Tinker 的吃瓜群众更快的上手热修补，可能后续还会写一些深入的文章，比如说如何打 realease 包，如何定制化的实现 tinker 的功能，如何自己搭建 tinker 后端。

