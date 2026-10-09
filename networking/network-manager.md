# NetworkManager

NetworkManager is a service for managing network settings. It manages:
- devices: network interfaces
- connections: configuration for a device. A device can have multiple connections.

To interact with NetworkManager via CLI, you can use `nmcli`.

To display the devices (network interfaces) and their status, use `nmcli dev status`:

```bash
[root@localhost ~]$ nmcli dev status
DEVICE   TYPE      STATE                   CONNECTION 
enp0s8   ethernet  connected               enp0s8     
lo       loopback  connected (externally)  lo         
docker0  bridge    connected (externally)  docker0    
```

To display the connections, use `nmcli con show`:

```bash
[root@localhost ~]$ nmcli con show
NAME     UUID                                  TYPE      DEVICE  
enp0s8   1f65bfe7-48e1-399c-ae10-e72cdd3766ca  ethernet  enp0s8  
lo       aaf6f6f6-8470-46cd-af54-78cc82442d9c  loopback  lo      
docker0  d04e5c12-e707-46cd-8813-28f7fc903955  bridge    docker0 
```

In this case there are 3 devices and a connection for each one of them.

NetworkManager provides also a terminal user interface called `nmtui`, which makes network configuration easier.

Another way of managing network is by directly modifying the configuration files in `/etc/NetworkManager/system-connections`.

The configuration files are in INI format:

```bash
[root@localhost ~]$ cat /etc/NetworkManager/system-connections/enp0s8.nmconnection 
[connection]
id=enp0s8
uuid=1f65bfe7-48e1-399c-ae10-e72cdd3766ca
type=ethernet
autoconnect-priority=-999
interface-name=enp0s8
timestamp=1789291224

[ethernet]

[ipv4]
method=auto

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

If you modify the files manually, remember to reload the connection with `nmcli con reload`:

```bash
[root@localhost ~]$ nmcli con reload
```
