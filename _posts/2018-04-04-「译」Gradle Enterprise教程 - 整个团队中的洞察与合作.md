---
layout:     post
title:      「译」Gradle Enterprise教程 - 整个团队中的洞察与合作
subtitle:   Gradle Enterprise Tutorials - Insights and collaboration for your whole team
date:       2018-04-04
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
原文地址：[https://docs.gradle.com/enterprise/tutorials/debug/](https://docs.gradle.com/enterprise/tutorials/debug/)
查看全部译文：[Gradle Enterprise教程](/tags/##GradleEnterprise教程)

软件开发是一场团队运动，Gradle Enterprise可以令分享和协作更容易。这使得开发者能够突破自我，开发团队能够积极参与解决问题、交付服务，从而节省每个人的时间和金钱。

阅读本教程需要

- 1分钟（只阅读简介）
- 5-10分钟（阅读简介和正文）
- 15-20分钟（阅读简介、正文并动手操作）

## 简介

为了让团队知道哪些构建正在进行中、谁正在执行构建、要花费多少时间以及失败的频率，也为了能够提供构建中每个任务的从配置到执行的方方面面的细节，一个存储所有构建的细节信息的中心化服务是必不可少的。

Gradle Enterprise提供了一项这样的服务，它和Gradle构建工具通过<ruby>构建扫描<rt>Build Scan</rt></ruby>来协作。

### 构建扫描是什么？

- 一个对构建中发生事件的持久化存储
- 永久的、可分享的长链接
- 帮助*开发者*互相帮助、突破自我的资源
- 帮助*构建和发布工程师*加速调试过程的资源

Gradle Enterprise收集你的所有构建的构建扫描，允许你查看、比较、分析、分享以及搜索类似失败构建、执行时间、动态依赖之类的信息。

下面是一些Gradle Enterprise能够帮助团队完成的：

1. 我的构建失败了，我需要帮助——我想把一个长链接分享给一个同事；
2. 我的CI构建在上周和本周之间有何差别？
3. 我收下有多少执行构建的开发者？
4. 我的失败率是多少？开发构建和CI构建有何差别？测试和非测试失败分别占多少比例？

也许最重要的是，你**永远**不需要为了查找问题或者寻求同事帮助而重新运行、重现任何东西。构建扫描包含诊断任何问题所需的一切，且通常（最佳实践）通过自动生成和收集团队执行的每一次构建得到。

## 构建扫描之旅：洞察与协作

### 将构建的细节分享给一个同事

假设你是一个构建工程师或者一个开发者，你注意到gradle输出中包含一些你无法理解的内容，或者你怀疑存在一些反常。过去，你可能把输出复制粘贴到邮件或者聊天窗口中发送，但是其他人可能很难仅仅根据这些信息提供帮助，因为缺少帮助他们理解的**上下文**。

现在，有了构建扫描，你就能轻易将想要讨论的细节放在一个**长链接**上分享出去。

请打开[这个构建扫描](https://enterprise-samples.gradle.com/example/s/share-console-output-you-dont-understand)。

这个链接把我们带到一个构建扫描的顶级页面，这里包含了所有有关此次构建的信息。如果我想要提醒你注意控制台输出的某一行，就可以指定特定的元素并分享这个长链接，如`<LINK_TO_BUILD_SCAN_ITSELF>/console-log#<LINE_NUMBER>`。

欲体验这一点，请访问[这个链接](https://enterprise-samples.gradle.com/example/s/share-console-output-you-dont-understand/console-log#L12)。注意这个链接突出显示了控制台输出的某一行。也可以选择一个范围。

构建扫描的长链接功能使得通过邮件或者消息系统将构建扫描分享给同事变得易如反掌。这还可以简化与工单系统、CI服务器和在线帮助论坛的集成。

另外，你永远不需要重新运行构建以获得帮助或者收集日志、调整日志等级、找到相关输出或者因为错误不稳定而反复重新构建。

### 了解依赖

了解拥有哪些依赖，哪些版本，何时选用新版本，是许多团队时间和金钱支出的重要部分。Gradle Enterprise给予你的工具能够帮助你理解并获取所有项目依赖图的相关信息，无论有多少子项目以及它们有多复杂、多深的嵌套。

#### 检查你的代码是否依赖特定依赖包，如果是，依赖哪个版本

我们想要知道我们是否拥有一个特定依赖。也许有人需要知道某个项目是否使用了它，或者一次代码审计正在进行。考虑[这个构建扫描](https://enterprise-samples.gradle.com/example/s/check-if-your-code-relies-on-a-specific-dependency)。从上个例子中，我们知道了我们可以直接前往我们所感兴趣的地方，即[依赖细节](https://enterprise-samples.gradle.com/example/s/check-if-your-code-relies-on-a-specific-dependency/dependencies?expandAll)。

我们可以看到，整个依赖图的完整信息占了好几页。对于真实世界中的构建可能是好几百页，所以我们可以搜索我们要找的目标。在这个例子中，我们想要知道我们是否依赖（使用）了Google的[Guava库](https://github.com/google/guava/wiki)。在依赖树顶部的搜索框中，键入“Guava”。如果搜索框不可见，只需要点击树顶端的放大镜图标令其可见。

我们可以看到我们确实在很多地方依赖了Guava，以及我们依赖的版本是17.0。你可以看到**为何**这个依赖生效了，这在很多情况下很有用。

#### 了解为何一个特定的第三方库出现在了你的classpath上

在[这个例子](https://enterprise-samples.gradle.com/example/s/understand-why-thirdparty-library-ends-up-on-classpath)中，我们想要知道为什么额外的日志库出现在了构建里，而非我们期望的那些。再回到构建扫描的[dependencies区域](https://enterprise-samples.gradle.com/example/s/understand-why-thirdparty-library-ends-up-on-classpath/dependencies)，搜索“log”（需要点击放大镜图标令其可见）。

我预期的结果是能够看到**slf4j**和**log4j**，但是我所**不期望**的是看到**commons-logging**，但是它出现了！那么问题就来了，为什么？如果你把鼠标悬浮在+commons-logging+上，右侧会出现一个小图标，点击它可以看到细节，如果点击*Required By*，我们就尅看到+commons-logging+是被+apache httpclient+库引入的。

我们可以采取相应的措施，如自定义依赖解析策略。

#### 找出你的项目中哪些依赖使用了动态版本

动态依赖是一个强大的工具，但是它也会为构建引入预料之外的问题。假设我们想要对一个构建进行审计，找出所有动态依赖，了解哪些版本被选中，以及决定是否应当采用更稳定的解析策略。考虑[这个构建](https://enterprise-samples.gradle.com/example/s/what-dependencies-use-dynamic-versions)。打开构建扫描的*Dependencies*区域。现在点击列表顶部展开所有依赖，这看上去像是一个左边带箭头的列表。你现在应该能够看到整个依赖图了。

再次点击列表顶部的搜索图标（放大镜）。你应该看到两个输入框。一个是我们已经了解的，另一个允许我们通过<ruby>解析过滤器<rt>resolution filter</rt></ruby>过滤这棵树。使用它，选择“Resolution: Dynamic version”。

现在，你可以看到五个使用了动态解析的依赖（所有依赖的子集），你还可以看到构建实际采用的版本。

### 了解你的构建使用的Gradle插件

构建工程师经常想知道一个特定的构建使用了哪些插件的那些版本，以及那些插件是Gradle的内部插件，那些是由第三方开发的。构建扫描提供了一种方式，将所有这些信息显示在一个区域内。考虑[这个构建](https://enterprise-samples.gradle.com/example/s/see-all-external-gradle-plugins-applied-to-your-build)。选中左侧导航栏上的*Plugins*。

在这里，一个简单的列表视图显示了所有的项目，我们可以看到所有三类插件，以及外部插件的版本。通过点击这些项，我们可以看到哪些工程使用了哪些插件。

## 动手实验室：Build scans for insights and collaboration

请继续阅读以更进一步。

### 先决条件

欲完成以下步骤，你需要：

1. 一个包含上述源代码的压缩包，用于重现上面的构建扫描。你可以在[这里](https://docs.gradle.com/enterprise/tutorials/debug/enterprise-tutorial-debug.zip)下载。请将它解压到随便什么地方，下文会以`LAB_HOME`指代这个位置。
2. 一个本地安装的JVM。对JVM的要求详见[https://gradle.org/install](https://gradle.org/install)。_注意：本地安装Gradle构建工具是可选而非必须的。所有的动手操作都会通过Gradle Wrapper来调用Gradle构建工具。_
3. 一个Gradle Enterprise服务器的实例。最方便的方法是通过[Gradle Enterprise试用版](https://gradle.com/enterprise/trial)，选择<ruby>在线版<rt>hosted</rt></ruby>，并按照指示和邮件操作。

> NOTE
>
> 在下文中，我们会假设你拥有一个Gradle Enterprise的实例，且位于`https://gradle-enterprise.mycompany.com`。
> 

### 实验室：检查一个构建扫描

在`LAB_HOME/02-browse-a-build-scan`目录中打开终端。

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

随后，请按照压缩包中`README.md`文件的指示进行操作。

在这个实验室中，你会运行一个构建，发布构建扫描到你的Gradle Enterprise实例，然后找出所有的与构建环境相关的基本信息——它们也是团队所希望看到的。例如，由于我们存储了JVM的供应商和版本，我们可以很容易地查看团队是否使用了相同的JVM，其不同的版本，以及谁在使用非标准JVM，以便我们采取措施。

### 实验室：找出公司中的开发者师傅运行了clean任务。

在`LAB_HOME/03-find-out-if-developers-run-clean`目录中打开终端。

随后，请按照压缩包中`README.md`文件的指示进行操作。

在这个实验室中，你可以探索Gradle Enterprise的搜索功能，搜索所有包含“clean”任务的构建。你可以并且应该尝试其他的过滤器，如时间范围、用户名以及构建输出。这些过滤器能够方便地执行常见过滤。

### 实验室：查看所有运行测试的构建

在`LAB_HOME/04-see-all-builds-that-ran-tests`目录中打开终端。

随后，请按照压缩包中`README.md`文件的指示进行操作。

按照第一个实验中的指示，使用文本编辑器修改`build.gradle`中的构建扫描配置，使之指向你自己的Gradle Enterprise实例。

随后，请按照压缩包中`README.md`文件的指示进行操作。

在这个实验室中，你会使用标签作为构建扫描搜索的过滤器。标签允许团队向构建扫描添加任意信息，令**你的**环境、你的特殊需求下的搜索更有针对性，更高效。

## 结论

Gradle Enterprise允许团队更高效地分享构建结果以达到调试和协作的目的。这可以减少你花在调试和重现构建问题上的时间，并增加开发、交付功能特性的时间，这些功能特性是对客户有用的，因此对于增加你的竞争力至关重要。结果是极大地节省了时间——这不仅仅是省钱，更重要的是将你最珍贵的资源从消极的、乏味的、不产生价值的工作中解放出来。