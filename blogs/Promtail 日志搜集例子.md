# Promtail 日志搜集例子

## Iptables 日志

`time=2025-06-26T23:28:50.904042+08:00 interface=eth2 src_ip=20.80.88.7 dst_ip=122.224.237.157 proto=TCP scr_port=45546 dst_port=8980`

```yaml
scrape_configs:
  - job_name: iptables
    static_configs:
      - targets:
          - localhost
        labels:
          job: iptables
          __path__: /var/log/iptables/input.log
    pipeline_stages:
      - regex:
          expression: "time=(?P<ts>[^ ]+) (?P<rest>.*)"
      - timestamp:
          source: ts
          format: RFC3339Nano
      - output:
          source: rest
      - regex:
          expression: 'interface=(?P<interface>\S+) src_ip=(?P<src_ip>\S+) dst_ip=(?P<dst_ip>\S+) proto=(?P<proto>\S+) scr_port=(?P<scr_port>\d+) dst_port=(?P<dst_port>\d+)'
      - labels:
          interface:
          src_ip:
          dst_ip:
          proto:
          scr_port:
          dst_port:

```
