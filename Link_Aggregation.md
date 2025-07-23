# Link Aggregation (LACP) and Bridging on Linux

## Link Aggregation on Debian/Ubuntu

### Bonding
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
    address 172.16.10.1/24
  bond-slaves ens3 ens4
  bond-mode 802.3ad
```
> IEEE 802.3ad – LACP (Link Aggregation Control Protocol) 

```shell
$ cat /proc/net/bonding/bond0
```

## Link Aggregation on RHEL
### Teaming
```shell
$ ip address

$ sudo nmcli conn add type team con-name team0 ifname team0 config '{"runner": {"name": "activebackup"}}'
$ sudo nmcli conn add type team-slave con-name team0-port1 ifname eth1
$ sudo nmcli conn add type team-slave con-name team0-port2 ifname eth2
```

```shell

```

### Қосымша ақпарат
> student@server:~$ iperf3 -s -p 1234
>
> student@desktop:~$ iperf3 -c server -p 1234
