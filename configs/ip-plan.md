# IP Plan

All addresses in this repo are private RFC1918 lab addresses used only inside the VirtualBox environment.

## Host Addressing

| Hostname | Role | Interface | IP / CIDR | Notes |
| --- | --- | --- | --- | --- |
| `zbx-srv` | Zabbix Docker host | `enp0s8` | `172.16.10.10/24` | main monitoring VM |
| `web-01` | web target | `enp0s8` | `172.16.10.20/24` | `nginx` + `zabbix-agent2` |
| `dns-01` | DNS target | `enp0s8` | `172.16.10.30/24` | `bind9` + `zabbix-agent2` |

## NAT-Side Notes

`enp0s3` remains DHCP via VirtualBox NAT on each VM.

That interface is used for:

- package downloads
- SSH access through forwarded ports
- frontend access through forwarded port `8080`

## Docker Networking

Observed Docker backend subnet on `zbx-srv`:

- `172.16.239.0/24`

Important consequence:

- `zabbix-agent2` passive checks arrive from this Docker subnet
- monitored hosts must allow both `172.16.10.10` and `172.16.239.0/24`

## NAT Port Forwarding

| VM | Host port | Guest port | Use |
| --- | --- | --- | --- |
| `zbx-srv` | `2221` | `22` | SSH |
| `web-01` | `2222` | `22` | SSH |
| `dns-01` | `2223` | `22` | SSH |
| `zbx-srv` | `8080` | `80` | Zabbix frontend |

## Service Endpoints

| Service | Address |
| --- | --- |
| Zabbix frontend from Windows | `http://127.0.0.1:8080` |
| Web target | `http://172.16.10.20/` |
| DNS target | `172.16.10.30:53` |

## Example Agent Config

```text
Server=172.16.10.10,172.16.239.0/24
ServerActive=172.16.10.10
Hostname=web-01
```
