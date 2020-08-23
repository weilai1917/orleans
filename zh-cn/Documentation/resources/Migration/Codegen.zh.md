---
layout: page
title: Code Generation in Orleans 2.0
---

# Orleans2.0中的代码生成

在Orleans 2.0中，代码生成得到了改进，从而缩短了启动时间并提供了更具确定性和可调试性的体验。与早期版本一样，Orleans提供生成时和运行时代码生成。

1.  **在构建期间**-这是推荐的选项，仅支持C＃项目。在这种模式下，代码生成将在每次编译项目时运行。将一个构建任务注入到项目的构建管道中，并在项目的中间输出目录中生成代码。要激活此模式，请添加其中一个软件包`Microsoft.Orleans.CodeGenerator.MSBuild`要么`Microsoft.Orleans.OrleansCodeGenerator.Build`所有包含Grains，Grain接口，序列化器或需要序列化器的类型的项目。包和其他代码生成信息之间的差异可以在相应的文件中找到[代码生成](../../grains/code_generation.md)部分。通过指定以下值可以在构建时发出其他诊断：`OrleansCodeGenLogLevel`在目标项目的*csproj*文件。例如，`<OrleansCodeGenLogLevel>跟踪</ OrleansCodeGenLogLevel>`。

2.  **配置期间**-这是F＃，Visual Basic和其他非C＃项目唯一受支持的选项。此模式在配置阶段生成代码。要访问它，请参阅配置文档。

两种模式都生成相同的代码，但是运行时代码生成只能为公共可访问类型生成代码。
