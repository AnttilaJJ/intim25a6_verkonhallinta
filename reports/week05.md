Osa 1: 

[Destination GeoIP: US, ASN 16509, Amazon.com, Inc.]
    [Destination GeoIP Country: United States]
    [Source or Destination GeoIP Country: United States]
    [Destination GeoIP ISO Two Letter Country Code: US]
    [Source or Destination GeoIP ISO Two Letter Country Code: US]
    [Destination GeoIP AS Number: 16509]
    [Source or Destination GeoIP AS Number: 16509]
    [Destination GeoIP AS Organization: Amazon.com, Inc.]
    [Source or Destination GeoIP AS Organization: Amazon.com, Inc.]
    [Destination GeoIP Latitude: 37.751]
    [Source or Destination GeoIP Latitude: 37.751]
    [Destination GeoIP Longitude: -97.822]
    [Source or Destination GeoIP Longitude: -97.822]


capture filtterillä "port 443 or port 53"
Lisäksi etsin määränpäät, joissa yle mainitaan ja lisäsin display filtteriin näyttämään kaikki samat kohteet. Tässä tapauksessa "ipv6.dst == 2600:9000:278f:8000:17:7a62:3540:93a1"




Osa 2: 

1) 

dns.qry.name constains "hamk"

learn.hamk.fi = 195.148.239.84
shibbo.hamk.fi = 195.148.239.10
SOA: ns1.hamk.fi
Käytetty DNS-palvelin: Elisa Oyj, 2001:998:20::20, Hanko
DNS-kyselyiden TTL (hop limit IPv6): 64


2) 

tls and frame contains "hamk"

Protokolla: TLSv1.3


ssl.handshake.ciphersuite

Client:
Transport Layer Security
    [Stream index: 3]
    TLSv1.3 Record Layer: Handshake Protocol: Client Hello
        Content Type: Handshake (22)
        Version: TLS 1.0 (0x0301)
        Length: 1916
        Handshake Protocol: Client Hello

Server:
Handshake Protocol: Server Hello
...
    Version: TLS 1.2 (0x0303)
        [Expert Info (Chat/Deprecated): This legacy_version field MUST be ignored. The supported_versions extension is present and MUST be used instead.]
    ...
    Extension: supported_versions (len=2) TLS 1.3
        Type: supported_versions (43)
        Length: 2
        Supported Version: TLS 1.3 (0x0304)



Osa 3: 

Pyri filteröimään kaapatusta liikenteestä http liikennettä ja etsi liikenteestä annetut käyttäjätunnukset. Palauta käyttämäsi filtterit ja kuvaile etsintä termit. Palauta myös kaappaustiedosto.

Hypertext Transfer Protocol
    GET / HTTP/1.1\r\n
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.1
    Host: localhost\r\n
    Connection: keep-alive\r\n
    Cache-Control: max-age=0\r\n
    Authorization: Basic bXl1c2VybmFtZTpteXBhc3N3b3Jk\r\n
        Credentials: myusername:mypassword
   

Capture filteriä ei käytetty. Kun ei heti alkuun vielä tiedä mitä lähtee etsimään, niin kannattaa taltiointivaiheessa jättää suodatus vähemmälle, varsinkin kun tässä demossa sisältöä tuli varsin vähän.
DisplayFilterinä käytettiin ihan pelkästään http:tä.Tämän jälkeen kävin ensimmäisestä lähtien HTTP-kohdat läpi kustakin liikenteestä.