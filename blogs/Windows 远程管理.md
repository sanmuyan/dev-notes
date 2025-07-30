# Windows 远程管理

## Winrm 配置

### HTTP

```powershell
# 查看 winrm 状态
winrm enumerate winrm/config/listener
winrm get winrm/config
# 如何没有 listener 可以快速创建
winrm quickconfig

# 可选配置
# winrm set winrm/config/client '@{TrustedHosts = "*"}'
# winrm set winrm/config/service '@{AllowUnencrypted="true"}'

```

### HTTPS

```powershell
# 创建证书
$dnsName = $env:COMPUTERNAME
$cert = New-SelfSignedCertificate `
    -DnsName $dnsName `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -KeyUsage DigitalSignature,KeyEncipherment `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1") `
    -NotAfter (Get-Date).AddYears(10)

# 也可手动获取证书
# Get-ChildItem -Path Cert:\LocalMachine\My\
# $cert = Get-ChildItem -Path Cert:\LocalMachine\My\xxx

# 创建 listener
New-Item -Path WSMan:\LocalHost\Listener -Transport HTTPS -Address * -CertificateThumbPrint $Cert.Thumbprint –Force
# 删除 listener
winrm delete winrm/config/Listener?Address=*+Transport=HTTTS
```

## PowerShell

```powershell
# 开启远程配置
Enable-PSRemoting
# 查看远程配置
Get-PSSessionConfiguration
# 创建远程会话
New-PSSession -ComputerName 192.168.128.1 -Credential
```

## WORKGROUP

WORKGROUP 环境下的远程管理，以 Hyper-V 为例

`服务端`

```powershell
# 开启 CredSSP Server
Enable-WSManCredSSP -Role Server
# 查看 CredSSP Server 状态
Get-WSManCredSSP
This computer is configured to receive credentials from a remote client computer.
# 关闭 CredSSP Server
# Disable-WSManCredSSP -Role Server
```

`客户端`

1. 编辑 `组策略\管理模版\凭据分配\允许分配新的凭据用于仅 NTLM 服务器身份验证`服务器列表可以设置为`*`
2. 连接时勾选`作为另外一个用户连接`，比如 `xxx-server\administrator`
