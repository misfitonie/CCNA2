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
============

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
* Remettre une route:
```bash
sudo ip route add 10.1.2.0/30
```
