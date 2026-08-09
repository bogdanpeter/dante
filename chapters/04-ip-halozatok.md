---
author: Peter Bogdan
chapter: 4
chapter_title: IP hálózatok
status: complete
title: DANTE -- A professzionális Audio over IP rendszerek kézikönyve
version: 1
---

# 4. IP hálózatok

> **A fejezet célja:** megérteni, hogyan működik az IP-hálózat az
> Ethernet fölött, és hogyan kapcsolódik ehhez a Dante. A fejezet végére
> képes leszel egy Dante-eszköz IP-konfigurációját értelmezni,
> kiszámolni egy egyszerű alhálózatot, megérteni az ARP és a gateway
> szerepét, valamint követni egy UDP/IP alapú Dante-forgalom útját.

## Hogyan tanuld ezt a fejezetet?

A 3. fejezetben megtanultuk:

``` text
Ethernet
   ↓
MAC
   ↓
Frame
   ↓
Switch
```

Most egy szinttel feljebb lépünk:

``` text
Dante
   ↓
UDP
   ↓
IP
   ↓
Ethernet
   ↓
Switch
```

### 🟢 1. szint -- kötelező alap

Első olvasáskor ezt értsd meg:

-   IP-cím;
-   subnet mask / prefix;
-   network és host rész;
-   ugyanazon subnet;
-   default gateway;
-   ARP;
-   routing;
-   UDP;
-   port;
-   unicast és multicast IP.

### 🟡 2. szint -- Dante szempontból fontos

Ezután:

-   DHCP vs statikus IP;
-   Dante discovery és IP-elérhetőség;
-   multicast;
-   UDP;
-   QoS és DSCP;
-   VLAN-ok és IP-subnetek kapcsolata;
-   Dante Controller hálózati problémák;
-   routing és VLAN-határok.

### 🔴 3. szint -- Deep Dive

A részletes bináris subnet-számítás, ARP cache, routing table, packet
fragmentation, TTL, ICMP, NAT és Wireshark-elemzés már olyan tudás,
amelyet később hibakeresésnél és hálózattervezésnél használunk.

> **Nem az a cél, hogy hálózati vizsgát tegyél. Az a cél, hogy amikor
> egy Dante-hálózatban valami nem működik, értsd, melyik rétegben keresd
> a problémát.**

------------------------------------------------------------------------

# 4.1 Mi az IP?

Az IP, vagy Internet Protocol, a hálózati réteg egyik alapvető
protokollja.

Az IPv4 32 bites címeket használ, és az IP feladata többek között az,
hogy datagramokat továbbítson összekapcsolt hálózatok között. Az RFC 791
az IP-címet, a routingot és a datagramok hálózatok közötti továbbítását
az IP alapvető funkciói között írja le. citeturn1search2

Egyszerűsített modell:

``` text
Alkalmazás
    │
    ▼
Transport
UDP / TCP
    │
    ▼
IP
    │
    ▼
Ethernet
    │
    ▼
Fizikai hálózat
```

A 3. fejezetben a switch és a MAC-cím volt a középpontban.

Most az IP-cím és a router kerül előtérbe.

------------------------------------------------------------------------

# 4.2 MAC és IP -- mi a különbség?

Ezt már láttuk, de most mélyebben kell értenünk.

``` text
Layer 2
MAC
 ↓
„Melyik Ethernet-interfész?”

Layer 3
IP
 ↓
„Melyik IP-hálózati cél?”
```

Például:

``` text
Eszköz
 ├── MAC: 00:11:22:33:44:55
 └── IP:  192.168.10.20
```

A MAC-cím a helyi Ethernet-továbbításban fontos.

Az IP-cím a hálózati rétegben használatos.

A kettő együtt dolgozik.

------------------------------------------------------------------------

# 4.3 IPv4-cím

Egy tipikus IPv4-cím:

``` text
192.168.10.20
```

Ez négy oktettből áll:

``` text
192 . 168 . 10 . 20
```

Minden oktett 8 bit:

``` text
8 + 8 + 8 + 8 = 32 bit
```

Ezért:

``` text
IPv4 = 32 bit
```

Az RFC 791 az IPv4-címet négy oktettből álló, 32 bites címként írja le.
citeturn1search2

------------------------------------------------------------------------

# 4.4 Mi az a subnet?

Egy IP-cím önmagában nem mondja meg teljesen, hogy melyik hálózathoz
tartozik az eszköz.

Ehhez kell a prefix vagy subnet mask.

Például:

``` text
192.168.10.20/24
```

A `/24` azt jelenti, hogy az első 24 bit a prefixhez tartozik.

``` text
11111111.11111111.11111111.00000000
```

decimálisan:

``` text
255.255.255.0
```

A CIDR-prefix jelölésben a perjel utáni szám adja meg a prefix hosszát.
citeturn2search0turn2search1

------------------------------------------------------------------------

# 4.5 Network rész és host rész

Nézzük:

``` text
192.168.10.20/24
```

A `/24` miatt:

``` text
Network:
192.168.10

Host:
20
```

Egyszerűsítve:

``` text
192.168.10 | 20
^^^^^^^^^^   ^^
network      host
```

Ez azonban csak a `/24` esetén ilyen könnyen látható.

------------------------------------------------------------------------

# 4.6 Mit jelent a /24?

Egy `/24` hálózatban 32 - 24 = 8 bit marad a hostok számára.

Ezért:

\[ 2\^8 = 256 \]

cím áll rendelkezésre.

A hagyományos IPv4 subnetben ebből tipikusan:

-   1 network address;
-   1 broadcast address;

nem használható normál hostcímként.

Így:

\[ 256 - 2 = 254 \]

hagyományos hostcím áll rendelkezésre.

Például:

``` text
Network:
192.168.10.0

Első host:
192.168.10.1

Utolsó host:
192.168.10.254

Broadcast:
192.168.10.255
```

> **Megjegyzés:** ez a klasszikus IPv4 /24 példa. Speciális prefixek,
> például /31 vagy /32, eltérő szabályok szerint használhatók.

------------------------------------------------------------------------

# 4.7 CIDR -- ne gondolkodj többé Class A/B/C-ben

Régi hálózati oktatási anyagokban találkozhatsz:

-   Class A;
-   Class B;
-   Class C.

A modern IP-hálózatok tervezésében azonban a CIDR-prefix a fontos.

Például:

``` text
10.0.0.0/8
172.16.0.0/12
192.168.10.0/24
```

A `/8`, `/12`, `/24` konkrétan megadja a prefix hosszát.

A CIDR lényege éppen az, hogy a hálózati prefix hosszát explicit módon
adjuk meg. citeturn2search0turn2search1

------------------------------------------------------------------------

# 4.8 Privát IPv4-címek

Dante-laborokban és helyi hálózatokban gyakran privát IPv4-címeket
használunk.

Az RFC 1918 három privát címtartományt határoz meg:

``` text
10.0.0.0/8

172.16.0.0/12

192.168.0.0/16
```

citeturn1search0turn1search1

Például:

``` text
192.168.10.10
192.168.10.20
192.168.10.30
```

