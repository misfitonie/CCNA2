# Rendu TP 3 - Utilisation de matériel Cisco

## I. Manipulation de switches et de VLAN

### 1. Mise en place du lab

* Sur les 3 clients

    * ajouter une ip
        ```bash
            cat /etc/sysconfig/network-scripts/ifcfg-enp0s8

            TYPE=Ethernet
            BOOTPROTO=static
            NAME=enp0s8
            DEVICE=enp0s8
            ONBOOT=yes
            IPADDR=10.1.2.1
            NETMASK=255.255.255.0

            sudo ifdown enp0s8
            sudo ifup enp0s8
        ```
    * Changer l'hostname
        ```bash
            sudo hostname client1.lab1.tp3
        ```

    * Test de ping client1->client3
        ```bash
            ping 10.1.1.3

            PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
            64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=3.72 ms
            64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=1.28 ms
            ^C
            --- 10.1.1.3 ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 2009ms
        ```
    
### 2. Configuration des VLANs

* Mise en place des ports en mode access

    * Sur le switch 1
        * Création de VLAN 10 
            ```bash
                conf t
                vlan 10
                name VLAN 10
                exit
            ```
        * Assigner une interface pour donner accès au VLAN
            ```bash
                interface Ethernet 0/0
                switchport mode access
                switchport access vlan 10
            ```

        * Création de VLAN 20
            ```bash
                conf t
                vlan 20
                name VLAN 20
                exit
            ```
        * Assigner une interface pour donner accès au VLAN
            ```bash
                interface Ethernet 0/1
                switchport mode access
                switchport access vlan 20
            ```

        * Configurer une interface entre les 2 Switch (mode Trunk)
            ```bash
                interface Ethernet 0/2
                switchport trunk encapsulation dot1q
                switchport mode trunk
            ```

    * Sur le switch 2
        * Création de VLAN 10 
            ```bash
                conf t
                vlan 10
                name VLAN 10
                exit
            ```
        * Assigner une interface pour donner accès au VLAN
            ```bash
                interface Ethernet 0/0
                switchport mode access
                switchport access vlan 10
            ```

        * Configurer une interface entre les 2 Switch (mode Trunk)
            ```bash
                interface Ethernet 0/1
                switchport trunk encapsulation dot1q
                switchport mode trunk
            ```
    * Verification :
        * Ping client1->client3
            ```bash
                ping 10.1.1.3
                
                PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
                64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=2.32 ms
                64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=1.27 ms
                ^C
                --- 10.1.1.3 ping statistics ---
                2 packets transmitted, 2 received, 0% packet loss, time 2001ms
            ```
        * Traceroute client1->client3
            ```bash
                traceroute 10.1.1.3
                
                traceroute to 10.1.1.3 (10.1.1.3), 30 hops max, 60 byte packets
                1  10.1.1.3 (10.1.1.3)  2.256 ms !X  2.124 ms !X  4.359 ms !X
            ```
        * Ping client1->client2
            ```bash
                ping 10.1.1.2
                
                PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
                From 10.1.1.1 icmp_seq=1 Destination Host Unreachable
                From 10.1.1.1 icmp_seq=2 Destination Host Unreachable
                From 10.1.1.1 icmp_seq=3 Destination Host Unreachable
                ^C
                --- 10.1.1.2 ping statistics ---
                4 packets transmitted, 0 received, +3 errors, 100% packet loss, time 4006ms
            ```

## II. Manipulation simple de routeurs

### 1. Mise en place du lab
* Client 1 
    - ip : 10.2.1.10 (lab2-net1)
    - hostname : client1.lab2.tp3

*  Client 2
    - ip : 10.2.1.11 (lab2-net1)
    - hostname : client2.lab2.tp3

* Server 1
    - ip : 10.2.2.10 (lab2-net2)
    - hostname : server1.lab2.tp3

* Router 1
    - ip : 10.2.1.254 (lab2-net1) && 10.2.12.1/30 (lab2-net12)
    - hostname : router1.lab2.tp3

* Router 2
    - ip : 10.2.2.254 (lab2-net2) && 10.2.12.2/30 (lab2-net12)
    - hostname : router2.lab2.tp3


