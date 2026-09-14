# Troubleshooting

## No NFS mounts are discovered

Check the live mount table:

```bash
cat /proc/self/mounts | grep -E ' nfs4? '
```

The template discovers active `nfs` and `nfs4` mounts automatically.

## An expected mount is reported missing

Compare `/etc/fstab` with the live mount table:

```bash
grep -E '[[:space:]]nfs4?[[:space:]]' /etc/fstab
cat /proc/self/mounts | grep -E ' nfs4? '
```

A persistent NFS entry in `/etc/fstab` that is not mounted is treated as a problem.

## Performance items are not present

This is expected when:

```text
{$NFS.PERFORMANCE}=0
```

Enable performance monitoring with:

```text
{$NFS.PERFORMANCE}=1
```

Then wait for the low-level discovery rule to run.

## Performance items are unsupported

Check whether the kernel exposes NFS mount statistics:

```bash
cat /proc/self/mountstats
```

For each NFS mount, a block similar to the following should exist:

```text
device server:/export mounted on /mountpoint with fstype nfs ...
```

If the mount exists in `/proc/self/mountstats`, inspect the raw master item `NFS performance: mountstats raw` in Zabbix.

## RPC Deep items are not present

This is expected when:

```text
{$NFS.RPC.DEEP}=0
```

Enable them with:

```text
{$NFS.RPC.DEEP}=1
```

RPC Deep Health is intended mainly for NFSv3.

## RPC Deep items are unsupported

Verify the required commands:

```bash
/usr/sbin/rpcinfo -p localhost
/usr/bin/timeout --version
```

The Zabbix Agent must allow the exact command pattern:

```ini
AllowKey=system.run[/usr/bin/timeout * /usr/sbin/rpcinfo *,wait]
```

Restart the agent after changing its configuration.

Do not use `AllowKey=system.run[*]`.

## UDP mountd check fails even though UDP is reachable

Some NFS services can return a wildcard universal address during RPC lookup. Use an explicit universal address to test the target directly.

Example for documentation address `192.0.2.10` and port `2048`:

```bash
rpcinfo -a 192.0.2.10.8.0 -T udp 100005 3
```

Port `2048` maps to suffix `.8.0` because `2048 = 8 * 256 + 0`.

## TCP/2049 is reachable but the filesystem is unavailable

TCP/2049 availability only proves that the NFS server accepts a TCP connection. It does not prove that the expected export is mounted on the client.

Check together:

- `Mount status`;
- `Correct source`;
- NFS server TCP/2049 availability.

## Why capacity alone cannot detect an unmount

After an NFS filesystem is unmounted, the mountpoint directory may expose the underlying local filesystem. `vfs.fs.size[]` can therefore continue returning valid values.

The template checks the exact source and mountpoint pair from `/proc/self/mounts` independently of capacity data.

## A mount needs different capacity thresholds

Use Zabbix macro context:

```text
{$NFS.SPACE.WARN:"/mnt/archive"}=90
{$NFS.SPACE.CRIT:"/mnt/archive"}=97
{$NFS.MIN.FREE:"/mnt/archive"}=20G
```
