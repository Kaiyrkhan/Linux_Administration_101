# VLAN and interVLAN Routing on Linux

### Топология
![Topology](Topology_interVLANRouting_Linux.png)

### Create VLAN
```shell
$ sudo nano /etc/network/interfaces
  auto ens3
  iface ens3 inet dhcp

  auto ens4.11
  iface ens4.11 inet static
  address 172.16.11.1/24
  vlan-raw-device ens4

  auto ens4.12
  iface ens4.12 inet static
  address 172.16.12.1/24
  vlan-raw-device ens4

$ sudo systemctl restart networking
$ sudo ifdown ens4.11 && sudo ifup ens4.11
$ sudo ifdown ens4.12 && sudo ifup ens4.12

$ ip -d link show ens4.11
$ ip -d link show ens4.12

$ sudo cat /proc/net/vlan/config
```

### Enable Packet IP Forwarding
```shell
$ sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1
$ sudo sysctl -p
```
