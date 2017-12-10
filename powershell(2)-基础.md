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

在powershell中一切都可以视为对象，包罗万象
New-Object可以创建一个对象
Add-Member可以添加属性和方法

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



