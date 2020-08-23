---
layout: page
title: Using Azure Web Apps with Azure Cloud Services
---

# 将azure web应用与azure云服务一起使用

如果你想从[azure web应用](http://azure.microsoft.com/en-gb/services/app-service/web/)而不是托管在同一个云服务中的web角色。

要使其安全工作，您需要将azure web应用和托管silos的工作角色都分配给[azure虚拟网络](http://azure.microsoft.com/en-gb/services/virtual-network/)是的。

首先，我们将设置azure web应用程序，您可以跟随[本指南](https://azure.microsoft.com/en-us/blog/azure-websites-virtual-network-integration/)它将创建虚拟网络并将其分配给azure web应用程序。

现在我们可以通过修改`服务配置`文件。

```xml
<NetworkConfiguration>
  <VirtualNetworkSite name="virtual-network-name" />
  <AddressAssignments>
    <InstanceAddress roleName="role-name">
      <Subnets>
        <Subnet name="subnet-name" />
      </Subnets>
    </InstanceAddress>
  </AddressAssignments>
</NetworkConfiguration>
```

还要确保已配置silos终结点。

```xml
<Endpoints>
  <InternalEndpoint name="OrleansSiloEndpoint" protocol="tcp" port="11111" />
  <InternalEndpoint name="OrleansProxyEndpoint" protocol="tcp" port="30000" />
</Endpoints>
```

现在您可以从web应用连接到集群的其余部分。

### 潜在问题

如果Web应用无法连接到silos：

-   确保你至少有**两个角色**，或者在你的azure云服务中一个角色的两个实例，或者`内部终结点`不能生成防火墙规则。
-   检查Web应用程序和silos是否使用相同的`棒状的`和`服务ID`是的。
-   确保网络安全组设置为允许内部虚拟网络连接。如果您没有一个，您可以创建和分配一个很容易使用以下`PowerShell`以下内容：

```c
New-AzureNetworkSecurityGroup -Name "Default" -Location "North Europe"
Get-AzureNetworkSecurityGroup -Name "Default" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName "virtual-network-name" -SubnetName "subnet-name"
```
