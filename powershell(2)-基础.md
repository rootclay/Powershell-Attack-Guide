# powershell(2)-基础


> 本节主要讲一下关于powershell一些简单的基础知识，推荐网站[http://www.pstips.net/](http://www.pstips.net/)学习Powershell的一些基础知识

这里是一些简单的基础，写的可能有些简陋，这里可能需要你有一些编程语言的基础就能看懂啦，这里对于后面的代码分析是非常有用的，所以还是希望大家简单的浏览一下基础知识。

## 变量

变量都是以`$`开头, 是强类型语言, 语言是大小写不敏感的

提一提变量保护与常量的声明：`New-Variable num -Value 100 -Force -Option readonly`这样就得到一个受保护的变量`$num`，如果要销毁它只能通过`del $num`删除。如果要声明常量则用`New-Variable num -Value 100 -Force -Option constant`


## 数组

### 数组的创建：
数组的创建可以通过下面五种方式来创建，在适当的条件下选择适当的方式创建即可

```powershell
$array = 1,2,3,4
$array = 1..4
$array=1,"2017",([System.Guid]::NewGuid()),(get-date)
$a=@()  # 空数组
$a=,"1" # 一个元素的数组
```

### 数组的访问
数组的访问和C类似，第一位元素实用下标0来访问即`$array[0]`,我们来看看ipconfig获取到的数据

```powershell
$ip = ipconfig
$ip[1] # 获取ipconfig第二行的数据
```

### 数组的判断
`$test -is [array]`

### 数组的追加:
`$books += "元素4"`



## 哈希表

### 哈希表的创建：
`$stu=@{ Name = "test";Age="12";sex="man" }`
### 哈希表里存数组：
`$stu=@{ Name = "hei";Age="12";sex="man";Books="kali","sqlmap","powershell" }`

### 哈希表的插入与删除:
```powershell
$Student=@{}
$Student.Name="hahaha"
$stu.Remove("Name")
```

## 对象

在powershell中一切都可以视为对象，包罗万象。
### 查看对象结构 (Get-Member)
由于对象在 Windows PowerShell 中扮演了如此重要的角色，因此存在几个用于处理任意对象类型的本机命令。 最重要的一个是 Get-Member 命令。

```powershell
Get-Process | Get-Member | Out-Host -Paging
TypeName: System.Diagnostics.Process

Name                           MemberType     Definition
----                           ----------     ----------
Handles                        AliasProperty  Handles = Handlecount
Name                           AliasProperty  Name = ProcessName
NPM                            AliasProperty  NPM = NonpagedSystemMemorySize
PM                             AliasProperty  PM = PagedMemorySize
VM                             AliasProperty  VM = VirtualMemorySize
WS                             AliasProperty  WS = WorkingSet
add_Disposed                   Method         System.Void add_Disposed(Event...
...
```
我们可以通过筛选想要查看的元素，让这个冗长的信息列表更易于使用。 Get-Member 命令仅允许你列出属性成员。 属性的形式有数种。 如果将 Get-Member MemberType 参数设置为值属性，则 cmdlet 将显示任何类型的属性 。 生成的列表仍会很长，但较之前更易于管理：

```powershell
PS> Get-Process | Get-Member -MemberType Properties

   TypeName: System.Diagnostics.Process

Name                       MemberType     Definition
----                       ----------     ----------
Handles                    AliasProperty  Handles = Handlecount
Name                       AliasProperty  Name = ProcessName
...
ExitCode                   Property       System.Int32 ExitCode {get;}
...
Handle                     Property       System.IntPtr Handle {get;}
...
CPU                        ScriptProperty System.Object CPU {get=$this.Total...
...
Path                       ScriptProperty System.Object Path {get=$this.Main...
...
```

### 选择对象部件 (Select-Object)
可以使用 Select-Object cmdlet 创建新的自定义 PowerShell 对象（包含从用于创建它们的对象中选择的属性）。 键入下面的命令以创建仅包括 Win32_LogicalDisk WMI 类的 Name 和 FreeSpace 属性的新对象：
```powershell
Get-CimInstance -Class Win32_LogicalDisk | Select-Object -Property Name,FreeSpace
Name      FreeSpace
----      ---------
C:      50664845312
```

可以使用 Select-Object 创建计算属性。 这样即可以以十亿字节为单位显示 FreeSpace，而非以字节为单位。

```PowerShell
Get-CimInstance -Class Win32_LogicalDisk |
  Select-Object -Property Name, @{
    label='FreeSpace'
    expression={($_.FreeSpace/1GB).ToString('F2')}
  }
  
Name    FreeSpace
----    ---------
C:      47.18
```
### 从管道中删除对象 (Where-Object)
在 PowerShell 中，你通常会生成和传递比预期更多的对象到管道中。 可以通过使用 Format-* cmdlet 指定特定对象的属性进行显示，但是这对从显示中删除整个对象的问题没有任何帮助。 你可能希望在管道末尾之前筛选对象，以便你可以只对初始生成对象的子集执行操作。
#### 使用 Where-Object 执行简单测试
借助 PowerShell 中的 Where-Object cmdlet，可以测试管道中的每个对象，并沿管道仅传递满足特定测试条件的对象。 将从管道中删除未通过测试的对象。 测试条件以 FilterScript 参数值的形式提供。
比较运算符	含义	示例（返回 True）
出于分析考虑，<、> 和 = 等符号不用作比较运算符。 相反，比较运算符由字母组成。比如-eq、-ne、-like等等。
Where-Object 脚本块使用特殊变量 $_ 来指代管道中的当前对象。 以下是其工作原理示例。 如果你有一个数字列表，且希望仅返回小于 3 的数字，则可使用 Where-Object 通过键入以下内容来筛选数字：

```powershell
1,2,3,4 | Where-Object {$_ -lt 3}
1
2
```

#### 基于对象属性进行筛选
由于 $_ 指代当前管道对象，因此可以访问它的属性进行测试。
例如，我们可以看看 WMI 中的 Win32_SystemDriver 类。 一个特定的系统上可能有数百个系统驱动程序，但是你可能只对特定一些系统驱动程序感兴趣，例如那些当前正在运行的程序。 对于 Win32_SystemDriver 类，相关属性是 State 。 你可以筛选系统驱动程序，通过键入以下内容仅选择正在运行的驱动程序：

```powershell
Get-CimInstance -Class Win32_SystemDriver |
  Where-Object {$_.State -eq 'Running'}
```
这仍会生成一个较长的列表。 你可能还希望进行筛选，以通过测试 StartMode 值仅选择自动启动的驱动程序集：

```powershell
DisplayName : RAS Asynchronous Media Driver
Name        : AsyncMac
State       : Running
Status      : OK
Started     : True

DisplayName : Audio Stub Driver
Name        : audstub
State       : Running
Status      : OK
Started     : True
...
```
这为我们提供了大量不再需要的信息，因为我们知道驱动程序正在运行。 事实上，此时我们可能需要的唯一信息就是名称和显示名。 下面的命令仅包括这两种属性，从而使输出更简单：

```powershell
Get-CimInstance -Class Win32_SystemDriver |
  Where-Object {$_.State -eq "Running"} |
    Where-Object {$_.StartMode -eq "Manual"} |
      Format-Table -Property Name,DisplayName
      
Name              DisplayName
----              -----------
AsyncMac               RAS Asynchronous Media Driver
bindflt                Windows Bind Filter Driver
bowser                 Browser
CompositeBus           Composite Bus Enumerator Driver
condrv                 Console Driver
HdAudAddService        Microsoft 1.1 UAA Function Driver for High Definition Audio Service
HDAudBus               Microsoft UAA Bus Driver for High Definition Audio
HidUsb                 Microsoft HID Class Driver
HTTP                   HTTP Service
igfx                   igfx
IntcDAud               Intel(R) Display Audio
intelppm               Intel Processor Driver
...
```

### 创建 .NET 对象
存在具有 .NET Framework 和 COM 接口的软件组件，使用它们可执行许多系统管理任务。 Windows PowerShell 允许你使用这些组件，因此你将不限于执行可通过使用 cmdlet 执行的任务。 Windows PowerShell 初始版本中的许多 cmdlet 对远程计算机无效。 我们将演示如何通过直接从 Windows PowerShell 使用 .NET Framework System.Diagnostics.EventLog 类在管理事件日志时绕过此限制。
#### 使用 New-Object 进行事件日志访问
.NET Framework 类库包括一个名为 System.Diagnostics.EventLog 的类，该类可用于管理事件日志。 可以通过使用具有 TypeName 参数的 New-Object cmdlet 创建 .NET Framework 类的新实例。 例如，以下命令将创建事件日志引用：

```powershell
PS> New-Object -TypeName System.Diagnostics.EventLog

  Max(K) Retain OverflowAction        Entries Name
  ------ ------ --------------        ------- ----
```
尽管该命令创建了 EventLog 类的实例，但该实例不包含任何数据。 这是因为我们未指定特定的事件日志。 如何获取真正的事件日志？
##### 将构造函数与 New-Object 一起使用
若要引用特定的事件日志，需要指定日志的名称。 New-Object 具有 ArgumentList 参数。 作为值传递到此形参的实参将由对象的特殊的启动方法使用。 此方法叫做构造函数，因为它将用于构造对象。 例如，若要对获取应用程序日志的引用，请指定字符串“Application”作为实参

```powershell
PS> New-Object -TypeName System.Diagnostics.EventLog -ArgumentList Application

Max(K) Retain OverflowAction        Entries Name
------ ------ --------------        ------- ----
16,384      7 OverwriteOlder          2,160 Application
```
##### 在变量中存储对象
你可能需要存储对对象的引用，以便在当前的 Shell 中使用。 尽管 Windows PowerShell 允许使用管道执行大量操作，减少了对变量的需求，但有时在变量中存储对对象的引用可以更方便地操纵这些对象。
Windows PowerShell 允许你创建实质上是命名对象的变量。 来自任何有效 Windows PowerShell 命令的输出都可以存储在变量中。 变量名始终以 $ 开头。 如果想要将应用程序日志引用存储在名为 $AppLog 的变量中，请键入该变量的名称，后跟一个等号，然后键入用于创建应用程序日志对象的命令：

```powershell
PS> $AppLog = New-Object -TypeName System.Diagnostics.EventLog -ArgumentList Application

PS> $AppLog

  Max(K) Retain OverflowAction        Entries Name
  ------ ------ --------------        ------- ----
  16,384      7 OverwriteOlder          2,160 Application
```

##### 使用 New-Object 访问远程事件日志
上一节中使用的命令以本地计算机为目标；Get-EventLog cmdlet 可做到这一点。 若要访问远程计算机上的应用程序日志，必须同时将日志名称和计算机名称（或 IP 地址）作为参数提供。

```powershell
PS> $RemoteAppLog = New-Object -TypeName System.Diagnostics.EventLog Application,192.168.1.81
PS> $RemoteAppLog

  Max(K) Retain OverflowAction        Entries Name
  ------ ------ --------------        ------- ----
     512      7 OverwriteOlder            262 Application
```

##### 使用对象方法清除事件日志
对象通常具有可调用以执行任务的方法。 可以使用 Get-Member 来显示与对象关联的方法。 下面的命令和已选的输出将显示 EventLog 类的一些方法：

```powershell
PS> $RemoteAppLog | Get-Member -MemberType Method

   TypeName: System.Diagnostics.EventLog

Name                      MemberType Definition
----                      ---------- ----------
...
Clear                     Method     System.Void Clear()
Close                     Method     System.Void Close()
...
GetType                   Method     System.Type GetType()
...
ModifyOverflowPolicy      Method     System.Void ModifyOverflowPolicy(Overfl...
RegisterDisplayName       Method     System.Void RegisterDisplayName(String ...
...
ToString                  Method     System.String ToString()
WriteEntry                Method     System.Void WriteEntry(String message),...
WriteEvent                Method     System.Void WriteEvent(EventInstance in...
```
Clear() 方法可以用于清除事件日志。 调用方法时，即使该方法不需要参数，也必须始终在方法名称后紧跟括号。 这使得 Windows PowerShell 方法能够区分该方法和具有相同名称的潜在属性。 键入以下命令以调用 Clear 方法：

```powershell
PS> $RemoteAppLog.Clear()
```

### 使用 New-Object 创建 COM 对象
可以使用 New-Object 来处理组件对象模型 (COM) 组件。 组件的范围从 Windows 脚本宿主 (WSH) 包含的各种库到 ActiveX 应用程序（如大多数系统上安装的 Internet Explorer）。

New-Object 使用 .NET Framework 运行时可调用的包装器创建 COM 对象，因此调用 COM 对象时它与 .NET Framework 具有相同的限制。 若要创建 COM 对象，需要为 ComObject 参数指定要使用的 COM 类的编程标识符（或 ProgId）。 COM 用途限制的全面讨论和确定系统上可用的 ProgId 已超出本用户指南的范围，但来自环境的大多数已知对象（如 WSH）都可在 Windows PowerShell 内使用。

可以通过指定以下 progid 来创建 WSH 对象：WScript.Shell 、WScript.Network 、Scripting.Dictionary 和 Scripting.FileSystemObject 。 以下命令将创建这些对象：

```powershell
New-Object -ComObject WScript.Shell
New-Object -ComObject WScript.Network
New-Object -ComObject Scripting.Dictionary
New-Object -ComObject Scripting.FileSystemObject
```
尽管在 Windows PowerShell 中可通过其他方法使用这些类的大多数功能，但使用 WSH 类执行某些任务（如创建快捷方式）仍更加简单。

#### 使用 WScript.Shell 创建桌面快捷方式
可以使用 COM 对象快速执行的一个任务是创建快捷方式。 假设你想要在桌面上创建链接到 Windows PowerShell 主文件夹的快捷方式。 首先需要创建对 WScript.Shell 的引用，我们会将其存储在名为 $WshShell 的变量中：

```powershell
$WshShell = New-Object -ComObject WScript.Shell
```
Ge-Member 可用于 COM 对象，因此你可以通过键入以下内容浏览对象的成员：

```powershell
PS> $WshShell | Get-Member

   TypeName: System.__ComObject#{41904400-be18-11d3-a28b-00104bd35090}

Name                     MemberType            Definition
----                     ----------            ----------
AppActivate              Method                bool AppActivate (Variant, Va...
CreateShortcut           Method                IDispatch CreateShortcut (str...
...
```
Get-Member 具有可选 InputObject 参数，你可以使用这个参数而不使用管道为 Get-Member 提供输入。 如果改用命令 Get-Member-InputObject $WshShell，你会得到与如上所示相同的输出。 如果使用 InputObject，它将视其参数为单个项。 这意味着，如果变量中有几个对象，那么 Get-Member 会将它们视为一个对象数组。 例如：

```powershell
PS> $a = 1,2,"three"
PS> Get-Member -InputObject $a
TypeName: System.Object[]
Name               MemberType    Definition
----               ----------    ----------
Count              AliasProperty Count = Length
...
```
WScript.Shell CreateShortcut 方法接受单个参数，即要创建的快捷方式文件的路径。 我们可以键入桌面的完整路径，但还有更简单的方法。 桌面通常由当前用户的主文件夹内名为 Desktop 的文件夹表示。 Windows PowerShell 具有变量 $Home，它包含此文件夹的路径。 我们可以通过使用此变量指定主文件夹的路径，然后通过键入以下内容添加 Desktop 文件夹的名称和要创建的快捷方式的名称：

```powershell
$lnk = $WshShell.CreateShortcut("$Home\Desktop\PSHome.lnk")
```
当你在双引号内使用外观类似变量名称的项时，Windows PowerShell 将尝试替换匹配的值。 如果使用单引号，Windows PowerShell 将不会替换该变量值。 例如，请尝试键入以下命令：

```powershell
PS> "$Home\Desktop\PSHome.lnk"
C:\Documents and Settings\aka\Desktop\PSHome.lnk
PS> '$Home\Desktop\PSHome.lnk'
$Home\Desktop\PSHome.lnk
```
我们现在有一个名为 $lnk 的变量，其中包含新的快捷方式引用。 如果想要查看其成员，你可以通过管道将它传递到 Get-Member。 下面的输出显示了完成创建快捷方式所需使用的成员：

```powershell
PS> $lnk | Get-Member
TypeName: System.__ComObject#{f935dc23-1cf0-11d0-adb9-00c04fd58a0b}
Name             MemberType   Definition
----             ----------   ----------
...
Save             Method       void Save ()
...
TargetPath       Property     string TargetPath () {get} {set}
```

我们需要指定 TargetPath（它是 Windows PowerShell 的应用程序文件夹），然后通过调用 Save 方法保存快捷方式 $lnk。 Windows PowerShell 应用程序文件夹路径存储在变量 $PSHome 中，因此我们可以通过键入以下内容执行此操作：

```powershell
$lnk.TargetPath = $PSHome
$lnk.Save()
```
## 控制语句
### 条件判断


#### 比较运算符
```powershell
-eq ：等于
-ne ：不等于
-gt ：大于
-ge ：大于等于
-lt ：小于
-le ：小于等于
-contains ：包含
$array -contains something

-notcontains :不包含
!($a): 求反
-and ：和
-or ：或
-xor ：异或
-not ：逆
```

#### if-else
```powershell
if-else:

if($value -eq 1){
	code1
}else{
	code2
}
```

### 循环语句

#### while
```powershell
while($n -gt 0){
	code
}
```

#### for
```powershell
$sum=0
for($i=1;$i -le 100;$i++)
{
    $sum+=$i
}
$sum
```
#### foreach
```powershell
# 打印出windows目录下大于1mb的文件名
foreach($file in dir c:\windows)
{
    if($file.Length -gt 1mb)
    {
        $File.Name
    }
}
```

#### foreach-object
```powershell
# 获取所有的服务，并获取对呀进程ID是否大于100
Get-WmiObject Win32_Service | ForEach-Object {"Name:"+ $_.DisplayName, ", Is ProcessId more than 100:" + ($_.ProcessId -gt 100)}
```
## 函数

```powershell
function Invoke-PortScan {
<#
.SYNOPSIS 
简介

.DESCRIPTION
描述
    
.PARAMETER StartAddress
参数

.PARAMETER EndAddress
参数

.EXAMPLE
PS > Invoke-PortScan -StartAddress 192.168.0.1 -EndAddress 192.168.0.254
用例
#>
code
}
```



## 异常处理

Try{
	$connection.open()
	$success = $true
}Catch{
	$success = $false
}