* Verification 
    * Client2->Router1
        ```bash
            ping 10.2.1.254
                
            PING 10.2.1.254 (10.2.1.254) 56(84) bytes of data.
            64 bytes from 10.2.1.254: icmp_seq=1 ttl=255 time=1.98 ms
            64 bytes from 10.2.1.254: icmp_seq=2 ttl=255 time=1.92 ms
            ^C
            --- 10.2.1.254 ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 2001ms
        ```

    * Server1->Router2
        ```bash
            ping 10.2.2.254
                
            PING 10.2.2.254 (10.2.2.254) 56(84) bytes of data.
            64 bytes from 10.2.2.254: icmp_seq=1 ttl=255 time=1.87 ms
            64 bytes from 10.2.2.254: icmp_seq=2 ttl=255 time=1.54 ms
            ^C
            --- 10.2.2.254 ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 2008ms
        ```
    
    * Router2->Route1
        ```bash
            ping 10.2.12.1
                
            PING 10.2.12.1 (10.2.12.1) 56(84) bytes of data.
            64 bytes from 10.2.12.1: icmp_seq=1 ttl=255 time=1.93 ms
            64 bytes from 10.2.12.1: icmp_seq=2 ttl=255 time=1.87 ms
            ^C
            --- 10.2.12.1 ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 2002ms
        ```

### 2. Configuration du routage statique

* Ajouter des routes :

    - Client2 :
        ```bash
            sudo ip route add 10.2.2.0/24 via 10.2.1.254 dev enp0s3
        ```
    - Server1 :
        ```bash
            sudo ip route add 10.2.1.0/24 via 10.2.2.254 dev enp0s3
        ```
    - Router1 :
        ```bash
            sudo ip route 10.2.2.0 255.255.255.0 10.2.12.2
        ```

    - Router2 :
        ```bash
            sudo ip route 10.2.1.0 255.255.255.0 10.2.12.1
        ```

* Verification

    * Client2->Server1
        ```bash
            ping 10.2.2.10
                
            PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
            64 bytes from 10.2.2.10: icmp_seq=1 ttl=255 time=1.98 ms
            64 bytes from 10.2.2.10: icmp_seq=2 ttl=255 time=1.92 ms
            64 bytes from 10.2.2.10: icmp_seq=2 ttl=255 time=1.95 ms
            ^C
            --- 10.2.2.10 ping statistics ---
            3 packets transmitted, 3 received, 0% packet loss, time 2002ms
        ```

    * Server1->Client2
        ```bash
            ping 10.2.1.11
                
            PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
            64 bytes from 10.2.1.11: icmp_seq=1 ttl=255 time=1.91 ms
            64 bytes from 10.2.1.11: icmp_seq=2 ttl=255 time=1.83 ms
            ^C
            --- 10.2.1.11 ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 2002ms
        ```

* Proposition de Topologie

```
    client1       switch                router1       router2
    +----+       +-------+              +------+      +------+
    |    +-------+       +--------------+      +------+      |
    +----+       +-------+              +------+      +------+
                     |                                    |
                     |                                    |
                     |                                    |
                   +----+                               +----+ 
                   |    |                               |    |
                   +----+                               +----+
                   client2                              server1
```

## III. Mise en place d'OSPF

### 1. Mise en place du lab

* Tableau d'adressage

    | Hosts            | 10.3.100.0/30 | 10.3.100.4/30 | 10.3.100.8/30  | 10.3.100.12/30 | 10.3.100.16/30 | 10.3.100.20/30 | 10.3.101.0/24   | 10.3.102.0/24   |
    |------------------|---------------|---------------|----------------|----------------|----------------|----------------|-----------------|-----------------|
    | client1.lab3.tp3 | x             | x             | x              | x              | x              | x              | 10.3.101.10/24  | x               |
    | server1.lab3.tp3 | x             | x             | x              | x              | x              | x              | x               | 10.3.102.10/24  |
    | router1.lab3.tp3 | 10.3.100.1/30 | x             | x              | x              | x              | 10.3.100.22/30 | x               | 10.3.102.254/24 |
    | router2.lab3.tp3 | 10.3.100.2/30 | 10.3.100.5/30 | x              | x              | x              | x              | x               | x               |
    | router3.lab3.tp3 | x             | 10.3.100.6/30 | 10.3.100.9/30  | x              | x              | x              | x               | x               |
    | router4.lab3.tp3 | x             | x             | 10.3.100.10/30 | 10.3.100.13/30 | x              | x              | 10.3.101.254/24 | x               |
    | router5.lab3.tp3 | x             | x             | x              | 10.3.100.14/30 | 10.3.100.17/30 | x              | x               | x               |
    | router6.lab3.tp3 | x             | x             | x              | x              | 10.3.100.18/30 | 10.3.100.21/30 | x               | x               |

