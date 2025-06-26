# Iptables 流量日志记录

## 开启日志

```shell
iptables -A INPUT -m state --state NEW ! -d 127.0.0.1 -j LOG --log-prefix "INPUT_LOG "
```

## Rsyslog 简单配置

`/etc/rsyslog.d/10-iptables.conf`

```shell
if $msg contains 'INPUT_LOG' then {
   /var/log/iptables/input.log
   stop 
}    
```

## Rsyslog 格式化输出

`/etc/rsyslog.d/10-iptables.conf`

```shell
template (
    name="IptablesFormat"
    type="string"
    string="time=%timereported:::date-rfc3339% interface=%!usr!interface% src_ip=%!usr!src_ip% dst_ip=%!usr!dst_ip% proto=%!usr!proto% scr_port=%!usr!src_port% dst_port=%!usr!dst_port%\n"
) {
    property(name="!usr!interface")
    property(name="!usr!src_ip")
    property(name="!usr!dst_ip")
    property(name="!usr!proto")
    property(name="!usr!src_port")
    property(name="!usr!dst_port")
}


if $msg contains 'INPUT_LOG' then {
    set $!usr!proto = re_extract($msg, "PROTO=([A-Za-z0-9]+)\\b", 0, 1, "0");
    set $!usr!src_ip = re_extract($msg, "SRC=([0-9.]+)\\b", 0, 1, "0");
    set $!usr!dst_ip = re_extract($msg, "DST=([0-9.]+)\\b", 0, 1, "0");
    set $!usr!src_port = re_extract($msg, "SPT=([0-9.]+)\\b", 0, 1, "0");
    set $!usr!dst_port = re_extract($msg, "DPT=([0-9.]+)\\b", 0, 1, "0");
    set $!usr!interface = re_extract($msg, "IN=([A-Za-z0-9]+)\\b", 0, 1, "0");

    action(type="omfile" file="/var/log/iptables/input.log" template="IptablesFormat")
    stop
}
```

### 日志效果

```shell
time=2025-06-26T23:28:50.904042+08:00 interface=eth0 src_ip=192.168.1.1 dst_ip=192.168.1.2 proto=TCP scr_port=45546 dst_port=80
```

## 日志分割

`/etc/logrotate.d/iptables`

```shell
/var/log/iptables/*.log {
        daily
        rotate 7
        missingok
        dateext
        postrotate
                systemctl restart rsyslog
        endscript
}


# 加入计划任务
crontab -e

0 0 * * * logrotate /etc/logrotate.d/iptables

```
