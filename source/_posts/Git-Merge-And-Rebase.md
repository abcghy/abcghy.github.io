---
title: Git Merge 和 Rebase 的用法
date: 2017-04-20
tags:
    - git
---
最近在找工作，总是会问到 merge 和 rebase 的区别，今天就写一篇文章整理一下 rebase 的用法。

这是一篇新手向的 rebase 初级用法教学，目的在于手把手的教会大家如何使 graph 变成线性，也可以在代码审查的时候，更容易理解大家所做的工作。 
<!-- more -->
## 理论
关于rebase的用法，已经有很多文章了，我在这里就不班门弄斧了。例如：
[Merging vs. Rebasing - Atlassian tutorials](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

## 实践
首先我们假设有两个程序员进行协同工作，一个是你自己，一个是小高。

### 初始化内容
本地创建一个 git 仓库。
``` bash
git init
```
默认是 master 分支。

我们创建一个文件`README.md`并往里面添加一些内容：
``` markdown
# Git Rebase Demo
This is a rebase demo.
```
之后将代码提交，以保证初始化。
``` bash
git add .
git commit -m "init"
```
此时 SourceTree 上看起来是这样的：![初始化时的样子](https://ws2.sinaimg.cn/large/006tNbRwly1fwnvwlcu9tj31kw10ewvp.jpg)

接下来我和小高都会根据 master 分支来完成自己的任务。
### 我的分支
首先新建我的分支
``` bash
git checkout -b myname
```
然后做自己的工作，比如添加一段话：
``` markdown
# Git Rebase Demo
This is a rebase demo.

## myname
This is the work that I done.
```
然后保存提交
``` bash
git add .
git commit -m "myname's work"
```
这样，我们就完成了我们的工作。

接下来按照同样的操作，添加一个小高的分支
### 模拟小高的操作
回到 master 分支，新建小高分支
``` bash
git checkout master
git checkout -b xiaogao
```
然后同样的添加一段话
``` markdown
# Git Rebase Demo
This is a rebase demo.

## xiaogao
This is xiaogao's work.
```
接着保存小高的工作成果
``` bash
git add .
git commit -m "xiaogao's work"
```

现在， SourceTree 上的图是这样的： ![两个人都完成工作之后](https://ws4.sinaimg.cn/large/006tNbRwly1fwnvzbwqx7j31kw10e7mg.jpg)

### 小高先提交自己的代码
由于小高速度比较快，所以他先提交了自己的代码。因为他是直接从公共分支(master)checkout出来的，并且后来master分支也没有合并新的代码，所以小高可以直接使用 merge 的方式讲代码合并进 master 分支。
我们先切换成 master 分支。
``` bash
git checkout master
git merge xiaogao
```
此时为这样：![在 master 分支中，合并小高的分支](https://ws3.sinaimg.cn/large/006tNbRwly1fwnw2l0ug9j31kw10eas2.jpg)
master 和小高分支同步了。

### 关键的 rebase
我们大可以用 merge 来将自己的工作合并到 master 分支当中（当然得处理冲突），但是这样的话， graph 就会不清晰。如果用 rebase 的话，graph 就会呈现为线性，更加容易理解。

#### 尝试用 merge
``` bash
git merge myname

Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```
可以看出，发生了冲突：
``` markdown
# Git Rebase Demo
This is a rebase demo.

<<<<<<< HEAD
## xiaogao
This is xiaogao's work.
=======
## myname
This is the work that I done.
>>>>>>> myname
```
这篇文章不详细说明如何解决冲突，只是简单处理当前冲突。

我们将 `<<< === >>>` 这些符号以及后面的文字都去掉，变成这样：
``` markdown
# Git Rebase Demo
This is a rebase demo.

## xiaogao
This is xiaogao's work.

## myname
This is the work that I done.
```
这样的话，两个人的工作成果都保存了下来，小高的在前，你的在后。于是我们保存一下处理冲突之后的代码
``` bash
git add .
git commit -m "merge myname to master"
```
使用 merge 是不是太简单了，只需要处理冲突之后保存就好，省去了很多不必要的麻烦。但是这个时候我们看看 SourceTree: ![merge 之后](https://ws1.sinaimg.cn/large/006tNbRwly1fwnwkplzwuj31kw10eh4j.jpg)
但是 graph 当中，有一个从初始化之后进行的分叉，最后合到 master 分支之后又有一次 commit 信息，如果只是两人协作或者只有两个子分支协作倒是不是很复杂，但是如果你是和四五个人同时协作，并且每个人都有自己的 feature 分支、 hotfix 分支，总共十几二十个分支的话，那么 merge 之后的 graph 界面只能说是惨不忍睹！

下面贴一个很久之前工作团队只使用 merge 的一个例子：![多人合作就很复杂](https://ws1.sinaimg.cn/large/006tNbRwly1fwnyrh0jthj31kw0wxwsn.jpg)从这样的提交历史当中，已经很难看出谁加了什么功能，graph 也是杂乱无章。 

#### 用今天的主角，rebase
我们重新回滚到merge之前的代码状态。![回滚的操作](https://ws4.sinaimg.cn/large/006tNbRwly1fwnz4g3h73j31kw0x4qtl.jpg)
选择 hard，点击确定就回到了 merge 之前的状态。

rebase 首先需要切换到 myname 分支下，接着 rebase
``` bash
git checkout myname
git rebase master
```
毫无疑问，还是会出现冲突，按照刚才解决冲突的方法再执行一遍。然后保存
``` bash
git add .
git rebase --continue
```
注意这里是 `git rebase --continue` 而不是 commit，完成之后我们会看到 SourceTree 里我们的 graph 就变得很清爽了：![rebase后的 graph](https://ws4.sinaimg.cn/large/006tNbRwly1fwnza2d8i0j31kw0yu4kq.jpg)

最后，你需要回到 master 分支，merge myname 分支，并且清理其他无用分支。
``` bash
git checkout master
git merge myname
git branch -d myname
git branch -d xiaogao
```
![最终效果](https://ws2.sinaimg.cn/large/006tNbRwly1fwnzine1lcj31kw0yu1em.jpg)这里的 graph 就呈现线性的状态，非常的清爽。相比 merge 操作，少了一个分叉，也少了一个 merge 的 commit。沿着 graph 一个一个往下观察，可以清晰的看到每个人的每一步操作。

#### 注意
merge 和 rebase 虽然都是将一个分支的代码合并进另一个分支，但是他们的理念不同，所以所在分支也不一样，操作的时候需要注意。

切记，rebase 操作不要在公共分支上进行。

## 总结
本篇文章虽然简单，但是熟练之后运用到自己的项目中会非常的有用。前半部分先告诉大家如何模拟多人协作，后面告诉大家 merge 和 rebase 的区别。

我建议大家不只是单纯的看一遍文章，就觉得理解了rebase的基本用法，还是跟着文章敲一遍，甚至是自己动手做个小demo比较好。
