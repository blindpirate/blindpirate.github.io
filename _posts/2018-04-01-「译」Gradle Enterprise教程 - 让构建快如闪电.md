---
layout:     post
title:      「译」Gradle Enterprise教程 - 让构建快如闪电
subtitle:   Gradle Enterprise Tutorials - Keeping builds fast
date:       2018-04-01
author:     Bo 
catalog: true
tags:
    - 中文
    - 译文
    - 技术
    - GradleEnterprise
    - 教程
    - GradleEnterprise教程
---

> 本文是Gradle Enterprise官方教程之一的译文。
原文地址：[https://docs.gradle.com/enterprise/tutorials/performance-build-scans/](https://docs.gradle.com/enterprise/tutorials/performance-build-scans/)
查看全部译文：[Gradle Enterprise教程](/tags/##GradleEnterprise教程)

优化构建性能需要长期的努力，不能一蹴而就。Gradle Enterprise所做的一切努力都是为了让你在代码和环境改变时，仍然拥有尽可能快的构建。

阅读本教程需要

- 1分钟（只阅读简介）
- 5-10分钟（阅读简介和正文）
- 15-20分钟（阅读简介、正文并动手操作）

## 简介

快速的构建是每个人追求的目标。对开发者而言，更快的构建能够产出更多的代码，提高功能特性交付的敏捷度和速度。对CI而言，更快的构建减少了反馈时间，能够使持续集成更好、更高效。

Gradle Enterprise的<ruby>远程构建缓存<rt>remote build cache</rt></ruby>是一个强大的工具，拥有它，团队就可以加速所有的构建。你可以通过阅读我们的[构建缓存教程](https://docs.gradle.com/enterprise/explainer-tutorials/caching)（[译文](/2018/03/31/译-Gradle-Enterprise教程-启用缓存以加速构建/)）来了解它。

本教程主要考虑两方面的构建性能优化。第一个方面是发现并消除那些构建缓存之外的、影响构建时间和内存消耗的主要因素——如二进制依赖的下载速度慢、构建脚本处理等。

另一个就是使用和优化那些可以利用缓存的任务。每个任务都可以决定是否参与缓存，前提是仔细分析和测试缓存的正确性，评估缓存的效率。在某些情况下，缓存慢于重新构建，因此缓存并不是需要无条件开启的。
自定义插件、使用了<ruby>注解和源代码生成处理器<rt>annotation and source code generation processors</rt></ruby>，以及其他特殊的原因都会使得任务无法高效地被缓存。

Gradle Enterprise的<ruby>构建扫描<rt>build scan</rt></ruby>是解决这两个方面性能问题的关键工具，它不仅允许你的构建*变得*迅速，而且允许你的构建*保持*迅速。

我们正在努力开发一种新的工具，利用构建扫描获取的数据检测<ruby>构建时间的回归问题<rt>build time regressions</rt></ruby>。它是解决所有团队的终极问题——永远让构建保持最快——的关键工具。

## 让构建保持迅速

### 优化缓存之外的构建性能

#### 了解本地和CI构建的时间差异

有时，了解为何CI的平均构建时间比本机开发的构建时间短（或长）能够传达很多信息。如果二者差异巨大，可能说明团队需要增加（或者减少）这一类构建的硬件资源，或规划预算。

对于示例项目，下面是一个寻找这种差异的快速方法。打开构建扫描的列表页[这里](https://enterprise-samples.gradle.com/scans)。

在tag中输入`CI`，project中输入`gradle`，点击搜索。然后将搜索结果按照时间倒序排列。我们可以看到5次构建，CI的用时大约是10秒。

将tag改成`local`，重复上述搜索，可以看到，开发者机器上的构建大约花了23秒。

基于这一点，我们可以知道，开发者的构建比CI构建慢了2.5倍，该信息可以用于辅助预算决策：需要升级开发者笔记本的硬件配置了。

#### 加速任意构建

大型的、复杂的构建总有提高构建速度的空间。任务列表视图能够帮助团队在大量任务中识别耗时最多的那些。这样，在重构构建时，团队就能迅速聚焦于那些能够带来最大收益的任务上。

打开[这个](https://enterprise-samples.gradle.com/example/s/make-a-build-faster)构建扫描。

打开左侧导航条上的*Timeline*视图。

顶端的图标可以立即告诉我们，我们有两个耗时很长的测试任务处于并行执行阶段，因此至少我们不能把它们改成串行执行。

如果你查看进度条下方的任务列表，在最右侧有一个下拉列表：*Order:*，默认的排序是执行顺序。将其改成*Longest*。

现在我们可以看到，优化这两个10秒的测试任务能够带来最大的收益。

#### 调查对<ruby>设定阶段时间<rt>configuration time</rt></ruby>影响最大的因素

在所有任务执行前，Gradle构建工具对构建进行<ruby>设定<rt>configuring</rt></ruby>的阶段可能消耗大量的时间。你可能使用了多个来源（Gradle，内部开发，以及第三方）的几十个甚至上百个插件，且随着时间增长，越来越多的新版插件可能出现在构建中。

Gradle Enterprise帮助你发现设定阶段耗时的数量和来源，从而帮助你减少这个阶段的耗时。

打开[这个构建扫描](https://enterprise-samples.gradle.com/example/s/investigate-what-has-the-biggest-impact-on-your-configuration-time)。

打开左侧导航栏上的*Performance*。选中右侧的*Configuration*选项卡。

现在你可以看到所有使用中的构建脚本和插件，按照消耗时间倒序排列。这可以帮助你决定，耗时最多的那些是否必要，以及是否能够通过重构减少消耗的时间。

#### 决定你的构建是否需要更多的内存

听上去可能令人吃惊，但是分配给Gradle构建工具JVM进程的内存数量不合适是构建性能低下的一个常见原因。

构建扫描可以检测到这一点，例如[这里](https://enterprise-samples.gradle.com/example/s/determine-if-your-build-needs-more-memory)。

打开左侧导航栏上的*Performance*。

检查页面上的`Peak heap memory usage`和`Total garbage collection time`。你会迅速得出结论，我们没有给JVM足够的内存，几乎一半的构建时间花费在了JVM的垃圾回收上。

现在选择左侧导航栏的*Infrastructure*，可以看到`Max JVM memory heap size`被设为了60MB。随后，你就可以逐渐调整JVM的内存大小，寻找令此构建性能最佳的值。

#### 了解依赖解析消耗的时间

Gradle能够自动创建本地的二进制缓存，用于存储从上游下载的二进制依赖。然而，网络的延迟，仓库缓存节点的可用性，以及可能出现在你的构建中的网络问题可能导致依赖解析阶段严重的构建性能下降。

欲从Gradle Enterprise处了解此问题，请打开[这个](https://enterprise-samples.gradle.com/example/s/determine-how-much-time-was-spent-resolving-dependencies)构建扫描。

选择左侧导航栏的*Performance*，然后选择顶端的*Network activity*选项卡。

可以看到，我们下载了23个文件，消耗了总构建时间中的5秒，带宽是706 kB/s。我们可能需要更好的网络连接，或者尝试将仓库缓存和构建放在一起。

### 优化任务缓存

这一节中的例子可以展示Gradle Enterprise是如何使用构建扫描来优化任务缓存的效率的。

#### 分析构建缓存的命中率

Gradle Enterprise能够展示构建缓存的整体统计数据，该信息有助于了解当前缓存的效率，以及下一步优化的目标。

请看[这个](https://enterprise-samples.gradle.com/example/s/analyze-build-cache-hit-rate)构建扫描。

选择左侧导航栏上的*Performance*，然后选择顶部的*Build cache*选项卡。

这里展示了本次构建中，本地和远程缓存的统计数据一览。我们可以看到两种缓存的命中数和未命中数；我们可以看到缓存的大小，以及存储目录；我们还可以看到从缓存中获取数据所消耗的时间。

现在点击顶部的*Task execution*选项卡。

找到视图中的`Not cacheable`并点击该链接。它显示，本次构件中，只有`clean`任务是不可缓存的，因为该任务并没有开启输出缓存——这是合理的，因为清理任务不应该被缓存。

#### 决定接下来缓存哪些任务

发现并处理任务缓存应当逐步、持续地进行。使用本节的方法来识别，接下来应该仔细检查并改造哪些任务，才能获得最大的收益。

请看[这个](https://enterprise-samples.gradle.com/example/s/determine-what-tasks-to-make-cacheable-next)构建扫描。

欲了解当前运行时间的最长的不可缓存任务，从左侧导航栏中选择*Timeline*，然后点击主页面的放大镜图标。

点击输入栏`Task output cacheability`，选择弹出的`Not cacheable: Any reason`。

然后找到主页面最右侧的`Order`下拉框，选择`最长`。

现在我们可以看到当前无法被缓存的任务：`task1`和`task3`是其中运行时间最长的两个。

我们可以进一步点击`task1`来获取更多信息：在这个例子中，我们可以看到这些任务完全没有开启输出缓存。我们可以通过向相关的任务类型中添加注解来开启输出缓存。`task3`没有定义任何输出，因此也无法被缓存。

#### 调查意外的构建缓存未命中的原因

最常见的调试缓存未命中的场景大概是，我们认为缓存应该命中，但是却没有。

请看[这个](https://enterprise-samples.gradle.com/example/s/investigate-why-you-are-getting-an%80%93unexpected-build-cache-miss)构建扫描，点击左侧导航栏的*Timeline*。

我们可以看到一些任务有`FROM-CACHE`标记，但是大多数没有这样的标记。我们尤其想知道，为什么`compileJava`任务是缓存未命中状态——我们认为它应该命中。

Gradle Enterprise有一个非常强大的功能，称为“<ruby>相邻扫描<rt>Adjacent Scans</rt></ruby>”。它允许我们查看并对比相同构建前一次和后一次的扫描结果。

在用户界面顶端，你可以看到一个带逆时针箭头的钟表图标。请点击它。

你可以看到一个3秒前的“相邻扫描”。选择它。

这就是那次我们认为应当产生缓存数据的构建，所以让我们调查一下。

我们可以看到一个名为`generateSomeSource`的任务，我们可以推断出，这个任务在每次构建中都会生成一些不同的代码，所以我们需要知道为什么，以及这样做是否必要，尤其是在缓存打开、开发者的构建本可以从缓存中受益的情况下。

## 动手操作：让构建保持迅速

请继续阅读以更进一步。

### 先决条件

欲完成以下步骤，你需要：

1. 一个包含上述源代码的压缩包，用于重现上面的构建扫描。你可以在[这里](https://docs.gradle.com/enterprise/tutorials/caching/enterprise-tutorial-performance-build-scans.zip)下载。请将它解压到随便什么地方，下文会以`LAB_HOME`指代这个位置。
2. 一个本地安装的JVM。对JVM的要求详见[https://gradle.org/install](https://gradle.org/install)。_注意：本地安装Gradle构建工具是可选而非必须的。所有的动手操作都会通过Gradle Wrapper来调用Gradle构建工具。_
3. 一个Gradle Enterprise服务器的实例。最方便的方法是通过[Gradle Enterprise试用版](https://gradle.com/enterprise/trial)，选择<ruby>在线版<rt>hosted</rt></ruby>，并按照指示和邮件操作。

> NOTE
>
> 在下文中，我们会假设你拥有一个Gradle Enterprise的实例，且位于`https://gradle-enterprise.mycompany.com`。
> 

### 实验室：寻找潜在的性能杀手

在`LAB_HOME/05-find-potential-performance-killers`目录中打开终端。

使用文本编辑器修改`build.gradle`中的构建扫描配置，使之指向你自己的Gradle Enterprise实例：

将：

{% highlight groovy %}
buildScan {
  server = '<<your Gradle Enterprise instance>>'
  publishAlways()
}
{% endhighlight %}

替换为

{% highlight groovy %}
buildScan {
  server = 'https://gradle-enterprise.mycompany.com' // <-- your personal Gradle Enterprise instance
  publishAlways()
}
{% endhighlight %}

在这个实验室里，通过创建一个构建并应用一些通用的性能优化手段，你会获得一些实践经验。

欲完成所有的实验操作，请按照压缩包中`README.md`文件的指示进行操作。

### 实验室：决定如何增加给定构建的<ruby>可缓存性<rt>cache-ability</rt></ruby>

在`LAB_HOME/06-increase-the-cacheability-of-the-build`目录中打开终端。

使用文本编辑器修改`build.gradle`中的构建扫描配置，使之指向你自己的Gradle Enterprise实例：

将：

{% highlight groovy %}
buildScan {
  server = '<<your Gradle Enterprise instance>>'
  publishAlways()
}
{% endhighlight %}

替换为

{% highlight groovy %}
buildScan {
  server = 'https://gradle-enterprise.mycompany.com' // <-- your personal Gradle Enterprise instance
  publishAlways()
}
{% endhighlight %}

在这个实验室里，通过创建一个构建并应用一些通用的性能优化手段，你会获得一些实践经验。

欲完成所有的实验操作，请按照压缩包中`README.md`文件的指示进行操作。

## 结论

优化构建性能需要长期的努力，不能一蹴而就。调优构建并不能让构建永远保持最快。若不认真处理，频繁的代码、环境改变会导致<ruby>性能回归问题<rt>performance regressions</rt></ruby>。毕竟，<ruby>熵永远在增大<rt>Entropy always wins</rt></ruby>。

Gradle Enterprise提供了完整的工具集，能够帮助你发现、修复导致两类性能回归问题的的根源，为性能优化实践奠定坚实的基础。