* Verification 

    * Client1->Router4
        ```bash
            ping 10.3.101.254

            PING 10.3.101.254 (10.3.101.254) 56(84) bytes of data.
            64 bytes from 10.3.101.254: icmp_seq=1 ttl=62 time=2.15 ms
            64 bytes from 10.3.101.254: icmp_seq=2 ttl=62 time=2.64 ms
            ^C
            --- 10.3.101.254 ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 2106ms
        ```

    * Server1->Router1
        ```bash
            ping 10.3.102.254

            PING 10.3.102.254 (10.3.102.254) 56(84) bytes of data.
            64 bytes from 10.3.102.254: icmp_seq=1 ttl=62 time=2.37 ms
            64 bytes from 10.3.102.254: icmp_seq=1 ttl=62 time=2.60 ms
            64 bytes from 10.3.102.254: icmp_seq=1 ttl=62 time=2.32 ms
            ^C
            --- 10.3.102.254 ping statistics ---
            3 packets transmitted, 3 received, 0% packet loss, time 2562ms
        ```
    
    * Router5->Router6
        ```bash
            ping 10.3.100.18

            PING 10.3.100.18 (10.3.100.18) 56(84) bytes of data.
            64 bytes from 10.3.100.18: icmp_seq=1 ttl=62 time=1.91 ms
            64 bytes from 10.3.100.18: icmp_seq=1 ttl=62 time=1.97 ms
            64 bytes from 10.3.100.18: icmp_seq=1 ttl=62 time=1.88 ms
            64 bytes from 10.3.100.18: icmp_seq=1 ttl=62 time=2.02 ms
            ^C
            --- 10.3.100.18 ping statistics ---
            4 packets transmitted, 4 received, 0% packet loss, time 2248ms
        ```

### 2. Configuration de OSPF

* OSPF sur tout les routers (Je montre la manip que pour 1)

    * Activation 
        ```bash
            router ospf 1
        ``` 
    * Attribution d'un id
        ```bash
            router-id 1.1.1.1
        ``` 
    * Partage de tous les réseaux auquel le routeur est connecté 
        ```bash
           network 10.3.100.0 0.0.0.3 area 0
           network 10.3.102.0 0.0.0.255 area 2
        ```

* Verification 
    
    * Router2->Router3
         ```bash
            ping 10.3.100.6

            PING 10.3.100.6 (10.3.100.6) 56(84) bytes of data.
            64 bytes from 10.3.100.6: icmp_seq=1 ttl=62 time=1.75 ms
            64 bytes from 10.3.100.6: icmp_seq=1 ttl=62 time=1.82 ms
            64 bytes from 10.3.100.6: icmp_seq=1 ttl=62 time=1.79 ms
            ^C
            --- 10.3.100.18 ping statistics ---
            3 packets transmitted, 3 received, 0% packet loss, time 2102ms
        ```
    
    * Client1->Server1 
        ```bash
            ping 10.3.102.10

            PING 10.3.102.10 (10.3.102.10) 56(84) bytes of data.
            64 bytes from 10.3.102.10: icmp_seq=1 ttl=62 time=2.17 ms
            64 bytes from 110.3.102.10: icmp_seq=1 ttl=62 time=2.09 ms
            ^C
            --- 10.3.102.10 ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 2025ms
        ```
        
## IV. Lab Final

* Topologie

    ```
        client1       switch1              router1       router2            switch2         server2
        +-----+       +-------+            +-----+       +-----+            +-------+       +-----+
        |     +-------+       +------------+     +-------+     +------------+       +-------+     |
        +-----+  V10  +---+---+            +--+--+       +--+--+            +-------+  V40  +-----+
                          |                   |             |                   |
                          | V20               |   router3   |                   | V30
                          |                   |   +-----+   |                   |
                       +--+--+                +---+     +---+                +--+--+ 
                       |     |                    +--+--+                    |     |
                       +-----+                       |                       +-----+
                       client2                       |                       server1
                                                     |
                                                  +--+--+
                                                  |     |
                                                  +-----+
                                                  nat
        *V = VLAN
    ```       
