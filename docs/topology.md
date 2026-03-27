# Topology And Build Notes

## Actual Lab Layout

This lab uses three Ubuntu Server VMs:

- `zbx-srv`
- `web-01`
- `dns-01`

There is no desktop GUI on the Linux side. Administration is done through:

- VirtualBox console for first boot tasks
- SSH from the Windows host
- Zabbix web UI in a browser on Windows

## Network Design

Each VM uses two VirtualBox adapters:

| Adapter | Type | Purpose |
| --- | --- | --- |
| `Adapter 1` | NAT | package installation, SSH port forwarding, frontend access |
| `Adapter 2` | Internal Network `zbx-lab` | monitoring and inter-VM traffic |

Inside Ubuntu:

| Interface | Purpose |
| --- | --- |
| `enp0s3` | NAT |
| `enp0s8` | lab network |

## IP Plan

| Host | IP |
| --- | --- |
| `zbx-srv` | `172.16.10.10/24` |
| `web-01` | `172.16.10.20/24` |
| `dns-01` | `172.16.10.30/24` |

Docker backend network on `zbx-srv`:

- `172.16.239.0/24`

## Access From Windows

Recommended NAT port forwarding:

| VM | Host port | Guest port | Purpose |
| --- | --- | --- | --- |
| `zbx-srv` | `2221` | `22` | SSH |
| `web-01` | `2222` | `22` | SSH |
| `dns-01` | `2223` | `22` | SSH |
| `zbx-srv` | `8080` | `80` | Zabbix frontend |

## Topology Diagram

```text
                      Windows Host
                   Browser + SSH client
                           |
                NAT port forwarding to VMs
                           |
    -------------------------------------------------------
    |                         |                          |
+---+----------------+  +-----+--------------+  +--------+-----------+
| zbx-srv            |  | web-01             |  | dns-01             |
| Ubuntu Server      |  | Ubuntu Server      |  | Ubuntu Server      |
| 172.16.10.10       |  | 172.16.10.20       |  | 172.16.10.30       |
| Docker + Zabbix    |  | nginx + agent2     |  | bind9 + agent2     |
+---+----------------+  +--------------------+  +--------------------+
    |
    +-- Docker backend bridge: 172.16.239.0/24

             Internal Network: zbx-lab (172.16.10.0/24)
```

## Deployment Notes

### `zbx-srv`

- Docker Engine installed directly on Ubuntu Server
- official `zabbix-docker` repo checked out on branch `7.4`
- `docker compose up -d` used from the repo root
- frontend published on VM port `80`

### `web-01`

- `nginx`
- `zabbix-agent2`
- web scenario `web-home`
- custom trigger `web-01 HTTP unavailable`

### `dns-01`

- `bind9`
- `zabbix-agent2`
- simple check `net.tcp.service[dns,172.16.10.30,53]`
- custom trigger `dns-01 DNS unavailable`

## Important Agent Note

When Zabbix server runs in Docker, passive checks do not come directly from `172.16.10.10`. They come from the Docker bridge subnet.

That means `zabbix-agent2` on monitored hosts must allow both:

- `172.16.10.10`
- `172.16.239.0/24`

Without that change, Zabbix may show:

- `Linux: Zabbix agent is not available (for 3m)`

even if the agent service itself is running.