Mind lehet ugyanannak a belső hálózatnak a része.

------------------------------------------------------------------------

# 4.9 Privát cím nem jelent automatikusan internet-hozzáférést

Fontos különbség:

``` text
Private IP
≠
Internet access
```

A privát címeknek nincs globális routingjelentésük.

Egy belső eszköz az internetet tipikusan valamilyen gateway/NAT
infrastruktúrán keresztül éri el.

Dante-rendszernél viszont sok esetben nincs szükség internetelérésre.

Egy Dante-only labor például lehet:

``` text
Stage Box
    │
    ├── 192.168.10.10
    │
    ▼
Switch
    │
    └── 192.168.10.20
         Console
```

és teljesen működőképes lehet internetkapcsolat nélkül.

------------------------------------------------------------------------

# 4.10 Statikus IP és DHCP

Egy IP-cím két gyakori módon kerülhet egy eszközre.

## Statikus

A rendszergazda kézzel állítja be:

``` text
IP:
192.168.10.20

Mask:
255.255.255.0

Gateway:
192.168.10.1
```

## DHCP

A DHCP-szerver adja a konfigurációt.

``` text
Device
  │
  │ DHCP
  ▼
DHCP Server
  │
  └── IP configuration
```

A Dante-eszközök és a Dante Controller által használt hálózat
kialakításától függően mindkét megoldással találkozhatsz.

------------------------------------------------------------------------

# 4.11 Miért fontos a DHCP?

Egy irodai hálózatban gyakori:

``` text
PC
 ↓
DHCP
 ↓
IP
```

Egy professzionális AV-rendszerben viszont gyakran szeretnénk
kiszámítható címzést és dokumentálhatóságot.

Ezért lehet célszerű:

-   statikus IP;
-   DHCP reservation;
-   dokumentált DHCP-scope.

Nem az a szabály, hogy „Dante-nál mindig statikus IP kell".

A helyes megoldás a rendszer üzemeltetési modelljétől függ.

------------------------------------------------------------------------

# 4.12 IP-cím, subnet mask, gateway -- a három alapadat

Egy tipikus eszköz:

``` text
IP:
192.168.10.20

Subnet:
255.255.255.0

Gateway:
192.168.10.1
```

Mit jelent?

### IP

Ez az eszköz hálózati címe.

### Subnet

Megmondja, mely címek tartoznak ugyanahhoz a helyi IP-hálózathoz.

### Gateway

Megadja azt a routert, amely felé a helyi subneten kívüli célokat kell
továbbítani.

------------------------------------------------------------------------

# 4.13 Mi történik, ha két eszköz ugyanazon subneten van?

Például:

``` text
A:
192.168.10.20/24

B:
192.168.10.30/24
```

Mindkettő:

``` text
192.168.10.0/24
```

hálózatban van.

A gép megállapítja, hogy a cél helyi subnetben van.

Ezért nem a default gateway felé küldi az adatot.

Helyette a helyi Layer 2 hálózaton keresztül próbálja elérni.

------------------------------------------------------------------------

# 4.14 Mi történik, ha másik subnetben van?

Például:

``` text
A:
192.168.10.20/24

B:
192.168.20.30/24
```

A két cím nem ugyanahhoz a `/24` subnethez tartozik.

Ezért A nem közvetlenül B Ethernet-címét keresi.

A csomagot a default gateway felé küldi.

``` text
A
│
│ 192.168.10.20
▼
Gateway
│
│ routing
▼
192.168.20.0/24
│
▼
B
```

Itt már routerre vagy Layer 3 switching funkcióra van szükség.

------------------------------------------------------------------------

# 4.15 Mi az a default gateway?

A default gateway a host azon útválasztási következő lépése, amelyet a
gép olyan célokhoz használ, amelyekhez nincs specifikusabb helyi
útvonala.

Egyszerűen:

> **„Ha nem tudom, hogyan jutok el oda közvetlenül, küldöm a gateway
> felé."**

Például:

``` text
PC:
192.168.10.20/24

Gateway:
192.168.10.1
```

Internet:

``` text
PC
 ↓
192.168.10.1
 ↓
Router
 ↓
ISP
 ↓
Internet
```

------------------------------------------------------------------------

# 4.16 Dante-hoz mindig kell gateway?

**Nem.**

Ha minden szükséges Dante-eszköz ugyanabban a helyi IP-hálózatban van, a
működéshez nem feltétlenül szükséges internet vagy külső gateway.

Például:

``` text
192.168.10.10
192.168.10.20
192.168.10.30
```

ugyanazon `/24` subneten.

Ha a Dante-rendszer teljes egészében ezen a hálózaton belül működik, a
gateway nem feltétlenül vesz részt az audió adatfolyam továbbításában.

Ez nagyon fontos:

> **A default gateway megléte nem azonos azzal, hogy a Dante-forgalom a
> gatewayn keresztül megy.**

------------------------------------------------------------------------

# 4.17 ARP -- hogyan lesz az IP-ből MAC?

Ez az egyik legfontosabb kapcsolat a 3. és 4. fejezet között.

Tegyük fel:

``` text
PC:
192.168.10.20

Cél:
192.168.10.30
```

A PC tudja a cél IP-címét.

De az Ethernet frame-hez destination MAC kell.

Mit csinál?

ARP segítségével felderíti a helyi IPv4-címhez tartozó MAC-címet.

Egyszerűsítve:

``` text
„Kié a 192.168.10.30?”

        ↓

„Én vagyok.
MAC:
AA:BB:CC:DD:EE:FF”
```

Ezután:

``` text
IP:
192.168.10.30

↓ ARP

MAC:
AA:BB:CC:DD:EE:FF
```

Az ARP klasszikus specifikációját az RFC 826 írja le.

------------------------------------------------------------------------

# 4.18 ARP cache

A gép nem minden egyes csomag előtt kérdezi újra:

> „Kié ez az IP?"

Az operációs rendszer ARP cache-t használhat.

Például:

``` text
192.168.10.30
      ↓
AA:BB:CC:DD:EE:FF
```

Ez ideiglenes állapot.

Az ARP cache megtekintése ezért hibakeresésnél hasznos lehet.

Windows alatt például:

``` powershell
arp -a
```

A pontos kimenet operációs rendszertől függ.

------------------------------------------------------------------------

# 4.19 ARP és Dante hibakeresés

Ha a Dante Controller lát egy eszközt, de a hálózati kommunikáció mégsem
működik megfelelően, ARP-szintű problémák is felmerülhetnek.

Nem ez az első dolog, amit minden hibánál ellenőrzünk.

De ha:

``` text
IP helyes
Subnet helyes
VLAN helyes
```

és mégsem működik az elérés, érdemes lehet megvizsgálni:

-   ARP cache;
-   switch MAC table;
-   port állapot;
-   VLAN;
-   IP-címütközés.

------------------------------------------------------------------------

# 4.20 IP-címütközés

Két eszköznek nem szabad ugyanazt az IP-címet használnia ugyanabban a
hálózati környezetben.

Hibás példa:

``` text
Stage Box:
192.168.10.20

DSP:
192.168.10.20
```

