 -------------------------------------------------------------------------------------------
TEHTÄVÄ 1.1 - Topologian kartoitus

r1			reititin käyttäjille
r2			verkon solmureititin käyttäjille, hallintatyökaluille ja palvelimille
r3			reititin ulkoverkon suuntaan
client1			käyttäjän työasema
attacker		KALI tietoturvan testaukseen
web1			Verkkopalvelut -palvelin
db1			tietokantapalvelin
branch-client		Työasema ulkoverkossa
ansible			konfigurointi, asennus, käyttöönotto ja rutiinitehtävien automatisointityökalu
Prometheus		valvonta- ja visualisointityökalu, hakee datan kohteista
grafana			valvonta- ja visualisointityökalu, ei kerää dataa itse, käyttää muiden kantoja
zabbix			valvonta- ja visualisointityökalu, käyttää agentteja.


 -------------------------------------------------------------------------------------------
TEHTÄVÄ 1.2 - Verkkokaavio

Erillinen drawio-tiedosto

 -------------------------------------------------------------------------------------------
TEHTÄVÄ 1.3 - IP-osoitteiden dokumentointi

VERKKO               Tarkoitus                   Yhdyskäytävä
10.10.10.0/24       Käyttäjien verkko           10.10.10.1
10.10.20.0/24       Palvelimien verkko          10.10.20.1
10.10.30.0/24       Erillisen toimiston verkko  10.10.30.1
10.10.99.0/24       Hallintapalvelut            10.10.99.1
10.255.12.0/30      R1-R2 -verkko               172.20.20.1 (Reitittimien default)
10.255.23.0/30      R2-R3 -verkko               172.20.20.1

Verkko on esimerkiksi jonkin yhtiön verkon hallintaan ja valvontaan.

 -------------------------------------------------------------------------------------------
TEHTÄVÄ 1.4 - Reitityksen tutkiminen

root@client1:/# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
105: eth0@if106: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:14:14:0a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.20.20.10/24 brd 172.20.20.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe14:140a/64 scope link
       valid_lft forever preferred_lft forever
129: eth1@if130: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9500 qdisc noqueue state UP group default
    link/ether aa:c1:ab:d8:26:f2 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    altname clab-o-05031180f95d8850
    inet 10.10.10.101/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a8c1:abff:fed8:26f2/64 scope link
       valid_lft forever preferred_lft forever
root@client1:/# ip route
default via 10.10.10.1 dev eth1
10.10.10.0/24 dev eth1 proto kernel scope link src 10.10.10.101
172.20.20.0/24 dev eth0 proto kernel scope link src 172.20.20.10
root@client1:/# ping -c 4 10.10.20.101
PING 10.10.20.101 (10.10.20.101) 56(84) bytes of data.
64 bytes from 10.10.20.101: icmp_seq=1 ttl=62 time=0.272 ms
64 bytes from 10.10.20.101: icmp_seq=2 ttl=62 time=0.069 ms
64 bytes from 10.10.20.101: icmp_seq=3 ttl=62 time=0.062 ms
64 bytes from 10.10.20.101: icmp_seq=4 ttl=62 time=0.076 ms

--- 10.10.20.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2998ms
rtt min/avg/max/mdev = 0.062/0.119/0.272/0.088 ms
root@client1:/# ping -c 4 10.10.30.101
PING 10.10.30.101 (10.10.30.101) 56(84) bytes of data.
64 bytes from 10.10.30.101: icmp_seq=1 ttl=61 time=0.273 ms
64 bytes from 10.10.30.101: icmp_seq=2 ttl=61 time=0.099 ms
64 bytes from 10.10.30.101: icmp_seq=3 ttl=61 time=0.057 ms
64 bytes from 10.10.30.101: icmp_seq=4 ttl=61 time=0.078 ms

--- 10.10.30.101 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2997ms
rtt min/avg/max/mdev = 0.057/0.126/0.273/0.085 ms
root@client1:/# traceroute 10.10.30.101
traceroute to 10.10.30.101 (10.10.30.101), 30 hops max, 60 byte packets
 1  10.10.10.1 (10.10.10.1)  0.731 ms  0.593 ms  0.565 ms
 2  10.255.12.2 (10.255.12.2)  0.556 ms  0.503 ms  0.484 ms
 3  10.255.23.2 (10.255.23.2)  0.465 ms  0.397 ms  0.359 ms
 4  10.10.30.101 (10.10.30.101)  0.333 ms  0.272 ms  0.243 ms


Muihin on yhteys, paitsi 10.10.99.0/24 -verkon laitteisiin, yhdyskäytävää lukuunottamatta. Syslogin hallintaosoitteeseen saa yhteyden (172.20.20.50)

traceroute to 10.10.30.101 (branch-client)
 1  10.10.10.1 R1 (oletus yhdyskäytävä)
 2  10.255.12.2 R1-R2 välisen yhteyden R2-portin osoite
 3  10.255.23.2 R2-R3 välisen yhteyden R3-portin osoite
 4  10.10.30.101


 -------------------------------------------------------------------------------------------
 

 YHTEENVETO

Verkon dokumentaatiossa lähtökohtaisesti kulutti eniten aikaa ja aiheutti vaivaa komentojen, työkalujen ja komentorivin hyödyntämisen osaamisen puute. Suuri osa ensimmäisen tehtävän vaatimista askeleista, tai ainakin niiden toteutus, oli täysin uutta (WSL, GIT, docker, ja containerlab).
Aikaa kului eniten yksinkertaisesti ymmärtämään annettua sisältöä Verkohallinta -reposta. Joka siis oli varmasti ensimmäisen tehtävän koko ajatus, että kun lähdetään tekemään temppuja niin ympäristö on jo auttavasti tuttu.

Dokumentaation ensimmäinen askel on ymmärtää ympäristöä. Kun tarpeelliset dokumentaatiot on tehty kyseisen IT-asiantuntijan toimesta, se on samalla auttanut ymmärtämään kokonaisuutta, ja vianmäärittäminen on jo rutkasti helpompaa. Samalla tarpeelliset tiedot ovat kirjattu ylös helposti omaksuttavaan ja tarkasteltavaan muotoon. Välillä itse työtä joutuu myös tekemään joku toinen, tuuraaja, ja yksinkertaiset dokumentaatiot ovat myös toisten käytettävissä.
"Ei se itse dokumentaatio, vaan dokumentointi."