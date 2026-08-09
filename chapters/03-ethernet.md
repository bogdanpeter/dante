---
author: Peter Bogdan
chapter: 3
chapter_title: Ethernet alapok
status: complete
title: DANTE -- A professzionális Audio over IP rendszerek kézikönyve
version: 1
---

# 3. Ethernet alapok

> **A fejezet célja:** megérteni azt a hálózati infrastruktúrát, amelyen
> a Dante működik. A fejezet végére nemcsak azt fogod tudni, hogy mi az
> Ethernet és mi a switch, hanem azt is, hogy ezeknek mely tulajdonságai
> számítanak egy professzionális Dante-rendszerben.

## Mit fogsz megtanulni?

A fejezet végére képes leszel:

-   megmagyarázni, mi az Ethernet;
-   megkülönböztetni a fizikai réteget és az adatkapcsolati réteget;
-   értelmezni az Ethernet frame alapvető mezőit;
-   megmagyarázni a MAC-cím szerepét;
-   megérteni, hogyan tanulja meg egy switch, melyik MAC-cím melyik
    porton található;
-   megkülönböztetni az unicast, broadcast és multicast forgalmat;
-   megérteni az unknown unicast flooding működését;
-   megérteni, miért különbözik egy switch a hubtól;
-   értelmezni a full duplex és link speed fogalmát;
-   megérteni a VLAN alapjait;
-   megérteni a trunk és access port közötti különbséget;
-   felismerni, miért fontos az uplink;
-   megérteni a broadcast domain és collision domain fogalmát;
-   megérteni az MTU és Ethernet overhead alapjait;
-   felismerni a hálózati loopok veszélyét;
-   elhelyezni a Spanning Tree működését a hálózatban;
-   megérteni, mikor releváns a QoS és IGMP egy Dante-hálózatban;
-   felismerni, milyen switch-tulajdonságokat kell vizsgálni Dante
    előtt;
-   megérteni, hogy egy **Aruba switch** ugyanúgy lehet Dante-hálózat
    infrastruktúrája, ha a szükséges funkciók és konfiguráció
    rendelkezésre állnak.

------------------------------------------------------------------------

## Hogyan tanuld ezt a fejezetet?

Ez a fejezet **rétegesen** épül fel. Nem kell mindent egyszerre
megjegyezned.

### 🟢 1. szint -- ezt mindenképpen értsd meg

``` text
Ethernet
   ↓
MAC-cím
   ↓
Ethernet frame
   ↓
Switch
   ↓
MAC table
   ↓
Unicast / Broadcast / Multicast
```

Ha ezt a láncot érted, már van egy használható mentális modelled.

### 🟡 2. szint -- Dante-hálózati gondolkodás

``` text
VLAN → Uplink → Multicast → IGMP → QoS → EEE
```

Itt már azt kell megértened, **miért számítanak ezek egy
Dante-rendszerben**.

### 🔴 3. szint -- Deep Dive

Az STP, DSCP, MTU, SFP/SFP+, port mirroring, CRC hibák és hasonló témák
elsőre nem feltétlenül szükségesek. Ezek azért szerepelnek, hogy később
valódi hálózatot is tudj hibakeresni és tervezni.

> **Első olvasáskor ne memorizálj. Értsd meg, mi történik.**

## Egy történet, amely végigkíséri a fejezetet

``` text
         Aruba Switch
        /      |      \
       /       |       \
Stage Box    FOH      DSP
```

A Stage Box egy Dante audiófolyamot küld a FOH konzolnak.

A kérdésünk végig ugyanaz:

> **Mi történik a hálózatban, amikor ez az adat elindul a Stage Boxból,
> és megérkezik a konzolhoz?**

------------------------------------------------------------------------

# 3.1 Mi az Ethernet?

Az Ethernet nem egyetlen kábel és nem egyszerűen „az internet".

Az Ethernet a helyi hálózati kommunikáció egyik alapvető technológiája,
amelynek szabványosítását az IEEE 802.3 szabványcsalád kezeli.

Az IEEE 802.3 szabvány az Ethernet hálózatok MAC- és fizikai rétegének
működését, valamint számos sebesség- és fizikai interfészváltozatot
határoz meg. citeturn0search7turn0search10

Egyszerűsített modell:

``` text
Alkalmazás
    │
    ▼
Dante / UDP / IP
    │
    ▼
Ethernet
    │
    ▼
Fizikai közeg
    │
    ├── réz
    └── optika
```

A Dante szempontjából ez azért fontos, mert a Dante nem saját fizikai
hálózatot épít minden eszköz közé.

A meglévő Ethernet-infrastruktúrát használja.

------------------------------------------------------------------------

# 3.2 Ethernet nem egyenlő kábellel

A „bedugtam az Ethernet-kábelt" kifejezés a hétköznapokban megfelelő.

Mérnöki szempontból azonban több különböző dolgot kell megkülönböztetni.

## Fizikai közeg

Például:

-   réz sodrott érpár;
-   optikai szál.

## PHY

A PHY a fizikai réteghez kapcsolódó adó-vevő funkciókat valósítja meg.

## MAC

A MAC az Ethernet adatkapcsolati működésének része.

## Switch

A switch Ethernet frame-ek továbbítását végzi, jellemzően MAC-cím
alapján.

Ezek együtt alkotják azt a rendszert, amit a felhasználó gyakran
egyszerűen „Ethernetnek" nevez.

------------------------------------------------------------------------

# 3.3 Az Ethernet helye a hálózati modellben

Az OSI-modellben az Ethernet elsősorban az 1. és 2. réteghez
kapcsolódik.

``` text
Layer 7  Application
Layer 6  Presentation
Layer 5  Session
Layer 4  Transport       ← UDP / TCP
Layer 3  Network         ← IP
Layer 2  Data Link       ← Ethernet / MAC
Layer 1  Physical        ← kábel / optika / PHY
```

A Dante megértéséhez ezt a modellt érdemes fejben tartani.

Például:

``` text
Dante audió
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
PHY
     │
     ▼
kábel
```

Ez nem azt jelenti, hogy minden Dante-forgalom minden pillanatban
pontosan ugyanilyen egyszerű protokollveremként jelenik meg; a Dante
rendszerében többféle forgalom és vezérlési mechanizmus létezik.

A modell arra szolgál, hogy tudjuk, **melyik problémát melyik rétegben
keressük**.

------------------------------------------------------------------------

# 3.4 A MAC-cím

Az Ethernet világában az egyik legfontosabb azonosító a MAC-cím.

Egy tipikus MAC-cím:

``` text
00:11:22:33:44:55
```

A MAC-cím 48 bites címzési formátum.

Hexadecimális alakban 12 hexadecimális számjeggyel írható le:

``` text
00 11 22 33 44 55
```

A cím két nagyobb részre is szemléltethető:

``` text
00:11:22
   │
   └── gyártóhoz kapcsolódó OUI-rész

33:44:55
   │
   └── interfészhez rendelt rész
```

A pontos címkiosztási részleteket az IEEE kezeli.

Fontos:

> A MAC-cím nem ugyanaz, mint az IP-cím.

------------------------------------------------------------------------

# 3.5 MAC-cím és IP-cím

A két cím szerepe eltérő.

### MAC

Ethernet szinten használjuk.

``` text
Layer 2
```

### IP

Hálózati szinten használjuk.

``` text
Layer 3
```

Egyszerűsített példa:

``` text
Eszköz
 ├── MAC: 00:11:22:33:44:55
 └── IP:  192.168.10.20
```

Egy Dante-eszköznek ezért lehet:

-   Ethernet/MAC-identitása;
-   IP-konfigurációja;
-   Dante-eszközneve.

Ezeket nem szabad összekeverni.

------------------------------------------------------------------------

# 3.6 Mi az Ethernet frame?

Az Ethernet nem „csak adatot" küld.

Az adatot egy frame-be csomagolja.

Egyszerűsített Ethernet II frame:

``` text
┌───────────────┬───────────────┬──────────────┬──────────┬───────┐
│ Destination   │ Source        │ EtherType    │ Payload  │ FCS   │
│ MAC           │ MAC           │              │          │       │
└───────────────┴───────────────┴──────────────┴──────────┴───────┘
```

A legfontosabb mezők:

-   Destination MAC;
-   Source MAC;
-   EtherType;
-   Payload;
-   FCS.

A fizikai átvitel során további mechanizmusok is részt vesznek, például
preamble és Start Frame Delimiter.

A könyv szempontjából most az a legfontosabb, hogy a switch számára a
**Destination MAC** és **Source MAC** kulcsfontosságú.

------------------------------------------------------------------------

# 3.7 Destination MAC

A destination MAC megmondja, hogy Ethernet szinten kinek szól a frame.

Például:

``` text
Destination:
AA:BB:CC:DD:EE:FF
```

A switch ezt az értéket felhasználhatja annak eldöntésére, melyik port
felé továbbítsa a frame-et.

Ez vezet el a switch működésének egyik legfontosabb részéhez.

------------------------------------------------------------------------

# 3.8 Source MAC és MAC learning

A switch nem előre tudja minden eszköz helyét.

Megtanulja.

Tegyük fel:

``` text
Port 1 ─── PC A
Port 2 ─── Dante Stage Box
Port 3 ─── FOH
Port 4 ─── DSP
```

A Stage Box küld egy frame-et.

A frame source MAC-je alapján a switch megtanulhatja:

``` text
MAC X → Port 2
```

A Cisco dokumentációja ezt a működést úgy írja le, hogy a switch a
beérkező frame-ek source MAC-címe alapján építi és frissíti a
MAC/forwarding táblát. citeturn0search1turn0search3

------------------------------------------------------------------------

# 3.9 MAC address table

A switch táblája leegyszerűsítve:

  VLAN   MAC                 Port
  ------ ------------------- ------
  10     AA:AA:AA:AA:AA:01   1
  10     AA:AA:AA:AA:AA:02   2
  10     AA:AA:AA:AA:AA:03   3
  20     BB:BB:BB:BB:BB:01   4

Fontos, hogy a MAC-címek kezelése a VLAN-környezethez is kötődhet.

Ezért ugyanaz a MAC-cím különböző VLAN-környezetekben eltérő forwarding
kontextusban jelenhet meg.

A switch tehát nem egyszerűen egyetlen globális „MAC → port" listával
gondolkodik.

------------------------------------------------------------------------

# 3.10 Hogyan továbbít egy switch?

Amikor frame érkezik:

``` text
Frame
  │
  ▼
Source MAC megtanulása
  │
  ▼
Destination MAC keresése
  │
  ├── ismert → célport
  │
  └── ismeretlen → flooding
```

A Cisco dokumentációja szerint:

-   ismert destination MAC esetén a switch az adott port felé továbbít;
-   ha a destination MAC ismeretlen, a frame-et az adott VLAN többi
    megfelelő portjára floodolja;
-   ha a destination MAC ugyanahhoz a porthoz tartozik, amelyen a frame
    érkezett, a switch szűrheti, és nem küldi vissza ugyanarra a portra;
-   broadcast esetén a frame-et az adott VLAN-on belül floodolja.
    citeturn0search1turn0search3

Ez az Ethernet switching alapja.

------------------------------------------------------------------------

# 3.11 Ismert unicast

``` text
PC A ──► Switch ──► PC B
```

Ha a switch tudja:

``` text
MAC B → Port 4
```

akkor:

``` text
Port 1
  │
  ▼
Switch
  │
  └────────► Port 4
```

A frame-et nem kell minden portra kiküldenie.

Ez a switching egyik legfontosabb előnye.

------------------------------------------------------------------------

# 3.12 Unknown unicast

Mi történik, ha a switch még nem ismeri a cél MAC-címét?

Például:

``` text
MAC X → ismeretlen
```

A switch a frame-et az adott VLAN többi portjára floodolhatja.

``` text
          ┌──► Port 2
          │
Port 1 ─► SW ──► Port 3
          │
          └──► Port 4
```

A célállomás válasza után a switch megtanulhatja a forrás MAC helyét.

Ezért a flooding bizonyos esetekben normális Ethernet-működés.

A probléma akkor kezdődik, ha túl sok vagy tartós flooding alakul ki.

A Cisco dokumentációja szerint a folyamatos flooding jelentős terhelést
okozhat, különösen szűkebb kapacitású linkeken. citeturn0search2

------------------------------------------------------------------------

# 3.13 Broadcast

A broadcast destination MAC:

``` text
FF:FF:FF:FF:FF:FF
```

A broadcast frame egy Layer 2 broadcast domainen belül minden releváns
végpont felé továbbítható.

``` text
              ┌──► A
              │
Broadcast ──► SW ──► B
              │
              └──► C
```

A VLAN-ok ezért fontosak.

Egy VLAN lényegében külön Layer 2 broadcast domainként működik.

A Cisco dokumentációja szerint a VLAN-ok logikailag szegmentálják a
hálózatot, és az adott VLAN-on belül továbbított/floodolt forgalom a
VLAN tagjaihoz tartozik. citeturn0search11

------------------------------------------------------------------------

# 3.14 Multicast

A multicast más logika.

Egy adó több vevőnek küldhet ugyanahhoz a multicast csoporthoz tartozó
forgalmat.

``` text
             ┌──► Receiver A
TX ──► Group ─┼──► Receiver B
             └──► Receiver C
```

A Dante szempontjából ez különösen fontos.

Az Audinate dokumentációja szerint a Dante támogat multicast audió- és
vezérlési forgalmat, és IGMP használható a multicast forgalom
kezelésére. citeturn0search36

------------------------------------------------------------------------

# 3.15 Miért veszélyes a multicast flooding?

A switchnek tudnia kell, mely portokon vannak érdeklődő vevők.

Ha ezt nem kezeli megfelelően, a multicast forgalom túl sok portra
juthat el.

Cisco dokumentáció szerint a multicast forgalom korlátozására az IGMP
Snooping és kapcsolódó mechanizmusok használhatók; ezek célja, hogy a
multicast ne legyen szükségtelenül floodolva az egész LAN-on.
citeturn0search0

Ez Dante esetén azért fontos, mert sok multicast audió-flow esetén a
fölöslegesen továbbított forgalom jelentős hálózati terhelést okozhat.

------------------------------------------------------------------------

# 3.16 Unicast, broadcast, multicast -- egyben

  -----------------------------------------------------------------------
  Típus                   Cél                     Egyszerű modell
  ----------------------- ----------------------- -----------------------
  Unicast                 egy konkrét vevő        1 → 1

  Broadcast               minden résztvevő az     1 → mindenki
                          adott broadcast         
                          domainben               

  Multicast               egy csoport             1 → több érdeklődő
  -----------------------------------------------------------------------

A Dante-ban mindhárom fogalommal találkozhatsz, de nem ugyanarra
használjuk őket.

------------------------------------------------------------------------

# 3.17 Hub vs switch

A régi Ethernet-hálózatokban hubokat is használtak.

A hub lényegében minden beérkező jelet továbbadott a többi portra.

``` text
       A
       │
B ─── HUB ─── C
       │
       D
```

A switch ezzel szemben megtanulja a MAC-címek helyét, és ismert unicast
esetén célzottan továbbít.

``` text
       A
       │
B ─── SWITCH ─── C
       │
       D
```

Ez nagy különbség.

A modern Dante-rendszerben természetesen menedzselhető Ethernet switch
használatát tervezzük, nem hubot.

------------------------------------------------------------------------

# 3.18 Full duplex

A modern switched Ethernet hálózatokban a full duplex alapvető.

Full duplex esetén az interfész egyidejűleg képes küldeni és fogadni.

``` text
A ─────────► B
A ◄───────── B
```

A két irány egyszerre működhet.

Ez különösen fontos megkülönböztetés a régi shared-medium, half-duplex
Ethernethez képest.