Ez IP address conflict.

A tünetek lehetnek:

-   intermittáló kapcsolat;
-   elérhetetlen eszköz;
-   furcsa ARP-viselkedés;
-   Dante Controllerben instabil állapot.

Ezért az IP-címzési terv dokumentálása fontos.

------------------------------------------------------------------------

# 4.21 IP-címzési terv

Egy professzionális rendszerben ne véletlenszerűen adjunk címeket.

Példa:

``` text
Network:
192.168.10.0/24

Gateway:
192.168.10.1

Infrastructure:
192.168.10.2 – .9

Dante endpoints:
192.168.10.10 – .99

Management:
192.168.10.100 – .149

Test:
192.168.10.200 – .219
```

Ez csak példa.

A tényleges címzési tervet a rendszer követelményei alapján kell
kialakítani.

------------------------------------------------------------------------

# 4.22 Mi az UDP?

Az UDP a User Datagram Protocol.

Az UDP transport layer protokoll.

Az RFC 768 szerint az UDP datagramok forrás- és célportot, hosszmezőt és
checksumot tartalmazó egyszerű transport protokollt biztosítanak.
citeturn1search48

Egyszerű modell:

``` text
Application
    │
    ▼
UDP
    │
    ▼
IP
    │
    ▼
Ethernet
```

------------------------------------------------------------------------

# 4.23 UDP vs TCP

A két legfontosabb transport protokoll:

``` text
TCP
UDP
```

Nagyon leegyszerűsítve:

### TCP

-   kapcsolat-orientált;
-   sorrendhelyességet biztosít;
-   újraküldéseket használ;
-   megbízható byte streamet biztosít.

### UDP

-   kapcsolat nélküli datagram-szolgáltatás;
-   nincs beépített újraküldési mechanizmus;
-   nincs beépített sorrendgarancia;
-   kisebb protokoll overhead;
-   alkalmas alacsony késleltetésű alkalmazásokhoz, ha maga az
    alkalmazás kezeli a szükséges időzítést/hibakezelést.

A fontos gondolat:

> **UDP nem azt jelenti, hogy „rossz" vagy „megbízhatatlan hálózat".**

Azt jelenti, hogy az alkalmazásnak más módon kell kezelnie a
megbízhatósággal, időzítéssel és veszteséggel kapcsolatos kérdéseket.

------------------------------------------------------------------------

# 4.24 Miért érdekes az UDP Dante esetén?

A Dante valós idejű médiafolyamokat továbbít.

Egy audiórendszerben a késleltetés és időzítés különösen fontos.

Ha minden elveszett audiócsomagot klasszikus TCP-s újraküldéssel
próbálnánk pótolni, az késleltetési és időzítési problémákat okozhatna.

A Dante saját protokolljai és mechanizmusai kezelik a valós idejű audió
és clocking követelményeit.

Ezért:

``` text
Dante
 ↓
real-time network traffic
 ↓
UDP/IP
```

A részletes Dante protokollműködést későbbi fejezetben bontjuk ki.

------------------------------------------------------------------------

# 4.25 UDP portok

Egy IP-cím önmagában nem azonosít egy konkrét szolgáltatást.

Például:

``` text
192.168.10.20
```

Egy eszközön több hálózati szolgáltatás működhet.

A portszám segít megkülönböztetni őket.

``` text
IP:
192.168.10.20

UDP port:
8800
```

A kettő együtt:

``` text
192.168.10.20:8800
```

A portszámok a transport protokollon belül szolgáltatások és végpontok
megkülönböztetésére szolgálnak. Az IANA a TCP/UDP portszámokat központi
nyilvántartásban kezeli. citeturn0search0

------------------------------------------------------------------------

# 4.26 Port ≠ switchport

Ez kezdőként nagyon fontos.

Két külön fogalom:

### UDP port

``` text
UDP port 8800
```

### Ethernet switchport

``` text
Switch Port 24
```

Az egyik:

``` text
Layer 4
```

A másik:

``` text
Layer 2 / fizikai interfész
```

Ezért:

> **A „port 24" és a „UDP 8800" két teljesen különböző dolog.**

------------------------------------------------------------------------

# 4.27 Dante által használt portok

Az Audinate „Dante Information for Network Administrators" dokumentuma
több Dante-forgalomhoz használt portot és multicast címtartományt
dokumentál.

A dokumentumban például szerepel:

``` text
UDP 8700–8708
```

multicast control/monitoring célokra, valamint több unicast UDP-port
Dante control/audio funkciókhoz, például:

``` text
UDP 4440
UDP 4444
UDP 4455
UDP 8751
UDP 8800
```

A pontos portlista és funkció az Audinate dokumentációjában található.
citeturn0search36

> **Ne tanuld meg ezeket vakon.** A későbbi firewall- és hibakeresési
> fejezetben lesz jelentőségük.

------------------------------------------------------------------------

# 4.28 IP routing

Ha egy cél másik subnetben található, routerre van szükség.

Például:

``` text
192.168.10.20/24
        │
        ▼
192.168.10.1
        │
        ▼
Router
        │
        ▼
192.168.20.30/24
```

A router a Layer 3 cím alapján választ útvonalat.

Ez a legfontosabb különbség:

``` text
Switch
→ Layer 2 forwarding

Router
→ Layer 3 forwarding
```

Egy modern Layer 3 switch természetesen mindkettőre képes lehet.

------------------------------------------------------------------------

# 4.29 Routing table

A router nem „gondolkodik emberi módon".

Routing table-t használ.

Egyszerű példa:

  Destination    Prefix   Next hop             Interface
  -------------- -------- -------------------- -----------
  192.168.10.0   /24      directly connected   VLAN 10
  192.168.20.0   /24      directly connected   VLAN 20
  0.0.0.0        /0       192.168.10.254       uplink

A:

``` text
0.0.0.0/0
```

a default route.

A CIDR dokumentációban a `/0` a teljes IPv4-címtartományt lefedő default
route-ként szerepel. citeturn2search0

------------------------------------------------------------------------

# 4.30 Longest Prefix Match

Ha több útvonal illeszkedik, a router a legspecifikusabb illeszkedő
prefixet választja.

Például:

``` text
10.0.0.0/8
10.20.0.0/16
10.20.30.0/24
```

Cél:

``` text
10.20.30.40
```

Mindhárom illeszkedik.

A `/24` a legspecifikusabb.

Ez a routing egyik alapelve.

Kezdőként nem kell minden routingprotokollt ismerned ahhoz, hogy ezt
megértsd.

------------------------------------------------------------------------

# 4.31 VLAN és IP subnet

A 3. fejezetben VLAN-ról beszéltünk.

Most kapcsoljuk össze az IP-vel.

Gyakori kialakítás:

``` text
VLAN 10
192.168.10.0/24

VLAN 20
192.168.20.0/24
```

A VLAN Layer 2 szegmentáció.

Az IP subnet Layer 3 szegmentáció.

A két fogalom gyakran együtt jelenik meg, de **nem ugyanaz**.

------------------------------------------------------------------------

# 4.32 Inter-VLAN routing

