---
layout: page
title: Documentation Guidelines
---

# 文件编制指南

Orleans文档是内置的[降价](https://help.github.com/articles/markdown-basics/). 我们使用一些简单的约定来确保整个文档集中的样式一致。

这些标准正在引入。如果您对这些指导原则有异议，请提出问题或请求。如果发现文档不符合指导原则，请进行修复并提交请求。另外，如果你使用的是Windows10，你可以去商店找到免费的降价编辑器，比如[这](https://www.microsoft.com/store/apps/9wzdncrdd2p3)

# 结构

## 语言

文件将遵循美国英语拼写。桌面工具，如<http://markdownpad.com>和[Visual Studio代码](https://code.visualstudio.com/)有拼写检查功能。

## 段落结构

每句话都应该写在一行，每行只能写一句话。这使得合并更改更容易，并有助于识别冗长的语言。Markdown中的段落只是一行或多行连续文本，后跟一行或多行空白行。

## 标题

标题应该用来组织文件。避免使用其他强调功能，如ALLCAPS，*斜体字*或**大胆的**确定一个新的话题。使用标头不仅更加一致，而且允许链接到标头。

## 页脚

在页面的末尾，链接到文档中的下一个逻辑页面很有帮助。如果该页是子节中的最后一页，则链接回索引页是有用的。

# 风格

## 代码格式

示例代码块的格式应为后面紧跟着语言的三个反勾号格式。

```csharp
    [StorageProvider(ProviderName="store1")]
    public class MyGrain<IMyGrainState>
    {
      ...
    }
```

将呈现为

```csharp
[StorageProvider(ProviderName="store1")]
public class MyGrain<IMyGrainState> ...
{
  ...
}
```

内联代码应该用一个反勾号标记(\`).

其中包括以下参考：

-   键入名称，例如。`Task<T>`
-   变量名，例如。`游戏`
-   名称空间，例如。`Orleans。存储。AzureTablesStorage`

如果显示输出的文本(例如文本文件内容或控制台输出)，则可以使用后三勾号而不指定语言，也可以缩进内容。例如：

```
1 said: Welcome to my team!
0 said: Thanks!
1 said: Thanks!
0 said: Thanks!
```

## 文件名和路径

引用文件名、目录/文件夹或URI时，请使用标准斜体来格式化。这可以通过在字符串周围加上一个星号来完成(`*`)或者一个下划线(`_`)

示例：

-   *奥尔勒ansRuntimeInterfaces.dll*
-   *C： \\二进制文件*
-   *../src/Grains.cs*

## 桌子

标记支撑[表格数据](https://help.github.com/articles/github-flavored-markdown/#tables). 可以使用表来构造数据，以便读者可以轻松地使用这些数据。

| 后缀 | 单位 |
| --- | --- |
| 女士 | 毫秒 |
| s | 秒 |
| 米 | 分钟 |

## 链接

引用另一个概念时，请提供指向该概念的链接。页面中的正向和反向引用可以通过页眉进行链接。e、 g.链接回[结构](#结构)指向其他文档的链接既可以链接到页面，也可以链接到页面中的子节/页眉。外部链接应作为完整链接公开。例如<https://github.com/dotnet/roslyn>

# 贡献

Orleans文档作为降价文件管理在托管在上的Git存储库中[gh pages分部的GitHub](https://github.com/dotnet/orleans/tree/gh-pages). 见[GitHub页面](https://pages.github.com/)有关如何使用`gh页面`“项目现场”文件的分支约定。
