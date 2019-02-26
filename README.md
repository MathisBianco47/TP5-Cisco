# TP5-Cisco

vous aurez besoin de :

+ Virtualbox
+ GNS3

les machines virtuelles Linux :

+ l'OS devra être CentOS 7 (en version minimale)
+ pas d'interface graphique (que de la ligne de commande)

les routeurs Cisco :

+ l'iOS devra être celui d'un Cisco 3640

# I. Préparation du lab #

**1.Préparation des VMs**
+ peu importe l'adresse on s'en servira juste pour faire du SSH
+ activez le DHCP comme ça on aura pas besoin de saisir les IPs

**2. Création des VMs**

+ On va juste cloner trois VMs :
+ server1.tp5.b1 est dans net1 et porte l'IP `10.5.1.10/24`
+ client1.tp5.b1 est dans net2 et porte l'IP `10.5.2.10/24`
+ client2.tp5.b1 est dans net2 et porte l'IP `10.5.2.11/24`
+ Ajoutez aux trois VMs une interface host-only en deuxième carte dans le host-only précédemment créé

**3. Vous clonez juste les VMs, vous ne les allumez pas.**

+ Ensuite RDV dans GNS3 : Preferences > VirtualBox VMs et vous ajoutez les trois VMs.

**4. Config réseau des dans GNS3**

+ Sur les 3 VMs
+ Configure > Network
+ faire 2 cartes réseau

### 2. Préparation Routeurs Cisco

Le réseau utilisé est 10.5.3.0/30 afin d'avoir comme IP possible 10.5.12.1 et 10.5.12.2 pour les deux routeurs.

Récapitulation des IPs:

| Machines       |    net1    |    net2    |    net3   |
|----------------|:----------:|:----------:|:---------:|
| client1.tp5.b1 |      X     |  10.5.2.10 |     X     |
| client2.tp5.b1 |      X     |  10.5.2.11 |     X     |
| router1.tp5.b1 | 10.5.1.254 |      X     | 10.5.12.1 |
| router2.tp5.b1 |      X     | 10.5.2.254 | 10.5.12.2 |
| server1.tp5.b1 |  10.5.1.10 |      X     |     X     |


## II. Lancement et configuration du lab

### Checklist IP VMs

+ Désactivation de SELinux  
+ carte NAT déjà désactivée dans le patron.

Définir les IPs statiques, les Noms de domaines et avoir une connexion SSH.

### Checklist IP Routeurs

Pour la définition des IPs statiques et des noms de domaines des routeurs `router1.tp5.b1` et `router2.tp5.b1`.


Pour le `router1`:

```
 ip address 10.5.1.254 255.255.255.0
```

```
 hostname router1.tp5.b1
```

### Checklist routes

- [ ] `router1`:

  ```
  ip route 10.5.2.0 255.255.255.0 10.5.12.2
  ```
 
 - [ ] `server1`:
 
  - Routes

    ```
     nano /etc/sysconfig/network-scripts/route-eth0

    10.2.0.0/24 via 10.2.0.254 dev eth0
    ```
  
  - `Hosts`
    
    ```
     nano /etc/hosts
    
    10.5.2.10 client1 client1.tp5.b1
    10.5.2.11 client2 client2.tp5.b1
    ```

Depuis client2:

```
 ping client1.tp5.b1 -c 2 && echo &&  ping server1.tp5.b1 -c 2
PING client1 (10.5.2.10) 56(84) bytes of data.
64 bytes from client1 (10.5.2.10): icmp_seq=1 ttl=64 time=6.56 ms
64 bytes from client1 (10.5.2.10): icmp_seq=2 ttl=64 time=4.80 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 4.830/5.683/6.537/0.856 ms

PING server1 (10.5.1.10) 56(84) bytes of data.
64 bytes from server1 (10.5.1.10): icmp_seq=1 ttl=62 time=27.8 ms
64 bytes from server1 (10.5.1.10): icmp_seq=2 ttl=62 time=33.8 ms

--- server1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 27.911/30.842/33.774/2.936 ms
```

## III. DHCP
### 1. Mise en place du serveur DHCP
#### 1. Renomme la machine

Machine en permanent

```
 nano /etc/hostname
hostname dhcp-net2.tp5.b1
```

#### 2. Installe le serveur DHCP
Activation de la carte NAT `ifup eth2`

```
 ip a show dev eth2
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:00:00:05:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.17/24 brd 192.168.20.255 scope global noprefixroute dynamic eth2
       valid_lft 7143sec preferred_lft 7143sec
    inet6 fe80::250:ff:fe00:502/64 scope link 
       valid_lft forever preferred_lft forever
```

le package installes `DHCP` sur les machines

```
 yum install -y dhcp
```

```
 ifup eth0 && ifdown eth2
```

#### 5. Démarre le serveur DHCP

Modification du fichier de configuration puis on le lance

```
 systemctl start dhcpd
```

Pour le démarrage automatique :

```
 systemctl enable dhcpd
```
Vérification du service

```
 systemctl status dhcpd -l
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since mer. 2019-02-20 17:27:20 CET; 2min 48s ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 3881 (dhcpd)
   Status: "Dispatching packets..."
   CGroup: /system.slice/dhcpd.service
           └─3881 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Listening on LPF/eth0/00:50:00:00:05:00/10.5.2.0/24
févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Sending on   LPF/eth0/00:50:00:00:05:00/10.5.2.0/24
févr. 20 17:27:20 dhcp-net2.tp5.b1 dhcpd[3881]: Sending on   Socket/fallback/fallback-net
févr. 20 17:27:20 dhcp-net2.tp5.b1 systemd[1]: Started DHCPv4 Server Daemon.
```

#### 6. Faire un test
En utilisant la VM `client1.tp5.b1`:
- Modification du fichier `/etc/sysconfig/network-scripts/ifcfg-eth0` pour une modification permanante
- `dhclient -v` pour demander une adresse ip.

```
 dhclient -v eth0
Internet Systems Consortium DHCP Client 4.2.5
Copyright 2004-2013 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/eth0/00:50:00:00:04:00
Sending on   LPF/eth0/00:50:00:00:04:00
Sending on   Socket/fallback
DHCPDISCOVER on eth0 to 255.255.255.255 port 67 interval 3 (xid=0x6ad2d7)
DHCPREQUEST on eth0 to 255.255.255.255 port 67 (xid=0x6ad2d7)
DHCPOFFER from 10.5.2.11
DHCPACK from 10.5.2.11 (xid=0x6ad2d7)
bound to 10.5.2.50 -- renewal in 254 seconds.
