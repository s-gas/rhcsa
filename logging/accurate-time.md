# Accurate time

## NTP

NTP stands for Network Time Protocol and it's the protocol through which machines synchronize their date and time over the internet.

You can use `timedatectl` to get date, time and to see if NTP is synchronized:

```bash
[user@rocky ~]$ timedatectl
               Local time: Fri 2026-10-16 15:04:05 CEST
           Universal time: Fri 2026-10-16 13:04:05 UTC
                 RTC time: Fri 2026-10-16 13:04:32
                Time zone: Europe/Berlin (CEST, +0200)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

You can also use `timedatectl` to change the system's settings:

```bash
[root@rocky ~]$ timedatectl set-ntp false
[root@rocky ~]$ timedatectl set-time 10:00:00
[root@rocky ~]$ timedatectl
               Local time: Fri 2026-10-16 10:00:02 CEST
           Universal time: Fri 2026-10-16 08:00:02 UTC
                 RTC time: Fri 2026-10-16 08:07:50
                Time zone: Europe/Berlin (CEST, +0200)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

## `chronyd`

The `chronyd` service synchronizes RTC (Real-Time Clock) with the NTP servers. If no internet is available, it calculates the drift and stores it in the `driftfile` specified in `/etc/chrony.conf`:

```bash
[user@rocky ~]$ grep 'driftfile' /etc/chrony.conf 
driftfile /var/lib/chrony/drift
```

The **stratum** of the NTP time source determines its quality. It is the number of hops that a machine is away from a high-performance reference clock.

For example, if a machine synchronizes its time from a NTP server directly connected to a reference clock, its stratum would be `stratum 2`:
- reference clock: `stratum 0`
- NTP server: `stratum 1`
- machine: `stratum 2`

In `/etc/chrony.conf` you can declare a server and a peer. The `server` is one stratum level above the local NTP server and the `peer` is at the same stratum level. `pool` is a variant of `server`:

```bash
[user@rocky ~]$ grep '^pool' /etc/chrony.conf 
pool 2.rocky.pool.ntp.org iburst
```

RHEL recommends always using the `iburst` option after the server domain name.
