# Dynamic Routing on Linux

### Топология
![Topology](Topology/Topology_Dynamic_Routing_Linux.png)

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

## Dynamic Routing on Debian

**R1**
```shell
Құрылғының атауын (Device Name) өзгерту
$ sudo hostnamectl set-hostname R1
$ bash
```

```shell
FRR (FRRouting) пакетін орнату және конфигурациялау

$ sudo apt update
$ sudo apt install frr frr-pythontools

$ sudo nano /etc/frr/daemons
ospfd=yes

$ sudo systemctl restart frr
$ sudo systemctl enable frr
```

```shell
OSPF конфигурациялау

$ sudo vtysh

router ospf
router-id 50.1.1.1
passive-interface ens3
network 10.10.10.0/24 area 0
int ens4
ip ospf network broadcast
ip ospf mtu-ignore

copy run start
exit
```