Az IEEE 802.3 szabvány a half-duplex és full-duplex működést egyaránt
kezeli, de a modern switched Ethernetben a full duplex a megszokott
működési mód. citeturn0search7

------------------------------------------------------------------------

# 3.19 Mi az a link speed?

A link speed az adott Ethernet-kapcsolat névleges sebessége.

Például:

-   100 Mbit/s;
-   1 Gbit/s;
-   2,5 Gbit/s;
-   5 Gbit/s;
-   10 Gbit/s;
-   25 Gbit/s;
-   40 Gbit/s;
-   100 Gbit/s.

Dante szempontból a legfontosabb kérdés nem az, hogy:

> „Mekkora a switch maximális sebessége?"

Hanem:

> **„Mekkora sebességgel működik az adott link, és elegendő-e a teljes
> forgalomhoz?"**

------------------------------------------------------------------------

# 3.20 100 Mbit/s és 1 Gbit/s

A 100 Mbit/s Ethernet:

``` text
100 Mbit/s
```

Az 1 Gbit/s:

``` text
1000 Mbit/s
```

Papíron tízszeres különbség.

A Dante számára azonban nemcsak az összkapacitás számít.

A hálózati késleltetés, QoS, multicast-kezelés, uplinkek és
eszközkompatibilitás is számít.

Az Audinate dokumentációja szerint QoS Dante esetén különösen fontos
vegyes 100 Mbps / 1 Gbps infrastruktúrában, illetve bizonyos mixed-use
hálózatokban. citeturn0search36

------------------------------------------------------------------------

# 3.21 A switch backplane és portsebesség

Egy switchnek sok portja lehet.

Például:

``` text
24 × 1 Gbit/s
```

Ez nem feltétlenül jelenti azt, hogy minden port teljes sebességű
forgalma ugyanazon belső útvonalon korlát nélkül továbbítható minden
irányban.

A gyártói specifikációkban ezért találkozhatsz olyan fogalmakkal, mint:

-   switching capacity;
-   forwarding rate;
-   backplane capacity;
-   packet forwarding performance.

A pontos értelmezés gyártónként eltérhet.

Dante-tervezésnél különösen fontos az **uplink kapacitás**.

------------------------------------------------------------------------

# 3.22 Uplink

Tegyük fel, hogy egy access switchhez sok Dante-eszköz csatlakozik.

``` text
Stage Box ─┐
DSP ───────┤
Console ───┤
Recorder ──┤
Amp ───────┤
           ▼
        Access SW
           │
           │ uplink
           ▼
        Core SW
```

Az access switch portjain összegződő forgalomnak az uplinken is át kell
jutnia.

Ha az access switch 24 darab 1 Gbit/s porttal rendelkezik, az nem
jelenti azt, hogy egyetlen 1 Gbit/s uplinken korlátlanul átvihetjük az
összes port teljes forgalmát.

A tervezés során meg kell vizsgálni:

-   milyen forgalom megy az uplinkre;
-   mennyi az összforgalom;
-   milyen a forgalom iránya;
-   milyen burstök vannak;
-   milyen multicast van;
-   van-e tartalék kapacitás.

------------------------------------------------------------------------

# 3.23 Oversubscription

Az oversubscription azt jelenti, hogy a switch hozzáférési portjainak
elméleti összkapacitása nagyobb, mint az adott uplink vagy belső
erőforrás kapacitása.

Példa:

``` text
8 × 1 Gbit/s access
        │
        ▼
   1 Gbit/s uplink
```

Elméletileg:

``` text
8 Gbit/s
   ↓
1 Gbit/s
```

Ez nem feltétlenül probléma.

A hálózatban általában nem minden végpont küld maximális sebességgel
egyszerre ugyanabba az irányba.

De Dante-rendszerben az ilyen tervezést **tudatosan** kell elvégezni.

------------------------------------------------------------------------

# 3.24 VLAN

A VLAN, vagy Virtual LAN, logikai Layer 2 szegmentáció.

Fizikailag ugyanaz a switch lehet:

``` text
Switch
├── VLAN 10
├── VLAN 20
└── VLAN 30
```

A VLAN-ok logikailag külön hálózatokat hoznak létre.

Egyszerű példa:

``` text
VLAN 10
Dante

VLAN 20
Control

VLAN 30
Office
```

Ez azonban nem jelenti automatikusan azt, hogy ezt a felosztást minden
Dante-rendszerben így kell megvalósítani.

A VLAN-tervezésnek kompatibilisnek kell lennie a Dante discovery,
control, clock és audio forgalmával.

------------------------------------------------------------------------

# 3.25 Access port

Egy access port jellemzően egyetlen VLAN-hoz tartozó végponti port.

``` text
Dante eszköz
     │
     │ untagged
     ▼
Access port
     │
     ▼
VLAN 10
```

A végpont sok esetben nem látja közvetlenül a VLAN taget az alkalmazási
szinten.

A switch kezeli a VLAN-hoz kapcsolódó keretezést.

------------------------------------------------------------------------

# 3.26 Trunk port

A trunk több VLAN forgalmát viheti egyetlen linken.

``` text
Switch A
   │
   │ VLAN 10
   │ VLAN 20
   │ VLAN 30
   ▼
Switch B
```

Ez különösen hasznos több switch összekapcsolásakor.

A Dante-rendszerekben azonban a trunk használata előtt mindig meg kell
érteni, hogyan működik az adott Dante-topológia és a használt eszközök.

A „minden Dante külön VLAN-ba" szabály nem univerzális megoldás.

------------------------------------------------------------------------

# 3.27 Miért lehet problémás a túlzott VLAN-komplexitás?

A Dante discovery és egyes vezérlési mechanizmusok Layer 2/multicast
viselkedésre támaszkodnak.

Ha egy egyszerű rendszert indokolatlanul sok VLAN-ra osztunk:

``` text
Dante VLAN
Control VLAN
PC VLAN
Management VLAN
Audio VLAN
Backup VLAN
...
```

akkor a rendszer működése összetettebbé válhat.

A routingot, multicastot, discoveryt és PTP-t külön kell megtervezni.

Ezért a jó hálózattervezés egyik alapelve:

> **Ne használj komplexitást csak azért, mert technikailag lehetséges.**

------------------------------------------------------------------------

# 3.28 MTU

Az MTU, Maximum Transmission Unit, egy hálózati interfész által egy
adott protokollszinten továbbítható maximális csomagméretre vonatkozó
fogalom.

Ethernet-környezetben gyakran a 1500 byte-os IP MTU-val találkozunk.

Jumbo frame-ek esetén ennél nagyobb értékek is használhatók.

Dante-rendszerben azonban nem az a cél, hogy automatikusan „jumbo
frame-re állítsunk mindent".

A teljes hálózatnak kompatibilisnek kell lennie.

``` text
Eszköz A
   │
   │ MTU 9000
   ▼
Switch
   │
   │ MTU 1500
   ▼
Eszköz B
```

A vegyes MTU-környezet problémát okozhat.

Ezért:

> **Jumbo frame-et Dante miatt önmagában nem kell bekapcsolni.**

------------------------------------------------------------------------

# 3.29 Ethernet overhead

Ha digitális audiót küldünk a hálózaton, nem csak a hasznos
audióbájtokat továbbítjuk.

A hálózati adatfolyamhoz különböző fejlécek és keretezési elemek
társulnak.

Egyszerűsített kép:

``` text
Ethernet
┌───────────────────────────────────────────────┐
│ Ethernet header │ IP │ UDP │ Audio payload │FCS│
└───────────────────────────────────────────────┘
```

A tényleges protokollstack és Dante-forgalom ennél összetettebb lehet.

A lényeg:

\[ `\text{wire rate}`{=tex} \> `\text{raw audio bitrate}`{=tex} \]

A különbséget overhead okozza.

------------------------------------------------------------------------

# 3.30 Miért nem kell minden byte-ot kiszámolnunk?

Egy rendszertervezésnél a pontos packet rate és wire rate természetesen
fontos lehet.

De kezdőként nem érdemes a könyv első Ethernet-fejezetében elveszni
minden egyes header byte-ban.

