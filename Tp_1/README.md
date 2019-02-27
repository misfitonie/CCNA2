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
### Capture réseau

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

### UDP

* Ouverture port 8888 pour client 1

```bash
sudo firewall-cmd --add-port=8888/udp --permanent
```

*On écoute le port 8888 via un netcat

```bash
nc -u -l 8888
```
* Pour client2

```bash
nc -u 10.1.1.2 8888
```
* Pour client1 (2éme shell)

```bash
ss -unp
Recv-Q Send-Q Local Address:Port               Peer Address:Port              
0      0      10.1.1.2:8888               10.1.1.3:43889              users:(("nc",pid=1512,fd=4))
```

* Pour client2 (2éme shell)

```bash
ss -unp
Recv-Q Send-Q Local Address:Port               Peer Address:Port              
0      0      10.1.1.3:43889             10.1.1.2:8888                users:(("nc",pid=1494,fd=3))
```

* Pour client1 (3ème shell)

```bash
sudo tcpdump -i enp0s8 -w nc-udp.pcap
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C6 packets captured
6 packets received by filter
0 packets dropped by kernel
```

*On remarque qu'il y a bien un échange de paquets entre le client et le serveur via un protocole UDP*

### TCP

* Même chose on ouvre le port 8888 et on écoute ce port avec netcat (pour le client1)

```bash
sudo firewall-cmd --add-port=8888/tcp --permanent
```
```bash
nc -l 8888
```

* Pour client2 

```bash
nc 10.1.1.2 8888
```

* Pour client1 (2ème shell)

```bash
ss -tnp
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
ESTAB       0      0      10.1.1.2:8888               10.1.1.3:45716              users:(("nc",pid=1474,fd=5))
ESTAB       0      0      10.1.1.2:22                 10.1.1.1:50020              
ESTAB       0      0      10.1.1.2:22                 10.1.1.1:50071 
```

* Pour client2 (2ème shell)

```bash
ss -tnp
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
ESTAB       0      0      10.1.1.3:45716             10.1.1.2:8888                users:(("nc",pid=1190,fd=3))
ESTAB       0      0      10.1.1.3:22                 10.1.1.1:50072              
ESTAB       0      0      10.1.1.3:22                 10.1.1.1:50021
```

* Pour client1 (3ème shell) 

```bash
sudo tcpdump -i enp0s8 -w nc-tcp.pcap 
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C15 packets captured
15 packets received by filter
0 packets dropped by kernel
```

### Firewall

* Client1: Fermeture du port 8888 UDP puis écouté netcat sur port TCP 8888 

```bash
sudo firewall-cmd --remove-port=8888/udp --permanent
```
```bash
nc -u -l 8888
```

* Client2 

```bash
nc -u 10.1.1.2 8888
```

* Client1 (2ème shell)

```bash
sudo tcpdump -i enp0s8 -w firewall.pcap 
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
^C10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

# III. Routage statique simple

* On transforme client1 en routeur

```bash
sudo sysctl -w net.ipv4.ip_forward=1 net.ipv4.ip_forward = 1
```

* On ajoute une route statique sur net2

```bash
sudo ip route add 10.1.2.0/30 via 10.1.2.2 dev enp0s9
```

*ce qui donne:*

```bash
10.1.2.0/30 via 10.1.1.2 dev enp0s9
```

FERREIRA Théo
CASTAING Alexandre
