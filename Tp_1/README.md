# I. Exploration du réseau d'une machine CentOS
===============================================
## 1) Mise en place
===================
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

[alt text]()