Először értsd meg:

``` text
Audióadat
   ↓
UDP/IP
   ↓
Ethernet frame
   ↓
PHY
   ↓
kábel
```

Később, amikor Dante flow-kat és hálózati kapacitást számolunk,
visszatérünk a részletesebb számításokhoz.

------------------------------------------------------------------------

# 3.31 Collision domain

A collision domain azt a hálózati környezetet írja le, ahol a közös
médium használata miatt ütközések alakulhatnak ki.

A régi hubos, half-duplex Ethernetnél ez fontos fogalom volt.

A modern switched full-duplex Ethernetnél az egyes switchportok
különálló linkeket jelentenek.

Ezért a klasszikus collision-domain probléma a modern Dante-hálózatban
sokkal kevésbé releváns, mint:

-   congestion;
-   queueing;
-   packet loss;
-   multicast;
-   uplink saturation.

------------------------------------------------------------------------

# 3.32 Broadcast domain

A broadcast domain viszont továbbra is nagyon fontos.

Egy VLAN tipikusan egy külön broadcast domain.

``` text
VLAN 10
┌────────────────────────────┐
│ Broadcast domain           │
│                            │
│ A ── B ── C ── D            │
└────────────────────────────┘

VLAN 20
┌────────────────────────────┐
│ Másik broadcast domain     │
│                            │
│ E ── F ── G                │
└────────────────────────────┘
```

A Layer 3 router vagy megfelelő Layer 3 switching választja el a
broadcast domaineket.

Dante esetén ennek azért van jelentősége, mert a discovery és egyes
vezérlési folyamatok hálózati határokon át történő működése külön
tervezést igényelhet.

------------------------------------------------------------------------

# 3.33 Loop

Egy Ethernet-hálózatban a fizikai redundancia önmagában veszélyes lehet.

Például:

``` text
      ┌─────────────┐
      │             │
SW1 ──┘             └── SW2
```

Ha a két switch között két útvonal van, és nincs megfelelő loop
prevention:

``` text
SW1 ─────────► SW2
 ▲              │
 └──────────────┘
```

a Layer 2 forgalom körbe-körbe járhat.

A broadcast és flooding mechanizmusok miatt ez súlyos hálózati
problémához vezethet.

Cisco dokumentáció szerint a redundáns fizikai hurkokat Spanning Tree
mechanizmusokkal lehet kezelni úgy, hogy a redundancia megmaradjon, de
az aktív Layer 2 forwarding topológia ne alkosson hurkot.
citeturn0search3

------------------------------------------------------------------------

# 3.34 Spanning Tree

A Spanning Tree Protocol család célja, hogy a Layer 2 hálózatban ne
maradjon aktív forwarding loop.

Egyszerű modell:

``` text
        SW1
       /   \
      /     \
    SW2 ─── SW3
```

Három switch esetén fizikailag lehet három kapcsolat.

A Spanning Tree egyes linkeket blocking/discarding állapotba helyezhet,
így logikailag fa alakú forwarding topológia marad.

Ha egy aktív link kiesik:

``` text
SW1 ──X── SW2
```

a korábban blokkolt alternatív útvonal aktiválható a használt
STP-változattól és konfigurációtól függően.

------------------------------------------------------------------------

# 3.35 Redundancia vs loop

Ez nagyon fontos Dante-szempontból.

A cél nem:

> „Ne legyen két kábel."

A cél:

> **Legyen redundáns infrastruktúra úgy, hogy a Layer 2 forwarding ne
> alkosson ellenőrizetlen hurkot.**

A későbbi Dante redundancia-fejezetben külön foglalkozunk azzal, hogyan
különbözik:

-   hálózati redundancia;
-   Dante Redundancy;
-   link aggregation;
-   Spanning Tree.

Ezek nem ugyanazok.

------------------------------------------------------------------------

# 3.36 QoS

A Quality of Service azt teszi lehetővé, hogy a hálózat különböző
forgalomtípusokat prioritással kezeljen.

Egy vegyes hálózatban például lehet:

``` text
PTP
Dante audio
Control
File transfer
Web
```

Ha egy porton torlódás keletkezik, nem mindegy, melyik forgalom milyen
sorba kerül.

Az Audinate dokumentációja szerint Dante valós idejű médiafolyamként
alacsony latencyből és jitterből profitál; vegyes hálózatokban QoS
használata szükséges vagy erősen indokolt a Dante clock- és
audióforgalmának prioritására, és az Audinate meghatározott
DSCP-prioritásokat dokumentál. citeturn0search36

------------------------------------------------------------------------

# 3.37 QoS és Dante

Az Audinate dokumentációjában a Dante forgalomhoz több prioritási szint
szerepel.

Egyszerűsítve:

``` text
Highest
   │
   ├── PTP time-critical
   │
   ├── Audio / PTP
   │
   └── Other traffic
Lowest
```

Az Audinate által dokumentált DSCP-értékek között szerepel például:

  Prioritás   Használat                 DSCP
  ----------- ------------------- ----------
  High        time-critical PTP     CS7 / 56
  Medium      audio, PTP v2          EF / 46
  Low         reserved               CS1 / 8
  None        egyéb                        0

A pontos switch-konfigurációt mindig az adott gyártó dokumentációjával
kell összevetni. citeturn0search36

------------------------------------------------------------------------

# 3.38 IGMP

Az IGMP az IP multicast csoporttagság kezelésére szolgál.

Egyszerűen:

``` text
Receiver
   │
   │ „érdekel ez a multicast group”
   ▼
IGMP
   │
   ▼
Switch
```

Az IGMP Snooping képes a multicast forgalmat a releváns portokra
korlátozni.

Az Audinate szerint IGMP használata különösen akkor fontos, ha:

-   mixed network van;
-   IP video is jelen van;
-   jelentős mennyiségű multicast audió-flow működik.
    citeturn0search36

Kis, Dante-only hálózatban kevés vagy nulla multicast-flow mellett nem
feltétlenül ugyanaz a szükséglet.

Ez fontos:

> **Nem minden Dante-hálózathoz kell ugyanaz a multicast-konfiguráció.**

------------------------------------------------------------------------

# 3.39 Energy Efficient Ethernet

Az IEEE 802.3az szabvány szerinti Energy Efficient Ethernetet gyakran
„Green Ethernet" néven is említik.

Az Audinate Dante hálózati adminisztrációs dokumentációja szerint az
EEE-t a Dante-forgalmat kezelő portokon ki kell kapcsolni, mert
problémát okozhat a szinkronizációban és időnként audiókimaradásokhoz
vezethet. citeturn0search36

Ez az egyik olyan switch-beállítás, amelyet Dante-rendszer telepítésénél
**kifejezetten ellenőrizni kell**.

------------------------------------------------------------------------

# 3.40 Managed switch vs unmanaged switch

## Unmanaged switch

Általában:

-   bedugod;
-   működik;
-   kevés konfigurációs lehetőség.

## Managed switch

Lehetővé tesz például:

-   VLAN;
-   QoS;
-   IGMP Snooping;
-   monitoring;
-   port konfiguráció;
-   STP;
-   link diagnosztika;
-   firmware kezelés.

Egy professzionális Dante-rendszernél a managed switch előnye jelentős,
mert a hálózatot nemcsak továbbítani kell, hanem **tervezni és
diagnosztizálni is**.

------------------------------------------------------------------------

# 3.41 Milyen switch kell Dante-hoz?

A válasz nem az, hogy:

> „Dante switch."

A Dante nem egyetlen speciális switch-gyártóhoz kötött Ethernet-hálózat.

A kérdés inkább:

> **Megfelelően konfigurálható és teljesítményű-e a switch az adott
> Dante-rendszerhez?**

Vizsgáld például:

-   linksebesség;
-   switching capacity;
-   forwarding performance;
-   VLAN;
-   QoS;
-   DSCP;
-   IGMP Snooping;
-   IGMP Querier lehetőség;
-   STP/RSTP/MSTP;
-   EEE;
-   port mirroring;
-   SNMP;
-   firmware támogatás;
-   SFP/SFP+ lehetőség;
-   redundáns táp;
-   környezeti követelmények.

