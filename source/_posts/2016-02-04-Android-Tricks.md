---
title: Android-Tricks
date: 2016-02-04 18:45:57
tags:
---
Material design 早在两年前就已经被 Google 推出，当时我就对他非常喜爱。富有真实感的阴影和炫酷的动画，现在我正式进入 Android 开发这个领域，自然对这一块有着浓厚的兴趣。

首先我找到了这个 github 网页:[Material-Animations](https://github.com/lgvalle/Material-Animations "lgvalle/Material-Animations")。首先，这些炫酷的动画只能在5.0之后实现，在其次，这个库只有对官方给出动画的实现，通过里面的sample我们可以更好的了解 Material Animations。

那么，这个项目就是我接下来要学习的内容，对其中我不懂的地方，我都会记下来。所以说，这将是我的一个笔记，而不是供别人学习的教程。事实上，这个项目中有一些比较详细的讲解，大家可以仔细阅读。我在这里只是对它的源码进行解读（幸运的是，源码并不是很难）。

首先看到，最低的sdk版本为21，那么之前的设备肯定就是用不上了。

![](image-1.png)
{% asset_img image-1.png minSdkVersion %}

---
未完待续