
## Netfilter on Debian
  1) Firewall, NAT using nftables;
  2) Firewall, NAT using iptables.

#### 🖧 Topology
![Topology](Topology/Topology_interVLANRouting_NAT_Linux.png)

### 1) Firewall, NAT using nftables

```shell
$ dpkg -l nftables
$ dpkg -s nftables
```

```shell
$ cat /etc/nftables.conf
```

```shell
Check Current Rules
$ sudo nft list ruleset
```

```shell
$ sudo systemctl status nftables

$ sudo systemctl start nftables
$ sudo systemctl enable nftables
```

```shell
Add a New Rule to the INPUT Chain
$ sudo nft add rule inet filter INPUT tcp dport 22 counter accept
```

```shell
Save nftables Configuration

$ sudo nft list ruleset | sudo tee /etc/nftables.conf
$ sudo systemctl restart nftables

$ sudo nft list ruleset                // жедел жадыда (RAM) сақталған kernel-дегі ережелер (running-config)
$ cat /etc/nftables.conf               // дискде (HDD/SSD) сақталған конфигурация (saved-config)
```
> $ sudo nano /etc/nftables.conf  
> $ sudo systemctl restart nftables  
> $ sudo nft list ruleset  

> $ sudo nano ~/rules.nftables  
> $ sudo nft -f ~/rules.nftables  
> $ sudo nft list ruleset  

```shell
Set Default Policy to DROP (INPUT and FORWARD Chains)

$ sudo nft chain inet filter INPUT { policy drop \; }
$ sudo nft сhain inet filter FORWARD { policy drop \; }
$ sudo nft сhain inet filter OUTPUT { policy accept \; }
```

**Қосымша ақпарат**
```shell
Create a New Table and Chains

$ sudo nft add table inet filter  
$ sudo nft add chain inet filter INPUT { type filter hook input priority 0 \; policy drop \; }  
$ sudo nft add chain inet filter FORWARD { type filter hook forward priority 0 \; policy drop \; }  
$ sudo nft add chain inet filter OUTPUT { type filter hook output priority 0 \; policy accept \; }
```

##### 1-мысал: Loopback интерфейске рұқсат ету
```shell
student@gateway:~$ ping -c4 127.0.0.1

$ sudo nft add rule inet filter INPUT iifname "lo" accept

student@gateway:~$ ping -c4 127.0.0.1
```

##### 2-мысал: ping H1,H2 to Gateway (ICMP хаттамаға рұқсат ету)
```shell
student@H1:~$ ping -c4 172.16.11.1
student@gateway:~$ ping -c4 172.16.11.101

$ sudo nft add rule inet filter INPUT ip protocol icmp accept

student@H1:~$ ping -c4 172.16.11.1
student@gateway:~$ ping -c4 172.16.11.101
```
*ICMP (IPv4) type: echo-reply, echo-request, destination-unreachable, time-exceeded, parameter-problem т.б.*

> $ sudo nft add rule inet filter INPUT icmp type echo-request accept  
> $ sudo nft add rule inet filter INPUT icmp type { echo-request, echo-reply, destination-unreachable, time-exceeded } accept
  
> $ sudo nft add rule inet filter INPUT ip protocol icmp accept  

##### 3-мысал: ping H1 to H2
```shell
student@H1:~$ ping -c4 172.16.12.101
student@H2:~$ ping -c4 172.16.11.101

$ sudo nft add rule inet filter FORWARD ip saddr 172.16.11.0/24 ip daddr 172.16.12.0/24
$ sudo nft add rule inet filter FORWARD ip saddr 172.16.12.0/24 ip daddr 172.16.11.0/24

student@H1:~$ ping -c4 172.16.12.101
student@H2:~$ ping -c4 172.16.11.101
```

##### 4-мысал: LAN желідегі құрылғыларды internet желісімен байланыстыру
```shell
student@gateway:~$ ping 8.8.8.8
student@gateway:~$ ping google.com
```
```shell
student@H1:~$ ping 8.8.8.8
student@H1:~$ ping google.com
```

Priority мәндері:
| Symbolic Name | Equivalent Numeric Value |
|---------------|--------------------------|
| `raw`         | -300                     |
| `mangle`      | -150                     |
| `dstnat`      | -100                     |
| `filter`      | 0                        |
| `srcnat`      | 100                      |
| `security`    | 150                      |


