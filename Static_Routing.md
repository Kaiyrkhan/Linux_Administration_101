# Static Route on Linux (permanent - тұрақты)

### Топология
![Topology](Topology/Topology_Static_Routing_Linux.png)

## Virtual PC Simulator
```shell
VPC1> ip 192.168.1.100/24 192.168.1.1
VPC1> show ip
VPC1> save
```
```shell
VPC2> ip 172.16.1.100/24 172.16.1.1
VPC2> show ip
VPC2> save
```

## Static Route on Debian

**R1**
```shell
Құрылғының атауын (Device Name) өзгерту
$ sudo hostnamectl set-hostname R1
$ sudo nano /etc/hosts
127.0.1.1  R1
$ bash
```

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

**R2**
```shell
Құрылғының атауын (Device Name) өзгерту
$ sudo hostnamectl set-hostname R2
$ sudo nano /etc/hosts
127.0.1.1  R2
$ bash
```

```shell
$ sudo nano /etc/network/interfaces
  auto ens3
  iface ens3 inet static
  address 10.1.1.102
  netmask 255.255.255.252
  up ip route add 192.168.1.0/24 via 10.1.1.101

  auto ens4
  iface ens4 inet static
  address 10.2.2.102
  netmask 255.255.255.252
  up ip route add 172.16.1.0/24 via 10.2.2.101
```

**R3**
```shell
Құрылғының атауын (Device Name) өзгерту
$ sudo hostnamectl set-hostname R3
$ sudo nano /etc/hosts
127.0.1.1  R3
$ bash
```

```shell
$ sudo nano /etc/network/interfaces
  auto ens3
  iface ens3 inet static
  address 172.16.1.1
  netmask 255.255.255.0

  auto ens4
  iface ens4 inet static
  address 10.2.2.101
  netmask 255.255.255.252
  up ip route add 192.168.1.0/24 via 10.2.2.102
```

#### Enable Packet IP Forwarding (R1, R2, R3)
```shell
$ sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
$ sudo sysctl -p
```

#### Verify
```shell
VPC1> ping 172.16.1.100
VPC2> ping 192.168.1.100
```

## Static Route on Ubuntu

**R1**
```shell
$ sudo nano /etc/netplan/*.yaml
network:
  ethernets:
    ens3:
      dhcp4: false
      addresses: [192.168.1.1/24]
    ens4:
      dhcp4: false
      addresses: [10.1.1.101/30]
      routers:
        - to: 172.16.1.0/24
          via: 10.1.1.102
version: 2
```

**R2**
```shell
$ sudo nano /etc/netplan/*.yaml
network:
  ethernets:
    ens3:
      dhcp4: false
      addresses: [10.1.1.102/30]
      routers:
        - to: 192.168.1.0/24
          via: 10.1.1.101
    ens4:
      dhcp4: false
      addresses: [10.2.2.102/30]
      routers:
        - to: 172.16.1.0/24
          via: 10.2.2.101
version: 2
```

**R3**
```shell
$ sudo nano /etc/netplan/*.yaml
network:
  ethernets:
    ens3:
      dhcp4: false
      addresses: [172.16.1.1/24]
    ens4:
      dhcp4: false
      addresses: [10.2.2.101/30]
      routers:
        - to: 192.168.1.0/24
          via: 10.2.2.102
version: 2
```

#### Enable Packet IP Forwarding (R1, R2, R3)
```shell
$ sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1
$ sudo sysctl -p
```

#### Verify
```shell
VPC1> ping 172.16.1.100
VPC2> ping 192.168.1.100
```

## Static Route on Oracle7 / RHEL7

**R1**
```shell
$ ip address
eth0
eth1
$ ls /etc/sysconfig/network-scripts/
ifcfg-eth0

Жаңа Profile құру
$ sudo nmcli conn add type ethernet con-name eth1 ifname eth1
$ ls /etc/sysconfig/network-scripts/
ifcfg-eth0
ifcfg-eth1

$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=192.168.1.1
NETMASK=255.255.255.0
$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth1
BOOTPROTO=static
IPADDR=10.1.1.101
NETMASK=255.255.255.252
$ sudo systemctl restart network

$ sudo vi /etc/sysconfig/network-scripts/route-eth1
172.16.1.0/24 via 10.1.1.102 dev eth1
```

**R2**
```shell
$ ip address
eth0
eth1
$ ls /etc/sysconfig/network-scripts/
ifcfg-eth0

Жаңа Profile құру
$ sudo nmcli conn add type ethernet con-name eth1 ifname eth1
$ ls /etc/sysconfig/network-scripts/
ifcfg-eth0
ifcfg-eth1

$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=10.1.1.102
NETMASK=255.255.255.252
$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth1
BOOTPROTO=static
IPADDR=10.2.2.102
NETMASK=255.255.255.252
$ sudo systemctl restart network

$ sudo vi /etc/sysconfig/network-scripts/route-eth0
192.168.1.0/24 via 10.1.1.101 dev eth0
$ sudo vi /etc/sysconfig/network-scripts/route-eth1
172.16.1.0/24 via 10.2.2.101 dev eth1
```

