Johdanto
Mikä on Infrastructure as Code?
IaC on automatisoitu lähestymistapa IT-ympäristöjen hallintaan. IaC:lla ympäristö määritetään konfiguraatiotiedostoissa, joiden avulla järjestelmien hallinta on nopeaa, turvallista, hallittua ja mahdollistaa laajojen kokonaisuuksien samanaikaisen hallinnan sen sijaan, että yksittäisiä kohteita hallittaisiin yksitellen yksi kerrallaan, joka tuo omat riskinsä puhumattakaan hitaudesta.

Inventory
Ympäristön rakenne.
Ymnpäristö sisältää kaikki opintojakson virtuaalisen ympäristön isännät, ansible mukaan lukien.
Isännät on jaettu ryhmiin käyttötarkoituksiensa mukaan
-routers
    r1
    r2
    r3
-clients
    client1
    attacker
    branch-client
-servers
    web1
    db1
-monitoring
    prometheus
    grafana
    zabbix
    cadvisor
-management
    ansible

loogisiin ryhmiin
-user_network
    client1
    attacker
-server_network
    web1
    db1
-branch_office
    branch-client

järjestelmällisesti
-network_devices
    routers
-linux_hosts
    clients
    servers
    monitoring
    management
-ubuntu_hosts
    client1
    web1
    db1
    branch-client

sovelluskohteellisesti
-node_exporter
    routers
    clients
    servers

Yksittäiset isännät kuuluvat omiin ryhmiinsä, mutta ryhmät voivat vielä lisäksi kuulua ryhmänä toiseen ryhmään.
Kohteiden tarkoitukset ympäristössä ovat keskenään hiukan lomittain. Tarkoittaen siis sitä, että selkeää jakoa ryhmiin ei voi tehdä, vaan tietyt kohteet voivat kuulua yhteen samaiseen ryhmään, mutta sen lisäksi keskenään eri ryhmiin. Ikään kuin luodaan kaikki kombinaatiot sille, mitä kaikkia kohteita voitaisiin joutua konfiguroimaan samaan aikaan ja samalla lailla; mitä yhteneväisyyksiä eri kohteilla on.



Esimerkkiplaybookien analyysi
Ping playbookissa käytetään isäntinä inventoryn linux_hosts, ei kerätä järjestelmätietoja, käytetään ansiblen omaa ping-moduulia. Palveluissa ei ole SSH:ta käytössä, joten kaikki muut paitsi palvelut vastaavat OK. Reitittimet eivät voi suorittaa python-scriptejä, joten ne yritetään ajaa ansiblessa hyödyntäen paramikon luomaa SSH yhteyttä reitittimiin. FRRouting-konteissa ei kuitenkaan ole SSH-palvelinta käytössä, joten yhteyttä ei saada ja täten playbook ei kykene vetämään reitittimien versio-tietoja playbookia ajettaessa.

SNMP -playbookissa asennetaan ja otetaan käyttöön SNMP ja -d inventoryn ubuntu_hosts -kohteisiin korotetuin käyttöoikeuksin. Määritelty yksi muuttuja. Muuttujilla vähennetään työn määrää ja helpotetaan kokonaisuuden hallittavuutta. Asennus on jaettu 4 vaiheeseen, viides ja kuudes kohta on varmistusviesti SNMP agentin tilasta. Lopussa on handlerina palvelun uudelleen käynnistämisestä. Handler ajetaan scriptin lopuksi, mikäli on tehty muutos johonkin kohteeseen jossa on notify, jossa kyseistä handleriä kutsutaan.
Ensin päivitetään päivitysvälimuisti, asennetaan SNMP sekä -d (state: present, eli ei ilmoiteta muutoksesta jos palvelut on jo asennettu), muokataan snmpd.conf-tiedostosta 4 kohtaa (tässä viitataan myös aikaisemmin mainittuun muuttujaan sekä lopuksi handleriin), enabloidaan snmpd eli palvelu käynnistyy järjestelmän käynnistyessä ja lopuksi vielä rekisteröidään snmpd:n prosessin tila joka palautetaan debug-viestissä.

