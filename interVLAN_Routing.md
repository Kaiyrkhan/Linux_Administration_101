# interVLAN Routing on Linux 

### Тақырыбы: Linux дистрибутивінде интернет шлюз (gateway) құру

### Жұмыстың орындалу қадамы: 
  1) Желілік интерфейсті конфигурациялау;
  2) 802.1Q VLAN құру;
  3) IP Packet Forwarding қызметін қосу (enable);
  4) Cisco Switch конфигурациялау;
  5) NAT конфигурациялау;
  6) Нəтижені тексеру.

### Physical Topology
![Physical Topology](TopologyPhysical_interVLANRouting_Linux.png)
### Logical Topology
![Logical Topology](TopologyLogical_interVLANRouting_Linux.png)

## Gateway

### 802.1Q VLAN құру
```shell
8021q модулін жүктеу
$ sudo modprobe 8021q
$ lsmod | grep 8021q
$ sudo cat /proc/net/vlan/config
```

```shell
$ ip address

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
~~ $ sudo ifdown ens4 && sudo ifup ens4
$ sudo ifup ens4.11
$ sudo ifup ens4.12 ~~

~~сызып тасталған~~

VLAN құрылғанын тексеру
$ sudo cat /proc/net/vlan/config

$ ip -d link show ens4.11
$ ip -d link show ens4.12
```

### Enable Packet IP Forwarding
```shell
$ sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1
$ sudo sysctl -p
```
