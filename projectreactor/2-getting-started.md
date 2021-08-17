# 2-入门

本章内容可以帮助你开始使用 Reactor，它包括以下部分：

- Reactor 简介
- 前置要求
- 了解 BOM 和版本控制方案
- 获取 Reactor
## 2.1.Reactor 简介
Reactor 是基于 JVM 的完整非阻塞反应式编程基础，具有高效的需求管理（以管理“背压”的形式实现）。它直接与 Java 8 函数式 API 集成，特别是 CompletableFuture、Stream 和 Duration。它提供了可组合的异步序列 API	——Flux（用于 N 个元素）和 Mono（用于0或1个元素），并完整实现了 Reactive Streams 规范。
​

Reactor 还支持与 reactor-netty 项目的非阻塞进程间通信。Reactor Netty 适合微服务架构，为 HTTP（包括 Websockets）、TCP 和 UDP 提供背压的网络引擎。完整支持响应式编解码。
## 2.2.前置要求
Reactor Core 在 Java 8 及更高版本上运行。
​

它对 org.reactivestreams:reactive-streams:1.0.3 有传递依赖。
> 安卓支持
> - Reactor 3 官方不提供针对 Android 的支持（如果对响应式需求强烈，请考虑使用 RxJava 2）。
> - 但是呢，它可以在 Android SDK 26 (Android O) 及更高版本上运行。
> - 启用脱糖后，它可以在 Android SDK 21 (Android 5.0) 及更高版本上运行。见 [https://developer.android.com/studio/write/java8-support#library-desugaring](https://developer.android.com/studio/write/java8-support#library-desugaring)
> - 我们愿意尽最大努力评估有利于 Android 支持的改进。但是，我们无法做出保证。具体情况具体分析吧。

## 2.3.了解 BOM 和版本控制方案
Reactor 3 使用 BOM（材料清单）模型（从 reactor-core 3.0.4 开始，是 Aluminium 发布版本）。这个策划列表将旨在很好地协同工作的模块分组，尽管这些构件中可能存在不同的版本控制方案，但仍提供相关版本。
​

请注意，版本控制方案在 3.3.x 和 3.4.x（Dysprosium 和 Europium）之间发生了变化。
​

构件遵循 MAJOR.MINOR.PATCH-QUALIFIER 的版本控制方案，而 BOM 使用受 CalVer 启发的 YYYY.MINOR.PATCH-QUALIFIER 方案进行版本控制，其中：

- MAJOR 是当前大版本的 Reactor，每一大版本的更新都会给项目结构带来根本性的变化（这可能意味着更重要的迁移工作）
- YYYY 是给定发布周期中第一个 GA 发布的年份（例如 3.4.0 的 3.4.x）
- .MINOR 是一个从0开始的数字，随着每个新的发布周期递增
   - 在 projects 的情况下，它通常反映更广泛的变化，可以表明适度的迁移工作
   - 在 BOM 的情况下，它允许区分发布周期，以防两个在同一年首次发布
   - .PATCH 是一个从0开始的数字，随着每个服务版本递增
   - -QUALIFIER 是一个文本限定符，在 GA 版本的情况下被省略（见下文）



因此，遵循该约定的第一个发布周期是 2020.0.x，代号为 Europium。该方案按顺序使用以下限定符（注意使用破折号分隔符）：

- -M1..-M9：里程碑（我们预计每个服务版本不会超过 9 个）
- -RC1..-RC9：候选版本（我们预计每个服务版本不会超过 9 个）
- -SNAPSHOT：快照
- 没有 GA 版本的限定符



> 快照按上述顺序出现在更靠前的位置，因为从概念上讲，它们始终是任何给定 PATCH 的“最新的预发布”。即使 PATCH 周期的第一个部署构件将始终是一个 -SNAPSHOT，一个类似命名但更新的快照也将在例如之后发布。里程碑或发布候选者之间。



每个发布周期也有一个代号，与之前的基于代号的方案保持一致，可以用来做非正式地名称（比如在讨论、博客文章等中……）。代号代表传统的 MAJOR.MINOR……编号。它们大部分来自元素周期表，按字母顺序递增。
> 在 Dysprosium 之前，BOM 是使用代号后跟限定符的版本序列方案进行版本控制的，并且限定符略有不同。例如：Aluminium-RELEASE（第一个 GA 版本，现在类似于 YYYY.0.0）、Bismuth-M1、Californium-SR1（服务版本现在类似于 YYYY.0.1）、Dysprosium-RC1、Dysprosium-BUILD-SNAPSHOT （在每个补丁之后，我们会回到相同的快照版本。现在类似于 YYYY.0.X-SNAPSHOT 所以我们每个补丁获得 1 个快照）。

## 2.4.获取 Reactor
如前所述，使用 Reactor 的最简单方法是使用 BOM 并将相关依赖项添加到您的项目中。请注意，当您添加此类依赖项时，您必须省略版本，以便从 BOM 中获取版本。
​

但是，如果您想强制使用特定构件的版本，则可以像往常一样在添加依赖项时指定它。您还可以完全放弃 BOM，并通过其构件版本指定依赖项。
> 从 reactor-core 3.4.9 版本开始，相关发布系列中最新的稳定 BOM 是 2020.0.10，这是下面的片段中使用的。从那时起可能会有更新的版本（包括快照、里程碑和新版本系列），请参阅 [https://projectreactor.io/docs](https://projectreactor.io/docs) 以了解最新的工件和 BOM。

### 2.4.1. Maven 添加
Maven 本身支持 BOM 概念。首先，您需要通过将以下代码段添加到 pom.xml 来导入 BOM：
```xml
<dependencyManagement>  //注意dependencyManagement 标签。这是常规依赖项部分的补充。
    <dependencies>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-bom</artifactId>
            <version>2020.0.10</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
如果顶部 (dependencyManagement) 已存在于您的 pom 中，则仅添加内容。
​

接下来，像往常一样，将您的依赖项添加到相关的 reactor 项目中，除了没有 <version>，如下所示：
```xml
<dependencies>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-core</artifactId>  //对核心库的依赖。
        //这里没有版本标签。
    </dependency>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-test</artifactId>  //reactor-test 提供了对响应式进行单元测试的工具。
        <scope>test</scope>
    </dependency>
</dependencies>
```
### 2.4.2. Gradle 添加
在 5.0 版本之前，Gradle 没有对 Maven BOM 的核心支持，但您可以使用 Spring 的 gradle-dependency-management 插件。
​

首先，从 Gradle 插件门户应用插件，如下所示：
```
plugins {
    id "io.spring.dependency-management" version "1.0.7.RELEASE"  //在撰写本文时，1.0.7.RELEASE 是该插件的最新版本。检查更新。
}
```
然后用它来导入BOM，如下：
```
dependencyManagement {
     imports {
          mavenBom "io.projectreactor:reactor-bom:2020.0.10"
     }
}
```
最后给你的项目添加一个依赖，没有版本号，如下：
```
dependencies {
     implementation 'io.projectreactor:reactor-core'  // 版本没有第三个 : 分隔部分。它取自 BOM。
}
```
从 Gradle 5.0 开始，您可以使用对 BOM 的原生 Gradle 支持：
```
dependencies {
     implementation platform('io.projectreactor:reactor-bom:2020.0.10')
     implementation 'io.projectreactor:reactor-core'  // 版本没有第三个 : 分隔部分。它取自 BOM。
}
```
### 2.4.3.里程碑和快照
里程碑和开发人员预览是通过 Spring Milestones 存储库而不是 Maven Central 分发的。要将其添加到您的构建配置文件中，请使用以下代码段：
示例 1. Maven 的里程碑版本
```xml
<repositories>
	<repository>
		<id>spring-milestones</id>
		<name>Spring Milestones Repository</name>
		<url>https://repo.spring.io/milestone</url>
	</repository>
</repositories>
```
对于 Gradle，请使用以下代码段：
示例 2. Gradle 的里程碑版本
```
repositories {
  maven { url 'https://repo.spring.io/milestone' }
  mavenCentral()
}
```
示例 3. Maven 的快照版本
```xml
<repositories>
	<repository>
		<id>spring-snapshots</id>
		<name>Spring Snapshot Repository</name>
		<url>https://repo.spring.io/snapshot</url>
	</repository>
</repositories>
```
示例 4. Gradle 的快照版本
```
repositories {
  maven { url 'https://repo.spring.io/snapshot' }
  mavenCentral()
}
```
## 2.5.支持和规则
以下条目是镜像 [https://github.com/reactor/.github/blob/main/SUPPORT.adoc](https://github.com/reactor/.github/blob/main/SUPPORT.adoc)
### 2.5.1.问题解答
> 先搜索 Stack Overflow；必要时可以讨论

如果你不确定为什么有些东西不起作用，或者想知道是否有更好的方法来姐姐问题。请先在 Stack Overflow 查询。如有必要，开始讨论。在我们为此目的关注的标签中使用相关标签：

- [**reactor-netty**](https://stackoverflow.com/questions/tagged/reactor-netty)：对于特定的 reactor-netty 问题。
- [**project-reactor**](https://stackoverflow.com/questions/tagged/project-reactor)：对于一般反应器问题。

如果您更喜欢实时讨论，我们还有一些 Gitter 频道：

- [**reactor**](https://gitter.im/reactor/reactor)：是历史上最活跃的一个频道，大多数问题社区都可以提供帮助。
- [**reactor-core**](https://gitter.im/reactor/reactor-core)：旨在围绕库的内部工作进行更高级的精确讨论。
- [**reactor-netty**](https://gitter.im/reactor/reactor-netty)：用于针对特定于 netty 的问题。



有关其他的信息，请参阅每个项目的自述文件。
​

我们通常不鼓励因为问题打开 GitHub issues，而是使用上述两个渠道。
### 2.5.2.已弃用的规则
在处理弃用时，给定版本 A.B.C，我们将确保：

- 版本 A.B.0 中引入的弃用将在版本 A.B+1.0 之前移除
- 版本 A.B.1+ 中引入的弃用将在版本 A.B+2.0 之前删除
- 我们将努力在弃用 javadoc 中提及以下内容：
   - 删除目标最低版本
   - 指向已弃用方法的替代品的指针
   - 不推荐使用该方法的版本
> 该政策自 2021 年 1 月起正式生效，适用于 2020.0 BOM 和更新版本系列中的所有模块，以及 Dysprosium-SR15 之后的 Dysprosium 版本。

> 弃用删除目标不是硬性承诺，弃用方法可以比这些最小目标 GA 版本更持久（即，只有最有问题的弃用方法才会被积极删除）。

> 也就是说，已超过其最小删除目标版本的已弃用代码可能会在任何后续版本（包括补丁版本，即服务版本）中被删除，恕不另行通知。所以用户仍然应该努力尽早更新他们的代码。

### 2.5.3.开发情况
下表总结了各种 Reactor 版本系列的开发状态：

| Version | Supported |
| --- | --- |
| 2020.0.0 (codename Europium) (core 3.4.x, netty 1.0.x) | :white_check_mark: |
| Dysprosium Train (core 3.3.x, netty 0.9.x) |
:white_check_mark: |
| Califonium 及以下（core < 3.3，netty < 0.9） | X |
| Reactor 1.x 和 2.x  | X |