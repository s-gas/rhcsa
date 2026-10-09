# Network Interfaces

Network interface names start with the interface type:
- `en`: Ethernet
- `wl`: WLAN (Wireless Local Area)
- `ww`: WWAN (Wireless Wide Area)

Network interfaces can have both IPv4 and IPv6 addresses and can be used in parallel (dual-stack mode):

```bash
[root@localhost sshd_config.d]$ ip a show enp0s8
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:0d:d5:7d brd ff:ff:ff:ff:ff:ff
    altname enx0800270dd57d
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s8
       valid_lft 41701sec preferred_lft 41701sec
    inet6 fd17:625c:f037:2:a00:27ff:fe0d:d57d/64 scope global dynamic noprefixroute 
       valid_lft 85835sec preferred_lft 13835sec
    inet6 fe80::a00:27ff:fe0d:d57d/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