* Tableau d'adressage

    | Hosts            | 10.3.10.0/24   | 10.3.11.0/24   | 10.3.12.4/30   | 10.3.13.12/30  | 10.3.14.20/30  |
    |------------------|----------------|----------------|----------------|----------------|----------------|
    | client1.lab4.tp3 | 10.3.10.1/24   | x              | x              | x              | x              |
    | client2.lab4.tp3 | 10.3.10.2/24   | x              | x              | x              | x              |
    | server1.lab4.tp3 | x              | 10.3.11.1/24   | x              | x              | x              |
    | server2.lab4.tp3 | x              | 10.3.11.2/24   | x              | x              | x              |
    | router1.lab4.tp3 | 10.3.10.254/24 | x              | 10.3.12.5/30   | 10.3.13.13/30  | x              | 
    | router2.lab4.tp3 | x              | 10.3.11.254/24 | 10.3.12.6/30   | x              | 10.3.14.21/30  | 
    | router3.lab4.tp3 | x              | x              | x              | 10.3.13.14/30  | 10.3.14.22/30  |


1. Configuration des clients et des servers
    * Ajouter une ip - *Exemple sur client1*
        ```bash
            cat /etc/sysconfig/network-scripts/ifcfg-enp0s8

            TYPE=Ethernet
            BOOTPROTO=static
            NAME=enp0s8
            DEVICE=enp0s8
            ONBOOT=yes
            IPADDR=10.3.10.1
            NETMASK=255.255.255.0

            sudo ifdown enp0s8
            sudo ifup enp0s8
        ```
    * Changer l'hostname
        ```bash
            sudo hostname client1.lab4.tp3
        ```
    
    * Test de ping client1->client2
        ```bash
            ping 10.3.10.2

            PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
            64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=2.72 ms
            64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=2.25 ms
            ^C
            --- 10.1.1.2 ping statistics ---
            2 packets transmitted, 2 received, 0% packet loss, time 2104ms
        ```

2. Configuration des 2 switch
    * Mise en place des ports en mode access

        * Sur switch1
            * Création de VLAN 10 
                ```bash
                    conf t
                    vlan 10
                    name VLAN 10
                    exit
                ```
            * Assigner une interface pour donner accès au VLAN
                ```bash
                    interface Ethernet 0/0
                    switchport mode access
                    switchport access vlan 10
                ```

            * Création de VLAN 20
                ```bash
                    conf t
                    vlan 20
                    name VLAN 20
                    exit
                ```
            * Assigner une interface pour donner accès au VLAN
                ```bash
                    interface Ethernet 0/1
                    switchport mode access
                    switchport access vlan 20
                ```

            * Configurer une interface entre switch1 et router1 (mode Trunk)
                ```bash
                    interface Ethernet 0/2
                    switchport trunk encapsulation dot1q
                    switchport mode trunk
                ```

        * Sur switch2
            * Création de VLAN 30 
                ```bash
                    conf t
                    vlan 30
                    name VLAN 30
                    exit
                ```
            * Assigner une interface pour donner accès au VLAN
                ```bash
                    interface Ethernet 0/0
                    switchport mode access
                    switchport access vlan 30
                ```
            * Création de VLAN 40
                ```bash
                    conf t
                    vlan 40
                    name VLAN 40
                    exit
                ```
            * Assigner une interface pour donner accès au VLAN
                ```bash
                    interface Ethernet 0/1
                    switchport mode access
                    switchport access vlan 40
                ```

        * Configurer une interface entre switch2 et router2 (mode Trunk)
                ```bash
                    interface Ethernet 0/2
                    switchport trunk encapsulation dot1q
                    switchport mode trunk
                ```

3. Configuration inter-vlan

    * sur router1
        ```bash
            interface FastEthernet1/0.10
            encap dot1Q 10 
            ip add 10.3.10.3 255.255.255.0 
            no shut
            exit
        ```

        ```bash
            interface FastEthernet1/0.20
            encap dot1Q 20 
            ip add 10.3.10.3 255.255.255.0 
            no shut
            exit
        ```
    * sur router2
        ```bash
            interface FastEthernet1/0.30
            encap dot1Q 30 
            ip add 10.3.11.3 255.255.255.0 
            no shut
            exit
        ```

        ```bash
            interface FastEthernet1/0.40
            encap dot1Q 40 
            ip add 10.3.11.3 255.255.255.0 
            no shut
            exit
        ```

4. Configuration des 3 routers
    * OSPF sur tout les routers 

        * Activation - *Exemple sur router1*
            ```bash
                router ospf 1
            ``` 
        * Attribution d'un id
            ```bash
                router-id 1.1.1.1
            ``` 
        * Partage de tous les réseaux auquel le routeur est connecté 
            ```bash
                network 10.3.10.0  0.0.0.255 area 0
                network 10.3.12.4 0.0.0.3 area 0
                network 10.3.13.12 0.0.0.3 area 0
            ```

