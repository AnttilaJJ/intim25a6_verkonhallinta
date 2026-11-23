1. Johdanto
Zabbix on yleisluontoinen valvontatyökalu. Zabbix kykenee seuraamaan useita eri parametrejä. Zabbixilla voidaan valvoa verkkoja, palvelimia, virtuaalikoneita, sovelluksia, palveluita, tietovarantoja, web-sivuja, pilviä ja paljon muuta. Zabbixissa on joustava hälytystoiminnallisuus, jolla voidaan lähettää ilmenevistä ongelmista viesti vaikka sähköpostiin.
Zabbix tukee sekä hakua, että työntävää tiedonvälitystä. Zabbixia käytetään selainpohjaisella käyttöliittymällä. Käyttöliittymän avulla Zabbixin monitorointia voidaan hyödyntää mistä tahansa lokaatiosta.

Hosts
Hosts eli kohdekoneet, isännät, joihin on asennettu zabbix agentti. Isäntiin lisätään käytettävät templatet ja triggerit.

Templates
Templatet ovat zabbixin ympäristössä valmiiksi olevia pohjia, joihin on lisätty useita eri vastaanotettavia parametrejä. Niillä voidaan valita, mitä tietoa isännästä vastaanotetaan.

Monitoring
Monitoroinnissa voi hyödyntää dashboardeja, ongelma-näkymää, kaivaa uusinta dataa kohteista tai esimerkiksi piirtää loogisen kartan seurattavista kohteista, johon saa visuaalisesti ilmennettyä onko isäntään esimerkiksi yhteys tai ei.

Dashboards
Dasboardiin saa näkyviin kaikki valvontatehtävän luonteen vaatimat tiedot visuaalisesti. Näkymä voi olla graafi tai kenties lista tapahtumalokista.

Alerts & reports
Ilmoitukset ja hälytykset saa näkyviin monitoring->problems välilehdeltä. Riippuen isäntiin kohdistetuista triggereistä, problems näkymässä voi olla ilmoitettuja ongelmia, joita zabbix on havainnut liittyen luotuihin triggereihin.


2. Hostien lisääminen
Valvontaan lisättiin web1 (jossa apache), db1 ja branch-client.
Jokaiseen lisättiin templateksi OS Linux by Zabbix agent active, lisäksi web1:en App Apache by Zabbix agent.

wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu24.04_all.deb

dpkg -i zabbix-release_latest_7.4+ubuntu24.04_all.deb

apt update

apt install zabbix-agent2

nano /etc/zabbix/zabbix_agent2.conf
Passiivinen ja aktiivinen sallittu osoite (172.20.20.0/24 ja 172.20.20.16 (zabbix))
hostname= isäntäkohteen nimi (web1, db1, branch-client)

service zabbix-agent2 start


3. Dashboard
Kuva: Zabbix_dashboard


4. Mittarit
CPU Usage
Memory Usage
Load Average
Uptime
Disk Usage
Network Traffic
Dokumentoi jokaisesta:

mittarin nimi
arvo
mitä mittari kertoo
miksi sitä kannattaa seurata

Mittari               Arvo                      Merkitys



5. Triggerit
Kuva: CPU80_Trigger-event

Free disk space below 20%
Warning
{branch-client:vfs.dev.util[sdd].last()}>80 or {db1:vfs.dev.util[sdd].last()}>80 or {web1:vfs.dev.util[sdd].last()}>80

CPU-usage over 80%
High
{branch-client:system.cpu.util.last()}>80 or {db1:system.cpu.util.last()}>80 or {web1:system.cpu.util.last()}>80


6. Hälytys- ja häiriötestit
Kuva: Apache2_restarting

for i in $(seq 1 $(nproc)); do yes > /dev/null & done

Komento tekoälyltä, palvelin jako taakan 16:lle ytimelle, piti rasittaa kaikkia, jotta päästiin yli 80%.

Lisäksii service apache2 restart web1:lle.

triggerit vastasi nopeasti, molemmista oli alle minuuttiin tieto problems-välilehdellä.


7. Vertailu
Ominaisuus      SNMP	                     Prometheus              	      Zabbix

Tiedonkeruu	   Pull hostilta                 Pull exporterilta                Agentit push/pull
Dashboardit	   Ei ole itsessään	             Käyttää toista sovellusta        Itsessään
Hälytykset	   Toisen sovelluksen kautta     Alert managerilla                Monipuolinen
Käyttöönotto   apt:n avulla                  TAR-paketin tuontia              Repon lisäys
Skaalautuvuus  Heikko		                 Kohtalainen                      Hyvä
Yrityskäyttö   Verkkoympäristössä.	         Pilvipalveluissa ja konteissa    Monipuolinen


8. Yhteenveto
Ensimmäisenä kurssi on parantanut ymmärrystä ja otetta ubuntun komennoista ja syntaksista. Ainahan voisi omaa aktiivisuttaan touhuta linuxin parissa enemmänkin, mutta kurssit antaa selvän suunnan kläytön kehittämiselle.
Kurssin aikana olen käsitellyt SNMP:tä, prometheusta, node-exporteria, grafanaa, netboxia, ansiblea, wiresharkia ja zabbixta.
Suurimman vaikutuksen kurssin sisällöstä teki itseeni ansible sekä IaC. Muokatessani zabbixien agenttien asennusten yhteydessä conffeja, aloin ohessa miettimään miten tämän tekisi ansiblessa. Valitettavasti tällä kertaa aikani ei riittänyt lähteä soveltamaan ja kokeilemaan, mutta mielenkiintoni on selkeästi vangittu.
Opintojakson aikana on kehittynyt siis linuxin peruskäyttö, tullut tutuksi verkonvalvonnan ja -hallinnoinnin työkalujen perussovellukset ja syvennytty miten laajoja ympäristöjä tulisi hallinnoida ja never trust, always verify.
