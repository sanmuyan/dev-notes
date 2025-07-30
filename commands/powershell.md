# PowerShell

## 基本操作

```powershell
rm
mv
ls
cd
cp
echo
history
mkdir
# 获取命令
Get-Command -Name *Process*
# 历史记录
cat $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt |Select-String env
```

## 环境变量

```powershell
# 查看变量
ls env:
ls env:ENV*
$env:ENV_NAME
# 设置变量
$env:ENV_NAME = "dev" 
# 设置永久环境变量
[Environment]::SetEnvironmentVariable("ENV_NAME","dev","User")
[Environment]::GetEnvironmentVariable("ENV_NAME")
# 关闭更新提示
[System.Environment]::SetEnvironmentVariable("POWERSHELL_UPDATECHECK", "Off", "Machine")
```

## 网络设置

```powershell
# 设置IP
netsh interface ip add address "vEthernet (WSL)" 192.168.128.254 255.255.255.0
# 设置路由
route -p add 157.0.0.0 MASK 255.0.0.0  157.55.80.1
route delete 157.0.0.0
# 设置专用网
Get-NetConnectionProfile
Set-netconnectionprofile -InterfaceIndex 23 -NetworkCategory Private
# 端口转发
netsh interface portproxy add v4tov4 listenport=6379 listenaddress=0.0.0.0 connectport=6379 connectaddress=192.168.128.1
```

## 进程管理

```powershell
# 获取进程
Get-Process -Id 99
# 搜索进程
Get-Process -Name *main*
# 停止进程
Stop-Process -Name Idle
Stop-Process -Name t*,e* -Confirm

```

## 虚拟机

```powershell
# 嵌套虚拟化
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true  
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## 电脑管理

```powershell
# 睡眠
rundll32.exe powrprof.dll SetSuspendState 0,1,0
# 锁屏
rundll32.exe user32.dll,LockWorkStation
# 注销
shutdown.exe -l
# 电脑状态
Stop-Computer
Restart-Computer
Restart-Computer -Force
```
