# Netfilter

#### Firewall, NAT on Linix
  1) NAT конфигурациялау (using nftables);
  2) NAT конфигурациялау (using iptables).

#### 🖧 Topology
![Logical Topology](Topology/Topology_interVLANRouting_NAT_Linux.png)

### 1) NAT конфигурациялау (using nftables)

```shell
$ dpkg -l nftables
$ dpkg -s nftables
```

```shell
$ sudo systemctl status nftables

$ sudo systemctl start nftables
$ sudo systemctl enable nftables
```

```shell
$ sudo nft list ruleset
```

```shell
$ sudo nft chain inet filter input { policy drop \; }
$ sudo nft сhain inet filter forward { policy drop \; }
$ sudo nft list ruleset
```

> sudo nft add table inet filter  
> sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }  
> sudo nft add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }  
> sudo nft add chain inet filter output { type filter hook output priority 0 \; policy accept \; }  

##### 1-мысал: Loopback интерфейске рұқсат ету
```shell
student@netfilter:~$ ping -c4 127.0.0.1

$ sudo nft add rule inet filter input iifname "lo" accept

student@netfilter:~$ ping -c4 127.0.0.1
```

##### 2-мысал: INPUT бойынша ICMP хаттамаға рұқсат ету
```shell
student@H1:~$ ping -c4 172.16.11.1

$ sudo iptables -A INPUT -p icmp -j ACCEPT

student@H1:~$ ping -c4 172.16.11.1
student@H1:~$ ping -c4 172.16.12.1
```

ping H1 to H2
```shell
student@H1:~$ ping -c4 172.16.11.1
student@H1:~$ ping -c4 172.16.12.101
```

Allow SSH, DNS, HTTP, HTTPS
```shell
$ sudo nft add rule inet filter input iifname "lo" accept
$ sudo nft add rule inet filter input icmp type echo-request accept
$ sudo nft add rule inet filter input tcp dport {22, 53, 80, 443} accept  
$ sudo nft add rule inet filter input udp dport 53 accept
```

Save configuration
```shell
$ sudo nft list ruleset | sudo tee /etc/nftables.conf
$ sudo systemctl restart nftables
```

Delete Rule
```shell
$ sudo nft -a list ruleset
немесе
$ sudo nft --handle list ruleset

$ sudo nft delete rule inet filter input handle 1
```

```shell
```

### 2) NAT конфигурациялау (using iptables)

##### iptables пакетін орнату және конфигурациялау
```shell
$ sudo apt update
$ sudo apt install iptables

Verify the Installation
$ sudo iptables -V
$ sudo iptables --version

Check Current Rules
$ sudo iptables -L
```

```shell
$ sudo iptables -A INPUT -i lo -j ACCEPT
$ sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

$ sudo iptables -vnL
немесе
$ sudo iptables -t filter -vnL
```

```shell
Inserting Rules
$ sudo iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT

Listing Rules
$ sudo iptables -vnL --line-numbers

Deleting Rules
$ sudo iptables -D INPUT 1
```

```shell
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP

sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
```

```shell
Clear input chain
$ sudo iptables -F INPUT

Flush the whole iptables
$ sudo iptables -F
```

```shell
Өзгерісті сақтау және қайта жүктеу (Saving and Reloading IPTables Rules)

$ sudo apt install iptables-persistent
$ sudo systemctl status netfilter-persistent

$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload

$ sudo iptables-save > /etc/iptables/rules.v4
$ sudo iptables-restore < /etc/iptables/rules.v4

/etc/network/if-pre-up.d/iptables
```

##### Network Address Translation (NAT)
```shell
$ sudo iptables -t nat -vnL

$ sudo iptables -t nat -A POSTROUTING -s 172.16.11.0/24 -o ens3 -j MASQUERADE
$ sudo iptables -t nat -A POSTROUTING -s 172.16.12.0/24 -o ens3 -j MASQUERADE
```

```shell
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

```shell
Нəтижені тексеру
VPC1> ping 8.8.8.8
VPC2> ping 8.8.8.8
```

### Troubleshooting
```shell
$ sudo iptables -F

sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP

$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```
###### 1-мысал: INPUT бойынша ICMP хаттамаға рұқсат ету
```shell
VPC1> ping 172.16.11.1
$ sudo iptables -A INPUT -p icmp -j ACCEPT
VPC1> ping 172.16.11.1
```

###### 2-мысал: FORWARD бойынша ICMP хаттамаға рұқсат ету
```shell
VPC1> ping 172.16.12.101
VPC1> ping 8.8.8.8
$ sudo iptables -A FORWARD -p icmp -j ACCEPT
VPC1> ping 172.16.12.101
VPC1> ping 8.8.8.8
```

###### 3-мысал: Conntrack және DNS портына рұқсат ету
```shell
VPC1> ping google.com

$ sudo iptables -A FORWARD -p udp --dport 53 -s 172.16.11.0/24 -j ACCEPT

$ sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

VPC1> ping google.com
```

###### 4-мысал: HTTP, HTTPS хаттамаларына рұқсат ету
```shell
$ sudo iptables -A FORWARD -p tcp -m multiport --ports 80,443 -s 172.16.11.0/24 -j ACCEPT

nc -w1 -vz 172.16.11.1 80
```

###### 5-мысал: Port Forwarding
```shell
Change external port 8080 to internal port 80:
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
```

### References

1) [Wikipedia Netfilter](https://en.wikipedia.org/wiki/Netfilter)
