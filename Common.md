# Powershell常用的命令与场景


winrm

```powershell
# Victim: Enable-PSRemoting

# Enable winrm/psremoting
Enable-PSRemoting # quick config

# Add remote host to trusted hosts:
Set-Item wsman:\localhost\client\trustedhosts 10.0.0.2
Test-WsMan 10.0.0.2 # test connection to remote host

# Check current remote session's permissions
Get-PSSessionConfiguration | Format-Table -Property Name, Permission -Auto
Set-PSSessionConfiguration -Name Microsoft.ServerManager -AccessMode Remote -Force

# Open remote session
Enter-PSSession -ComputerName 10.0.0.2 -Credential Domain\Username -Authentication Default
#$SS = New-PSSession -ComputerName 10.0.0.2 -Credential Domain\Username -Authentication Default
#Get-PSSession Remove-PSSession
#Invoke-Command -Session $SS -ScriptBlock {Get-Culture}
# Enter/New-PSSession -SkipCACheck -SkipCNCheck -UseSSL

# Disable remoting in powershell
Disable-PSRemoting
#Stop-Service winrm
#Set-Service -Name winrm -StartupType Disabled
```

powershell BitsTransfer

```powershell
# PowerShell
 # Import-Module BitsTransfer
Start-BitsTransfer -Priority Foreground -Source "https://nmap.org/ncrack/dist/*.tar.gz" -Destination "C:\temp\"

$Cred = Get-Credential
Start-BitsTransfer -Authentication ntlm -Credential $Cred -Priority Foreground -Source "\\192.168.1.51\a\test.txt" -Destination "C:\temp\test.txt"
```

获取补丁：`get-hotfix | out-gridview`

SDDL操作

```powershell
$sddl = "D:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCLCSWRPWPLORC;;;SO)(A;;CCLCSWRPLORC;;;IU)(A;;CCLCSWRPWPDTLOCRRC;;;LS)(A;;CCLCSWRPWPDTLOCRRC;;;NS)"
$ACLObject = New-Object -TypeName System.Security.AccessControl.DirectorySecurity
$ACLObject.SetSecurityDescriptorSddlForm($sddl)
$ACLObject.Access
```

创建凭据

```powershell
# get credentials interactive:
$creds = Get-Credential

# non-interactive
$secpasswd = ConvertTo-SecureString "PlainTextPassword" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ("username", $secpasswd)
```

RDP开启：`powershell.exe -w hidden -nop -c "reg add \"HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\" /v fDenyTSConnections /t REG_DWORD /d 0 /f; if($?) {$null = netsh firewall set service type = remotedesktop mod = enable;$null = reg add \"HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\" /v UserAuthentication /t REG_DWORD /d 0 /f }"`


Disable RDP: `powershell.exe -w hidden -nop -c "reg add \"HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\" /v fDenyTSConnections /t REG_DWORD /d 1 /f; if ($?) { $null = reg add \"HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\" /v UserAuthentication /t REG_DWORD /d 1 /f }"`



```powershell
```