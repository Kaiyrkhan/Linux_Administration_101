# VLAN and interVLAN Routing on Linux

### Топология
![Topology](Topology_interVLANRouting_Linux.png)

## Configure
```shell
$ sudo nano /etc/network/interfaces
  auto ens3
  iface ens3 inet dhcp

# VLAN 11
  auto ens4.11
  iface ens4.11 inet static
  address 172.16.11.1/24
  vlan-raw-device ens4
# VLAN 12
  auto ens4.12
  iface ens4.12 inet static
  address 172.16.12.1/24
  vlan-raw-device ens4
```
