---
layout:     post
title:      「译」Gradle Enterprise教程 - 启用缓存以加速构建
subtitle:   Gradle Enterprise Tutorials - Caching for faster builds
date:       2018-03-31
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
原文地址：[https://docs.gradle.com/enterprise/tutorials/caching](https://docs.gradle.com/enterprise/tutorials/caching)
查看全部译文：[Gradle Enterprise教程](/tags/##GradleEnterprise教程)

Gradle Enterprise的<ruby>构建缓存<rt>build cache</rt></ruby>可以使构建更快——无论是对开发者还是持续集成服务器（CI）而言。

阅读本教程需要

- 1分钟（只阅读简介）
- 5-10分钟（阅读简介和正文）
- 15-20分钟（阅读简介、正文并动手操作）

## 简介：避免重复构建

逻辑上，Gradle有三层复用机制，避免潜在的昂贵任务重复执行：

1. 对工作空间中任务的输出执行Up-to-date检查。
2. 在<ruby>本地缓存<rt>local cache</rt></ruby>中查找任务的输出。
3. 在<ruby>远程缓存<rt>remote cache</rt></ruby>中查找任务的输出，如[Gradle Enterprise](https://gradle.com/build-cache)自带的远程缓存。


![3层缓存 - 本图是SVG格式，部分浏览器可能支持有限](/img/caching-3-layers.svg)

这三个等级的支持能够在三种不同的目标场景下加速你的构建：

1. 在两次连续运行的Gradle构建之间，通常不会有太多东西发生改变。Gradle的增量构建功能能够只执行和上次构建相比<ruby>过时的<rt>not up-to-date</rt></ruby>任务。
2. 典型情况下，开发者在不同的分支上维护着不同的工作空间，以执行逻辑上独立的任务。本地缓存使得输出结果能被迅速地跨工作空间复用，而无需访问网络。
3. 许多时候，CI节点和开发者运行的任务相同，工作空间内的变更也相同。远程缓存允许输出结果在用户和构建服务器上被复用，确保整个团队无需多次重复运行相同的构建。

## Gradle Enterprise构建缓存之旅

接下来，让我们深入探究远程缓存是如何加速你的构建的。

### <ruby>全新构建<rt>Clean build</rt></ruby>

考虑一个非常简单的Java示例项目。[这个页面](https://enterprise-samples.gradle.com/s/enhrtnwl2jah6/timeline)是一次<ruby>构建扫描<rt>build scan</rt></ruby>（对一次Gradle构建过程的完整记录）的时间线。这次构建是一次全新构建，它运行了工作空间内的所有任务，且没有从缓存中获取任何数据。我们可以通过观察任务的时间线分辨出这一点：没有任何一个任务的状态是`FROM-CACHE`。这个人为构造的示例项目花了大约8秒来执行所有的任务，其中大部分时间花在了编译Java源代码上。

点击任务`:compileJava`可以看到，构建缓存<ruby>未命中<rt>miss</rt></ruby>，从而发生了一次<ruby>储存<rt>store</rt></ruby>过程。我们执行了重新编译，并将结果储存到了远程缓存中。

通过构建扫描的[构建缓存性能页面](https://enterprise-samples.gradle.com/s/enhrtnwl2jah6/performance/buildCache)，我们还可以发现，这次构建虽然没有能够命中缓存，却存储了3个输出结果到远程缓存中，从而使得后续的构建受益。

我们知道，本次构建的几乎所有时间都花在了编译Java源代码上，若将这些输出结果储存起来，后续能够命中缓存并利用它们的构建就会快得多，从而节省开发者的时间。

### 从远程构建中获取输出结果的全新构建

将`compileJava`任务的输出储存起来之后，我们期望，后续使用了构建缓存的构建能够避免重新编译源代码。

[这里](https://enterprise-samples.gradle.com/s/cy4nezuzvj3qi/timeline)是第二次构建的构建扫描的时间线页面，它在第一次构建之后运行，且被配置为使用构建缓存。第二次构建可能是由同一个开发者在不同或者<ruby>清理<rt>clean</rt></ruby>后的工作空间内运行的，也可能是由不同的开发者，甚至是CI运行的，它们的共同点是都开启了构建缓存，从而能够从构建缓存获取数据。

随即，我们可以看到所有的编译和测试任务都被标记成了`FROM-CACHE`，这次构建只花了大概不到一秒。使用缓存的输出结果极大地节省了构建的时间。

我们还可以通过[构建缓存性能页面](https://enterprise-samples.gradle.com/s/cy4nezuzvj3qi/performance/buildCache)发现，远程缓存有3次<ruby>命中<rt>hit</rt></ruby>，以及具体的缓存查找和传输信息。

真实构建的运行通常要花费几分钟甚至几小时。通常，构建会反复地完成相同的工作，无论是复杂的CI<ruby>流水线<rt>pipeline</rt></ruby>构建还是开发者个人的构建。使用Gradle Enterprise的构建缓存，在开发者工时和所需CI设施上都可以节省大量时间。若我们将节省下来的时间乘以开发者的数量以及每天运行的CI构建的数量，总数将是巨大的。

更快的构建意味着开发者可以每天运行更多的构建，更快地发现问题，更高效地交付软件变更。

结果就是，一个能更多、更快、更省地交付的，拥有无与伦比的生产力的团队。

### 配置示例

欲达到较高的缓存命中率，我们可以令构建流水线中位置靠前的CI构建向远程缓存推送任务的输出结果，下游的CI构建和开发者构建从构建缓存中拉取任务的输出结果。

在下图中，你可以看到CI服务器向远程缓存推送数据，以及开发者从远程缓存拉取数据的过程。

![缓存的典型场景 - 本图是SVG格式，部分浏览器可能支持有限](/img/caching-typical-scenario.svg)

1. 一个运行在CI服务器上的任务。该构建被设置为向配置过的远程缓存推送数据，这样任务的输出结果就可以被流水线中的其他CI构建或者开发者构建所复用。
2. 开发者在本地修改了一个问题并运行相同的任务。Gradle依次试图从本地和远程缓存中加载任务的输出结果，由于缓存未命中，任务得以执行。此任务生成的输出结果被储存到工作空间和配置过的本地缓存中。储存在本地缓存中的输出结果可以被本机上的其他工作空间以及相同的工作空间复用，即使在这些工作空间运行过`clean`任务后。
3. 第二个开发者在CI构建过的一个提交版本上，不进行任何改变运行相同的任务。这一次，远程缓存查找命中，缓存后的任务输出结果被下载下来并直接拷贝到工作空间和本地缓存内，这样，该任务就无需被重复执行。

## 动手操作：Gradle Enterprise构建缓存

接下来，我们将会更进一步，通过实际运行构建来重现上面的示例。

### 先决条件

欲完成以下步骤，你需要：

1. 一个包含上述源代码的压缩包，用于重现上面的构建扫描。你可以在[这里](https://docs.gradle.com/enterprise/tutorials/caching/enterprise-tutorial-caching.zip)下载。请将它解压到随便什么地方，下文会以`LAB_HOME`指代这个位置。
2. 一个本地安装的JVM。对JVM的要求详见[https://gradle.org/install](https://gradle.org/install)。_注意：本地安装Gradle构建工具是可选而非必须的。所有的动手操作都会通过Gradle Wrapper来调用Gradle构建工具。_
3. 一个Gradle Enterprise服务器的实例。最方便的方法是通过[Gradle Enterprise试用版](https://gradle.com/enterprise/trial)，选择<ruby>在线版<rt>hosted</rt></ruby>，并按照指示和邮件操作。

> NOTE
>
> 在下文中，我们会假设你拥有一个Gradle Enterprise的实例，且位于`https://gradle-enterprise.mycompany.com`。
> 

在`LAB_HOME/01-using-the-gradle-enterprise-build-cache`目录中打开终端。

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


还需要修改将相同目录下的`settings.gradle`，将

{% highlight groovy %}
buildCache {
  local {
    enabled = false
  }
  remote(HttpBuildCache) {
    push = true
    url = '<<your Gradle Enterprise instance>>/cache/'
  }
}
{% endhighlight %}

替换为你自己的Gradle Enterprise构建缓存服务地址：

{% highlight groovy %}
buildCache {
  local {
    enabled = false
  }
  remote(HttpBuildCache) {
    push = true
    url = 'https://gradle-enterprise.mycompany.com/cache/' // <-- your personal Gradle Enterprise instance's build cache access point
  }
}
{% endhighlight %}

### 全新构建

现在，构建已经配置完成，随时可以运行了。我们会运行带`clean`任务的构建，将工作空间内所有任务的输出结果清除掉，然后将所有<ruby>可缓存任务<rt>cacheable task</rt></ruby>的输出结果推送到你的Gradle Enterprise实例上的远程缓存中去。

标准的Gradle构建是这样和配置好的Gradle Enterprise实例交互的：

1. 本地缓存已关闭（在`settins.gradle`中配置）。
2. 在每个可缓存任务执行期间，Gradle会通过任务的输入计算一个哈希值，并向Gradle Enterprise服务器查找匹配的结果。
3. 若缓存命中，任务的输出会被拷贝到工作空间中，而非重新运行任务。
4. 若干缓存未命中，任务会被执行，其输出结果和生成的缓存键值一起被拷贝到远程缓存中（因为`settins.gradle`中的`push`被设置为了`true`）。
5. 在构建结束后，Gradle将捕获的构建数据发送给Gradle Enterprise实例。命令行随后显示一个指向生成的构建扫描的唯一URL。此构建扫描包含十分精细的、缓存操作的各个方面的细节等信息。

Gradle Enterprise构建缓存的管理员页面能够显示缓存的统计数据，详见`https://gradle-enterprise.mycompany.com/cache-admin`。此时，假设你的Gradle Enterprise实例是新的，这个页面显示的应该是全零。

接下来我们要更进一步，运行我们的第一次全新构建——清除所有的任务输出，重新构建所有任务。然后请查看构建扫描和缓存管理员页面。通过下列命令执行全新构建：

{% highlight shell %}
$ ./gradlew clean check --build-cache
{% endhighlight %}

输出结果应该类似于：

{% highlight shell %}
$ ./gradlew clean check --build-cache

BUILD SUCCESSFUL in 7s
5 actionable tasks: 5 executed

Publishing build scan...
https://gradle-enterprise.mycompany.com/s/annggkicyjzro
{% endhighlight %}

打开构建扫描链接，你会看到和之前章节一模一样的构建扫描结果：所有的任务都被重新构建，3次缓存未命中，3个结果写入了缓存。

现在，打开缓存管理员页面`https://gradle-enterprise.mycompany.com/cache-admin`，你会看到3个结果被储存起来，以及3次缓存未命中。

![缓存管理员页面1](/img/cache-admin-1.png)

### 从远程构建中获取输出结果的全新构建

随后，我们会运行第二次构建，同样清除工作空间内的所有任务输出结果——不过，这一次缓存会命中。完成后请查看构建扫描和缓存管理员页面。通过下列命令执行第二次全新构建：

{% highlight shell %}
$ ./gradlew clean check --build-cache
{% endhighlight %}

输出结果应该类似于：

{% highlight shell %}
$ ./gradlew clean check --build-cache

BUILD SUCCESSFUL in 1s
5 actionable tasks: 1 executed, 3 from cache, 1 up-to-date

Publishing build scan...
https://gradle-enterprise.mycompany.com/s/juoqa37xwy2tw
{% endhighlight %}

打开构建扫描链接，你会看到和之前章节一模一样的第二个构建扫描结果：3个任务的输出结果缓存命中，因此被复用。注意，第二次构建在1秒之内就完成了（相比第一次的7秒）。

现在，重新打开缓存管理员页面，你会看到缓存命中的记录。

![缓存管理员页面2](/img/cache-admin-2.png)

## 结论

总而言之，利用Gradle Enterprise的构建缓存可以极大地缩短所有构建的构建时间，从而使得你的团队能够完成更多的工作，更快、更高效地向用户交付更有价值的产品特性，节省时间和成本。