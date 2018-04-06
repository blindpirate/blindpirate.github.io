---
layout:     post
title:      JUnit 5 Unroll Extension for Kotlin
subtitle:   JUnit 5 Unroll 扩展
date:       2018-03-25
author:     Bo 
catalog: true
tags:
    - 中文
    - 技术
    - 原创
    - Kotlin
    - Test
    - OpenSource
    - JUnit 5
---

二月份的时候一直在开发Gradle的JUnit 5支持，最终赶上了[Gradle 4.6的发布](https://docs.gradle.org/4.6/release-notes.html)。虽然还有一些issue，但看上去Gradle 4.6对JUnit 5的集成还是非常稳定的。

JUnit 5最大的特点就是扩展系统（Extension）。在JUnit 4时代，自定义JUnit行为的方法是[Runner](https://github.com/junit-team/junit4/wiki/test-runners)系统，然鹅，每个测试类只能定义一个`Runner`，多了不行。这就好像，你入了我的党，就不能再入别的党。

那你说，我既想那啥，又想那啥，应该咋办？

```
@RunWith(当婊子.class)
@RunWith(立牌坊.class)
public class 矛盾的我Test {
    ...
}
```

答案是……没辙。

JUnit 4的这个设定，让人民群众非常痛苦。JUnit 5采用了全新的扩展系统（Extension），优雅的解决了这个问题。有关JUnit 5扩展系统的细节以后详谈，简而言之，JUnit 5允许你同时加入多个党。本文的主要内容是我开发的一个扩展：[JUnit 5 Unroll Extension](https://github.com/blindpirate/junit5-unroll-extension)。

# 一个例子

我们假设你想测试`Math.max(int a, int b)`。在JUnit 4中，[官方做法](https://github.com/junit-team/junit4/wiki/Parameterized-tests)是这么干的：

```
@RunWith(Parameterized.class)
public class MaxTest {
    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {
                 { 1, 3, 3 }, { 7, 4, 7 }, { 0, 0, 0 }  
           });
    }

    @Parameter // first data value (0) is default
    public int a;

    @Parameter(1)
    public int b;

    @Paramater(2)
    public int result;

    @Test
    public void test() {
        assertTrue(Math.max(a, b) == result);
    }
}
```

这个……反正我看着晕。

非官方做法[JUnitParams](https://github.com/Pragmatists/JUnitParams)要优雅一点：

```
@RunWith(JUnitParamsRunner.class)
public class MaxTest {
    @Test
    @Parameters({"1, 3, 3", 
                 "7, 4, 7",
                 "0, 0, 0"}) 
    public void test(int a, int b, int result){
        assertTrue(Math.max(a, b) == result);
    }
}
```

不过你仔细看的话就会发现数据实际上是内嵌在字符串里的，这是受注解的限制所致，因此也没有编译器检查。

# Data Driven Test

如果你用过[Spock测试框架](http://spockframework.org/)，你应该会被它处理这个问题的做法惊艳到：

```
class MathSpec extends Specification {
  def "maximum of #a and #b is #result"() {
    expect:
    Math.max(a, b) == result

    where:
    a | b | result
    1 | 3 | 3
    7 | 4 | 7
    0 | 0 | 0
  }
}
```

这样会生成三个测试：

- maximum of 1 and 3 is 3
- maximum of 7 and 4 is 7
- maximum of 0 and 0 is 0

Spock号称这是“数据驱动测试”。

Gradle的几乎所有测试都是基于Spock的（刚刚粗略统计了一下它有4000多个测试文件，60多万行测试代码），私以为，这是Spock最强大的功能，没有之一。

既然这么强大，那我就要开始山寨了。花了几天的时间，实现了一个JUnit 5的扩展，旨在向Kotlin提供类似Spock的数据驱动测试体验。这也是我自己的第一个Kotlin项目，有关Kotlin的体验见我的另一篇帖子：[Kotlin初体验](/2018/03/25/Kotlin初体验/)。

# JUnit 5 Unroll Extension

Kotlin是静态语言，因此语法上对这种天马行空的写法多有限制。还好，最后我想方设法地在Kotlin语法的铜墙铁壁上挖了个洞：

```
import com.github.blindpirate.junit.extension.unroll.Param
import com.github.blindpirate.junit.extension.unroll.Unroll
import com.github.blindpirate.junit.extension.unroll.where

class Math {
    @Unroll
    fun `max number of {0} and {1} is {2}`(
            a: Int, b: Int, c: Int, param: Param = where {
                1 _ 3 _ 3
                7 _ 4 _ 7 
                0 _ 0 _ 0
            }) {
        assert(Math.max(a, b) == c)
    }
}
```

虽然没有Spock那么优雅，但是还是比原生的[JUnit 5 `@ParameterizedTest`](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)方便了不少。在JUnit 5中，原生的`@ParameterizedTest`只支持`int`/`String`/`Enum`等数据类型（受Java的`Annotation`所限）；如果你想搞一些其他的类型，比如下面的测试，原生的`@ParameterizedTest`是做不到的，只能自己定义[`ArgumentsProvider`](https://junit.org/junit5/docs/current/api/org/junit/jupiter/params/provider/ArgumentsProvider.html)和[`ArgumentConverter`](https://junit.org/junit5/docs/current/api/org/junit/jupiter/params/converter/ArgumentConverter.html)：

```
@Unroll
fun `first element of {0} is {1}`(
        list: List<Any>, element: Any, param: Param = where {
            listOf(1)                       _ 1
            listOf(2.0)                     _ 2.0
            listOf(Instant.ofEpochMilli(3)) _ Instant.ofEpochMilli(3)
        }) {
    assert(list.first() == element)
}
```

在IDEA中运行的结果如下：

![1](/img/unroll-test-result.png)

没错，因为Kotlin支持反引号包裹的特殊方法名，你可以像这个例子一样把测试用例的描述放在方法名里。`{0}`、`{1}`会被自动渲染为参数的值。这其实也是从Spock里面山寨过来的。

最后，这货的Maven坐标是：

```
<dependency>
    <groupId>com.github.blindpirate</groupId>
    <artifactId>junit5-unroll-extension</artifactId>
    <version>0.1.1</version>
    <scope>test</scope>
</dependency>
```

或者

```
repositories {
    jcenter()
}

dependencies {
    testCompile 'com.github.blindpirate:junit5-unroll-extension:0.1.1'
}
```

# 一点限制

这个扩展有一个小小的限制：在`where {}`的花括号里面必须是一个函数，而不能是一个闭包，即不能引用外围实例，如：

```
class UnsupportedTest {
    @Unroll
    fun `this is not supported`(
            a: Int, b: Int, param: Param = where {
                abs(-1) _ 1
            }) {
    }

    private fun abs(i: Int): Int = Math.abs(i)
}
```

这种情况是不支持的，因为`abs()`是外围实例的方法，而在处理参数的过程中外围类的实例还没有创建。

Enjoy!