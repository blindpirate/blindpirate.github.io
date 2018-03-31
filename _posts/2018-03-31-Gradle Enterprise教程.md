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

Gradle Enterprise的远程缓存（remote cache）可以使构建更快——无论是对开发者还是CI而言。

阅读本教程需要

- 1分钟（只阅读简介）
- 5-10分钟（阅读简介和正文） (read the Introduction and Tour)
- 15-20分钟（阅读简介、正文并动手操作） minutes (read the Introduction and Tour and perform the Hands-on Lab)

## 简介：避免重复构建

逻辑上，Gradle有三层复用机制，避免潜在的昂贵任务重复执行：

1. 对工作空间中任务的输出执行Up-to-date检查。
2. 在本地缓存中查找任务的输出。
3. 在远程缓存中查找任务的输出，如[Gradle Enterprise](https://gradle.com/build-cache)自带的远程缓存。


![3层缓存 - 本图是SVG格式，部分浏览器可能支持有限](/img/caching-3-layers.svg)

这三个等级的支持能够在三种不同的目标场景下加速你的构建：

1. 在两次连续运行的Gradle构建之间，通常不会有太多东西发生改变。Gradle的增量构建功能能够只执行和上次构建相比“过时”（not up-to-date）的任务。
2. 典型情况下，开发者在不同的分支上维护着不同的工作空间，以执行逻辑上独立的任务。本地缓存使得输出结果能被迅速地跨工作空间复用，而无需访问网络。
3. 许多时候，CI节点和开发者运行的任务相同，工作空间内的变更也相同。远程缓存允许输出结果在用户和构建服务器上被复用，确保整个团队无需多次重复运行相同的构建。

## Gradle Enterprise构建缓存之旅

接下来，让我们深入探究远程缓存是如何加速你的构建的。

### 全新构建（Clean build）

考虑一个非常简单的Java示例项目。[这个页面](https://enterprise-samples.gradle.com/s/enhrtnwl2jah6/timeline)是一次构建扫描（build scan，对一次Gradle构建过程的完整记录）的时间线。这次构建是一次全新构建，它运行了工作空间内的所有任务，且没有从缓存中获取任何数据。我们可以通过观察任务的时间线分辨出这一点：没有任何一个任务的状态是`FROM-CACHE`。这个示例项目是人造构建花了大约8秒来执行所有的任务，其中大部分时间花在了编译Java源代码上。

点击任务`:compileJava`可以看到，构建缓存未命中（miss），从而发生了一次存储（store）过程。我们执行了重新编译，并将结果储存到了远程缓存中。

通过构建扫描的[构建缓存性能页面](https://enterprise-samples.gradle.com/s/enhrtnwl2jah6/performance/buildCache)，我们还可以发现，这次构建虽然没有能够命中缓存，却存储了3个输出结果到远程缓存中，从而使得后续的构建受益。

我们还可以看到，由于本次构建的大多数时间都花在了编译Java源代码上，通过将输出结果储存起来，我们可以知道，若某次构建能够命中缓存并利用这个输出结果，它的速度会极大地提高，从而节省开发者的时间。

### 从远程构建中获取输出结果的全新构建

将`compileJava`任务的输出储存起来之后，我们预期的记过是后续使用了构建缓存的构建能够避免重新编译源代码。


The build, a simple contrived example, takes about 8 seconds to execute tasks, mostly to compile Java sources.

Click on the task `:compileJava` to see that the build cache result was a miss, followed by a store.
We recompiled, and then stored the output in the remote cache.

We can also look at the https://enterprise-samples.gradle.com/s/enhrtnwl2jah6/performance/buildCache[build cache performance page]
of the build scan to see that this build enjoyed no cache hits, but it did store 3 outputs in the remote cache to the benefit of subsequent builds.

We can also see that since almost all of the time taken for this build was spent on compiling Java sources, and we stored that output,
we would expect builds that can use this output via a cache hit to be much faster, saving developers time.

=== Clean build with outputs taken from the remote cache

Having stored the output of the `compileJava` tasks, we would expect that subsequent builds that use the build cache
would not have to recompile the source.

https://enterprise-samples.gradle.com/s/cy4nezuzvj3qi/timeline[Here] is the timeline page of a build scan for a second build, run after the first one
and configured to pull from the build cache. It could have been run by the same developer in a different or clean workspace,
a different developer, or a CI build that has pulling from the build cache turned on.

We can immediately see that all the compilation and tests tasks are annotated `FROM-CACHE`, and that
the build only took about a half second to complete. Using cached outputs has saved most of the time of the build.

We can also examine the build cache specifically on https://enterprise-samples.gradle.com/s/cy4nezuzvj3qi/performance/buildCache[the build cache performance page],
which shows 3 hits to the remote cache along with detailed information about the cache lookups and transfers.

Real builds often take many minutes or even hours to run. Often builds have to do the same work over and over. This is true of complex CI pipelines
down to individual developer builds. Using the Gradle Enterprise build cache saves time, both in developer hours and required CI infrastructure. When we multiply
the savings by the number of developer and CI builds per day, the savings are immense.

Faster builds mean developers can run more builds per day, find issues more quickly, and deliver changes more efficiently.

The result is an exceptionally productive team that can deliver more, faster, at lower cost.

=== Configuration example

We can achieve a high cache hit rate by having CI builds early in the build pipeline push task outputs to the remote cache,
and downstream CI builds as well as developer builds pull task outputs from the remote cache.

In the diagram below, you can see the flow of CI agents pushing to the remote cache, and developers pulling from the remote cache.

[role="shadow"]
image::caching-typical-scenario.svg[]

1. A task runs on a CI server. The build is configured to push to a configured remote cache, so that task outputs can be reused
   by other CI pipeline builds or developer builds.
2. A developer runs the same task with a local change to a file. Gradle tries to load the task output from the local then the remote cache,
   misses, and the task executes. The task generates the task output which is stored both in the workspace and in the
   configured local cache. Outputs stored in the local cache can be reused in other workspaces local to that developer's machine,
   or in the same workspace, even after running the `clean` task.
3. A second developer runs that task without any local changes from the commit that CI built. This time the remote cache
   lookup is a hit, the cached task output is downloaded and directly copied to the workspace and the local cache, and the task does not need to be executed.

== Hands-on Lab: Gradle Enterprise build cache

Read on to go one level deeper and run builds to recreate the cache examples you viewed above.

=== Prerequisites
To follow these steps, you will need:

1. A zip file that includes the source code to recreate the build scans we discussed previously. +
You can get this from link:enterprise-tutorial-caching.zip[here]. +
Unzip this anywhere, we will refer to that location as `LAB_HOME` in what follows.
2. A local JVM installed. +
JVM requirements are specified at https://gradle.org/install.
_Note that a local Gradle Build Tool install is optional and not required. All hands-on labs will execute the Gradle Build Tool using the Gradle Wrapper._
3. An instance of a Gradle Enterprise server. +
The most convenient way to get this is to https://gradle.com/enterprise/trial[sign up for a Gradle Enterprise trial],
select the fully hosted option and follow the instructions and emails provided.

[NOTE]
====
For the rest of the document, we will assume you have a Gradle Enterprise instance, available at `\https://gradle-enterprise.mycompany.com`.
====

Open a terminal window in `LAB_HOME/01-using-the-gradle-enterprise-build-cache`.

Using the text editor of your choice, modify the build scan configuration in `build.gradle` to point to your Gradle Enterprise server instance:
[horizontal]
Replace::
+
[source,groovy]
----
buildScan {
  server = '<<your Gradle Enterprise instance>>'
  publishAlways()
}
----
with::
+
[source,groovy]
----
buildScan {
  server = 'https://gradle-enterprise.mycompany.com' // <-- your personal Gradle Enterprise instance
  publishAlways()
}
----

You will also need to modify `settings.gradle` in the same directory,
with your Gradle Enterprise build cache service location:
[horizontal]
Replace::
+
[source,groovy]
----
buildCache {
  local {
    enabled = false
  }
  remote(HttpBuildCache) {
    push = true
    url = '<<your Gradle Enterprise instance>>/cache/'
  }
}
----
with::
+
[source,groovy]
----
buildCache {
  local {
    enabled = false
  }
  remote(HttpBuildCache) {
    push = true
    url = 'https://gradle-enterprise.mycompany.com/cache/' // <-- your personal Gradle Enterprise instance's build cache access point
  }
}
----

=== Clean build

At this point, the build is fully configured and ready to run.
We will run this build with a `clean`, which will remove all task outputs from the workspace, execute all tasks, and then push all cacheable task outputs to the remote cache in your Gradle Enterprise instance.

Here is how this standard Gradle build will now interact with the configured Gradle Enterprise instance:

1. The local cache is disabled (according to `settings.gradle`).
2. Each cacheable task does a lookup in the Gradle Enterprise server to see if there is a match between
the cache key computed as a hash of task inputs with a matching element in the remote cache, at task execution time.
3. For a cache hit, task outputs are copied to the workspace, instead of rerunning the task.
4. For a cache miss, task outputs are rebuilt, and a cache key is generated and copied into the remote cache (since `push` is set to `true` for this cache in the `settings.gradle` file).
5. At the end of the build, Gradle sends the captured build data to the Gradle Enterprise instance. A unique URL pointing to the generated build scan is printed on the command line.
This build scan contains very fine-grained details of all aspects of cache operations, and more.

The Gradle Enterprise build cache has an administrative page that shows cache statistics, that can be viewed from `\https://gradle-enterprise.mycompany.com/cache-admin`.
At this point, assuming your Gradle Enterprise instance is fresh, you should see all zeros on this page.

We will now go ahead and run our first clean build which will remove all task outputs, rebuild them all, and then view the build scan and cache admin pages.
Perform the clean build with

{% highlight shell %}
$ ./gradlew clean check --build-cache
{% endhighlight %}

----
Your output should look something like:
----
$ ./gradlew clean check --build-cache

BUILD SUCCESSFUL in 7s
5 actionable tasks: 5 executed

Publishing build scan...
https://gradle-enterprise.mycompany.com/s/annggkicyjzro
----

If you follow the build scan link, you will see an identical build scan to the first build scan discussed in the earlier section,
and you will be able to see that all the tasks rebuilt, that the build had 3 cache misses, and that 3 artifacts were written to cache.

At this point, if you revisit the cache admin page at `\https://gradle-enterprise.mycompany.com/cache-admin`, you will see 3 artifacts now stored, along with the 3 cache misses.

[role="shadow"]
image::cache-admin-1.png[]

=== Clean build with outputs taken from the remote cache

We will now go ahead and run our second clean build which will remove all task outputs from the workspace, but now experience cache hits,
and then view the build scan and cache admin pages. Perform the second clean build with
----
$ ./gradlew clean check --build-cache
----
Your output should look something like:
----
$ ./gradlew clean check --build-cache

BUILD SUCCESSFUL in 1s
5 actionable tasks: 1 executed, 3 from cache, 1 up-to-date

Publishing build scan...
https://gradle-enterprise.mycompany.com/s/juoqa37xwy2tw
----

If you follow the build scan link, you will see an identical build scan to the second build scan discussed in the earlier section,
and you will be able to see that 3 tasks reused outputs that were cache hits.
Also note that this second build completed in 1 second (versus 7 for the first).

At this point, if you revisit the cache admin page, you will see a record of the cache hits.

[role="shadow"]
image::cache-admin-2.png[]

== Conclusion

To summarize, utilizing Gradle Enterprise’s build cache can result in dramatic improvements in build times for all your builds,
returning time to your team to do more work and deliver more valuable features to your users more quickly and efficiently,
which saves you time and your organization money.