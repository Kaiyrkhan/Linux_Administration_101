# Dynamic Routing on Linux

### Топология
![Topology](Topology/Topology_Dynamic_Routing_Linux.png)

## Dynamic Routing on Debian

**R1**
```shell
$ sudo nano /etc/network/interfaces
  auto ens3
  iface ens3 inet static
  address 192.168.1.1
  netmask 255.255.255.0

  auto ens4
  iface ens4 inet static
  address 10.1.1.101
  netmask 255.255.255.252
  up ip route add 172.16.1.0/24 via 10.1.1.102
```
