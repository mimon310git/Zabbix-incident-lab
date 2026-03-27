# Monitoring Matrix

This file reflects the checks used or planned in the current three-VM Docker-based lab.

## Host Monitoring

| Host | Check | Method | Status |
| --- | --- | --- | --- |
| `web-01` | agent availability | `Linux by Zabbix agent` | implemented |
| `dns-01` | agent availability | `Linux by Zabbix agent` | implemented |
| `web-01` | CPU, memory, disk | template items | implemented |
| `dns-01` | CPU, memory, disk | template items | implemented |

## Service Monitoring

| Host | Check | Method | Status |
| --- | --- | --- | --- |
| `web-01` | homepage availability | web scenario `web-home` | implemented |
| `web-01` | HTTP failure alert | `web.test.fail[web-home]` trigger | implemented |
| `dns-01` | DNS TCP port 53 | `net.tcp.service[tcp,172.16.10.30,53]` | implemented |
| `dns-01` | DNS unavailable alert | trigger on simple check | implemented |
| `web-01` | ICMP response time | `icmppingsec[...]` | planned |
| `web-01` | ICMP packet loss | `icmppingloss[...]` | planned |

## Trigger Expressions

### Web

`web-01 HTTP unavailable`

```text
last(/web-01/web.test.fail[web-home])>0
```

### DNS

`dns-01 DNS unavailable`

```text
last(/dns-01/net.tcp.service[tcp,172.16.10.30,53])=0
```

### Planned Latency

`web-01 High latency`

```text
avg(/web-01/icmppingsec[172.16.10.20,4,200,56,1000],5m)>0.15
```

`web-01 Packet loss high`

```text
min(/web-01/icmppingloss[172.16.10.20,4,200,56,1000],5m)>10
```

## Operational Notes

- `Zabbix server` default host inside the container deployment may show an unnecessary agent problem.
- In this lab, the fix is to unlink `Linux by Zabbix agent` from the default `Zabbix server` host.
- The real monitored targets are `web-01` and `dns-01`.
- The working DNS port check uses `tcp` as the service type, not `dns`.