Ha:

``` text
Dante VLAN
192.168.10.0/24

Management VLAN
192.168.20.0/24
```

és a két hálózat között kommunikáció kell, Layer 3 routing szükséges.

``` text
VLAN 10
   │
   ▼
L3 Gateway
   │
   ▼
VLAN 20
```

Ezt nevezhetjük inter-VLAN routingnak.

------------------------------------------------------------------------

# 4.33 Miért fontos ez Dante-nál?

Mert a Dante-rendszerben nem minden kommunikáció egyszerűen „átmegy a
routeren".

Bizonyos Dante discovery és multicast mechanizmusok Layer
2/VLAN-határokhoz kötődnek, ezért a VLAN-ok és routing megtervezése
külön figyelmet igényel.

Az Audinate dokumentációja szerint multicast Dante-környezetben IGMP
használható a multicast kezelésére, és VLAN-onként egy IGMP Querier
választása javasolt olyan környezetekben, ahol IGMP szükséges.
citeturn0search36

Ezért:

> **A „ping működik" önmagában nem bizonyítja, hogy egy Dante-rendszer
> minden szükséges hálózati funkciója működik.**

------------------------------------------------------------------------

# 4.34 Ping és ICMP

A `ping` nagyon hasznos hibakereső eszköz.

Például:

``` powershell
ping 192.168.10.20
```

A ping tipikusan ICMP Echo Request / Echo Reply üzeneteket használ.

Fontos:

``` text
Ping működik
≠
Dante működik
```

A ping csak egy bizonyos IP-szintű elérést igazol.

Nem bizonyítja:

-   Dante discovery működését;
-   multicast működését;
-   PTP működését;
-   UDP-portok megfelelő elérését;
-   QoS helyes működését;
-   audio subscription működését.

------------------------------------------------------------------------

# 4.35 IP multicast

A multicast IP-címzés egy csoportot azonosít.

Egyszerű modell:

``` text
TX
 │
 ▼
239.x.x.x
 │
 ├── RX1
 ├── RX2
 └── RX3
```

A multicast nem ugyanaz, mint a broadcast.

### Broadcast

``` text
mindenki
```

### Multicast

``` text
az érdeklődő csoport
```

Dante-ban a multicast különösen fontos lehet több vevőnek továbbított
flow-k esetén.

------------------------------------------------------------------------

# 4.36 Dante multicast és IGMP

Az Audinate dokumentációja szerint Dante implementál IGMP v2 vagy v3
támogatást.

Olyan hálózatokban, ahol sok multicast audió-flow vagy például IP video
is jelen van, az IGMP fontos multicast-management eszköz.

Az Audinate ugyanakkor azt is kiemeli, hogy kevés vagy nulla multicast
audió-flow-t használó Dante-only hálózatban az IGMP nem feltétlenül
követelmény. citeturn0search36

Ezért nem tanácsos:

> „Dante = mindig IGMP Snooping."

A helyes gondolkodás:

> **A multicast mennyisége és a hálózat topológiája alapján kell
> eldönteni, milyen multicast-kezelés szükséges.**

------------------------------------------------------------------------

# 4.37 PTP előkészítés

A Dante rendszerben a pontos időzítés és órajel-szinkronizáció alapvető.

A PTP, Precision Time Protocol, ezt a témát fogja később részletesen
lefedni.

Most csak azt kell megértened:

``` text
IP hálózat
   │
   ├── audio traffic
   ├── control traffic
   └── timing / PTP
```

Ezért a hálózat nem pusztán „bájtokat továbbít".

A Dante-rendszer számára az időzítés is hálózati funkció.

Az Audinate a Dante QoS-dokumentációban a time-critical PTP-forgalom
számára magas prioritást ír le. citeturn0search36

------------------------------------------------------------------------

# 4.38 QoS és IP

A QoS-t a 3. fejezetben már megismertük.

Most egy fontos részletet kapcsolunk hozzá:

A Dante csomagok DSCP értékeket használhatnak, amelyek alapján a switch
QoS-sorba sorolhatja őket.

Az Audinate dokumentációja szerint a dokumentált prioritások között
szerepel:

  Forgalom            DSCP            Decimal
  ------------------- ------------- ---------
  Time-critical PTP   CS7                  56
  Audio / PTP v2      EF                   46
  Reserved            CS1                   8
  Other               Best Effort           0

A konkrét QoS-konfigurációt mindig az adott switch gyártói
dokumentációjával együtt kell értelmezni. citeturn0search36

------------------------------------------------------------------------

# 4.39 Mi az a TTL?

Az IPv4 fejléc egyik mezője a TTL, Time To Live.

Egyszerűsítve:

> A TTL korlátozza, hogy egy IP-csomag hány router-hopon haladhat
> keresztül.

Minden router továbbításkor csökkenti.

Ha eléri a nullát, a csomag nem haladhat tovább.

Ez segít megakadályozni, hogy routing-hiba esetén egy csomag örökké
körbe-körbe járjon.

Dante kezdőként nem kell, hogy TTL-specialista legyél.

De Wireshark használatakor látni fogod.

------------------------------------------------------------------------

# 4.40 IP header

Egy IPv4 packet többek között olyan mezőket tartalmaz, mint:

-   Version;
-   IHL;
-   DSCP/ECN;
-   Total Length;
-   Identification;
-   Flags;
-   Fragment Offset;
-   TTL;
-   Protocol;
-   Header Checksum;
-   Source Address;
-   Destination Address.

A kezdő számára most a legfontosabb:

``` text
Source IP
Destination IP
Protocol
TTL
DSCP
```

------------------------------------------------------------------------

# 4.41 UDP header

Az UDP fejléc egyszerű.

``` text
┌──────────────────────┐
│ Source Port          │
├──────────────────────┤
│ Destination Port     │
├──────────────────────┤
│ Length               │
├──────────────────────┤
│ Checksum             │
└──────────────────────┘
```

Az RFC 768 ezeket a mezőket definiálja. citeturn1search48

Ezután jön az UDP payload.

------------------------------------------------------------------------

# 4.42 A teljes csomagolási lánc

Most rakjuk össze a 2., 3. és 4. fejezetet.

``` text
Hang
 ↓
ADC
 ↓
PCM
 ↓
Dante transport
 ↓
UDP
 ↓
IP
 ↓
Ethernet frame
 ↓
PHY
 ↓
kábel
 ↓
Switch
 ↓
kábel
 ↓
FOH Console
```

Ez a könyv egyik legfontosabb mentális modellje.

------------------------------------------------------------------------

# 4.43 Egy konkrét példa

Tegyük fel:

``` text
Stage Box
IP:
192.168.10.20

FOH Console
IP:
192.168.10.30
```

Mindkettő:

``` text
/24
```

### Stage Box

``` text
192.168.10.20/24
```

### Console

``` text
192.168.10.30/24
```

A Stage Box megállapítja:

``` text
192.168.10.30
```

ugyanabban a subnetben van.

Ezért a helyi Ethernet-szintű eléréshez ARP segítségével felderítheti a
cél MAC-címét.

Ezután:

``` text
Destination IP
192.168.10.30

        ↓

Destination MAC
AA:BB:CC:DD:EE:FF
```

A switch pedig MAC alapján továbbít.

------------------------------------------------------------------------

# 4.44 Másik subnet esetén

Most:

``` text
Stage Box:
192.168.10.20/24

Console:
192.168.20.30/24
```

A Stage Box látja:

``` text
másik subnet
```

Ezért:

``` text
Stage Box
   │
   ▼
Default Gateway
192.168.10.1
   │
   ▼
Router / L3 switch
   │
   ▼
192.168.20.30
```

A router Layer 3 döntést hoz.

Ez a különbség a helyi Layer 2 forwarding és a Layer 3 routing között.

------------------------------------------------------------------------

# 4.45 Miért lehet problémás a routing Dante esetén?

Mert a Dante nem egyetlen egyszerű TCP/UDP alkalmazás.

Többféle forgalom van:

``` text
Audio
Control
Discovery
Clock / PTP
Multicast
```

Ezek hálózati viselkedése nem feltétlenül azonos.

Ezért egy olyan konfiguráció, amelyben:

``` text
ping működik
```

nem feltétlenül jelenti azt, hogy:

``` text
Dante discovery működik
Dante clock működik
Dante audio működik
```

Ez az egyik legfontosabb hibakeresési szemlélet.

------------------------------------------------------------------------

# 4.46 NAT

A NAT, Network Address Translation, IP-címeket és gyakran portokat
fordít egy hálózati határon.

Tipikus otthoni hálózat:

``` text
192.168.1.20
      │
      ▼
NAT router
      │
      ▼
Public IP
      │
      ▼
Internet
```

Dante-rendszerben a NAT általában nem része egy egyszerű helyi
Dante-hálózatnak.

Ha azonban Dante-forgalom több hálózati zónán, tűzfalon vagy távoli
kapcsolaton keresztül megy, a NAT és a routing kérdései előkerülhetnek.

------------------------------------------------------------------------

# 4.47 Firewall

A tűzfal forgalmat engedélyezhet vagy blokkolhat például:

-   source IP;
-   destination IP;
-   protocol;
-   source port;
-   destination port;
-   interface;
-   zone

alapján.

Ezért:

``` text
Ping működik
```

de:

``` text
UDP szolgáltatás blokkolva
```

is előfordulhat.

Dante esetén ezért firewall-szabályokat nem lehet kizárólag
„engedélyezzük az IP-címet" szinten megtervezni.

Az Audinate által dokumentált Dante portok és multicast tartományok
ismerete ilyenkor fontos. citeturn0search36

------------------------------------------------------------------------

# 4.48 Wireshark -- első találkozás

A Wiresharkkal később külön fejezetben dolgozunk.

Most csak azt jegyezd meg:

A Wireshark megmutathatja például:

``` text
Source IP
Destination IP
Protocol
UDP port
DSCP
TTL
Packet timing
```

Ez óriási segítség Dante-hibakeresésnél.

Például ha azt mondjuk:

> „A Dante nem működik."

a Wiresharkkal már feltehetjük a kérdést:

> **Látjuk egyáltalán a várt UDP-forgalmat?**

------------------------------------------------------------------------

# 4.49 Tipikus IP-hibák

## Hibás subnet mask

``` text
Device A:
192.168.10.20/24

Device B:
192.168.10.30/16
```

Ez különböző logikai hálózati értelmezést eredményezhet.

## Hibás gateway

A helyi kommunikáció működik, de más subnet nem.

## IP conflict

Két eszköz ugyanazt az IP-t használja.

## Hibás VLAN

Az eszközök fizikailag ugyanazon switchen vannak, de Layer 2 szinten nem
ugyanabban a hálózatban.

## DHCP-probléma

Az eszköz nem kap megfelelő címet.

## Firewall

Az IP-szintű elérés bizonyos irányokban blokkolva van.

------------------------------------------------------------------------

# 4.50 Hibakeresési sorrend

Ha egy Dante-eszközt nem látsz:

``` text
1. Link
   ↓
2. Link speed
   ↓
3. VLAN
   ↓
4. IP address
   ↓
5. Subnet mask
   ↓
6. IP conflict
   ↓
7. ARP
   ↓
8. Ping / ICMP
   ↓
9. UDP / multicast
   ↓
10. Dante discovery
   ↓
11. PTP / clock
   ↓
12. Subscription
```

Ez nem merev szabály minden hibára.

De kezdőként kiváló gondolkodási sorrend.

------------------------------------------------------------------------

# 4.51 Gyakorlati feladat -- subnet felismerés

Döntsd el, hogy ugyanabban a subnetben vannak-e.

### A

``` text
A: 192.168.10.20/24
B: 192.168.10.30/24
```

**Igen.**

### B

``` text
A: 192.168.10.20/24
B: 192.168.20.30/24
```

**Nem.**

### C

``` text
A: 10.10.5.20/16
B: 10.10.200.30/16
```

**Igen.**

Mindkettő:

``` text
10.10.0.0/16
```

hálózatban van.

------------------------------------------------------------------------

# 4.52 Gyakorlati feladat -- /26

Van:

``` text
192.168.10.0/26
```

A `/26` esetén:

\[ 32-26=6 \]

hostbit marad.

\[ 2\^6=64 \]

cím van.

A klasszikus subnetben:

``` text
Network:
192.168.10.0

Host:
192.168.10.1 – 192.168.10.62

Broadcast:
192.168.10.63
```

A következő `/26`:

``` text
192.168.10.64/26
```

majd:

``` text
192.168.10.128/26
192.168.10.192/26
```

Ez a subnetelés alapja.

------------------------------------------------------------------------

# 4.53 Gyakorlati feladat -- IP-címzési terv Dante-hoz

Tervezd meg:

``` text
Dante network:
192.168.50.0/24
```

Legyen:

``` text
Gateway:
192.168.50.1

Core:
192.168.50.2 – .9

Dante endpoints:
192.168.50.10 – .79

Management:
192.168.50.100 – .129

Test:
192.168.50.200 – .219
```

Készíts táblázatot:

  Eszköz        IP              MAC   Switch      Port
  ------------- --------------- ----- ----------- ------
  Stagebox-01   192.168.50.10   ...   Aruba-SW1   1
  Console-01    192.168.50.20   ...   Aruba-SW1   2
  DSP-01        192.168.50.30   ...   Aruba-SW2   8

A cél nem az, hogy ez legyen az egyetlen helyes címzési terv.

A cél az, hogy **legyen terv**.

------------------------------------------------------------------------

# 4.54 Gyakorlati feladat -- UDP port

Tegyük fel, hogy egy szolgáltatás:

``` text
IP:
192.168.50.20

UDP:
8800
```

Hogyan különbözteted meg a következőktől?

``` text
Switch Port 24
```

Válasz:

``` text
UDP 8800
→ transport-layer port

Switch Port 24
→ fizikai / Layer 2 interfész
```

Ha ezt már automatikusan meg tudod különböztetni, sok későbbi hálózati
dokumentáció könnyebb lesz.