5. Configuration de la NAT
    * sur router3
        ```bash
            conf t
            interface FastEthernet0/0 
            ip address dhcp
            no shut
        ```
    * On vient de recupérer l'ip : 192.168.122.82

    * Acces aux autres router 
         ```bash
            interface FastEthernet0/0
            ip nat outside
            exit

            interface FastEthernet0/1
            ip nat outside
            exit

            interface FastEthernet0/2
            ip nat outside
            exit

            ip nat inside source list 1 interface fastEthernet0/0 overload
            access-list 1 permit any
        ```
    * Default passerelle
        ```bash
            router ospf 1
            default-information originate
        ```
    * Reste à tester si les clients et les servers ont accès à internet à présent
        * client1
            ```bash
                ping 8.8.8.8

                PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
                64 bytes from 8.8.8.8: icmp_seq=1 ttl=64 time=3.01 ms
                64 bytes from 8.8.8.8: icmp_seq=2 ttl=64 time=2.97 ms
                64 bytes from 8.8.8.8: icmp_seq=2 ttl=64 time=2.91 ms
                ^C
                --- 8.8.8.8 ping statistics ---
                3 packets transmitted, 3 received, 0% packet loss, time 2108ms
            ```

6. Test si les clients peuvent ping les servers
    * Avant faut mettre les *GATEWAY* (J'avais oublié)
        * Sur client1 & client2
            ```bash
                cat /etc/sysconfig/network-scripts/ifcfg-enp0s8

                TYPE=Ethernet
                BOOTPROTO=static
                NAME=enp0s8
                DEVICE=enp0s8
                ONBOOT=yes
                IPADDR=10.3.10.1
                NETMASK=255.255.255.0
                GATEWAY=10.3.10.254
            ```
        * Sur server1 et server2
            ```bash
                cat /etc/sysconfig/network-scripts/ifcfg-enp0s8

                TYPE=Ethernet
                BOOTPROTO=static
                NAME=enp0s8
                DEVICE=enp0s8
                ONBOOT=yes
                IPADDR=10.3.11.1
                NETMASK=255.255.255.0
                GATEWAY=10.3.11.254
            ```
        
    * ping client1->server1
        ```bash
            ping 10.3.11.1

            PING 10.3.11.1 (10.3.11.1) 56(84) bytes of data.
            64 bytes from 10.3.11.1: icmp_seq=1 ttl=64 time=2.92 ms
            64 bytes from 10.3.11.1: icmp_seq=1 ttl=64 time=2.27 ms
            64 bytes from 10.3.11.1: icmp_seq=1 ttl=64 time=2.32 ms
            64 bytes from 10.3.11.1: icmp_seq=1 ttl=64 time=2.59 ms
            ^C
            --- 10.3.11.1 ping statistics ---
            4 packets transmitted, 4 received, 0% packet loss, time 2731ms
        ```

7. Installation d'un service d'infra
    * Comme dans le Tp précédent, on va utiliser nginx
        * Installation sur server1
            ```bash
                sudo yum install -y nginx
            ```
        * On ouvre le port 80 (http)
            ```bash
                sudo firewall-cmd --add-port=80/tcp --permanent
            ```
        * On reload la conf
            ```bash
                sudo firewall-cmd --reload
            ```
        * On lance le server
            ```bash
                sudo systemctl start nginx
            ```

    * Depuis client1
        * Curl du site de server1
            ```bash
                curl 10.3.11.1
            ```

        * Résultat
            ```html
                <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
                    <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">

                    <head>
                        <title>Test Nginx HTTP Server by Lucas Consejo</title>
                        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
                    </head>

                    <body>
                        <h1>Welcome to my test page</h1>

                        <div class="content">
                            <p>WOOOOOOOW !!!!!! If you see this page, it means that your config is working properly! YOUHOU! You finished the TP !!!!!!!! </p>

                            <p>Here is the repo git of the rendering of the TP : <a href="https://github.com/lucasconsejo/CCNA2-TPs/tree/master/tp-3">link to the repo git</a></p>
                        </div>
                    </body>
                </html>
            ```

## Auteur
Rédigé par [Lucas Consejo](https://github.com/lucasconsejo) - Etudiant Ingésup B2B [Ynov Bordeaux](https://www.ynov.com/)