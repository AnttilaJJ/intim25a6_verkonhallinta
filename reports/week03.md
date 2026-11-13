1. Johdanto
Prometheus-monitoroinnin toimintaperiaate.
Linux->Node-exporter->http:9100->Pull->Prometheus->PromQL->Grafana->Käyttäjä
Node-exporterit keräävät datan järjestelmästä ja valmistelee sen Prometheuksen käyttämään muotoon. Prometheus kerää datan kohteen exporterilta tietokantaansa ja tallentaa sen aikajanalle. Dataa voidaan hyödyntää esimerkiksi Grafanalla, jolla tieto saadaan esitettävään ja helposti seurattavaan muotoon PromQL:n, eli kyselyidenn, avulla.


2. Node Exporterin käyttöönotto
Asennusvaiheet ja testit.
Node-exporter asennetaan kohde(target)-järjestelmään.
Haetaan wget:llä uusin versio, tässä tapauksessa se tehtiin githubista. Paketti puretaan (tar xvf) ja sovellus käynnistetään ./node_exporter
Minun tapauksessani taustalla oli jo pyörimässä vanha versio, joten tapoin prosessin ja siirsin uuden exporterin oikeaan kansioon ja käynnistin uudelleen.
Node exporterin toiminta testataan uudella ikkunalla "curl http://localhost:9100/metrics" tuloksena tulee "suuri määrä mittareita".


3. Prometheus
Targets-sivun tarkastelu ja havainnot.
Prometheuksen status->target health sivulta tarkastettiin kohteiden kunto. Halutut kohteet tulee olla UP-tilassa ja vihreänä.
Lisäksi querystä voi hakea "up", jolloin saa vastaukseksi (toivottavasti) sopivan määrä isäntiä ja instansseja joiden perässä on numero 1. Pelkkä UP-tila ei riitä, vaan numero 0 tai 1 kertoo kohteen yhteydestä.


4. Dashboard
Kuvakaappaus koko dashboardista.
Dashboardissa näkyi useampi isäntä useamman kerran, kuten levyn käytössä ja molemmissa network-näkymissä. Oletettavasti tämä johtuu siitä, että kohteella on useampi rajapinta käytössä tai ominaisuus on jaettu useampaan osaan.

5. Kuormitustesti
Mittareiden käyttäytyminen kuormituksen aikana.
Mittareista huomaa, että useampi kohde käyttää todennäköisesti samaa isäntäpalvelinta, kun useamman kohteen käyrät menevät käsi kädessä.

Kuormituksessa käytettiin kahta komentoa ajalla 14:27:
dd if=/dev/zero of=testfile.img bs=1M count=500
stress-ng --cpu 4 --timeout 60s

CPU:n käyttöaste nousi noin 5%, eli ei merkittävästi. Käyttöaste pysyi 6,65%:ssa 60 sekunttia ja laski sen jälkeen takaisin.
Levyn käyttö nousi 6,11%:sta 6,14%:in. Datan tallennuksen luonteesta johtuen, käyrä tietenkin pysyi samalla tasolla myös jälkeenpäin.
Verkkoliikenteessä vastaanotto nousi 170 kiloon sekunnissa kahdeksi minuutiksi ja lähetys 0,58:en bittiin sekunnissa samaksi ajaksi. En kyennyt kuitenkaan selvittämään mistä tämä johtui, sillä tiedonsiirto tapahtui testien jälkeen ja lähetyksessä on tasainen viiva 900 bitin korkeudessa, jonka oletan olevan yhteys prometheukseen.


6. SNMP vs Prometheus
Vertailutaulukko ja johtopäätökset.
Tiedonkeruu
Tiedonkeruu on SNMP:ssä aktiivinen, kun taas Prometheuksessase on passiivnen. SNMP saa datan "vetämällä" niin kuin Prometheuskin, mutta se vaatii aktiivisia toimia välittyäkseen. Prometheuksessa toiminta on hyvinkin automatisoitua.
Käyttöönotto		
Olettaen, että prometheus-palvelin on jo määritelty, niin uuden kohteen lisääminen on yhtä vaivatonta kuin SNMP:ssäkin. Tosin SNMP:ssä joutuu kohdekoneella käydä muuttamassa SNMPD.conf tietoja, jotta sen saa toimimaan, ja toivottavasti tietoturvallisesti.
Mittarien määrä		
Prometheukses käyttää neljää metriikkaa: Counter, gauge, histogram ja summary. (Laskuri, mittari, histogrammi ja yhteenveto)
SNMP:ltä löytyy vastine vain laskurille ja mittareille.