------------------------------------------------------------------------

# 4.55 Gyakorlati feladat -- Dante elérés

Topológia:

``` text
             Aruba Switch
             /          \
            /            \
     Stage Box          Console
192.168.10.20        192.168.10.30
```

Mindkettő:

``` text
/24
```

Ellenőrzési sorrend:

``` powershell
ipconfig
ping 192.168.10.30
arp -a
```

Ezután:

-   ellenőrizd a switch MAC table-t;
-   ellenőrizd a VLAN-t;
-   nézd meg a Dante Controllerben az eszközöket;
-   ellenőrizd a clock státuszt;
-   ellenőrizd a subscriptiont.

------------------------------------------------------------------------

# 4.56 Aruba labor

A 3. fejezetben már dolgoztunk Aruba switch-csel.

Most egészítsük ki IP-szinten.

``` text
             Aruba Switch
            /      |      \
           /       |       \
      Stage Box   PC       Console
        .10       .100       .20
```

Példa:

``` text
Network:
192.168.50.0/24

Stage Box:
192.168.50.10

Console:
192.168.50.20

PC:
192.168.50.100
```

Ellenőrizd:

``` text
PC → ping Stage Box
PC → ping Console
Stage Box → Console
```

Majd:

``` text
arp -a
```

és a switchen:

``` text
MAC table
```

A cél az, hogy ugyanazt a kapcsolatot két nézőpontból lásd:

``` text
IP / ARP
    +
MAC / Switch
```

------------------------------------------------------------------------

# 4.57 Deep Dive -- Mi történik pontosan egy helyi IP-küldésnél?

Tegyük fel:

``` text
Source:
192.168.10.20

Destination:
192.168.10.30

Subnet:
192.168.10.0/24
```

A host:

1.  megállapítja, hogy a cél helyi subnetben van;
2.  ellenőrzi, hogy van-e ARP cache bejegyzés;
3.  ha nincs, ARP segítségével felderíti a cél MAC-címét;
4.  létrehozza az Ethernet frame-et;
5.  a destination MAC a cél eszköz MAC-je lesz;
6.  a switch továbbítja a frame-et;
7.  a cél host feldolgozza az IP/UDP adatot.

Ez az a pont, ahol a 3. és 4. fejezet szorosan összekapcsolódik.

------------------------------------------------------------------------

# 4.58 Deep Dive -- Mi történik másik subnet esetén?

Tegyük fel:

``` text
Source:
192.168.10.20/24

Destination:
192.168.20.30/24

Gateway:
192.168.10.1
```

A source nem a 192.168.20.30 MAC-címét akarja közvetlenül felderíteni.

A Layer 3 döntés alapján a következő hop a gateway:

``` text
Gateway IP:
192.168.10.1
```

Ezért az Ethernet frame destination MAC-je a gateway MAC-címe lesz.

``` text
IP Destination:
192.168.20.30

Ethernet Destination:
Gateway MAC
```

A router ezután új Ethernet frame-et készít a következő hálózati
szegmenshez.

Ez a „rétegenkénti" gondolkodás egyik legfontosabb példája.

------------------------------------------------------------------------

# 4.59 Deep Dive -- Az IP-cím nem „a gép teljes címe"

Egy eszköznek lehet:

``` text
MAC
IP
hostname
Dante device name
UDP port
```

Ezek különböző azonosítók.

Például:

``` text
Dante Device Name:
FOH-MAIN

IP:
192.168.50.20

MAC:
00:11:22:33:44:55
```

A név embernek kényelmes.

Az IP a Layer 3 kommunikációhoz kell.

A MAC a helyi Ethernethez kell.

A port egy transport-layer végpont.

A Dante név pedig az alkalmazási rendszer szintjén lehet azonosító.

------------------------------------------------------------------------

# 4.60 Deep Dive -- Miért nem elég a ping?

Ez az egyik leggyakoribb hibakeresési csapda.

``` text
ping működik
```

Azt jelenti, hogy egy ICMP-alapú IP-elérés működik.

Nem jelenti automatikusan:

``` text
UDP működik
multicast működik
PTP működik
Dante discovery működik
QoS jó
audio subscription jó
```

Ezért a jó hibakereső mindig megkérdezi:

> **Pontosan mit bizonyított ez a teszt?**

------------------------------------------------------------------------

# 4.61 Deep Dive -- Miért fontos a subnet mask?

Két eszköz IP-címe önmagában nem elég.

Például:

``` text
A:
192.168.10.20

B:
192.168.10.30
```

A maszk nélkül nem tudjuk biztosan megmondani, hogy ugyanazon
IP-subnetben vannak-e.

Lehet:

``` text
/24
```

vagy:

``` text
/16
```

vagy más prefix.

Ezért egy hálózati dokumentációban az IP-címet mindig a prefixszel
együtt érdemes megadni:

``` text
192.168.10.20/24
```

nem csak:

``` text
192.168.10.20
```

------------------------------------------------------------------------

# 4.62 Deep Dive -- Miért jó a dokumentált IP-terv?

Egy Dante-rendszerben akár több tucat vagy több száz hálózati végpont
lehet.

Ha nincs címzési terv:

``` text
192.168.1.17
192.168.1.42
192.168.1.83
192.168.1.104
...
```

akkor később nehéz lesz megmondani:

> „Melyik eszköz ez?"

Egy dokumentált terv:

``` text
.10–.79 Dante endpoints
.100–.129 management
.200–.219 test
```

már önmagában sok információt ad.

------------------------------------------------------------------------

# 4.63 Egy Dante-csomag teljes útja -- a 3. és 4. fejezet összekapcsolása

Most álljunk meg, és rakjuk össze **egy konkrét Dante audiófolyam
útját** az eddig tanult összes réteggel.

Tegyük fel, hogy:

``` text
Stage Box
IP: 192.168.50.10/24

FOH Console
IP: 192.168.50.20/24
```

Mindkét eszköz ugyanazon Aruba switchhez csatlakozik:

``` text
             Aruba Switch
             /          \
            /            \
     Stage Box          Console
 .50.10/24             .50.20/24
```

## 1. A Stage Boxból elindul az audió

A 2. fejezetben már láttuk:

``` text
Hang
 ↓
Mikrofon
 ↓
ADC
 ↓
PCM
```

A Stage Box tehát már **digitális audióadatot** kezel.

A Dante-réteg ezt a hálózati továbbításhoz szükséges formában kezeli.

------------------------------------------------------------------------

## 2. A Dante-forgalom UDP/IP fölött jelenik meg

Egyszerűsített modellben:

``` text
Dante
 ↓
UDP
 ↓
IP
```

A csomagnak van:

``` text
Source IP
Destination IP
UDP source port
UDP destination port
```

Például:

``` text
Source IP:
192.168.50.10

Destination IP:
192.168.50.20
```

------------------------------------------------------------------------

## 3. A Stage Box eldönti: helyi vagy távoli cél?

A Stage Box címe:

``` text
192.168.50.10/24
```

A cél:

``` text
192.168.50.20
```

Mindkettő ebbe a hálózatba tartozik:

