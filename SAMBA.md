# SMB Protocol on Linux

## SMB on Ubuntu Linux

##### SMB Server
```shell
$ sudo apt update
$ sudo apt install samba
```

```shell
$ sudo mkdir /mnt/data1
$ sudo mkdir /mnt/data2
```

```shell
$ sudo nano /etc/samba/smb.conf

[global]
        netbios name = server
        server string = server
        workgroup = WORKGROUP
        security = user

[data1]
        comment = data1
        path = "/mnt/data1"
        public = yes
        guest ok = yes
        read only = no

[data2]
        comment = data2
        path = "/mnt/data2"
        public = yes
        guest ok = yes
        read only = no
```

```shell
$ sudo pdbedit -a -u user1
new password: 1234
```

```shell
$ sudo systemctl restart smbd
$ sudo systemctl restart nmbd

$ sudo service smbd restart
$ sudo service nmbd restart
```

##### SMB Client
```shell
Microsoft Windows 11
Windows+R -> \\172.16.11.1
```

```shell
Linux Distro
$ sudo nano /etc/fstab
172.16.1.1:/mnt/data1  /mnt/data1  nfs  defaults,x-systemd.mount-timeout=5  0  0
172.16.1.1:/mnt/data2  /mnt/data2  nfs  defaults,x-systemd.mount-timeout=5  0  0
```
