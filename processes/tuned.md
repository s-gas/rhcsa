# `tuned`

`tuned` is a daemon that provides tuning adjustments by providing tuning profiles, which can tune performance either statically or dynamically.

## Static tuning

The parameters specified in the tuning profiles are adjusted during boot. This is the default. You can see that from the main configuration file of `tuned`, which is `/etc/tuned/tuned-main.conf`:

```bash
[s-gas@localhost /]$ grep -B1 '^dynamic' /etc/tuned/tuned-main.conf 
# Dynamically tune devices, if disabled only static tuning will be used.
dynamic_tuning = 0
```

## Dynamic tuning

With dynamic tuning, in order to monitor and to continuously adjust parameter settings, the `tuned` daemon uses **monitor plug-ins** and **tuning plug-ins**.

Monitor plug-ins analyze the system and collect information. The current (RHEL 9) plug-ins are:
- `disk`: monitors the disk load based on the number of I/O operations for every disk device
- `net`: monitors the network load based on the number of transferred packets for every network card
- `load`: monitors the CPU load for every CPU

Tuning plug-ins use the data collected by the monitor plug-ins to adjust the settings. Some of these plug-ins are:
- `disk`: configures different disk parameters (disk scheduler, power management, etc)
- `net`: configures the interface speed and Wake-on-Lan (WoL) functionality
- `load`: configures CPU parameters

To turn on dynamic tunic, modify set `dynamic_tuning` to `1` in `/etc/tuned/tuned-main.conf`.
