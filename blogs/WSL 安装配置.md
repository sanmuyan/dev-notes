# WSL

## 安装和配置

```powershell
wsl --install
```

```powershell
cd $env:UserProfile
.wslconfig
[wsl2]
# wsl虚拟机cpu核数
processors=8

# wsl虚拟机内存
memory=4GB

swap=1GB

# 开启localhost 访问
localhostforwarding=true

# 关闭嵌套虚拟化
nestedVirtualization=false
```

```bash
/etc/wsl.conf
[automount]
# window 分区挂载文件权限掩码
options = "metadata,umask=022,fmask=033"

[interop]
# 是否开启互操作，也就是在linux使用win命令
enabled = true
appendWindowsPath = true

[boot]
# 开机启动命令一些想开机启动的命令可以放进来
command = /etc/init.d/start.sh
```

## 安装GUI应用

```bash
sudo apt update

# 安装常用软件
sudo apt install gedit nautilus -y

# 安装输入法
sudo apt install fcitx fcitx-googlepinyin -y

# 启动语言检查，可以完整一下中文环境
sudo echo "sudo /etc/init.d/dbus start" >> /etc/init.d/start.sh
sudo /etc/init.d/dbus start
sudo gnome-language-selector

# 安装edge浏览器
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo install -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/
sudo sh -c 'echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/edge stable main" > /etc/apt/sources.list.d/microsoft-edge.list'
sudo rm microsoft.gpg
sudo apt install microsoft-edge-stable -y
```

## 中文输入法

```bash
if [ $(ps -ef |grep fcitx-dbus-watcher |wc -l) -eq 1 ]; then
  fcitx-autostart >/tmp/fcitx.log 2>&1
fi

export GTK_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
export QT_IM_MODULE=fcitx
```

## 安装Docker

```bash
# 安装docker
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 自动启动
sudo echo "sudo /etc/init.d/dbus start" >>/etc/init.d/start.sh
```

## 设置固定IP

```powershell
netsh interface ip add address "vEthernet (WSL)" 192.168.128.254 255.255.255.0
```

```bash
sudo ip addr add 192.168.128.1/24 broadcast 192.168.128.255 dev eth0 label eth0:1

# 开机自动设置
sudo echo "ip addr add 192.168.128.1/24 broadcast 192.168.128.255 dev eth0 label eth0:1" >>/etc/init.d/start.sh
```