``` text
192.168.50.0/24
```

Tehát a cél **helyi subnetben van**.

Ez nagyon fontos következménnyel jár:

> A Stage Boxnak az alapvető helyi kommunikációhoz nem a default
> gatewayhez kell küldenie a frame-et.

------------------------------------------------------------------------

## 4. A Stage Boxnak szüksége van a cél MAC-címére

Az IP-cél:

``` text
192.168.50.20
```

de az Ethernet frame-hez destination MAC-cím kell.

Ha a Stage Boxnak nincs megfelelő ARP cache bejegyzése, ARP segítségével
felderítheti:

``` text
Kié a 192.168.50.20?
```

A Console válaszol:

``` text
192.168.50.20
       ↓
AA:BB:CC:DD:EE:FF
```

Most már rendelkezésre áll:

``` text
Destination IP:
192.168.50.20

Destination MAC:
AA:BB:CC:DD:EE:FF
```

------------------------------------------------------------------------

## 5. Elkészül az Ethernet frame

A magasabb rétegek adata Ethernet frame-be kerül.

Egyszerűsítve:

``` text
┌─────────────────────────────┐
│ Destination MAC             │
│ AA:BB:CC:DD:EE:FF           │
├─────────────────────────────┤
│ Source MAC                  │
│ 00:11:22:33:44:55           │
├─────────────────────────────┤
│ Ethernet payload            │
│                             │
│   IP                        │
│    └── UDP                  │
│         └── Dante data      │
└─────────────────────────────┘
```

Most már pontosan összeér a 3. fejezet és a 4. fejezet:

``` text
Dante
  ↓
UDP
  ↓
IP
  ↓
Ethernet frame
  ↓
MAC
```

------------------------------------------------------------------------

## 6. Az Aruba switch megkapja a frame-et

Az Aruba switch nem azt látja, hogy:

> „Ez egy Dante Stage Boxból érkező audió."

A switch Layer 2 szinten elsősorban ezt látja:

``` text
Source MAC
Destination MAC
VLAN
Ethernet frame
```

A MAC table alapján megkeresi:

``` text
AA:BB:CC:DD:EE:FF → Port 12
```

és a frame-et a megfelelő portra továbbítja.

------------------------------------------------------------------------

## 7. A Console megkapja a frame-et

A Console hálózati interfésze fogadja az Ethernet frame-et.

A feldolgozás visszafelé halad a protokollrétegeken:

``` text
Ethernet
   ↓
IP
   ↓
UDP
   ↓
Dante
   ↓
Audió
```

A Dante-rendszer ezután a megfelelő audióadatot feldolgozza.

------------------------------------------------------------------------

## 8. Az egész folyamat egyetlen ábrán

``` text
┌──────────────────────┐
│      Stage Box       │
│  192.168.50.10/24    │
└──────────┬───────────┘
           │
           │ Dante
           ▼
┌──────────────────────┐
│         UDP          │
└──────────┬───────────┘
           │
           │ IP
           ▼
┌──────────────────────┐
│         IP           │
│ Src:  .50.10         │
│ Dst:  .50.20         │
└──────────┬───────────┘
           │
           │ Ethernet
           ▼
┌──────────────────────┐
│    Aruba Switch      │
│                      │
│ MAC table            │
│ VLAN                 │
└──────────┬───────────┘
           │
           │ Ethernet
           ▼
┌──────────────────────┐
│     FOH Console      │
│  192.168.50.20/24    │
└──────────────────────┘
```

### A legfontosabb gondolat

> **Az IP-cím megmondja, melyik hálózati célhoz akarunk eljutni. Az ARP
> a helyi IPv4-célhoz szükséges MAC-cím felderítésében segít. Az
> Ethernet frame-et a switch a MAC-cím alapján továbbítja.**

------------------------------------------------------------------------

## 9. Mi változik, ha a Console másik subnetben van?

Most legyen:

``` text
Stage Box:
192.168.50.10/24

Console:
192.168.60.20/24
```

A cél már nem helyi.

A folyamat:

``` text
Stage Box
192.168.50.10
      │
      ▼
Default Gateway
192.168.50.1
      │
      ▼
Router / Layer 3 Switch
      │
      ▼
192.168.60.0/24
      │
      ▼
Console
192.168.60.20
```

Ebben az esetben az első Ethernet frame destination MAC-je **a gateway
MAC-címe**, nem közvetlenül a Console MAC-címe.

Ez az a pont, ahol a Layer 2 forwarding és a Layer 3 routing különválik.

------------------------------------------------------------------------

## 10. Mit bizonyít a ping?

Ha a PC-ről:

``` powershell
ping 192.168.50.20
```

sikeres, akkor bizonyos IP/ICMP kommunikáció működik.

De ez önmagában nem bizonyítja:

``` text
Dante discovery
Dante audio
Multicast
PTP
QoS
Subscription
```

Ezért Dante-hibakeresésnél mindig azt kérdezzük:

> **Melyik rétegig bizonyítottuk, hogy működik a rendszer?**

``` text
Link
 ↓
Ethernet
 ↓
IP
 ↓
UDP
 ↓
Multicast / Control
 ↓
PTP
 ↓
Dante
 ↓
Audio
```

Ez a szemlélet később a teljes hibakeresési módszertan egyik alapja
lesz.

------------------------------------------------------------------------

# 4.64 Összefoglalás

Ebben a fejezetben az Ethernet fölötti IP- és UDP-világot építettük fel.

A legfontosabb fogalmak:

### IPv4

32 bites hálózati cím.

### Prefix / subnet mask

Megmutatja, mely bitek tartoznak a hálózati prefixhez.

### Subnet

Egy logikai IP-hálózati tartomány.

### Default gateway

A helyi hálózaton kívüli célok felé használt következő hop.

### ARP

IPv4 helyi hálózati címhez tartozó MAC-cím felderítésének klasszikus
mechanizmusa.

### UDP

Egyszerű, datagram-alapú transport protokoll.

### Port

Transport-layer végpont azonosítására használt szám.

### Routing

IP-csomagok hálózatok közötti továbbítása.

### VLAN

Layer 2 szegmentáció.

### Multicast

Egy adó → egy érdeklődő csoport.

### QoS

Forgalompriorizálás.

------------------------------------------------------------------------

# 4.65 A teljes könyv eddigi mentális modellje

Most már három fejezetet tudunk egymásra rakni:

``` text
                DANTE
                  │
                  ▼
        Digitális audió / PCM
                  │
                  ▼
          Dante transport
                  │
                  ▼
                UDP
                  │
                  ▼
                 IP
                  │
                  ▼
              Ethernet
                  │
                  ▼
               Switch
                  │
                  ▼
                PHY
                  │
                  ▼
                Kábel
```

A 2. fejezetből tudod:

``` text
PCM
```

A 3. fejezetből:

``` text
Ethernet / MAC / Switch
```

A 4. fejezetből:

``` text
IP / UDP / routing
```

A következő nagy lépés:

``` text
Dante hálózati működése
```

------------------------------------------------------------------------

# 4.66 Ellenőrző kérdések

## Alap

