# Iptables 流量日志记录

## 开启日志

```shell
iptables -A FORWARD -j LOG --log-prefix " FORWARD: "
```

## Rsyslog 简单配置

`/etc/rsyslog.d/iptables.conf`

```shell
if $msg contains 'FORWARD' then {
   /var/log/iptables/forward.log
   stop 
}    
```

## Rsyslog 格式化输出

`/etc/rsyslog.d/iptables.conf`

```shell

# template (
#         name="IptablesFormat"
#         type="string"
#         string="%timegenerated% src_ip=%msg:R,ERE,1,BLANK:SRC=([0-9.]+)\\b--end%\n"
# )


template (
    name="IptablesFormat"
    type="string"
    string="%timegenerated% src_ip=%!usr!src_ip% dst_ip=%!usr!dst_ip% proto=%!usr!proto% scr_port=%!usr!src_port% dst_port=%!usr!dst_port%\n"
) {
    property(name="!usr!src_ip")
    property(name="!usr!dst_ip")
    property(name="!usr!proto")
    property(name="!usr!src_port")
    property(name="!usr!dst_port")
}


if $msg contains 'FORWARD' then {
    set $!usr!proto = re_extract($msg, "PROTO=([A-Za-z0-9]+)\\b", 0, 1, "0");
    set $!usr!src_ip = re_extract($msg, "SRC=([0-9.]+)\\b", 0, 1, "0");
    set $!usr!dst_ip = re_extract($msg, "DST=([0-9.]+)\\b", 0, 1, "0");
    set $!usr!src_port = re_extract($msg, "SPT=([0-9.]+)\\b", 0, 1, "0");
    set $!usr!dst_port = re_extract($msg, "DPT=([0-9.]+)\\b", 0, 1, "0");
        
    action(type="omfile" file="/var/log/iptables/forward.log" template="IptablesFormat")
    stop
}

```

### 日志效果

```shell
Jan  6 15:45:58 src_ip=192.168.1.1 dst_ip=192.168.2.1 proto=TCP scr_port=42342 dst_port=80
```

## 日志分割

`/etc/logrotate.d/iptables`

```shell
/var/log/iptables/*.log {
        daily
        rotate 30
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