------------------------------------------------------------------------

# 3.42 Aruba switch a Dante-rendszerben

A könyv gyakorlati részeiben Aruba switch-csel is dolgozunk.

Ez azért fontos, mert a Dante-rendszer integrátora nem feltétlenül
egyetlen gyártó ökoszisztémájában dolgozik.

Egy Aruba switch esetében is ugyanazokat a hálózati alapelveket
vizsgáljuk:

``` text
Dante végpont
     │
     ▼
Aruba switch
     │
     ▼
uplink
     │
     ▼
core / másik switch
```

A konkrét konfigurációs parancsok az Aruba platformtól és firmware-től
függenek.

Ezért később külön megvizsgáljuk az Aruba környezetet is:

-   VLAN;
-   QoS;
-   IGMP Snooping;
-   IGMP Querier;
-   STP/RSTP;
-   port konfiguráció;
-   EEE;
-   monitoring.

A cél nem az lesz, hogy egyetlen switch-gyártó parancsait bemagoljuk.

A cél az, hogy értsd:

> **mit kell beállítani, miért kell beállítani, és hogyan ellenőrzöd,
> hogy valóban működik.**

------------------------------------------------------------------------

# 3.43 Egy egyszerű Dante switch

Tegyük fel:

``` text
                 ┌── FOH Console
                 │
Stage Box ───────┤
                 │
DSP ─────────────┤
                 │
Recorder ────────┤
                 │
Aruba Switch ────┤
                 │
Amp ─────────────┘
```

Egy ilyen kis rendszerben a hálózat viszonylag egyszerű.

De már itt is ellenőrizni kell:

-   minden link felépült-e;
-   megfelelő sebességen működik-e;
-   van-e IP-kapcsolat;
-   működik-e a Dante discovery;
-   van-e clock;
-   van-e subscription;
-   van-e packet loss;
-   nincs-e EEE-probléma.

------------------------------------------------------------------------

# 3.44 Több switch

Nagyobb rendszer:

``` mermaid
flowchart TB
    CORE["Core Switch"]
    A["Aruba Access Switch A"]
    B["Aruba Access Switch B"]
    FOH["FOH"]
    STAGE["Stage Boxes"]
    DSP["DSP"]
    AMP["Amplifiers"]

    CORE --> A
    CORE --> B
    A --> FOH
    A --> STAGE
    B --> DSP
    B --> AMP
```

Itt már megjelenik az uplink-tervezés.

A kérdések:

-   1 Gbit/s vagy 10 Gbit/s uplink?
-   milyen hosszú a link?
-   réz vagy optika?
-   mennyi audióforgalom megy rajta?
-   milyen multicast van?
-   milyen QoS van?
-   hogyan működik a redundancia?

------------------------------------------------------------------------

# 3.45 Optika

Nagyobb távolságoknál az optikai Ethernet fontos lehet.

Előnye lehet:

-   nagyobb távolság;
-   elektromos leválasztás;
-   kisebb elektromágneses érzékenység bizonyos környezetekben;
-   nagyobb sávszélességű uplinkek támogatása.

Például:

``` text
Stage
  │
  │ réz
  ▼
Access switch
  │
  │ fiber
  ▼
Core switch
```

Dante-rendszerben az optikai link nem „másik Dante".

Ugyanazt az Ethernet-hálózatot használjuk, csak más fizikai közeggel.

------------------------------------------------------------------------

# 3.46 SFP és SFP+

A switcheken gyakran találkozunk SFP/SFP+ portokkal.

Ezek moduláris interfészhelyek, amelyekhez különböző optikai vagy réz
modulok csatlakoztathatók.

Egyszerűen:

``` text
Switch
 ┌─────────────┐
 │ SFP         │──── Fiber
 │ SFP+        │──── Fiber
 └─────────────┘
```

Az SFP és SFP+ nem egyetlen konkrét optikai szabványt jelent.

Mindig ellenőrizni kell:

-   modul típusa;
-   sebesség;
-   hullámhossz;
-   single-mode / multimode;
-   távolság;
-   kompatibilitás.

------------------------------------------------------------------------

# 3.47 PoE

A Power over Ethernet lehetővé teszi energia továbbítását
Ethernet-kábelen.

``` text
Switch
  │
  │ data + power
  ▼
PoE device
```

Dante-szempontból ez azért lehet érdekes, mert egyes hálózati
AV-eszközök PoE-ről is működhetnek.

De fontos:

> **Nem minden Dante-eszköz PoE-kompatibilis.**

A switch PoE-képességét és az eszköz fogyasztását mindig együtt kell
vizsgálni.

------------------------------------------------------------------------

# 3.48 Link negotiation

Két Ethernet-eszköz csatlakozásakor a portoknak meg kell állapodniuk a
használható linkparaméterekben.

Egy egyszerű példában:

``` text
Switch
1 Gbit/s capable
       │
       │ negotiation
       ▼
Device
1 Gbit/s capable
```

Ha a kábel vagy egyik eszköz miatt csak 100 Mbit/s érhető el:

``` text
1 Gbit/s ──► 100 Mbit/s
```

akkor a kapcsolat a kisebb közös sebességen működhet.

Ezért egy Dante-hibakeresésnél fontos kérdés:

> **Milyen link speeden működik ténylegesen a port?**

Nem elég azt tudni, hogy a switch „gigabites".

------------------------------------------------------------------------

# 3.49 Hibás kábel, jó Dante?

Képzeljük el:

``` text
Dante Stage Box
       │
       │ kábel
       ▼
Switch
```

A kábel hibás.

A link lehet:

-   teljesen down;
-   intermittáló;
-   alacsonyabb sebességre visszaálló;
-   hibás frame-eket okozó.

A Dante Controllerben ez később audióhibaként jelenhet meg.

Ezért:

> **A Dante hibakeresése Ethernet-hibakeresés is.**

------------------------------------------------------------------------

# 3.50 Port counters

A managed switch egyik legfontosabb hibakeresési eszköze a
portstatisztika.

Például:

-   RX packets;
-   TX packets;
-   errors;
-   drops;
-   CRC errors;
-   collisions;
-   multicast;
-   broadcast.

Ha egy Dante-porton folyamatosan CRC errorokat látunk:

``` text
RX:
1 000 000 packets
CRC errors:
12 000
```

az erős jelzés arra, hogy a fizikai kapcsolatot kell vizsgálni.

Lehetséges ok:

-   kábel;
-   csatlakozó;
-   transceiver;
-   port;
-   elektromos környezet.

Nem a Dante subscription újrakattintása az első lépés.

------------------------------------------------------------------------

# 3.51 Port mirroring

A managed switch képes lehet egy port forgalmát egy másik, monitorozó
portra tükrözni.

``` text
Dante Port
    │
    ├────► Network
    │
    └────► Mirror
             │
             ▼
        Capture PC
```

Ezzel például Wiresharkkal vizsgálható a hálózati forgalom.

A későbbi hibakeresési fejezetben erre külön visszatérünk.

------------------------------------------------------------------------

# 3.52 Dante és Wireshark

A Wireshark hálózati packet analyzer.

Egy Dante-hálózat hibakeresésénél használható például:

-   multicast forgalom vizsgálatára;
-   UDP-forgalom vizsgálatára;
-   PTP-forgalom vizsgálatára;
-   IP-címzés ellenőrzésére;
-   packet loss körüli vizsgálatokra.

Fontos azonban:

> A Wireshark nem helyettesíti a switch monitoringot.

A két eszköz különböző szintű információt ad.

``` text
Switch counters
       +
Wireshark capture
       +
Dante Controller
       =
jobb hibakeresés
```

------------------------------------------------------------------------

# 3.53 Egy teljes Layer 2 hibakeresési példa

A FOH nem kap hangot.

### 1. Ellenőrzés

``` text
Stage Box → Switch
```