1.  Mi az IPv4?
2.  Hány bites egy IPv4-cím?
3.  Mit jelent a `/24`?
4.  Mi a subnet mask?
5.  Mi a network address?
6.  Mi a broadcast address?
7.  Mi a host address?
8.  Mi a default gateway?
9.  Mi az ARP?
10. Mi az ARP cache?
11. Mi az UDP?
12. Mi az UDP port?
13. Mi a különbség az UDP port és a switchport között?
14. Mi a routing?
15. Mi az IP multicast?

## Dante

16. Miért fontos az IP-cím a Dante-rendszerben?
17. Miért nem elég a MAC-cím?
18. Miért fontos az UDP?
19. Miért nem bizonyítja a ping a Dante működését?
20. Mikor kell különösen figyelni a multicast kezelésére?
21. Miért fontos a QoS?
22. Miért lehet szükség routingra?
23. Miért fontos a subnet és VLAN összhangja?
24. Miért fontos az IP-címzési terv?
25. Miért hasznos az ARP és a MAC table együttes vizsgálata?

------------------------------------------------------------------------

# 4.67 Vizsgafeladat -- gondolkodj végig egy Dante-kapcsolatot

Adott:

``` text
Stage Box:
192.168.50.10/24

Console:
192.168.50.20/24

Switch:
Aruba
```

Kérdések:

1.  Ugyanazon subnetben vannak?
2.  Kell-e router az alapvető helyi kommunikációhoz?
3.  Melyik Layer 2 címre van szükség a helyi Ethernet frame-hez?
4.  Hogyan derülhet ki a cél MAC-címe?
5.  Milyen eszköz továbbítja a frame-et?
6.  Mi történik, ha a destination MAC ismert?
7.  Mi történik, ha ismeretlen?
8.  Hol helyezkedik el az UDP a protokollstackben?
9.  Miért nem bizonyítja a ping, hogy van Dante audio?
10. Milyen további vizsgálatokat végeznél?

### Elvárt gondolkodás

``` text
IP subnet
 ↓
helyi / távoli cél
 ↓
ARP / gateway
 ↓
Ethernet destination MAC
 ↓
switch forwarding
 ↓
UDP/IP feldolgozás
 ↓
Dante
```

------------------------------------------------------------------------

# 4.68 Laborprojekt -- IP hibakeresés

Építs:

``` text
PC
 │
 ▼
Aruba Switch
 ├── Stage Box
 └── Console
```

Használj például:

``` text
Network:
192.168.50.0/24

Stage Box:
192.168.50.10

Console:
192.168.50.20

PC:
192.168.50.100
```

## Teszt 1

``` powershell
ping 192.168.50.10
```

## Teszt 2

``` powershell
arp -a
```

## Teszt 3

Ellenőrizd a switch MAC table-t.

## Teszt 4

Ellenőrizd a VLAN-t.

## Teszt 5

Ellenőrizd a Dante Controllerben:

-   eszköz látható?
-   clock látható?
-   subscription működik?

## Teszt 6

Ha van Wireshark:

-   látod-e az IP-forgalmat?
-   milyen UDP-portok jelennek meg?
-   van-e multicast?
-   milyen DSCP értékek jelennek meg?

------------------------------------------------------------------------

# 4.69 Laborprojekt -- szándékos hibák

A laborban hozz létre egy-egy hibát.

### Hiba A

Rossz subnet:

``` text
Stage Box:
192.168.50.10/24

PC:
192.168.60.100/24
```

### Hiba B

Rossz gateway.

### Hiba C

IP conflict.

### Hiba D

Rossz VLAN.

### Hiba E

Multicast kezelés problémája.

Ezután minden esetben dokumentáld:

``` text
Tünet
↓
Mérés
↓
Bizonyíték
↓
Hiba
↓
Javítás
↓
Újrateszt
```

Ez már valódi hálózati hibakeresési gondolkodás.

------------------------------------------------------------------------

# 4.70 Műszaki ellenőrzési megjegyzés

A fejezet kritikus hálózati állításait elsődleges vagy szabványos
forrásokkal ellenőriztük.

Különösen:

-   IPv4 és IP működés -- RFC 791;
-   UDP -- RFC 768;
-   privát IPv4-címtartományok -- RFC 1918;
-   CIDR és prefix notation -- RFC 4632;
-   Dante portok, multicast, QoS, IGMP és EEE -- Audinate „Dante
    Information for Network Administrators" dokumentáció.

Az Audinate dokumentációja szerint a Dante QoS használata vegyes
hálózatokban fontos, a dokumentált DSCP prioritások között pedig CS7 és
EF is szerepel; az IGMP multicast-management célokra használható, az EEE
pedig Dante-forgalmat kezelő portokon kikapcsolandó.
citeturn0search36

A Dante által használt konkrét portokat nem tekintjük örök, minden
jövőbeli verzióra érvényes „varázsszámoknak"; a könyvben mindig az
aktuális Audinate dokumentációval kell őket összevetni.

------------------------------------------------------------------------

# 4.71 Források

1.  **RFC 791 -- Internet Protocol**\
    https://www.rfc-editor.org/rfc/rfc791.html

2.  **RFC 768 -- User Datagram Protocol**\
    https://www.rfc-editor.org/rfc/rfc768.html

3.  **RFC 1918 -- Address Allocation for Private Internets**\
    https://www.rfc-editor.org/rfc/rfc1918.html

4.  **RFC 4632 -- Classless Inter-domain Routing (CIDR)**\
    https://www.rfc-editor.org/rfc/rfc4632.html

5.  **IANA -- Service Name and Transport Protocol Port Number
    Registry**\
    https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml

6.  **Audinate -- Dante Information for Network Administrators**\
    https://assets.audinate.com/wp-content/uploads/2022/03/dante-information-for-network-admins.pdf

------------------------------------------------------------------------

# 4.72 Fejezeti állapot

**Állapot: COMPLETE**

A fejezet tartalmaz:

-   IPv4 alapokat;
-   CIDR és prefix notation;
-   subnet mask;
-   network / host / broadcast;
-   privát IP-címeket;
-   statikus IP-t;
-   DHCP-t;
-   default gatewayt;
-   ARP-t;
-   ARP cache-t;
-   IP conflictot;
-   IP-címzési tervet;
-   UDP-t;
-   UDP portokat;
-   Dante által használt hálózati portokat;
-   routingot;
-   routing table-t;
-   longest prefix match alapot;
-   VLAN és IP subnet kapcsolatát;
-   inter-VLAN routingot;
-   ICMP/pinget;
-   IP multicastot;
-   IGMP-t;
-   PTP előkészítést;
-   DSCP/QoS kapcsolatot;
-   TTL-t;
-   IPv4 és UDP headert;
-   NAT-ot;
-   firewallt;
-   Wireshark alapokat;
-   Dante IP hibakeresési sorrendet;
-   Aruba laborfeladatokat;
-   subnet számítási feladatokat;
-   Deep Dive részeket;
-   ellenőrző kérdéseket;
-   vizsgafeladatot;
-   műszaki ellenőrzési megjegyzést és forrásokat.
