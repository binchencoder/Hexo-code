title: 认识Bazel
date: 2019-08-16 20:53:08
tags:
    - Bazel
categories:
    - 工具
---

# Bazel入门

最近一直在研究网关这玩意，想借鉴我们公司的网关架构自己实现一下。思路是引入grpc-gateway，只是在这套系统的基础上增加一些定制的功能，如：负载均衡，权限验证，API参数检查。因为grpc-gateway采用的是Bazel来构建的，所以我要实现的网关也必须是用Bazel来构建。虽然在公司里接触过Bazel，但那也是在比较完善的平台上照葫芦画瓢。

为了完成自己的目标，只能硬着头皮边学边用，边用边学了

## Bazel是什么

Bazel 是一个开源的构建和测试工具，类似于Make、Maven及Gradle。它使用一种人易于理解的高级构建语言。Bazel 支持多种开发语言的项目，能够基于多个平台来构建。Bazel支持跨多个制品库和大规模用户的大型代码仓库。


## 为什么使用Bazel

Bazel具有以下优势：

- **高级构建语言** Bazel使用一种抽象的、人易于理解的、语义级别的高级语言来描述项目的构建属性。与其他工具不同，Bazel基于库，二进制文件，脚本和数据集的概念进行操作，使您免于陷入将单个调用编写到编译器和链接器等工具的复杂性。
- **Bazel高效可靠** Bazel缓存以前完成的所有工作，并跟踪文件内容和构建命令的更改。通过这种方式，Bazel知道何时需要重建某些东西，并仅重建那些东西。为了进一步加快构建速度，您可以将项目设置为以并行和增量的方式构建。
- **Bazel是跨平台的** Bazel可以在Linux，macOS和Windows上运行。Bazel可以为同一个项目中的多个平台(包括桌面,服务器和移动设备)构建二进制文件和可部署软件包。
- **Bazel扩展性强** Bazel在使用100k+源文件处理构建时仍然保持良好的性能表现。它适用于多个制品存储库和10K用户规模。

## 核心概念

Bazel根据在称为工作空间(WORKSPACE)的目录中组织的源代码构建软件。工作空间中的源文件以包的嵌套层次结构进行组织，其中每个包都是包含一组相关源文件和一个BUILD文件的目录。BUILD文件指定可以从源构建哪些软件输出。

### 工作空间

工作空间是文件系统上的目录，每个工作空间目录都有一个名为WORKSPACE的文本文件，该文件可能为空(不建议这样做，至少定义一个name)，或者可能包含对构建输出所需的外部依赖项的引用。

Sample:

```
workspace(name = "com_github_binchencoder_ease_gateway")
```

### 程序包

工作空间中代码组织的主要单元是包。包是相关文件的集合，以及它们之间的依赖关系的规范。每个程序包中包含一个BUILD文件，此文件中描述了此工具包的生成构建方式。

### 目标

生成的目标，每个target又可以作为另外一个规则的输入。
绝大部分的target属于两种基本类型中的一种，file和rule。另外，还有一种其他的target类型，package group。但是他们很少见。

![](./target_tree.png)


## 进阶

上述就是bazel最简单的描述，如果要学习bazel的详细进阶，可以访问起[官网](https://bazel.build)，官网上的文档非常详细，不过是英文的。

另外，我的github有几个更多的例子，可以进一步学习和理解bazel。

References:

- https://bazel.build
- https://github.com/grpc-ecosystem/grpc-gateway
- https://github.com/binchencoder/ease-gateway