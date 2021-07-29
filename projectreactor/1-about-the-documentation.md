# 1-关于本文档

本章是 Reactor 参考文档的简要概述。你不用线性阅读这份文档[@FCG](https://github.com/AsuKa1989) 。每章的内容都是独立的。当然每一章经常引用其它章节的内容。

## 1.1.最新版本及版权声明
Reactor 参考文档可以通过HTML网页查看。[最新版本](https://projectreactor.io/docs/core/release/reference/index.html) 

你可以制作本文件的副本供你自己使用，也可以分发给其他人，但前提是你不对副本收取任何费用，而且每一份副本均包含本版权声明，无论以印刷体或电子形式分发。([@FCG](https://github.com/AsuKa1989)为尊重版权，以下是版权声明的英文原文）

> Copies of this document may be made for your own use and for distribution to others, provided that you do not charge any fee for such copies and further provided that each copy contains this Copyright Notice, whether distributed in print or electronically.

## 1.2.为文档做贡献
参考文档是用 Asciidoc 编写的，你可以在 https://github.com/reactor/reactor-core/tree/main/docs/asciidoc 查看原始文件。

如果你觉得文档有改进的地方或者建议，我们将很高兴收到您的 PR。
​
我们建议你下载仓库到本地，这样你就可以通过运行 asciidoctor 命令来渲染并生成文档。因为文档部分内容需要某些源文件，所以 GitHub 的渲染并不总是完整的。

> 为了方便文档编辑，大多数章节的末尾都有一个链接，可以直接在 GitHub 上打开该部分主要源文件的编辑 UI。这些链接仅出现在本参考指南的 HTML5 版本中。示例：关于文档的编辑建议（Suggest Edit to About the Documentation.）。

## 1.3.获取帮助
你可以通过多种方式获取关于 Reactor 的帮助：

- 在 [Gitter](https://gitter.im/reactor/reactor) 上获取帮助。
- 在 stackoverflow 的 [project-reactor](https://stackoverflow.com/tags/project-reactor) 板块提问。
- 在 Github 的 issues 反馈 bug。我们密切关注以下仓库：[reactor-core](https://github.com/reactor/reactor-core/issues)（基本功能）和 [reactor-addons](https://github.com/reactor/reactor-addons/issues)（ reactor-test 和 adapters）。
> Reactor 的所有内容都是开源的，包括[本文档](https://github.com/reactor/reactor-core/tree/main/docs/asciidoc)。如果你发现文档存在问题或想要改进它们，请[参与](https://github.com/reactor/.github/blob/main/CONTRIBUTING.md)。

## 1.4.导航

- 如果你想直接从代码开始，请查看第2章-[入门](./2-getting-started.md)。
- 如果你不熟悉响应式编程，请查看第3章-[响应式编程简介](./3-introduction-to-reactive-programming.md)。
- 如果你熟悉 Reactor 概念，并且只是在为工作寻找合适的工具，但不知道用什么操作符。请查看[附录 A-我需要哪个操作符](./appendix-a-which-operator-do-i-need.md)。
- 为了更深入地了解 Reactor 的核心功能，请查看第4章-[Reactor 核心功能](./4-reactor-core-features.md)。
   - 更多介绍关于 Reactor 的响应类型：**Flux**(包含0个或者 N 个数据的异步数据流)和 **Mono**(包含0个或者1个数据的异步数据流)。
   - 如何使用调度程序切换线程上下文。
   - 如何处理错误。
- 单元测试？是的，reactor-test 是可以的，请查看第6章-[测试](./6-testing.md)。
- 更高级的创建响应式源的方法，请查看第4章的第4小节-[以编程方式创建流](./4-reactor-core-features.md)。
- 其他高级主题，请查看第9章-[高级功能和概念](./9-advanced-features-and-concepts.md)。

[Suggest Edit](https://github.com/reactor/reactor-core/edit/main/docs/asciidoc/aboutDoc.adoc) to "[About the Documentation](https://projectreactor.io/docs/core/release/reference/#about-doc)"