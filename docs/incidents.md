# Incident Scenarios

This file documents the incidents used in the current lab and reflects the actual working configuration that was tested on the three-VM Docker-based setup.

## Scenario 1: Web Server Down

**Target:** `web-01`  
**Goal:** Detect a stopped HTTP service while the host itself remains reachable.

### Lab Implementation

- host: `web-01`
- service: `nginx`
- web scenario: `web-home`
- trigger: `web-01 HTTP unavailable`
- trigger expression:

```text
last(/web-01/web.test.fail[web-home])>0
```

### Simulate

```bash
sudo systemctl stop nginx
```

### Expected Zabbix Result

- `web-01` stays available by agent
- the web scenario step `homepage` fails
- `Monitoring -> Problems` shows `web-01 HTTP unavailable`

### Recovery

```bash
sudo systemctl start nginx
```

## Scenario 2: DNS Failure

**Target:** `dns-01`  
**Goal:** Detect a DNS service outage while the host itself remains reachable.

### Lab Implementation

- host: `dns-01`
- service: `bind9`
- item: `net.tcp.service[tcp,172.16.10.30,53]`
- trigger: `dns-01 DNS unavailable`
- trigger expression:

```text
last(/dns-01/net.tcp.service[tcp,172.16.10.30,53])=0
```

### Simulate

```bash
sudo systemctl stop bind9
```

### Validation

```bash
dig @127.0.0.1 localhost
```

Expected failure after stopping `bind9`:

- `connection refused`
- `no servers could be reached`

### Expected Zabbix Result

- `dns-01` stays available by agent
- the TCP 53 simple check returns `0`
- `Monitoring -> Problems` shows `dns-01 DNS unavailable`

### Recovery

```bash
sudo systemctl start bind9
```

## Scenario 3: High Latency Or Packet Loss

**Target:** `web-01`  
**Goal:** Detect degraded network quality rather than a hard outage.

### Planned Items

```text
icmppingsec[172.16.10.20,4,200,56,1000]
icmppingloss[172.16.10.20,4,200,56,1000]
```

### Planned Triggers

```text
avg(/web-01/icmppingsec[172.16.10.20,4,200,56,1000],5m)>0.15
min(/web-01/icmppingloss[172.16.10.20,4,200,56,1000],5m)>10
```

### Simulate

```bash
sudo tc qdisc add dev enp0s8 root netem delay 250ms 40ms loss 15%
```

### Recovery

```bash
sudo tc qdisc del dev enp0s8 root
```

## Scenario 4: Blocked Port Or Interface Down

**Target:** `web-01` or `dns-01`  
**Goal:** Show the difference between service isolation and full reachability loss.

### Option A: Block HTTP Port

```bash
sudo iptables -I INPUT -p tcp --dport 80 -j REJECT
```

Recovery:

```bash
sudo iptables -D INPUT -p tcp --dport 80 -j REJECT
```

### Option B: Bring Lab Interface Down

```bash
sudo ip link set enp0s8 down
```

Recovery:

```bash
sudo ip link set enp0s8 up
```

## Troubleshooting Notes

### Agent Not Available After Docker Deployment

If Zabbix shows `Linux: Zabbix agent is not available (for 3m)` while the service is running, verify that monitored hosts allow the Docker bridge subnet.

Working value in this lab:

```text
Server=172.16.10.10,172.16.239.0/24
```

### DNS Check Fix

An earlier attempt used:

```text
net.tcp.service[dns,172.16.10.30,53]
```

That did not produce the expected alert in this lab. The working item key was:

```text
net.tcp.service[tcp,172.16.10.30,53]
```

### Default `Zabbix server` Host Noise

The default host created by the container deployment may show a false agent alert.

Recommended cleanup:

- unlink `Linux by Zabbix agent`
- keep `Zabbix server health`

## Evidence Captured

The current repo already includes:

- [01-healthy-dashboard.png](/d:/zabbix-incident-lab/docs/screenshots/01-healthy-dashboard.png)
- [02-web-http-unavailable.png](/d:/zabbix-incident-lab/docs/screenshots/02-web-http-unavailable.png)
- [03-dns-unavailable.png](/d:/zabbix-incident-lab/docs/screenshots/03-dns-unavailable.png)
- [04-resolved-state.png](/d:/zabbix-incident-lab/docs/screenshots/04-resolved-state.png)
