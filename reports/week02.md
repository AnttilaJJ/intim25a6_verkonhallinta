1. Johdanto 
Mikä on SNMP? 
SNMP (simple network management protocol) on tiedon jäsentely ja keräys protokolla. Protokollassa on palvelimilla olevia agentteja, jotka jäsentelevät tiedot standartoidusti, jollakin tietokoneella olevalle, managerille annettavaksi sitä kysyttäessä. 
Sen avulla kerätään tietoja palvelimien sen hetkisestä tilasta ja käytettävästä infrastruktuurista, ja se on myös kanava jota kautta voi muokata määritettyjä arvoja.


2. Asennus 
Miten SNMP-agentti asennettiin? 
agentti kohteeseen:
apt install snmp snmpd -y 
agentaddress udp:161 (sallii liikenteen ulkopuolelta)

Manageri kohteeseen:
apt install snmp -y (ei snmpd:tä managerille)
apt install -y snmp-mibs-downloader (jotta voi käyttää nimiä, eikä tarvitse osoittaa OID:lla)
download-mibs (lopun kirjaston lataus)


3. Kerätyt tiedot 
Suoritetut SNMP-kyselyt 
snmpwalk -v2c -c public web1 system

Nimi: SNMPv2-MIB::sysName.0 = STRING: web1
Käyttöjärjestelmä: SNMPv2-MIB::sysDescr.0 = STRING: Linux web1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64
Uptime: DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (1643) 0:00:16.43

snmpget -v2c -c public web1 sysName.0

root@ansible:/# snmpget -v2c -c public web1 sysName.0
SNMPv2-MIB::sysName.0 = STRING: web1


snmpget -v2c -c public web1 sysDescr.0

root@ansible:/# snmpget -v2c -c public web1 sysDescr.0
SNMPv2-MIB::sysDescr.0 = STRING: Linux web1 6.18.33.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 18 21:54:43 UTC 2026 x86_64


snmpget -v2c -c public web1 sysUpTime.0

root@ansible:/# snmpget -v2c -c public web1 sysUpTime.0
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (263716) 0:43:57.16


Laite, nimi, käyttöjärjestelmä ja uptime
web1
web1
Linux 6.18.33.2-microsoft-standard-WSL2 x86_64
1:12:46.31 (h:m:s)

db1
db1
Linux 6.18.33.2-microsoft-standard-WSL2 x86_64
0:04:26.25

branch-client
branch-client
Linux 6.18.33.2-microsoft-standard-WSL2  x86_64
0:01:33.06


4. Verkkorajapinnat 
SNMP:n avulla kerätyt verkkorajapintatiedot 

root@ansible:/# snmpwalk -v2c -c public web1 ifDescr
IF-MIB::ifDescr.1 = STRING: lo
IF-MIB::ifDescr.89 = STRING: eth0
IF-MIB::ifDescr.120 = STRING: eth1

Päällisin puolin ei voi tietää kumpi portti tekee mitä, joten selvitetään sisään tulevat ja ulos menevät bitit

root@ansible:/# snmpget -v2c -c public web1 .1.3.6.1.2.1.2.2.1.10.89 .1.3.6.1.2.1.2.2.1.16.89
IF-MIB::ifInOctets.89 = Counter32: 2799300
IF-MIB::ifOutOctets.89 = Counter32: 13312662
root@ansible:/# snmpget -v2c -c public web1 .1.3.6.1.2.1.2.2.1.10.120 .1.3.6.1.2.1.2.2.1.16.120
IF-MIB::ifInOctets.120 = Counter32: 12931948
IF-MIB::ifOutOctets.120 = Counter32: 116264

Todennäköisesti eth0 (89) on ulkoverkkoon osoittava portti, sillä ulos menevä bittimäärä on suurempi kuin sisään tuleva. Verkkopalvelin ymmärtääkseni harvemmin hakee/saa tietoa, kun taas käyttäjät saattavat ladata palvelimelta tietoja.


5. OID-analyysi 
Käytetyt OID-objektit 
OID                 Tarkoitus
https://www.rfc-editor.org/info/rfc3418/
sysName.0	        An administratively-assigned name for this managed
               node.  By convention, this is the node's fully-qualified
               domain name.  If the name is unknown, the value is
               the zero-length string.
sysDescr.0	        A textual description of the entity.
                This value should include the full name and version identification of the system's hardware type, software operating-system, and networking software.
sysUpTime.0	        The time (in hundredths of a second) since the
               network management portion of the system was last
               re-initialized.
https://www.rfc-editor.org/info/rfc2863/
ifDescr	            A textual string containing information about the
            interface.  This string should include the name of the
            manufacturer, the product name and the version of the
            interface hardware/software.
ifOperStatus        The current operational state of the interface.  The
            testing(3) state indicates that no operational packets can
            be passed.  If ifAdminStatus is down(2) then ifOperStatus
            should be down(2).  If ifAdminStatus is changed to up(1)
            then ifOperStatus should change to up(1) if the interface is
            ready to transmit and receive network traffic; it should
            change to dormant(5) if the interface is waiting for
            external actions (such as a serial line waiting for an
            incoming connection); it should remain in the down(2) state
            if and only if there is a fault that prevents it from going
            to the up(1) state; it should remain in the notPresent(6)
            state if the interface has missing (typically, hardware)
            components.


6. Pohdinta 
Omat havainnot SNMP:n hyödyistä ja rajoituksista.

SNMP:n käyttöön ottajan ja asentajan tulee varmistua, että agenttien välittämä tieto vaatii tunnistautumisen ja enkryptauksen, sekä minne tietoa verkossa jaetaan (portit, osoitteet). Ulkoisen hyökkääjän on helppo hyödyntää mahdollisia saamiaan tietojaan tunkeutumisen jatkamiseksi.

Puutteellisella MIB-tietokannalla, tai sen kanssakin, tietojen tulkitseminen on aikaa vievää ilman GUI:ta.
Protokollalla saa paljon hyödyllistä tietoa, mutta laajemmassa käytössä se vaatii lisätyökaluja.