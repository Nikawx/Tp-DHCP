0. Setup
Lancement GNS3
🌞 Déterminer l'adresse MAC de vos deux machines
SHOW de la node1 :

~~~
NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
VPCS1  10.1.1.1/24          255.255.255.0     00:50:79:66:68:00  10004  127.0.0.1:10005
       fe80::250:79ff:fe66:6800/64

~~~

SHOW de la node2 :

~~~


NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
VPCS1  10.1.1.2/24          255.255.255.0     00:50:79:66:68:01  10002  127.0.0.1:10003
       fe80::250:79ff:fe66:6801/64
~~~

🌞 Définir une IP statique sur les deux machines
show ip : aucune IP aucun DNS et aucune Gateway

node1 :

~~~

ip 10.1.1.1 255.255.255.0

~~~

node2 :

~~~

ip 10.1.1.2 255.255.255.0

~~~

Les IP ont été attribuées les deux machines se ping mutuellement

🌞 Effectuer un ping d'une machine à l'autre
Après le ping nous pouvons avoir l'adresse MAC durant quelques secondes

~~~

VPCS> show arp

00:50:79:66:68:00  10.1.1.1 expires in 112 seconds


~~~

Le Switch a été ajouté et la troisième machine Node3 aussi.

~~~

ip 10.1.1.3/24
~~~

pour ajouter l'ip

Toutes les machines ping les autres

Déterminons l'adresse MAC des trois maachines :
~~~

Node1 : 00:50:79:66:68:00 10.1.1.1 expires in 112 seconds
Node2 : 00:50:79:66:68:01 10.1.1.2 expires in 107 seconds
Node3 : 00:50:79:66:68:02 10.1.1.3 expires in 115 seconds

~~~
Définir une IP statique sur les 3 machines :

~~~

ip 10.1.1.1/24
ip 10.1.1.2/24
ip 10.1.1.3/24
~~~

🌞 Effectuer un ping d'une machine à l'autre
Node 1 / Node 2 :

~~~

VPCS> ping 10.1.1.2
84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.682 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.733 ms
84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=1.153 ms
84 bytes from 10.1.1.2 icmp_seq=4 ttl=64 time=0.959 ms
84 bytes from 10.1.1.2 icmp_seq=5 ttl=64 time=0.617 ms
~~~

Node 2 / Node 3 :

~~~

VPCS> ping 10.1.1.3
84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=0.879 ms
84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=0.721 ms
84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=1.002 ms
84 bytes from 10.1.1.3 icmp_seq=4 ttl=64 time=0.950 ms
84 bytes from 10.1.1.3 icmp_seq=5 ttl=64 time=1.233 ms
~~~
Node 1 / Node 3 :
~~~

VPCS> ping 10.1.1.3
84 bytes from 10.1.1.3 icmp_seq=1 ttl=64 time=0.643 ms
84 bytes from 10.1.1.3 icmp_seq=2 ttl=64 time=1.140 ms
84 bytes from 10.1.1.3 icmp_seq=3 ttl=64 time=1.338 ms
84 bytes from 10.1.1.3 icmp_seq=4 ttl=64 time=0.854 ms
84 bytes from 10.1.1.3 icmp_seq=5 ttl=64 time=0.848 ms
~~~

🌞 Wireshark !
Installation de Rocky Linux

Nous avons accès a internet :
~~~

ping 8.8.8.8
54 bytes from 8.8.8.8 icmp_seq=1 ttl=116 time=22.2 ms

~~~

Installation du DHCP en NAT sinon rien ne marche

~~~

sudo su

dnf -y install dhcp-server
vi /etc/dhcp/dhcpd.conf

option domain-name "example.local";

option domain-name-servers 8.8.8.8, 8.8.4.4; # Google DNS, modifiez si besoin

default-lease-time 600;

max-lease-time 7200;

authoritative;

subnet 10.1.1.0 netmask 255.255.255.0 {

    range 10.1.1.10 10.1.1.50;

    option broadcast-address 10.1.1.255;

    option routers 10.1.1.1;

}
~~~

:wq pour enregister et quitter
~~~

systemctl enable --now dhcpd

systemctl start dhcpd

firewall-cmd --add-service=dhcp
firewall-cmd --runtime-to-permanent

~~~

Sur mes VPCS node1 node2 et Node3, j'ai écris dhcp.

Résultats obtenus :

~~~

Node1 :

VPCS > dhcp
DORA IP 10.1.1.12/24 GW 10.1.1.1

Node2 :

VPCS > dhcp
DORA IP 10.1.1.13.24 GW 10.1.1.1

Node3 :

VPCS > dhcp
DORA IP 10.1.1.11/24 GW 10.1.1.1

~~~
Après avoir fais le WireShark voici les résultats obtenus :

Wireshark sur le cable du Switch au Node1;

Détection du DHCP DORA sur Wireshark :
~~~
51 270.100929 0.0.0.0   255.255.255.255 DHCP 406 DHCP Discorver - Transaction ID 0x4121d366
52 270.104831 10.1.1.10 10.1.1.12       DHCP 342 DHCP Offer     - Transaction ID 0x4121d366
53 271.100967 0.0.0.0   255.255.255.255 DHCP 406 DHCP Request   - Transaction ID 0x4121d366
54 271.119047 10.1.1.10 10.1.1.12       DHCP 342 DHCP ACK       - Transaction ID 0x4121d366

~~~

DHCP Spoofing
Je viens de créer une VM Rocky #2 afin de faire les tests

Ma VM est bien dans le réseau l'ip est 10.1.1.14 attribuée par le DHCP
~~~

dnf install -y dnsmasq

~~~
Configuration de dnsmasq :
~~~

port=0
dhcp-range=10.1.1.210,10.1.1.250,255.255.255.0,12h
dhcp-authoritative
interface=enp0s3

~~~
~~~

Node 1 :


VPCS > dhcp
DORA IP 10.1.1.245/24 GW 10.1.1.50

VPCS > show ip
NAME        : VPCS
IP/MASK     : 10.1.1.245/24

~~~
~~~
Node 2:

VPCS > dhcp
DORA IP 10.1.1.246/24 GW 10.1.1.50


Node 3 :

VPCS > dhcp
DORA IP 10.1.1.27/24 GW 10.1.1.50