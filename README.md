# Zabbix Linux NFS Enterprise

<p align="center">
  <img src="https://img.shields.io/badge/Zabbix-7.0-D40000?logo=zabbix&logoColor=white" alt="Zabbix 7.0">
  <img src="https://img.shields.io/badge/Linux-Agent%20%7C%20Agent%202-FCC624?logo=linux&logoColor=black" alt="Linux">
  <img src="https://img.shields.io/badge/NFS-v3%20%7C%20v4-2F6F9F" alt="NFSv3 and NFSv4">
  <img src="https://img.shields.io/badge/Mode-Active%20Checks-5CB85C" alt="Active checks">
  <img src="https://img.shields.io/badge/Setup-Plug%20%26%20Play-0078D4" alt="Plug and Play">
</p>

Plug-and-play Zabbix 7.0 template for automatic NFS/NFSv4 monitoring on Linux hosts.

## Features

The template automatically discovers:

- active NFS/NFSv4 mounts;
- NFS servers;
- exports and mountpoints;
- NFS version;
- persistent NFS mounts defined in `/etc/fstab`.

By default it monitors:

- mount availability;
- expected NFS source;
- read-only/read-write state;
- total, used and free space;
- used space percentage;
- capacity forecast;
- NFS server TCP/2049 availability;
- TCP/2049 connection time.

No static server or mountpoint lists are required.

## Requirements

- Zabbix 7.0
- Zabbix Agent or Agent 2
- Active checks
- Linux access to:
  - `/proc/self/mounts`
  - `/proc/self/mountstats`
  - `/etc/fstab`

## Installation

1. Import `zabbix_template_linux_nfs_enterprise_7.0.yaml` into Zabbix.
2. Link `Linux NFS by Zabbix agent active` to the target Linux hosts.
3. Wait for the discovery rules to run.

No NFS server IP, export or mountpoint needs to be configured in Zabbix.

Example mount:

```text
10.0.0.10:/data /mnt/data nfs rw,hard,vers=3,proto=tcp 0 0
```

The template automatically detects:

```text
Server:      10.0.0.10
Export:      /data
Mountpoint:  /mnt/data
NFS version: 3
Transport:   tcp
```

## Macros

| Macro | Default | Description |
|---|---:|---|
| `{$NFS.SPACE.WARN}` | `80` | Warning threshold for used space percentage. |
| `{$NFS.SPACE.CRIT}` | `90` | High threshold for used space percentage. |
| `{$NFS.MIN.FREE}` | `5G` | High alert when free space drops below this value. |
| `{$NFS.FORECAST.DAYS}` | `7d` | Alert when space exhaustion is forecast within this period. |
| `{$NFS.TCP.CONNECT.WARN}` | `0.5` | Warning threshold for TCP/2049 connection time, in seconds. |
| `{$NFS.PERFORMANCE}` | `0` | Enables advanced metrics from `/proc/self/mountstats`. |
| `{$NFS.RPC.RTT.WARN}` | `50` | Warning threshold for average RPC RTT, in milliseconds. |
| `{$NFS.RPC.RETRANS.WARN}` | `5` | Warning threshold for RPC retransmissions, in percent. |
| `{$NFS.RPC.ERROR.WARN}` | `1` | Warning threshold for RPC errors, in percent. |
| `{$NFS.RPC.DEEP}` | `0` | Enables functional RPC checks through `rpcinfo`. |

## Performance monitoring

Performance monitoring is disabled by default.

Enable it with:

```text
{$NFS.PERFORMANCE}=1
```

This adds metrics such as:

- READ/WRITE throughput;
- READ/WRITE operations per second;
- RPC RTT;
- RPC execution time;
- RPC queue time;
- retransmission percentage;
- RPC error percentage.

When `{$NFS.PERFORMANCE}=0`, these items are not created on monitored hosts.

## RPC Deep Health

RPC Deep Health is disabled by default.

Enable it with:

```text
{$NFS.RPC.DEEP}=1
```

For NFSv3, it functionally checks:

- rpcbind TCP/UDP;
- mountd TCP/UDP;
- NFSv3;
- NLM/nlockmgr.

It requires `rpcinfo` and `timeout` on the Linux client.

Recommended Zabbix Agent configuration:

```ini
AllowKey=system.run[/usr/bin/timeout * /usr/sbin/rpcinfo *,wait]
```

Do not use:

```ini
AllowKey=system.run[*]
```

## Per-mount thresholds

Zabbix macro contexts can be used for specific mountpoints.

Example:

```text
{$NFS.SPACE.WARN:"/mnt/archive"}=90
{$NFS.SPACE.CRIT:"/mnt/archive"}=97
{$NFS.MIN.FREE:"/mnt/archive"}=20G
```

All other mounts continue to use the global defaults.

## Troubleshooting

See [`docs/TROUBLESHOOTING.md`](./docs/TROUBLESHOOTING.md).