Node-exporter -playbookissa asennetaan ja otetaan käyttöön node-exporter inventoryn ubuntu_hosts -kohteisiin korotetuin käyttöoikeuksin. Määritelty yksi muuttuja, asennuksessa käytettävän node-exporterin versio, jolloin päivittäminen onnistuu helposti. Asennus jaettu kuuteen vaiheeseen, skriptin 7. ja 8. kohta varmistaa playbookin käyttäjälle node-exportereiden tilan. Ensin päivitetään apt update, asennetaan litännäiset, joilla kyetään hakemaan tar-paketti ja sen jälkeen purkamaan se ja mikäli uusimmat liitännäiset on jo asennettu, ei muutosta. Luodaan node-exporterille asennuskansio ja/tai päivitetään käyttöoikeudet mikäli kumpaakaan on tarvetta tehdä. Haetaan tar-paketti tietystä URL:sta tiettyyn kohteeseen (tässä viitataan aikaisemmin määritettyyn node-exporterin versioon), puretaan paketti käyttäen kohteella olevaa tiedostoa (remote_src: yes), asennetaan exporteri ja annetaan sille tietyt käyttöoikeudet ja kerrotaan taas ansiblelle, että tiedosto on jo kohteessa. Viimeiseksi lopetetaan aikaisemmin käynnissä ollut node-exporter prosessi ( ||true mikäli prosessi ei ollut jo käynnissä, ja virhe-ilmoitus voidaan ohittaa), varmistetaan sovelluksen ajo taustalla, ohjataan virheviestit samaan lokiin ohjelman muiden viestien kanssa ja kerrotaan käytettävän bash-komentotulkkia.

Kaikissa playbookeissa tulee viimeisenä vielä play recap, joka kertaa mitkä kohteet ovat onnistuneet, tuliko virheitä ja mitä, jos lainkaan, muutettiin.



Oma playbook: palvelimen asennus

root@ansible:/ansible/playbooks# ansible-playbook -i ../inventory.ini install-postgresql.yml

PLAY [Install postgresql and add an user] *************************************************************************************

TASK [Update package cache] ***************************************************************************************************
ok: [db1]

TASK [Install postgres and psycopg2 library] **********************************************************************************
ok: [db1]

TASK [Ensure postgresql service is running] ***********************************************************************************
changed: [db1]

TASK [Configure PostgreSQL access] ********************************************************************************************
ok: [db1]

TASK [Restart PostgreSQL service] *********************************************************************************************
changed: [db1]

TASK [Create a new user] ******************************************************************************************************
changed: [db1]

TASK [Show verification result] ***********************************************************************************************
ok: [db1] => {
    "msg": "Postgresql running on db1"
}

PLAY RECAP ********************************************************************************************************************
db1                        : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   



Automaatio VS. manuaalinen
Ansiblen hyödyllisyys tuli hyvin ilmi jo node-exporterin ja SNMP:n asennuksissa. Olisi ollut hidasta ja virhealtista suorittaa jokainen kohde yksitellen ja siitä olisi pitänyt muodostaa erillinen muistiinpano, että mitä on minnekin asennettu.
Playbook suorittaa dokumentoinnin samalla eikä virheitä pääse syntymään.
Kun tekee playbookit itselleen tarkasti, niitä voi aina myöhemmin hyödyntää uudelleen tai tarvittaessa muoakta mikäli ilmenee ominaisuus jota ei aikaisemmin otettu huomioon tai johon täytyy tarttua.



Yhteenveto
Oman näkemykseni mukaan ansiblen, ja ylipäätään IaC, on varmasti verkko- ja palvelinympäristön ylläpitäjän tietotaitojen ytimessä. Playbookkien avulla luodaan jo itsessään dokumentaatiota ympäristön tilasta, mitä on asennettu, minne ja miten. Tietenkin ajan myötä on varmaan syytä ylläpitää myös versionhallintaa ja -historiaa, mikäli playbookkeja muokataan ja ympäristö kehittyy. Playbookit mahdollistaa suurien kokonaisuuksien hallinnan tarvittaessa massamaisesti nopeuttaen muutoksen tekoa. Muutoksia voidaan joutua tekemään useammastakin syystä ja sen takia toiminnan nopeus voi olla jopa kriittistä (tietoturva, palvelut, käyttäjät...).