```shell
Network Address Translation (NAT)

$ sudo nft add table inet nat
$ sudo nft add chain inet nat POSTROUTING { type nat hook postrouting priority srcnat \; policy accept \; }
$ sudo nft add rule inet nat POSTROUTING ip saddr 172.16.11.0/24 oifname "ens3" masquerade
```

```shell
Netfilter Connection Tracking (Conntrack)

$ sudo nft add rule inet filter INPUT ct state established,related accept
$ sudo nft add rule inet filter INPUT ct state invalid drop
```
*ct state-тің негізгі аргументтері:* new, established, related, invalid  
> $ sudo nft add rule inet filter INPUT tcp dport { 22, 80, 443 } ct state **new** accept  
> $ sudo nft add rule inet filter INPUT ct state **established,related** accept  
> $ sudo nft add rule inet filter INPUT ct state **invalid** drop  

> $ ... ct state snat log  
> $ ... ct state dnat log  
> $ ... ct status dnat  
> $ ... ct status snat  

```shell
student@gateway:~$ ping 8.8.8.8
student@gateway:~$ ping google.com
```

```shell
Allow LAN IP addresses
$ sudo nft add rule inet filter FORWARD ip saddr 172.16.11.0/24 oifname "ens3" accept
$ sudo nft add rule inet filter FORWARD ip daddr 172.16.11.0/24 iifname "ens3" accept
```
```shell
student@H1:~$ ping 8.8.8.8
student@H1:~$ ping google.com
```

Allow SSH, HTTP, HTTPS, DNS
```shell
$ sudo nft add rule inet filter INPUT tcp dport { 22, 80, 443 } ct state new accept
$ sudo nft add rule inet filter INPUT tcp dport 53 accept  
$ sudo nft add rule inet filter INPUT udp dport 53 accept
```

Add a New Rule (жаңа ереже қосу)
```shell
$ sudo nft add rule inet filter INPUT tcp dport 22 ct state new accept
```
> *add – жаңа қосқан ережені position бойынша СОҢЫНА қоюды*  
> position – ережелердің орналасу/орындалу реті

insert a New Rule (жаңа ереже енгізу)
```shell
$ sudo nft insert rule inet filter INPUT tcp dport 22 ct state new accept
```
> *insert – жаңа қосқан ережені position бойынша БАСЫНА қоюды*  
> position – ережелердің орналасу/орындалу реті

handle және index нөмір қолдану арқылы ережелердің орналасу ретін (position) алмастыру
```shell
$ sudo nft -a list ruleset        // Ережелердің handle нөмірін көру

$ sudo nft insert rule inet filter INPUT handle 2  tcp dport 22 counter accept      // енгізген ережені, handle 2 нөмірдегі ереженің алдына қою арқылы орын алмасады

$ sudo nft insert rule inet filter input index 2 tcp dport 22 ct state new accept      // орналасу реті бойынша үшінші орынға қоюды, себебі index мәні "0"-ден басталады
$ sudo nft insert rule inet filter input index 0 tcp dport 22 ct state new accept      // ережелердің ең басына қоюды
```

Delete a Rule (ережені жою)
```shell
$ sudo nft -a list ruleset        // Ережелердің handle нөмірін көру

$ sudo nft delete rule inet filter INPUT handle 13
```

Replace a Rule (ережені ауыстыру)
```shell
$ sudo nft -a list ruleset

$ sudo nft replace rule inet filter INPUT handle 14   tcp dport 22 ct state new accept      // енгізген ережені, handle 14 нөмірдегі ережемен ауыстырады
```

Clear all Rules (ережелерді тазалау)
```shell
$ sudo nft flush ruleset
```

##### Қосымша ақпарат

```shell
$ sudo nft list ruleset

$ sudo nft list tables
$ sudo nft list tables inet
$ sudo nft list table inet filter

$ sudo nft list chains
$ sudo nft list chains inet
$ sudo nft list chain inet filter input
```

### 2) Firewall, NAT using iptables

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

###### Қосымша ақпарат

```shell
Translating from iptables to nftables

$ sudo iptables-translate -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
  nft add rule ip filter input tcp dport 22 ct state new counter accept
$ sudo ip6tables-translate -A FORWARD -i eth0 -o eth2 -p udp -m multiport --dport 111,222 -j ACCEPT
немесе
$ sudo iptables-save > save.txt
$ sudo iptables-restore-translate -f save.txt > ruleset.nft
```

### References

1) [Wikipedia Netfilter](https://en.wikipedia.org/wiki/Netfilter)