Link LED rendben?

### 2. Port speed

``` text
1000 Mbps?
```

### 3. Errors

``` text
CRC = 0?
Drops = 0?
```

### 4. MAC

A switch látja a Stage Box MAC-címét?

### 5. IP

A Dante Controller látja az eszközt?

### 6. Clock

Van PTP?

### 7. Subscription

Van TX → RX kapcsolat?

Ez a hibakeresési sorrend sokkal gyorsabb, mint véletlenszerűen
konfigurációkat változtatni.

------------------------------------------------------------------------

# 3.54 Dante switch-tervezési checklist

Egy új switch kiválasztásakor:

## Fizikai

-   [ ] megfelelő portszám;
-   [ ] megfelelő linksebesség;
-   [ ] megfelelő kábelezés;
-   [ ] szükséges SFP/SFP+;
-   [ ] szükséges PoE;
-   [ ] redundáns táp, ha szükséges.

## Layer 2

-   [ ] MAC table kapacitás;
-   [ ] VLAN;
-   [ ] STP/RSTP;
-   [ ] multicast kezelés;
-   [ ] IGMP Snooping.

## QoS

-   [ ] DSCP támogatás;
-   [ ] prioritási queue-k;
-   [ ] megfelelő mapping;
-   [ ] congestion kezelés.

## Dante

-   [ ] EEE kikapcsolható;
-   [ ] megfelelő linksebesség;
-   [ ] PTP-forgalom megfelelő kezelése;
-   [ ] multicast igény felmérve;
-   [ ] Dante hálózati dokumentációval összevetve.

## Monitoring

-   [ ] port counters;
-   [ ] SNMP;
-   [ ] syslog;
-   [ ] port mirroring;
-   [ ] firmware management.

------------------------------------------------------------------------

# 3.55 Mit nem csinálunk?

Egy rossz Dante-hálózat gyakran így készül:

> „Vegyünk egy drága switch-et, az biztos jó lesz."

Ez nem mérnöki tervezés.

Egy drága switch lehet rosszul konfigurálva.

Egy olcsóbb, megfelelően kiválasztott és konfigurált switch pedig lehet
teljesen megfelelő egy adott rendszerben.

A helyes folyamat:

``` text
Követelmények
      ↓
Forgalmi modell
      ↓
Topológia
      ↓
Switch kiválasztás
      ↓
Konfiguráció
      ↓
Teszt
      ↓
Dokumentáció
```

------------------------------------------------------------------------

# 3.56 Dante hálózat: minimális modell

Egy nagyon egyszerű rendszer:

``` mermaid
flowchart LR
    PC["Dante Controller PC"] --> SW["Ethernet Switch"]
    ST["Dante Stage Box"] --> SW
    FOH["Dante Console"] --> SW
```

Itt már három fontos komponensünk van:

1.  Dante végpont;
2.  Ethernet switch;
3.  vezérlő PC.

A switch feladata nem az, hogy „értse a Dante-t".

A switch Ethernet-szinten továbbítja a forgalmat.

A Dante-specifikus működés a végpontok és a hálózati protokollok
együttműködéséből áll.

------------------------------------------------------------------------

# 3.57 Mi történik egy subscription után?

Tegyük fel:

``` text
Stage Box TX1
      │
      ▼
FOH RX1
```

A Dante Controllerben létrehozzuk a subscriptiont.

A háttérben a hálózatban Dante audióforgalom kezd közlekedni.

Az Ethernet-hálózat szempontjából:

``` text
Dante audio
    ↓
UDP/IP
    ↓
Ethernet frame
    ↓
Switch
    ↓
Ethernet frame
    ↓
Receiver
```

A switch nem „hangcsatornát" lát.

Ethernet frame-eket lát.

Ezért fontos a hálózat megértése.

------------------------------------------------------------------------

# 3.58 Az Ethernet frame és a Dante közötti kapcsolat

A korábbi fejezetben ezt láttuk:

``` text
Analóg
 ↓
ADC
 ↓
PCM
 ↓
Dante
```

Most ezt egészítjük ki:

``` text
PCM
 ↓
Dante transport
 ↓
UDP/IP
 ↓
Ethernet frame
 ↓
Switch
 ↓
Ethernet
 ↓
Receiver
```

A következő fejezetekben ezt a középső részt fogjuk részletesen
lebontani.

------------------------------------------------------------------------

# 3.59 Gyakorlati feladat -- MAC learning

Képzelj el három eszközt:

``` text
Port 1 → PC
Port 2 → Stage Box
Port 3 → FOH
```

Kezdetben a switch MAC táblája üres.

A Stage Box frame-et küld.

### Kérdések

1.  Melyik MAC-címet tanulja meg a switch?
2.  Melyik porthoz társítja?
3.  Mit tesz, ha a destination MAC ismeretlen?
4.  Mi történik később, amikor már ismeri a cél MAC-jét?

### Megoldás

A switch a beérkező frame source MAC-címét a beérkezési porttal tanulja
meg.

Ha a destination MAC ismeretlen, az adott VLAN-on belül floodolhat.

Amikor a cél MAC már ismert, a frame-et célzottan a megfelelő port felé
továbbítja. citeturn0search1turn0search3

------------------------------------------------------------------------

# 3.60 Gyakorlati feladat -- Uplink számítás

Van:

``` text
16 × 1 Gbit/s access port
1 × 1 Gbit/s uplink
```

### Kérdés

Lehetséges-e, hogy az uplink szűk keresztmetszet legyen?

**Igen.**

Az access portok összes elméleti kapacitása:

\[ 16`\text{ Gbit/s}`{=tex} \]

Az uplink:

\[ 1`\text{ Gbit/s}`{=tex} \]

Ez nem jelenti azt, hogy a hálózat automatikusan rossz.

A tényleges forgalmat kell megvizsgálni.

### Következő kérdés

Mi lenne, ha az uplink:

``` text
10 Gbit/s
```

lenne?

A rendelkezésre álló összkapacitás sokkal nagyobb lenne, és az
oversubscription arány jelentősen javulna.

------------------------------------------------------------------------

# 3.61 Gyakorlati feladat -- VLAN

Tervezd meg:

``` text
VLAN 10 – Dante
VLAN 20 – Management
VLAN 30 – Office
```

Ezután válaszolj:

1.  Melyik VLAN-ban van a Dante Stage Box?
2.  Melyik VLAN-ban legyen a menedzsment?
3.  Hogyan éri el a PC a Dante-eszközöket?
4.  Mi történik a VLAN-ok között?
5.  Hol történik Layer 3 routing?
6.  Hogyan biztosítod, hogy a Dante discovery és PTP megfelelően
    működjön?

A 6. kérdésre nincs univerzális egyetlen mondatos válasz.

A konkrét topológia, Dante-eszközök, routing és multicast-környezet
alapján kell tervezni.

------------------------------------------------------------------------

# 3.62 Gyakorlati feladat -- Aruba

Építs egy kis laborhálózatot:

``` text
                 ┌── Dante Stage Box
                 │
PC ───────────── Aruba Switch
                 │
                 └── Dante Console
```

Ellenőrizd:

-   link speed;
-   MAC table;
-   VLAN;
-   IP-kapcsolat;
-   multicast;
-   QoS;
-   EEE;
-   port counters.

A cél nem egy konkrét Aruba parancssor bemagolása.

A cél:

> **tudd megmondani, mit ellenőrzöl és miért.**

A következő switch-fejezetekben platformonként is fogunk konfigurálni.

------------------------------------------------------------------------

# 3.63 A teljes folyamat egyetlen Dante audiócsomaggal

Most rakjuk össze az eddigieket. Nem az a cél, hogy minden
protokollrészletet már most tudj, hanem hogy **lásd a folyamatot**.

Tegyük fel:

``` text
Stage Box
    │
    │ Dante TX
    ▼
Aruba Switch
    │
    ▼
FOH Console
```

### 1. A Stage Box digitális audiót kezel

A 2. fejezetben láttuk:

