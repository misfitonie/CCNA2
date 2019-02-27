# I. Exploration du réseau d'une machine CentOS

## 1) Mise en place

* Combien d'adresses disponible dans un /24 ?
    
    *Il y en a 254 disponibles*

* Combien d'adresses disponible dans un /30 ?

    *Il y en a 2 disponibles*

* Quelle utilité a un /30 ?

    *Comme il n'a que 2 adresses dispo il est facile de voir qui est sur le réseau*

* NAT
```bash
curl google.com
```

*le code d'erreur 301 signifie une redirection permanente*

* NET1/NET2

```bash
ping 10.1.1.1
```
```bash
ping 10.1.2.1
```
*les deux fonctionnent*

## 2) Basics

### Routes 

* Afficher les routes:

```bash
ip route show
```

![ip_route_show](https://github.com/misfitonie/CCNA2/blob/master/Tp_1/img/iproute.PNG)

*La route par défault est la 10.0.2.2*
*enp0s3 est la route (NAT) qui permet l'accès à 10.0.2.0/24*

*enp0s8 est la route (NET1) qui permet l'accès à 10.1.1.0/24*

*enp0s9 est la route (NET2) qui permet l'accès à 10.1.2.0/30*

* Supprimer une route:

```bash
sudo ip route del 10.1.2.0/30
```
![del_route](https://github.com/misfitonie/CCNA2/blob/master/Tp_1/img/Capture3.PNG)

* Remettre une route:
```bash
sudo ip route add 10.1.2.0/30 via 10.1.2.2 dev enp0s9
```

```bash
ip route show

default via 10.0.2.2 dev enp0s3 proto static metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 100 
10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 100 
```
*10.1.2.0/30 c'est bien rajouter*

### Table ARP

* Afficher la table ARP

```bash
ip neight show

10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
```

*Cela veut dire que j'ai déjà échanger avec 10.1.1.0*

* Vider la table ARP

```bash
sudo ip neight flush all
```

* Requête vers l'hôte

```bash
ping 10.1.1.2
```

```bash
ip neight show 

10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
```
###Capture réseau

* On capture 10 paquets

```bash
sudo tcpdump -i enp0s9 -w ping.pcap 
tcpdump: listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
^C10 packets captured
10 packets received by filter
0 packets dropped by kernel
```
![table whireshark](https://github.com/misfitonie/CCNA2/blob/master/Tp_1/img/Capture.PNG)

# II. Communication entre deux machines

## 2. Basics

### Ping et ARP

* Vider les ARP

```bash
sudo ip meight flush all
```

* Ping client2

```bash
ping -c 4 client2 (10.1.1.3)
```
*Il y a des changements dans la table ARP*

```bash
ip neight show

10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:06 REACHABLE
10.1.1.3 dev enp0s8 lladdr 08:00:27:26:2e:dc REACHABLE
```

###UDP

* Ouverture port 8888 pour client 1

```bash
sudo firewall-cmd --add-port=8888/udp --permanent
```

   *On écoute le port 8888 via un netcat

```bash
nc -u -l 8888
```