Visualisointi
SNMP:ssä ei ole visualisointikeinoja. Prometheuksessa on jo itsessään oma auttava graafinen näkymä.
Molemmissa voi/tulee käyttää kuitenkin toista alustaa visualisointiin. Näissä erona on kuitenkin yhteensopivuus ja yhteentoimivuus. Esimerkiksi Grafana toimii mutkatta Prometheusen kanssa, kun taas SNMP:lle ei ole vastinetta joka olisi yhtä hyvin yhteensopiva jonkin toisen sovelluksen kanssa.

Hälytysmahdollisuudet
SNMP:ssä laite (eli agentti) itse hälyttää poikkeamista toiselle osapuolelle, kun taas prometheus kykenee oman jatkuvan datan keruun ja säilömisen myötä reagoimaan poikkeamiin. Saadakseen tästä kuitenkin viesti eteenpäin, vaatii prometheus alertmanagerin käytön joka voi lähettää esimerkiksi sähköpostin.

Soveltuvuus pilviympäristöihin
Prometheus soveltuu paremmin monimutkaisiin laajoihin dynaamisiin ympäristöihin, kun taas SNMP:tä käyttäisin enemmän verkonvalvontaan Zabbixin kanssa staattisessa ympäristössä.

7. Yhteenveto
Mitä opit harjoituksesta?
Lisäämään kohteen Prometheus valvontatyökalun seurantaan node_exporterin avulla.
Ymmärtämään mitä eroja SNMP:ssä ja Prometheuksessa on.
Miten Prometheuksen prosessi toimii kohteelta aina Grafanalle asti dashboardiin.
Prometheuksin queryjen tekeminen ja neljä metriikkaa.
Dashboardien luonti Grafanaan ja paneelien luonti PromQL:llä.

Mitä hyötyä Prometheuksesta on verrattuna SNMP:hen?
Laajempi metriikka, yksinkertaisempi kohdekirjasto (vrt. SNMP OIB ja MIB), paremmat mahdollisuudet visualisointiin, yksinkertaisempi toteutus datan keruuseen, säilömiseen ja hälyttämiseen, sopivampi dynaamisiin ja monimutkaisempiin verkkoympäristöihin.

Millaisia mittareita ylläpitäjän kannattaa seurata jatkuvasti?
Tiedon välitys ja resurssien käyttöä. Suorituskyvyllisiä metriikoita. Paikkoja joissa voi syntyä palveluiden pullonkauloja.

Mitä tietoa dashboardisi tarjoaa ylläpitäjälle?
Rakentamani dashboardi tarjoaa ylläpitäjälle välittömän ja reaaliaikaisen tiedon järjestelmän resursseista ja mahdollisista palvelunalenemisista, ja ennakkovaroituksen asiakkaiden yhteydenotoista ja murheista. Tavoitteena kuitenkin etupainoisesti korjata ongelmatilanteet ennen kuin niistä on haittaa.

Mitä uusia mittareita lisäisit dashboardiin?
Mahdollisesti tiedon verkkoliikenteeseen, mistä näkee joko portit, palvelut tai sovellukset jotka syövät kaistaa. Mahdollistaa kenties paremman optimoinnin.
Lisäksi RAM-muistin määrä ja käyttö voisi olla hyödyllinen tieto jo senkin takia, että voi paremmin havaita jos nytkyinen muistimäärä on normaalitoimminnoissa riittämätön.
 

Miten monitorointitiedosta voisi olla hyötyä vianetsinnässä?
Mikäli ongelmia ilmenee, voi tiedoista nähdä missä ongelma mahdollisesti piilee. Lisäksi valvonnassa on aikaleimoja, jolloin on mahdolista arvioida milloin ongelma on alkanut ja tällöin voi selvittää mitä juuri silloin on tapahtunut, että ongelma on ilmennyt.

