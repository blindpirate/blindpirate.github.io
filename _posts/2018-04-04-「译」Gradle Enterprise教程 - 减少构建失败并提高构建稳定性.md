---
layout:     post
title:      「译」Gradle Enterprise教程 - 减少构建失败并提高构建稳定性
subtitle:   Gradle Enterprise Tutorials - Reduce build failures and increase build reliability
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
原文地址：[https://docs.gradle.com/enterprise/tutorials/build-reliability/](https://docs.gradle.com/enterprise/tutorials/build-reliability/)
查看全部译文：[Gradle Enterprise教程](/tags/##GradleEnterprise教程)

网络异常、构建脚本有误、各种环境因素都会使你的构建莫名其妙地失败，且难以重现，最终拖慢甚至阻碍你的<ruby>应用部署生产线<rt>application deployment production line</rt></ruby>。Gradle Enterprise能够让团队迅速地发现并解决这些问题。

阅读本教程需要

- 1分钟（只阅读简介）
- 5-10分钟（阅读简介和正文）
- 15-20分钟（阅读简介、正文并动手操作）

## 简介

在扩展规模时，团队经常被莫名其妙的构建失败所困扰：特定类型的构建失败是间歇性的、可重现的吗？它们是由于某些未知的环境因素改变所引起的吗？这些因素是构建失败的根源吗？

Gradle Enterprise能够使团队用比其他方式更快的速度发现并解决这些失败的测试。无需重新运行构建，修改日志等级，等待错误再次发生，然后在通讯工具、工单、邮件或者日志收集系统之间来回复制粘贴大段的日志，<ruby>构建扫描<rt>build scan</rt></ruby>能够提供更好的发现失败构建原因的方法。

部署Gradle Enterprise的最佳实践是令*所有的*构建将构建扫描发送到中心化的Gradle Enterprise服务器上。CI服务器确实保存了失败的CI构建，但是通常情况下，团队是看不到开发者本地构建的失败的。若所有的构建都被中心化地收集，团队就可以迅速发现特定的开发者失败构建或者CI失败构建，并将其与最近或最相似的成功构建相比较，寻找变更之处。Gradle Enterprise的用户界面被特别优化，以允许构建/发布团队成员或者开发团队自己迅速地发现、分析错误。

团队可以采取激进的方案配置构建：令所有的构建发布构建扫描。由于Gradle Enterprise的服务器（包含一个构建扫描服务和一个构建缓存服务）的设计目标之一就是令最大的团队能轻易扩容而不受单点故障影响。如果Gradle Enterprise的构建扫描服务（或者Gradle Enterprise远程构建缓存服务）宕机、中断或者运行缓慢，构建能正常进行而不受影响。Gradle构建工具支持等待服务恢复后再发送上一次构建扫描。

通过设立常规的、系统性的错误监控流程，以及使用构建扫描量化持续变化的构建，团队就能持续集成越来越复杂的代码；遇到的无法解释的错误、不稳定测试会越来越少，也会花更少的时间在寻找和修复莫名其妙的失败上，从而能够将宝贵的时间节省下来产出价值。

## 减少构建失败和增加构建稳定性之旅

### 在测试失败时寻求帮助

首先，让我们点击[这里](https://enterprise-samples.gradle.com/example/s/pull-in-help-for-an-unexpectedly-failing-test)，查看Gradle Enterprise服务器上的构建扫描结果。

_注意，本教程中的链接包含的构建扫描ID是经过命名的，通常情况下的构建扫描ID看上去更像是机器生成的_。

你可以看到失败构建的总结信息（顶部的红色“x”），其中，`:test`任务失败了。在左侧导航栏上，点击*Tests*。

我们可以看到其中有一些测试（准确的说是25个），它们都来自同一个项目，也就是刚刚失败的那个。我们可以点击顶部的`1 failed`链接，然后页面就会变成只显示失败的那一行测试。请试一下。

现在点击失败的单行测试，你会看到失败的详情。失败是一个Java的`AssertionError`，你还可以看到<ruby>栈轨迹<rt>stacktrace</rt></ruby>。

点击展开栈轨迹，注意浏览器中显示的URL。你可以看到它是`https://enterprise-samples.gradle.com/example/s/pull-in-help-for-an-unexpectedly-failing-test/tests/a5a6sydx5wc2o?openStackTraces=WzBd`。

这是一个很长的链接，在构建扫描ID后面，我们可以看到，这是_tests_区域，而我们正在浏览的是由ID标记的某个特定测试，以及它的栈轨迹。

现在我们可以将这个链接分享给相关人员了。也许是最后修改这个文件，添加了这个测试的开发者？

### 查看所有项目的本地失败测试

下面的例子建立在上一个的基础上。在上一个例子中，我们看到了如何在一个<ruby>单项目<rt>single project</rt></ruby>中显示并过滤错误结果。
Gradle Enterprise的构建扫描服务还能将一个<ruby>多项目<rt>multi-project</rt></ruby>中的所有测试聚合起来，使得它们能够以同样轻松的方式被处理。

让我们看一看[这里](https://enterprise-samples.gradle.com/example/s/see-all-locally-failing-tests-across-all-projects)的构建扫描。

同样，点击左侧导航栏上的`Tests`。

这一次，我们看到了100个测试，它们来自于5个不同的项目。如果你移动滚动条查看全部的100行测试结果，你会看到一些失败，顶部`14 failed`的红色文字告诉我们有多少个测试失败了。

点击列表顶部的放大镜图标。

弹出了一个搜索框。我们现在可以像之前的例子一样过滤，只显示失败的测试，也可以搜索特定文本以过滤行。在搜索框内键入`get`。

我们发现，100行中的12个测试被匹配到了。清空搜索框，回到100行测试的状态。现在我们可以点击`5 projects`链接，看到2个<ruby>子项目<rt>sub-project</rt></ruby>没有错误，3个有错误。

和上个例子一样，我们可以点击`14 failed`，只看失败的测试，或者深入一行查看错误细节和栈轨迹。

### 确定动态的改变的依赖是否破坏了构建

最常见的构建失败之一就是动态解析的上游二进制依赖改变了。

Gradle Enterprise的对比功能对解决此类失败有奇效。

点击[这里](https://enterprise-samples.gradle.com/scans)查看Gradle Enterprise的扫描列表。

最近，我们注意到`08-determine-if-changed-dynamic-dependency-broke-the-build`项目失败了。

让我们搜索该项目的构建——在构建扫描列表显示的标题栏中的*Project*栏里键入项目名的起始字符`08`。

现在我们找到了两个结果，一个失败的和一个成功的。我们可以看到两次构建所花时间相同，而且几乎是同时开始的，所以我们觉得对比两次构建，看看二者之间的差异可能有助于我们寻找失败原因。

将鼠标移到第一行的最左侧，点击设置比较的`A`面，然后在第二行的最左侧选择比较的`B`面。

你可以看到这两行的背景都变灰了，最右侧出现了一个`Compare`图标。

点击`Compare`。

现在你应该能看到构建扫描差异视图。左侧导航栏用蓝点显示了存在不同的区域。

我们注意到依赖有所差异，所以点击左侧导航栏上的`Dependencies`。

我们看到确实存在一处不同，失败构建中的`com.acme.utils`被解析到了1.6，而成功构建中的对应版本是1.5。

现在我们可以合理地怀疑这就是构建失败的根本原因。

### 调查为何你的项目无法在同事的机器上编译

最常见、令人沮丧而且费时的失败之一就是“在我这里一切正常但在你那里构建失败，可是它们看起来一模一样”。

通常，这种情况会导致花费大量时间寻找原因——却一无所获。让我们点击[这里](https://enterprise-samples.gradle.com/example/s/jdk-outdated)查看失败的构建扫描。

现在让我们点击[这里](https://enterprise-samples.gradle.com/example/s/jdk-latest)查看成功的构建扫描。

如果它们之间的差异不明显，我们可以使用Gradle Enterprise的构建比较功能寻找失败的根本原因。

和之前一样，我们点击[这里](https://enterprise-samples.gradle.com/scans)打开构建列表。

在顶部的项目过滤栏中键入`ratpack`只显示`ratpack`项目的扫描结果。

在列表底部，我们可以看到两个由用户`Mark`和`Rene`提交的结果。和上一个例子一样，我们在最左侧选中它们，进行一次比较，然后点击顶部最右侧灰色区域中的`Compare`cue，以显示对比视图。

这一次，我们在左侧的导航栏上看到*Infrastructure*上有一个代表差异的蓝点，因此这里可能有我们所感兴趣的东西。

我们看到了两次构建使用的JDK版本不同，因此，他们两人使用的JDK的供应商不同。我们知道，我们只支持Oracle JDK，所以另外一次构建会失败。我们可以采取合适的措施以保证不再使用其他供应商的JDK，这样，这个问题就解决了。

### 当本地构建失败时寻求帮助

你是否曾苦苦挣扎，寻找构建失败的原因，最终却发现原因是一些未提交、未测试的代码变更？

在接下来的例子中我们会看到，Gradle Enterprise的构建扫描具有可扩展性：在构建扫描中精心设计的自定义数据可以提供额外信息，使得错误排查的过程更快、更高效。

我们会在讲述定制化的教程中看到如何编写此类扩展信息。

[这里](https://enterprise-samples.gradle.com/example/s/reach-out-for-help-when-local-build-fails)是一个错误，有人把它发给我们以寻求帮助。请打开它。

我们也许会上钩，深入研究测试中的错误，不过，我们很快注意到在构建扫描的标题栏上有一个链接`Diff`，因此，在开始深入研究之前，我们先看一眼这个链接是什么——它是一个Github *gist*。


我们可以分辨出，在当前失败的测试中，存在一些本地未提交的代码，因此我们可能会告诉提交者这可能是失败的原因，甚至指出问题在代码的哪一行。

这样的扩展能够极大地缩短调试时间，使团队更快地发现并修复引起构建失败的原因。

## 动手实验室：减少构建失败，提高构建稳定性

请继续阅读以更进一步。

### 先决条件

欲完成以下步骤，你需要：

1. 一个包含上述源代码的压缩包，用于重现上面的构建扫描。你可以在[这里](https://docs.gradle.com/enterprise/tutorials/build-reliability/enterprise-tutorial-build-reliability.zip)下载。请将它解压到随便什么地方，下文会以`LAB_HOME`指代这个位置。
2. 一个本地安装的JVM。对JVM的要求详见[https://gradle.org/install](https://gradle.org/install)。_注意：本地安装Gradle构建工具是可选而非必须的。所有的动手操作都会通过Gradle Wrapper来调用Gradle构建工具。_
3. 一个Gradle Enterprise服务器的实例。最方便的方法是通过[Gradle Enterprise试用版](https://gradle.com/enterprise/trial)，选择<ruby>在线版<rt>hosted</rt></ruby>，并按照指示和邮件操作。

> NOTE
>
> 在下文中，我们会假设你拥有一个Gradle Enterprise的实例，且位于`https://gradle-enterprise.mycompany.com`。
> 

### 实验室：添加一些有成功有失败的测试

在`LAB_HOME/04-see-all-builds-that-ran-tests`目录中打开终端。

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

README会指引你运行一次构建，第一次不运行测试任务，第二次运行测试任务。

一个方便的TAG扩展会在构建扫描的标题栏上醒目地告诉你，测试是否运行。我们会看到如何编写这些标签

You observe this by using a convenience TAG extension that indicates very prominently at the build scan header whether tests were run or not. We will look at how such tags are authored in the extensibility tutorial.

在第二次构建扫描中，点击左边导航栏中的`Tests`，我们可以看到只有一个测试，且它成功了。

修改测试源代码，增加4个测试，如

`vi src/test/java/HelloTest.java`

将测试文件替换为：

```
import org.junit.Test;
import static org.junit.Assert.fail;

public class HelloTest {
  @Test
  public void hello() {
    // Pass
  }
  @Test
  public void foo() {
    // Pass
  }
  @Test
  public void bar() {
    // Pass
  }
  @Test
  public void baz() {
    // Fail
    fail("this is the first simulated test failure");
  }
  @Test
  public void fred() {
    // Fail
    fail("this is the second simulated test failure");
  }
}
```

查看产生的构建扫描的*`Tests`*栏，你会看到增加的4个测试，以及看到两个模拟失败的细节。

## 结论

Gradle Enterprise大大简化了团队寻找和修复构建失败原因的过程。

通过系统化地收集团队构建活动的细节（CI和开发者构建），团队就可以在实践中系统化地跟踪代码和构建环境的问题以及趋势，从而提高构建的稳定性，减少构建失败的的时间和频率。