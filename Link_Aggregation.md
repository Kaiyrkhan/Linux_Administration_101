# Link Aggregation (LACP) on Linux

## Link Aggregation on Debian/Ubuntu
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
  bond-mode 4
  bond-slaves ens3 ens4
```

```shell
$ cat /proc/net/bonding/bond0
```

> student@server:~$ iperf3 -s -p 1234
>
> student@desktop:~$ iperf3 -c server -p 1234
