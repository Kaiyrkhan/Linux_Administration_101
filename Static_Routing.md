# Static route on Linux

### Static route on Debian

Топология
![Topology](Topology_Static_Route.png)

**R2**
```shell
$ sudo nano /etc/network/interfaces
  auto ens3
  iface ens3 inet static
  address 10.1.1.102
  netmask 255.255.255.252
  up ip route add 192.168.1.0/24 via 10.1.1.101

  auto ens4
  iface ens4 inet static
  address 10.2.2.101
  netmask 255.255.255.252
  up ip route add 172.16.1.0/24 via 10.2.2.102
```