``` text
Hang
 ↓
Mikrofon
 ↓
ADC
 ↓
PCM
```

### 2. A Dante továbbítási lánca kezeli az audiót

Egyszerűsítve:

``` text
PCM audió
   ↓
Dante transport
   ↓
UDP/IP
```

### 3. Az IP-adat Ethernet frame-be kerül

Az Ethernet-szinten többek között megjelenik:

``` text
Destination MAC
Source MAC
EtherType
Payload
FCS
```

A switch számára itt már nem az a kérdés, hogy „ez egy
mikrofoncsatorna".

**A switch Ethernet frame-et lát.**

### 4. A switch megvizsgálja a destination MAC-et

Például:

``` text
Destination MAC:
AA:BB:CC:DD:EE:FF
```

A switch MAC táblájában lehet:

``` text
AA:BB:CC:DD:EE:FF → Port 12
```

Ekkor a switch a frame-et a Port 12 felé továbbítja.

### 5. Mi történik, ha nem ismeri?

Ha a destination MAC még nincs a táblában, az adott VLAN-on belül
flooding történhet.

### 6. A FOH konzol megkapja a frame-et

A konzol hálózati interfésze fogadja a frame-et, majd a magasabb
protokollrétegek feldolgozzák az adatot.

## Az egész folyamat egy ábrán

``` text
Stage Box
   │
   ▼
Dante transport
   │
   ▼
UDP / IP
   │
   ▼
Ethernet frame
   │
   ├── Destination MAC
   ├── Source MAC
   └── Payload
   │
   ▼
Aruba Switch
   │
   ├── MAC table
   ├── VLAN
   └── QoS / multicast
   │
   ▼
FOH Console
```

### Mini ellenőrzés

**Mit lát a switch?**\
Ethernet frame-eket.

**Mi alapján dönt ismert unicast esetén?**\
A destination MAC és a forwarding/MAC table alapján.

**Miért fontos a VLAN?**\
Mert meghatározza azt a Layer 2 környezetet/broadcast domaint, amelyben
a forwarding és flooding történik.

**Miért érdekes mindez Dante esetén?**\
Mert a Dante hálózati kommunikációját az Ethernet-infrastruktúra
továbbítja.

------------------------------------------------------------------------

# 3.64 Deep Dive -- Miért MAC alapján kapcsol a switch?

Az IP-cím önmagában nem Ethernet forwarding információ.

A Layer 2 switch a frame destination MAC-címét használja a Layer 2
forwardinghoz.

``` text
IP packet
   │
   ▼
Ethernet frame
   │
   ▼
Destination MAC
   │
   ▼
Switch forwarding table
   │
   ▼
Port
```

A router ezzel szemben Layer 3 információ alapján hoz forwarding
döntést.

Ezért:

``` text
Switch ≠ Router
```

Bár egy modern Layer 3 switch mindkét funkciót képes lehet ellátni.

------------------------------------------------------------------------

# 3.65 Deep Dive -- Miért nem lát minden eszköz minden frame-et?

A hubos Ethernethez képest a switch célzottan továbbítja az ismert
unicast frame-eket.

``` text
A ──► Switch ──► B
```

C nem feltétlenül kapja meg a B-nek küldött unicast frame-et.

Ez fontos biztonsági és teljesítménybeli előny.

Viszont:

-   broadcast;
-   unknown unicast;
-   bizonyos multicast

forgalom floodolható.

Ezért a hálózati viselkedés megértéséhez nem elég azt mondani:

> „A switch nem küldi mindenkinek."

A helyes állítás:

> **Az ismert unicastot célzottan továbbítja, míg bizonyos ismeretlen
> vagy csoportos forgalmak floodolhatók.**

------------------------------------------------------------------------

# 3.66 Deep Dive -- Miért számít a multicast Dante-nál?

Tegyük fel, hogy egy audióforrást tíz vevőnek kell eljuttatni.

Unicast esetén leegyszerűsítve:

``` text
TX ─► RX1
TX ─► RX2
TX ─► RX3
...
TX ─► RX10
```

Multicast esetén:

``` text
TX ─► multicast group ─► RX1
                       ├► RX2
                       ├► RX3
                       └► RX10
```

A hálózati forgalom és a switch terhelése másképp alakul.

A multicast előnye nem azt jelenti, hogy minden esetben multicastot kell
használni.

A routing és flow-tervezés alapján kell választani.

------------------------------------------------------------------------

# 3.67 Deep Dive -- Miért nem a legdrágább switch a legjobb?

Mert a Dante-kompatibilitás nem pusztán ár kérdése.

Egy switch lehet:

-   48 portos;
-   10 Gbit/s uplinkes;
-   komoly Layer 3 képességekkel rendelkező;

és mégis problémás Dante-rendszerben, ha például:

-   EEE nem kapcsolható ki;
-   QoS rosszul konfigurálható;
-   multicast kezelése nem megfelelő;
-   PTP-forgalom problémás;
-   firmware bug van;
-   a hálózati topológia rosszul van megtervezve.

A jó választás:

``` text
követelmény
   ↓
kompatibilitás
   ↓
kapacitás
   ↓
konfigurálhatóság
   ↓
monitoring
   ↓
megbízhatóság
```

------------------------------------------------------------------------

# 3.68 Hálózati dokumentáció

Egy professzionális Dante-hálózatot nem csak megépíteni kell.

Dokumentálni is kell.

Legalább:

``` text
Switch neve
IP-címe
Firmware
Port
VLAN
Csatlakoztatott eszköz
Link speed
Uplink
```

Például:

  Switch        Port Eszköz           VLAN   Speed
  ----------- ------ ------------- ------- -------
  Aruba-SW1        1 Stagebox-01        10      1G
  Aruba-SW1        2 Stagebox-02        10      1G
  Aruba-SW1        3 FOH                10      1G
  Aruba-SW1       24 Core uplink     trunk     10G

Ez később hibakeresésnél aranyat ér.

------------------------------------------------------------------------

# 3.69 Ethernet hibakeresési sorrend

Ha Dante-n nincs hang:

``` text
1. Fizikai link
      ↓
2. Link speed
      ↓
3. Port errors
      ↓
4. MAC learning
      ↓
5. VLAN
      ↓
6. IP
      ↓
7. Multicast / unicast
      ↓
8. QoS
      ↓
9. PTP
      ↓
10. Dante subscription
```

A sorrend nem merev szabály minden hibára, de jó kiindulási modell.

A lényeg:

> **Ne ugorjunk rögtön a Dante Controllerhez, ha az Ethernet-réteget még
> nem ellenőriztük.**

------------------------------------------------------------------------

# 3.70 Kezdő szótár

  -----------------------------------------------------------------------
  Fogalom                             Egyszerű jelentés
  ----------------------------------- -----------------------------------
  Ethernet                            A helyi hálózati kommunikáció egyik
                                      alapvető technológiája

  MAC-cím                             Ethernet-szintű cím

  IP-cím                              Hálózati szintű cím

  Frame                               Ethernet-szintű adat-egység

  Switch                              Ethernet frame-eket továbbító
                                      hálózati eszköz

  MAC table                           A switch által tanult MAC → port
                                      információ

  Unicast                             Egy adó → egy vevő

  Broadcast                           Egy Layer 2 broadcast domain összes
                                      releváns résztvevője

  Multicast                           Egy adó → egy érdeklődő csoport

  VLAN                                Logikai Layer 2 hálózati
                                      szegmentáció

  Uplink                              Switch-ek közötti vagy magasabb
                                      hálózati réteg felé vezető link

  QoS                                 Forgalmi prioritás és sorbaállítás

  IGMP                                IP multicast csoporttagság
                                      kezeléséhez kapcsolódó protokoll

  MTU                                 A hálózati adat-egység méretének
                                      egyik fontos korlátja

  STP                                 Layer 2 loopok kezelésére szolgáló
                                      protokollcsalád

  EEE                                 Energiahatékonysági
                                      Ethernet-funkció, amelyet
                                      Dante-portokon az Audinate ajánlása
                                      szerint ellenőrizni/kikapcsolni
                                      kell
  -----------------------------------------------------------------------

