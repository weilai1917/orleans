---
layout: page
title: Contributing to Orleans
---

# 对Orleans的贡献

为那些想为Orleans做出贡献的开发者提供一些注释和指导。

## 对本项目的贡献

这里有一些建议，任何人寻找迷你特性和工作项目，将对Orleans作出积极的贡献。

这些只是一些想法，所以如果你想到其他有用的方法，那么就把[讨论线索](https://github.com/dotnet/orleans/issues)在GitHub上讨论这个提议，并付诸行动！

-   **[OrleansGitHub存储库](https://github.com/dotnet/orleans)**

拉请求总是受欢迎的！

-   **[实习生和学生项目](student_projects.md)**

对可能的实习生/学生项目的一些建议。

-   **[文件编制指南](documentation_guidelines.md)** 

用于编写此网站文档的样式指南。

## 代码贡献

这个项目和另一个项目使用相同的贡献过程**[DotNet项目](http://dotnet.github.io/)**在GitHub上。

-   **[DotNet项目贡献指南](https://github.com/dotnet/corefx/wiki/Contributing)**

为GitHub上的DotNet项目提供帮助的指南和工作流。

-   **[互联网CLA](https://cla.dotnetfoundation.org/)**

GitHub上DotNet项目的贡献许可协议。

-   **[.NET框架设计指南](https://github.com/dotnet/corefx/wiki/Framework-Design-Guidelines-Digest)**

一些基本的API设计规则、编码标准和.NETFramework API的样式指南。

## 编码标准和惯例

对于编码风格的战争，我们尽量不要过于拘泥于强迫症，但在发生争议时，我们确实会回到GitHub上其他DotNet OSS项目使用的两本“.NET编码标准”书籍中的核心原则：

-   [编码风格指南](https://github.com/dotnet/corefx/blob/master/Documentation/coding-guidelines/coding-style.md)

-   [.NET框架设计指南](https://github.com/dotnet/corefx/blob/master/Documentation/coding-guidelines/framework-design-guidelines-digest.md)

还有很多其他有用的文档[.NET核心清除器](https://github.com/dotnet/coreclr/tree/master/Documentation)和[.NET核心框架](https://github.com/dotnet/corefx/tree/master/Documentation)值得一读的文档站点，尽管大多数有经验的C开发人员可能已经通过渗透获得了许多这样的最佳实践，特别是在性能和内存管理方面。

## 源代码组织

Orleans并没有严格遵循“每个文件一个类”的规则，但是我们尝试使用实际的判断来最大限度地改变团队中开发人员的“代码理解能力”。如果许多小的ish类共享一个“公共主题”和/或总是一起处理，那么在大多数情况下，将它们放在一个源代码文件中是可以的。例如，不同的“LogConsumer”类最初放在单个源文件中，因为它们表示代码理解的单个单元。

作为推论，如果类位于与类同名的文件中，那么查找该类的源代码就容易得多[类似于Java文件命名规则]因此，在代码查找能力和最小化/限制解决方案中的项目数量和项目中的文件数量之间存在着一种张力和价值判断[对于大型项目，这两者都直接影响到visualstudio的“开放”和“构建”时间"].

在VS和ReSharper中的代码搜索工具绝对有帮助。

## 依赖关系和项目间引用

我们非常严格的一个主题是关于组件和子系统之间的依赖引用。

### 组件/项目参考

解决方案中项目之间的引用必须始终使用“**项目参考文献**“而不是”*DLL引用*“以确保构建工具知道组件构建关系。

**赖特**:

```
<ProjectReference Include="..\Orleans\Orleans.csproj">
  <Project>{BC1BD60C-E7D8-4452-A21C-290AEC8E2E74}</Project>
  <Name>Orleans</Name>
</ProjectReference>
```

*错了*:

```
<Reference Include="Orleans" >
   <HintPath>..\Orleans\bin\Debug\Orleans.dll</HintPath>
</Reference>
```

为了帮助确保项目间引用保持干净，然后在构建服务器上[和本地的`生成.cmd`脚本]我们故意使用并排输入`.\src`和输出`.\二进制文件`目录而不是更正常的就地构建目录结构(例如。`[项目]\bin\发布`)由本地开发计算机上的VS使用。

### 统一组件版本

我们在整个Orleans代码库中使用相同的外部组件统一版本，因此不需要添加`绑定重定向`中的条目`应用程序配置`文件夹。

而且，一般来说，几乎没有必要`私有=真`元素，但重写与Windows/VS“system”组件的冲突除外。一些包管理工具在进行版本更改时有时会感到困惑，有时会认为我们在一个解决方案中使用同一程序集的多个版本，当然，我们从来没有这样做过。

我们渴望有一天.NET包管理工具可以进行事务性版本更改！在此之前，偶尔需要通过手动编辑.csproj文件(它们只是XML文本文件)来“修复”某些.NET包管理工具的错误操作，并/或使用“Discard Edited Line”函数，这些函数是大多数优秀的Git工具，如[阿特拉斯资源树](https://www.sourcetreeapp.com/)提供。

使用“sort”引用和统一的组件版本可以避免在Orleans运行时组件和/或外部组件之间创建脆弱的链接，并且在过去几年中被证明在重要部署里程碑期间为Orleans团队降低压力水平非常有效。:)
