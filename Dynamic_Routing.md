# Dynamic Routing on Linux

### Топология
![Topology](Topology/Topology_Dynamic_Routing_Linux.png)

## Dynamic Routing on Debian

**R1**
```shell
Құрылғының атауын (Device Name) өзгерту
$ hostnamectl set-hostname firewall
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