> Ha egy fogalmat nem értesz, ne ugord át. A későbbi Dante-fejezetekben
> ugyanazok a fogalmak újra vissza fognak térni.

------------------------------------------------------------------------

# 3.71 Összefoglalás

Ebben a fejezetben az Ethernetet nem általános informatikai témaként
kezeltük, hanem Dante-rendszerintegrációs szemmel.

A legfontosabb fogalmak:

### Ethernet

Az IEEE 802.3 által szabványosított hálózati technológiacsalád.

### MAC

Layer 2-es címzés.

### Frame

Az Ethernet adatkapcsolati szintű adat-egysége.

### Switch

MAC-címek alapján továbbít Ethernet frame-eket.

### Unicast

Egy adó → egy vevő.

### Broadcast

Egy broadcast domain összes releváns résztvevője.

### Multicast

Egy adó → egy multicast csoport.

### VLAN

Logikai Layer 2 szegmentáció.

### QoS

Forgalmi prioritás és sorbaállítás.

### IGMP

IP multicast csoporttagság kezeléséhez kapcsolódó protokoll.

### Uplink

Switch-ek közötti vagy access/core kapcsolat.

### MTU

A továbbítható csomagméret egyik fontos korlátja.

### STP

Layer 2 loopok megelőzésére szolgáló protokollcsalád.

------------------------------------------------------------------------

# 3.72 Ellenőrző kérdések

## Alap

1.  Mi az Ethernet?
2.  Mi a MAC-cím?
3.  Mi az IP-cím?
4.  Mi a különbség a MAC és IP között?
5.  Mi az Ethernet frame?
6.  Mi a destination MAC?
7.  Mi a source MAC?
8.  Hogyan tanulja meg a switch a MAC-címeket?
9.  Mi az unknown unicast?
10. Mi a broadcast?
11. Mi a multicast?
12. Mi a VLAN?
13. Mi a QoS?
14. Mi az IGMP?
15. Mi az MTU?

## Dante

16. Miért fontos a switch a Dante-rendszerben?
17. Miért nem elég azt mondani, hogy „gigabites a switch"?
18. Miért fontos az uplink?
19. Miért lehet szükség QoS-ra?
20. Mikor fontos különösen az IGMP?
21. Miért kell figyelni az EEE-re?
22. Miért lehet probléma egy Layer 2 loop?
23. Miért fontos a port counters használata?
24. Miért lehet hasznos a port mirroring?
25. Miért használható Aruba switch Dante-rendszerben?

------------------------------------------------------------------------

# 3.73 Laborprojekt -- első Dante Ethernet hálózat

Építs fel egy laborhálózatot:

``` mermaid
flowchart LR
    PC["PC / Dante Controller"] --> SW["Aruba Managed Switch"]
    ST["Dante Stage Box"] --> SW
    CON["Dante Console"] --> SW
    DSP["Dante DSP"] --> SW
```

## 1. lépés -- Fizikai

Ellenőrizd:

-   kábelek;
-   link LED;
-   port speed.

## 2. lépés -- Layer 2

Ellenőrizd:

-   MAC table;
-   VLAN;
-   broadcast domain.

## 3. lépés -- Layer 3

Ellenőrizd:

-   IP-cím;
-   subnet;
-   elérhetőség.

## 4. lépés -- Dante

Ellenőrizd:

-   discovery;
-   clock;
-   subscription.

## 5. lépés -- Switch

Ellenőrizd:

-   QoS;
-   IGMP;
-   EEE;
-   errors;
-   drops.

A labor végén készíts egy egyszerű hálózati dokumentációt.

------------------------------------------------------------------------

# 3.74 Laborprojekt -- hibás hálózat

Most szándékosan hozz létre egy hibát.

Példák:

-   húzd ki a Dante Stage Box kábelét;
-   állítsd át hibás VLAN-ra;
-   hozz létre nem megfelelő multicast konfigurációt laborban;
-   szimulálj hibás uplinket;
-   ha a platform támogatja és a tesztkörnyezet biztonságos, vizsgáld
    meg az EEE hatását.

Ezután ne a Dante Controllerben kezdd.

Használd:

``` text
Switch
 ↓
Port status
 ↓
Counters
 ↓
MAC table
 ↓
VLAN
 ↓
IP
 ↓
Dante Controller
```

A cél az, hogy megtanuld:

> **A hálózati hibát bizonyítékok alapján keresd, ne találgatással.**

------------------------------------------------------------------------

# 3.75 Következő fejezet

Az Ethernet alapjai után tovább kell lépnünk a hálózati protokollokra.

A következő fejezetben:

# 4. IP és UDP

megvizsgáljuk:

-   IPv4;
-   subnet mask;
-   gateway;
-   ARP;
-   UDP;
-   portok;
-   multicast IP;
-   broadcast IP;
-   Dante által használt hálózati kommunikáció;
-   és azt, hogyan kerül az Ethernet frame-be az IP/UDP adat.

A mentális modellünk:

``` text
Dante
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
PHY
```

A 3. fejezetben az alsó két réteget értettük meg.

A 4. fejezetben egy szinttel feljebb lépünk.

------------------------------------------------------------------------

# 3.76 Források

1.  **IEEE -- IEEE 802.3 Ethernet Standard**\
    https://standards.ieee.org/ieee/802.3-2022.html

2.  **Cisco -- Configure MAC / Layer 2 forwarding**\
    https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr2-fwd/cdp-lldp-mac-udld/cdp-lldp-mac-udld-configuration-guide/c-configure-mac.html

3.  **Cisco -- Troubleshoot LAN Switching Environments**\
    https://www.cisco.com/c/en/us/support/docs/lan-switching/ethernet/12006-chapter22.html

4.  **Cisco -- Multicast in a Campus Network / IGMP Snooping**\
    https://www.cisco.com/c/en/us/support/docs/switches/catalyst-6500-series-switches/10559-22.html

5.  **Audinate -- Dante Information for Network Administrators**\
    https://assets.audinate.com/wp-content/uploads/2022/03/dante-information-for-network-admins.pdf

------------------------------------------------------------------------

# 3.77 Műszaki ellenőrzési megjegyzés

A fejezet kritikus hálózati állításait elsődleges szabvány- és gyártói
dokumentációval ellenőriztük.

Különösen ellenőrzött témák:

-   Ethernet és IEEE 802.3;
-   MAC learning és Layer 2 forwarding;
-   unknown unicast és broadcast flooding;
-   VLAN;
-   multicast és IGMP Snooping;
-   Dante QoS;
-   Dante multicast;
-   EEE;
-   STP és Layer 2 loopok.

Ahol egy gyártói dokumentáció platformfüggő működést ír le, azt nem
általánosítottuk minden switchre. A konkrét Aruba-konfigurációkat a
későbbi Aruba-laborokban az adott ArubaOS/AOS-CX verzió
dokumentációjához kötjük.

------------------------------------------------------------------------

# 3.78 Fejezeti állapot

**Állapot: COMPLETE**

A fejezet tartalmaz:

-   Ethernet alapokat;
-   OSI-rétegkapcsolatot;
-   MAC-címeket;
-   Ethernet frame-et;
-   switch forwardingot;
-   MAC learninget;
-   unicast/broadcast/multicast működést;
-   floodingot;
-   full duplexet;
-   link speedet;
-   uplinket;
-   oversubscriptiont;
-   VLAN-t;
-   access/trunk portot;
-   MTU-t;
-   Ethernet overheadet;
-   collision/broadcast domain fogalmakat;
-   STP-t;
-   QoS-t;
-   IGMP-t;
-   EEE-t;
-   managed switch koncepciót;
-   Aruba switch környezetet;
-   port counters-t;
-   port mirroringot;
-   Dante Ethernet hibakeresést;
-   switch-tervezési checklistet;
-   laborprojekteket;
-   Deep Dive részeket;
-   ellenőrző kérdéseket;
-   műszaki forrásokat.
