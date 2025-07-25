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
$ cat /usr/share/doc/ifenslave/examples/two_ethernet
  auto bond0
  iface bond0 inet dhcp
    bond-slaves ens3 ens4
    bond-mode 1
    bond-miimon 100
    bond-primary ens3 ens4
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

$ sudo systemctl restart networking
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

### NIC Teaming using nmcli

```shell
Create a Team interface
nmcli conn add type team con-name <connection-name> ifname <device-name> config '{"runner":{"name":"<runners-mode>"}}'

$ sudo nmcli conn add type team con-name team0 ifname team0 config '{"runner": {"name": "lacp"}}'
$ sudo nmcli conn add type team con-name team0 ifname team0 config '{"runner": {"name": "activebackup"}}'

$ sudo nmcli conn add type team con-name team0 ifname team0 \
  config '{"runner": {"name": "lacp"}}'
$ sudo nmcli conn add type team con-name team0 ifname team0 \
  config '{"runner": {"name": "activebackup"}}'
```

##### The Modes/Runners of Teaming:
- `broadcast:` - Transmits data over all ports
- `roundrobin:` - Transmits data over all ports in turn
- `activebackup:` - Transmits data over one port while the others are kept as a backup
- `loadbalance:` - Transmits data over all ports with active Tx load balancing and Berkeley Packet Filter (BPF)-based Tx port selectors
- `random:` - Transmits data on a randomly selected port
- `lacp:` - Implements the 802.3ad Link Aggregation Control Protocol (LACP)

```shell
Add NIC interfaces to the Team
nmcli conn add type team-slave con-name <connection-name> ifname <device-name> master <teamed-interface>
$ sudo nmcli conn add type team-slave con-name team0-port1 ifname eth1 master team0
$ sudo nmcli conn add type team-slave con-name team0-port2 ifname eth2 master team0
немесе
$ sudo nmcli connection add type ethernet slave-type team con-name team0-port1 ifname eth1 master team0
$ sudo nmcli connection add type ethernet slave-type team con-name team0-port2 ifname eth2 master team0
```

```shell
Configure the IPv4 settings
$ sudo nmcli conn modify team0 ipv4.addresses 172.16.11.101/24
$ sudo nmcli conn modify team0 ipv4.gateway 172.16.11.1
$ sudo nmcli conn modify team0 ipv4.dns 8.8.8.8
$ sudo nmcli conn modify team0 ipv4.dns-search edu.local
$ sudo nmcli conn modify team0 ipv4.method manual

Configure the IPv6 settings
$ sudo nmcli conn modify team0 ipv6.addresses 2001:db8:1::1/64
$ sudo nmcli conn modify team0 ipv6.gateway 2001:db8:1::fffe
$ sudo nmcli conn modify team0 ipv6.dns 2001:db8:1::fffd
$ sudo nmcli conn modify team0 ipv6.dns-search edu.local
$ sudo nmcli conn modify team0 ipv6.method manual

$ sudo systemctl restart NetworkManager

$ sudo nmcli conn down eth1 && sudo nmcli conn up eth1
$ sudo nmcli conn down eth2 && sudo nmcli conn up eth2
$ sudo nmcli conn down team0 && sudo nmcli conn up team0
```

```shell
Verification
$ teamdctl team0 state
```

```shell
Troubleshooting
$ teamdctl team0 state
$ sudo nmcli conn down eth1
$ teamdctl team0 state
```

```shell
2-ші тәсіл: Configure the IPv4 settings

$ sudo nmcli dev dis team0
$ sudo systemctl stop NetworkManager
$ sudo systemctl disable NetworkManager

$ ls -l /etc/sysconfig/network-script/
ifcfg-team0
ifcfg-team0-port1
ifcfg-team0-port2

$ cat /etc/sysconfig/network-script/ifcfg-team0-port1
$ cat /etc/sysconfig/network-script/ifcfg-team0-port2

$ sudo vi /etc/sysconfig/network-script/ifcfg-team0
BRIDGE=brteam0
:wq

$ sudo vi /etc/sysconfig/network-script/ifcfg-brteam0
DEVICE=brteam0
ONBOOT=yes
TYPE=Bridge
IPADDR0=172.16.11.101
PREFIX0=24
:wq

$ sudo systemctl restart network
$ ip address
$ teamdctl team0 state

$ ping -I brteam0 172.16.11.1
$ reboot
```

### Қосымша ақпарат
> student@server:~$ iperf3 -s -p 1234
>
> student@desktop:~$ iperf3 -c 172.16.11.1 -p 1234

### References

1) [Configuring a NIC team by using nmcli](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/configuring-network-teaming_configuring-and-managing-networking#configuring-a-network-team-by-using-nmcli_configuring-network-teaming)
2) [How To Configure Network Bonding & Network Teaming In Linux](https://tekneed.com/configure-network-bonding-network-teaming-in-linux/#what-is-the-difference-between-teaming-and-bonding-in-linux)
