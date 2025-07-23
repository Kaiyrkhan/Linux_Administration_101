# NFS (Network File System) on Linux

> NFS - Network File Sharing Protocol

## NFS on Ubuntu Linux

##### NFS Server
```shell
$ sudo apt update
$ sudo apt install nfs-kernel-server
```

```shell
$ sudo nano /etc/exports
/mnt/data1  172.16.11.0/24(rw,sync,no_subtree_check)
/mnt/data2  172.16.11.0/24(rw,sync,no_subtree_check)
```

```shell
$ sudo service nfs-kernel-server restart
```

##### NFS Client
```shell
$ sudo nano /etc/fstab
172.16.11.101:/mnt/data1  /mnt/data1  nfs  defaults  0  0
172.16.11.101:/mnt/data2  /mnt/data2  nfs  defaults  0  0
```

```shell
```