**R3**
```shell
$ ip address
eth0
eth1
$ ls /etc/sysconfig/network-scripts/
ifcfg-eth0

Жаңа Profile құру
$ sudo nmcli conn add type ethernet con-name eth1 ifname eth1
$ ls /etc/sysconfig/network-scripts/
ifcfg-eth0
ifcfg-eth1

$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=172.16.1.1
NETMASK=255.255.255.0
$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eth1
BOOTPROTO=static
IPADDR=10.2.2.101
NETMASK=255.255.255.252
$ sudo systemctl restart network

$ sudo vi /etc/sysconfig/network-scripts/route-eth1
192.168.1.0/24 via 10.2.2.102 dev eth1
```

#### Enable Packet IP Forwarding (R1, R2, R3)
```shell
$ sudo vi /etc/sysctl.conf
net.ipv4.ip_forward=1
$ sudo sysctl -p
```

#### Verify
```shell
VPC1> ping 172.16.1.100
VPC2> ping 192.168.1.100
```

## Static Route on Rocky9 / RHEL9

**R1**
```shell
$ ip address
ens3
ens4
$ ls /etc/NetworkManager/system-connections/
ens3.nmconnection

Жаңа Profile құру
$ sudo nmcli conn add type ethernet con-name ens4 ifname ens4
$ ls /etc/NetworkManager/system-connections/
ens3.nmconnection
ens4.nmconnection

$ sudo nmcli conn modify ens3 ipv4.addresses 192.168.1.1/24
$ sudo nmcli conn modify ens3 ipv4.method manual
$ sudo nmcli conn modify ens4 ipv4.addresses 10.1.1.101/30
$ sudo nmcli conn modify ens4 ipv4.method manual

$ sudo systemctl restart NetworkManager
$ sudo nmcli conn down ens3 && sudo nmcli conn up ens3
$ sudo nmcli conn down ens4 && sudo nmcli conn up ens4

$ sudo nmcli connection modify ens4 +ipv4.routes "172.16.1.0/24 10.1.1.102"
$ sudo nmcli connection up ens4
$ ip route
172.16.1.0/24 via 10.1.1.102 dev ens4
```

**R2**
```shell
$ ip address
ens3
ens4
$ ls /etc/NetworkManager/system-connections/
ens3.nmconnection

Жаңа Profile құру
$ sudo nmcli conn add type ethernet con-name ens4 ifname ens4
$ ls /etc/NetworkManager/system-connections/
ens3.nmconnection
ens4.nmconnection

$ sudo nmcli conn modify ens3 ipv4.addresses 10.1.1.102/30
$ sudo nmcli conn modify ens3 ipv4.method manual
$ sudo nmcli conn modify ens4 ipv4.addresses 10.2.2.102/30
$ sudo nmcli conn modify ens4 ipv4.method manual

$ sudo systemctl restart NetworkManager
$ sudo nmcli conn down ens3 && sudo nmcli conn up ens3
$ sudo nmcli conn down ens4 && sudo nmcli conn up ens4

$ sudo nmcli conn modify ens3 +ipv4.routes "192.168.1.0/24 10.1.1.101"
$ sudo nmcli conn up ens3
$ ip route
192.168.1.0/24 via 10.1.1.101 dev ens3

$ sudo nmcli conn modify ens4 +ipv4.routes "172.16.1.0/24 10.2.2.101"
$ sudo nmcli conn up ens4
$ ip route
172.16.1.0/24 via 10.2.2.101 dev ens4
```

**R3**
```shell
$ ip address
ens3
ens4
$ ls /etc/NetworkManager/system-connections/
ens3.nmconnection

Жаңа Profile құру
$ sudo nmcli conn add type ethernet con-name ens4 ifname ens4
$ ls /etc/NetworkManager/system-connections/
ens3.nmconnection
ens4.nmconnection

$ sudo nmcli conn modify ens3 ipv4.addresses 172.16.1.1/24
$ sudo nmcli conn modify ens3 ipv4.method manual
$ sudo nmcli conn modify ens4 ipv4.addresses 10.2.2.101/30
$ sudo nmcli conn modify ens4 ipv4.method manual

$ sudo systemctl restart NetworkManager
$ sudo nmcli conn down ens3 && sudo nmcli conn up ens3
$ sudo nmcli conn down ens4 && sudo nmcli conn up ens4

$ sudo nmcli conn modify ens4 +ipv4.routes "192.168.1.0/24 10.2.2.102"
$ sudo nmcli conn up ens4
$ ip route
192.168.1.0/24 via 10.2.2.102 dev ens4
```

#### Enable Packet IP Forwarding (R1, R2, R3)
```shell
$ echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
$ sudo sysctl -p
```

#### Verify
```shell
VPC1> ping 172.16.1.100
VPC2> ping 192.168.1.100
```
