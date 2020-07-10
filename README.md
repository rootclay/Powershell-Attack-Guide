# Powershell-Attack-Guide
Powershell攻击指南----黑客后渗透之道

## 前言
> 时隔许久再来更新曾经的文章，对其中一些知识点重新理解记录。（2020-4-13）另外，重新更新了更可读的gitbook：https://rootclay.gitbook.io/powershell-attack-guide/

> 本文首发于安全客，原文专题页面:https://www.anquanke.com/subject/id/90541

> 一段时间以来研究Powershell，后来应朋友们对Powershell的需求，让我写一个Powershell安全入门或者介绍方面的文章，所以这篇文章就出现了。但又因为各种各样的事情搞得有些拖延，同时作者文笔不好，文章可能有不流畅的地方，还请多多见谅。这里做一些总结，来让新人对此有个大致了解，能对Powershell或是内网有更多的理解。

> 那么开始之前我们先来思考一下powershell一个常见的问题，那么我们知道powershell的后缀是ps1，哪为什么是ps1而不是ps2,ps3呢？那么理解这个问题呢我们可以看看powershell的特性，powershell是对下完全兼容的，也就是说你使用powershell 5.x的版本来执行powershell v1.0的代码也是完全没有问题的。那么我个人理解一下为什么是ps1，可以这么说，当我们见到ps2后缀之时就是powershell进行大的更新，也就是不对下兼容的时候，所以这里一直是使用ps1后缀。


> 那么对于我们的安全人员来说我们用什么版本呢？毫无疑问是v2,为什么呢，应为在win7当中默认安装了v2,而且之后的版本都是兼容v2的，v1版本所有的功能对于我们的需求很多都不能瞒住，所以v2成为了我们目前来说独一无二的选择，通过下面的方式我们可以看到我们的powershell的版本与一些详细的信息，后面我们的代码，大多都是以v2.0来讨论的。（经过最新的修改后可能部分功能用到最新的Powershell7.0）



通过命令`Get-Host`可以获取当前的PS版本信息等

```powershell
Name             : ConsoleHost
Version          : 2.0
InstanceId       : 388599a6-35cd-4bba-bedb-cf00d2a39389
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : zh-CN
CurrentUICulture : en-US
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace
``` 

对于安全人员学习ps主要有以下两个场景：

1. 第一种我们需要获得免杀或者更好的隐蔽攻击对方的win机器，可以通过钓鱼等方式直接执行命令。
2. 第二种我们已经到了对方网络，再不济也是一台DMZ的win-server，那么我们利用ps做的事情那么自然而然的是对内网继续深入。

那么本powershell系列主要是内容涉及和安全测试相关的内容，所以面向的读者主要是安全或者运维人员，不管你是在网络世界中扮演什么角色，在这里应该是能收获到你想要的。文章主要包含下面一些内容:

1. powershell基础语法
2. powershell脚本编写与调用执行
3. powershell的Socket编程
4. powershell端口扫描与服务爆破
5. powershell多线程
6. powershell操作wmi与.net
7. powershell操作win32API
8. powershell操作Dll注入&shellcode注入&exe注入
9. powershell混淆
10. powershell事件日志
11. powershell实例使用场景
12. Powershell渗透工具集
