---
layout: page
title: PowerShell Client Module
---

# powershell客户端模块

Orleans Powershell客户端模块是一组[powershell cmdlet](https://technet.microsoft.com/en-us/library/dd772285.aspx)那包裹着[grains客户端](https://github.com/dotnet/orleans/blob/master/src/Orleans/Core/GrainClient.cs)在一组方便的命令中，不仅可以与[管理grain](https://github.com/dotnet/orleans/blob/master/src/Orleans.Runtime/Core/ManagementGrain.cs)但是任何`伊格拉因`就像普通的Orleans应用程序可以通过使用powershell脚本一样。

这些cmdlet通过利用powershell脚本，从开始维护任务、测试、监视或任何其他类型的自动化，启用一系列场景。

使用方法如下：

## 安装模块

### 来源

您可以从源代码构建`奥利恩斯普提尔斯`投射并导入：

```powershell
PS> Import-Module .\projectOutputDir\Orleans.psd1
```

尽管您可以这样做，但是有一种更简单有趣的方法可以通过从**powershell库**是的。

### 来自powershell库

现在的powershell模块很容易像nuget包一样共享，但是它们托管在[powershell库](https://www.powershellgallery.com/)是的。

-   要将其安装到特定文件夹，请运行：

```powershell
PS> Save-Module -Name OrleansPSUtils -Path <path>
```

-   在powershell模块路径上安装（**推荐的方法**），只需运行：

```powershell
PS> Install-Module -Name OrleansPSUtils
```

-   如果您计划在[azure自动化](https://azure.microsoft.com/en-us/services/automation/)，只需单击下面的按钮：<button style="border:none;background-image:none; background-color:transparent " type="button" title="Deploy this module to Azure Automation." onclick="window.open('https://www.powershellgallery.com/packages/Orleans/DeployItemToAzureAutomation?itemType=PSModule', target = '_blank')">
    	<img src="https://www.powershellgallery.com/Content/Images/DeployToAzureAutomationButton.png">
    </button>

## 使用模块

无论您决定以何种方式安装它，要实际使用它，首先需要在当前的powershell会话中导入模块，以便通过运行以下命令使cmdlet可用：

```powershell
PS> Import-Module OrleansPSUtils
```

**注意**：如果是从源代码生成，则必须使用`.psd1段`而不是使用模块名，因为它不在`$env:psmodulepath路径`powershell运行时变量。同样，强烈建议您改为从powershell库安装。

导入模块（这意味着该模块已加载到powershell会话中）后，将有以下cmdlet可用：

-   `启动GrainClient`
-   `停止GrainClient`
-   `获取Grains`

#### 启动GrainClient

这个模块是一个包装器`grainclient.initialize（）`以及它的超载。

**用法**以下内容：

-   **`启动GrainClient`**

    -   与Call相同`grainclient.initialize（）`它将查找已知的Orleans客户端配置文件名

-   **`启动grainclient[-configfilepath]<string>[[-timeout]<timespan>]`**

    -   将使用提供的文件路径，如中所示`grainclient.initialize（文件路径）`

-   **`启动grainclient[-configfile]<fileinfo>[[-timeout]<timespan>]`**

    -   使用`系统文件信息`类表示配置文件，就像`grainclient.initialize（文件信息）`

-   **`启动grainclient[-config]<clientconfiguration>[[-timeout]<timeSPAN>]`**

    -   使用`Orleans.runtime.configuration.clientconfiguration`就像在`grainclient.initialize（配置）`

-   **`启动grainclient[-gatewayaddress]<ipendpoint>[[-overrideconfig]<bool>[[-timeout]<timespan>]`**

    -   采用Orleans群集网关地址终结点

**注意**：的`超时`参数是可选的，如果它被告知并且大于`System.TimeSpan.Zero系统.TimeSpan.Zero`，它将调用`orleans.grainclient.setResponseTimeout（超时）`在内部。

#### 停止GrainClient

不接受任何参数，当调用时，如果`grains客户端`初始化将正常取消初始化。

#### 获取Grains

包装物`grainclient.grainfactory.getgrain<t>（）`以及它的超载。

强制参数是`-粒状`以及`-xxx键`对于Orleans支持的当前Grains密钥类型（`一串`，请`指导方针`我是说，`长的`）还有`-键扩展`可以用在有复合键的Grains上。

此cmdlet返回作为上的参数传递的类型的粒度引用`-粒状`是的。

## 例子：

一个简单的调用示例`MyInterfacesNamespace.IMyGrain.SayHeloTo`grains法：

```powershell
PS> Import-Module OrleansPSUtils
PS> $configFilePath = Resolve-Path(".\ClientConfig.xml").Path
PS> Start-GrainClient -ConfigFilePath $configFilePath
PS> Add-Type -Path .\MyGrainInterfaceAssembly.dll
PS> $grainInterfaceType = [MyInterfacesNamespace.IMyGrain]
PS> $grainId = [System.Guid]::Parse("A4CF7B5D-9606-446D-ACE9-C900AC6BA3AD")
PS> $grain = Get-Grain -GrainType $grainInterfaceType -GuidKey $grainId
PS> $message = $grain.SayHelloTo("Gutemberg").Result
PS> Write-Output $message
Hello Gutemberg!
PS> Stop-GrainClient
```

我们计划更新这个页面，因为我们在powershell上引入了更多的cmdlet，如useobservators、streams和其他orleans核心特性。我们希望这能帮助人们成为自动化的起点。一如既往，这是一项正在进行的工作，我们热爱贡献！：）

请注意，目的不是在powershell上重新实现整个客户端，而是给it和devops团队一种与grains交互的方式，而无需实现一个.net应用程序。
