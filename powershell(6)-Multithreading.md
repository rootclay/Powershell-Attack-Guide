## powershell(6)-Multithreading

powershell的多线程是我们在使用powershell进行渗透过程中必须使用到的功能！为什么呢？试想，当你到达对方内网，你需要列出用户，或者下载文件等等操作的时候你是选择等待几天还是几分钟搞定呢？我们通过内存和CPU的占用来提高效率，也就是我们通常算法上说的用空间来换取时间。机器配置高，有的用，而不用就是浪费。

### powershell自带的Job

这里使用网上一个例子


```powershell
# 不使用多线程
$start = Get-Date
$code1 = { Start-Sleep -Seconds 5; 'A' }
$code2 = { Start-Sleep -Seconds 5; 'B'}
$code3 = { Start-Sleep -Seconds 5; 'C'}
 
$result1,$result2,$result3= (& $code1),(& $code2),(& $code3)
 
$end =Get-Date
$timespan= $end - $start
$seconds = $timespan.TotalSeconds
 
Write-Host "总耗时 $seconds 秒."
```

```powershell
# 使用多线程
$start = Get-Date
$code1 = { Start-Sleep -Seconds 5; 'A' }
$code2 = { Start-Sleep -Seconds 5; 'B'}
$code3 = { Start-Sleep -Seconds 5; 'C'}
 
$job1 = Start-Job -ScriptBlock $code1
$job2 = Start-Job -ScriptBlock $code2
$job3 = Start-Job -ScriptBlock $code3
 
$alljobs =  Wait-Job $job1,$job2,$job3
$result1,$result2,$result3 = Receive-Job $alljobs
 
$end =Get-Date
 
$timespan= $end - $start
$seconds = $timespan.TotalSeconds
Write-Host "总耗时 $seconds 秒."
```

那么可以测试到这两个脚本确实感觉上是使用了多线程，因为第二个版本使用时间只有9s左右的时间，但如果分来执行是需要15s的，就如第一个版本。那么这里是真的使用了多线程么？其实真实情况是多进程，最简单的查看方式，打开任务管理器，再执行脚本你可以看到多出3个powershell.exe的进程。

那么我们可以用这个多进程么？是可以用，但是需要注意每个进程都需要跨进程交换数据，而且没有节流的机制，所以我们还是来看看真正的多线程吧。

### 多线程

直接来看一段代码

```powershell
$code = { Start-Sleep -Seconds 2; "Hello" }
$newPowerShell = [PowerShell]::Create().AddScript($code)
$newPowerShell.Invoke()
```
这样我们通过powershell的API运行一个代码块，就算是在一个进程内执行了代码，不会创建新的进程。这是单线程，那么如何多线程呢？下面的代码就可以实现啦，那么测试过程中推荐windows的`process explorer`来查看进程对应的线程，可以清晰的看到创建的线程。

```powershell

# 设置线程限制为4，那么如果一起启动超过4线程就需要排队等待
$throttleLimit = 4
# 创建线程池
$SessionState = [system.management.automation.runspaces.initialsessionstate]::CreateDefault()
$Pool = [runspacefactory]::CreateRunspacePool(1, $throttleLimit, $SessionState, $Host)
$Pool.Open()
 
# 代码块
$ScriptBlock = {
    param($id)
 
    Start-Sleep -Seconds 2
    "Done processing ID $id"
}
 
$threads = @()

# 创建40个线程
$handles = for ($x = 1; $x -le 40; $x++) {
    $powershell = [powershell]::Create().AddScript($ScriptBlock).AddArgument($x)
    $powershell.RunspacePool = $Pool
    $powershell.BeginInvoke()
    $threads += $powershell
}
 
# 获取数据
do {
  $i = 0
  $done = $true
  foreach ($handle in $handles) {
    if ($handle -ne $null) {
      if ($handle.IsCompleted) {
        $threads[$i].EndInvoke($handle)
        $threads[$i].Dispose()
        $handles[$i] = $null
      } else {
        $done = $false
      }
    }
    $i++
  }
  if (-not $done) { Start-Sleep -Milliseconds 500 }
} until ($done)
```

大家可以试一试下面的代码和单独执行`get-hotfix`的速度差别：

```powershell
$throttleLimit = 40
$SessionState = [system.management.automation.runspaces.initialsessionstate]::CreateDefault()
$Pool = [runspacefactory]::CreateRunspacePool(1, $throttleLimit, $SessionState, $Host)
$Pool.Open()

$ScriptBlock = {
    get-HotFix
}
$threads = @()
$handles = for ($x = 1; $x -le 40; $x++) {
    $powershell = [powershell]::Create().AddScript($ScriptBlock)
    $powershell.RunspacePool = $Pool
    $powershell.BeginInvoke()
    $threads += $powershell
}

do {
  $i = 0
  $done = $true
  foreach ($handle in $handles) {
    if ($handle -ne $null) {
      if ($handle.IsCompleted) {
        $threads[$i].EndInvoke($handle)
        $threads[$i].Dispose()
        $handles[$i] = $null
      } else {
        $done = $false
      }
    }
    $i++
  }
  if (-not $done) { Start-Sleep -Milliseconds 500 }
} until ($done)
```

那么以后大家需要执行的代码就写在脚本块区域即可。这里和前面的爆破脚本结合起来就是一个完美的爆破脚本和信息收集脚本。