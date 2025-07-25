# Link Aggregation (LACP) and Bridging on Linux

## Link Aggregation on Debian

### NIC Bonding
```shell
$ sudo apt update
$ sudo apt install ifenslave

$ sudo nano /etc/modules
bonding
CTRL+O, Enter, CTRL+X

$ sudo modprobe bonding
$ lsmod | grep bonding
```

```shell
$ sudo nano /etc/network/interfaces
  auto ens3
  iface ens3 inet manual
    bond-master bond0

  auto ens4
  iface ens4 inet manual
    bond-master bond0

  auto bond0
  iface bond0 inet static
    address 172.16.11.101/24
    gateway 172.16.11.1
    bond-mode 802.3ad
    bond-slaves ens3 ens4

    bond-miimon 100
    bond-lacp-rate 1
    bond-slaves none
```
> IEEE 802.3ad – LACP (Link Aggregation Control Protocol) 

```shell
$ cat /proc/net/bonding/bond0
```

### Қосымша ақпарат
> student@server:~$ iperf -s
>
> student@desktop:~$ iperf -c 172.16.11.1
>
> student@desktop:~$ iperf -c 172.16.11.1 -P2

## Link Aggregation on RHEL

### NIC Teaming
```shell
$ ip address

$ sudo nmcli conn add type team con-name team0 ifname team0 config '{"runner": {"name": "activebackup"}}'
$ sudo nmcli conn add type team-slave con-name team0-port1 ifname eth1
$ sudo nmcli conn add type team-slave con-name team0-port2 ifname eth2

$ teamdctl team0 state

$ sudo nmcli dev dis team0
$ sudo systemctl stop NetworkManager
$ sudo systemctl disable NetworkManager

$ ls -l /etc/sysconfig/network-script/
$ sudo vi /etc/sysconfig/network-script/ifcfg-team0
BRIDGE=brteam0
:wq
$ sudo vi /etc/sysconfig/network-script/ifcfg-team0-port1
$ sudo vi /etc/sysconfig/network-script/ifcfg-team0-port2

$ sudo vi /etc/sysconfig/network-script/ifcfg-brteam0
DEVICE=brteam0
ONBOOT=yes
TYPE=Bridge
IPADDR0=172.16.11.1
PREFIX0=24
:wq

$ sudo systemctl restart network
$ ip address

$ ping -I brteam0 172.16.11.101
$ reboot
```

### Қосымша ақпарат
> student@server:~$ iperf3 -s -p 1234
>
> student@desktop:~$ iperf3 -c 172.16.11.1 -p 1234
