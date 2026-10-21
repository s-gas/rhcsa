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

## Tuning profiles

The tuning profiles are stored in `/usr/lib/tuned/` and `/etc/tuned/`. Every profile has its own directory which contains the main configuration file `tuned.conf` and, optionally, other files:

```bash
[s-gas@localhost ~]$ ls /usr/lib/tuned/profiles/
accelerator-performance  balanced          desktop      intel-sst            network-latency     optimize-serial-console  throughput-performance  virtual-host
aws                      balanced-battery  hpc-compute  latency-performance  network-throughput  powersave                virtual-guest
[s-gas@localhost ~]$ ls /usr/lib/tuned/profiles/balanced/
tuned.conf
```

A configuration file looks like this:

```bash
[s-gas@localhost ~]$ cat /usr/lib/tuned/profiles/aws/tuned.conf 
#
# tuned configuration
#

[main]
summary=Optimize for aws ec2 instances
include=throughput-performance

# Marvell ThunderX
[bootloader.thunderx]
# rhbz:1836058
type=bootloader
uname_regex=aarch64
cmdline=+iommu.strict=0

[bootloader]
# https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/nvme-ebs-volumes.html#timeout-nvme-ebs-volumes
# set nvme_core.io_timeout to 4294967295
# https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking-ena.html
# set net.ifnames to 0
cmdline=+net.ifnames=0 nvme_core.io_timeout=4294967295
```

The `[main]` section can have the `include` parameter, so you can import the configuration of another profile. In this case the `aws` profile uses the `throughput-performance` profile as a base.

To modify a profile, copy its configuration file from `/usr/lib/tuned/` to `/etc/tuned/` and then apply the changes to the configuration file in `/etc/tuned/`.

## `tuned-adm`

The `tuned` daemon can be controlled with the command `tuned-adm`.

To see the active profile, run `tuned-adm active`:

```bash
[s-gas@localhost system]$ tuned-adm active
Current active profile: throughput-performance
```

To list all the available profiles, run `tuned-adm list`:

```bash
[s-gas@localhost system]$ tuned-adm list
Available profiles:
- accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
- aws                         - Optimize for aws ec2 instances
- balanced                    - General non-specialized tuned profile
- balanced-battery            - Balanced profile biased towards power savings changes for battery
- desktop                     - Optimize for the desktop use-case
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- optimize-serial-console     - Optimize for serial console use.
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: throughput-performance
```

To have more information about a profile, run `tuned-adm profile_info`:

```bash
[s-gas@localhost system]$ tuned-adm profile_info balanced
Profile name:
balanced

Profile summary:
General non-specialized tuned profile

Profile description:

```

To change to a different profile, run `tuned-adm profile`:

```bash
[root@localhost ~]# tuned-adm profile virtual-guest
[root@localhost ~]# tuned-adm active
Current active profile: virtual-guest
```
