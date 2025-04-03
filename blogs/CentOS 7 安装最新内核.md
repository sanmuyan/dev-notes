# CentOS 7 安装最新内核

```shell
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm -y

yum --disablerepo="*" --enablerepo="elrepo-kernel" list available

yum --enablerepo=elrepo-kernel install  kernel-ml-devel kernel-ml -y


awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg

grub2-set-default 0

grub2-mkconfig -o /boot/grub2/grub.cfg

reboot

```
