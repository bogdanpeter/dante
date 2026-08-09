---
author: Peter Bogdan
chapter: 5
chapter_title: Dante architektúra
status: complete
title: DANTE -- A professzionális Audio over IP rendszerek kézikönyve
version: 1
---

# Dante architektúra

> **A fejezet célja:** megérteni, hogy a Dante nem egyszerűen „hang
> küldése Etherneten", hanem egy több komponensből álló hálózati
> audio-architektúra. A fejezet végére képes leszel megkülönböztetni a
> Dante eszközöket, a Dante Controller szerepét, a subscription és flow
> fogalmát, az unicast és multicast audio működését, a PTP clocking
> alapját, valamint azt, hogyan áll össze mindez egy működő
> Dante-rendszerré.

## Előismeretek

Ehhez a fejezethez már tudnod kell:

-   mi az Ethernet frame;
-   mi a MAC-cím;
-   mi az IP-cím és a subnet;
-   mi az UDP;
-   mi a switch;
-   mi az unicast és multicast alapelve;
-   miért fontos a QoS és az IGMP.

Ha ezek közül valami bizonytalan, nem probléma. A szükséges fogalmakat
itt újra összekapcsoljuk, de nem tanítjuk újra teljes részletességgel.

------------------------------------------------------------------------

## Hogyan tanuld ezt a fejezetet?

Most egy fontos váltás következik.

Az első négy fejezetben elsősorban azt tanultuk meg, **milyen
technológiákból épül fel egy Dante-rendszer**.

Most azt nézzük meg:

> **Hogyan működik maga a Dante rendszer?**

### 🟢 1. szint -- ezt mindenképpen értsd

``` text
Dante Device
      ↓
Dante Controller
      ↓
Subscription
      ↓
Flow
      ↓
Audio
```

És:

``` text
Audio
  +
Clock
  +
Control / Discovery
  +
Network
```

### 🟡 2. szint -- rendszerintegrátori gondolkodás

Ezután:

-   transmit és receive channel;
-   unicast flow;
-   multicast flow;
-   fanout;
-   clock leader és follower;
-   Dante Controller és hálózati discovery;
-   Primary és Secondary interface;
-   redundancy;
-   latency;
-   Dante Domain Manager alapjai.

### 🔴 3. szint -- Deep Dive

A részletes PTP működés, flow-allokáció, multicast routing, Dante Domain
Manager, AES67 és SMPTE ST 2110 interoperabilitás már a következő szint.

> **Első olvasáskor ne a portszámokat vagy protokollrészleteket
> memorizáld. A rendszer működésének logikáját értsd meg.**

------------------------------------------------------------------------

# 5.1 Mi a Dante valójában?

A Dante egy professzionális audio-over-IP platform.

A legfontosabb gondolat:

> **A Dante nem egyetlen protokoll.**

Egy Dante-rendszerben többféle hálózati funkció dolgozik együtt:

``` text
                 DANTE
                   │
       ┌───────────┼───────────┐
       │           │           │
      Audio      Clock       Control
       │           │           │
       ▼           ▼           ▼
      UDP         PTP      Discovery /
                             Routing
                   │
                   ▼
                Ethernet
                   │
                   ▼
                 Switch
```

A Dante modern IP-hálózati technológiákra épül, ezért a Dante-eszközök a
hálózaton IP-végpontokként viselkednek, és hagyományos Ethernet
infrastruktúrán is működhetnek. citeturn0search20turn0search4

Ez az első fontos mentális modell:

> **Dante = audio + időzítés + vezérlés + hálózati infrastruktúra
> együttese.**

------------------------------------------------------------------------

# 5.2 Dante eszközök

Egy Dante-rendszerben különböző típusú eszközökkel találkozhatsz:

-   Dante audio interface;
-   mixing console;
-   stage box;
-   DSP;
-   amplifier;
-   speaker processor;
-   broadcast interface;
-   computer interface;
-   Dante Virtual Soundcard;
-   Dante AV-eszközök;
-   egyéb Dante-enabled végpontok.

Ezek funkciója különbözhet, de hálózati szempontból közös bennük, hogy
Dante-képes végpontként vesznek részt a rendszerben.

------------------------------------------------------------------------

# 5.3 Transmit és Receive

A Dante routing alapja két fogalom:

``` text
TX = Transmit
RX = Receive
```

Például:

``` text
Stage Box
TX Channel 1
"Vocal 1"

        ↓

Console
RX Channel 17
"Vocal 1"
```

A Stage Box **küld**.

A Console **fogad**.

Ez még nem jelenti azt, hogy a kapcsolat létrejött.

A kettőt össze kell kapcsolni.

------------------------------------------------------------------------

# 5.4 Mi az a subscription?

A Dante-ban a routingot subscriptionnel hozzuk létre.

Egyszerűen:

> **A subscription megmondja, hogy egy adott Receive csatorna melyik
> Transmit csatornát fogadja.**

Például:

``` text
TX:
StageBox-01
Channel 1
"Lead Vocal"

        ↓ subscription

RX:
Console-01
Channel 17
"Lead Vocal"
```

A Dante Controllerben ezt a routing mátrixban állítjuk be.

Az Audinate dokumentációja szerint a subscription egy Receive channel
hozzárendelése egy másik Dante eszköz Transmit channeljéhez.
citeturn0search3

------------------------------------------------------------------------

# 5.5 A subscription nem audioadat

Ez kezdőként fontos különbség.

``` text
Subscription
=
„Ki kitől kérjen / fogadjon?”
```

Nem maga az audió.

A tényleges audioadat ezután a létrejött flow-kon keresztül közlekedik.

Ezért:

``` text
Subscription
       ↓
Flow
       ↓
Audio packets
```

A routing konfiguráció és a tényleges médiaforgalom két külön fogalom.

------------------------------------------------------------------------

# 5.6 Mi az a flow?

A Dante flow egy hálózati médiafolyam.

Egyszerűen:

``` text
Transmitter
     │
     │ flow
     ▼
Receiver(s)
```

Egy flow több audiócsatornát is hordozhat, az adott Dante eszköz
képességeitől és konfigurációjától függően.

Az Audinate dokumentációja szerint egy tipikus unicast audio flow
kapacitása legfeljebb négy audio channel, míg a multicast flow-k
támogatott csatornaszáma eszköztípustól függően eltérhet.
citeturn0search1

> **Ne keverd össze a channel és a flow fogalmát.**

``` text
Channel
→ egy audiócsatorna

Flow
→ hálózati médiafolyam, amely több channel adatát is hordozhat
```

------------------------------------------------------------------------

# 5.7 Egy egyszerű példa

Tegyük fel:

``` text
Stage Box

TX 1  Vocal
TX 2  Guitar
TX 3  Bass
TX 4  Keys
```

A Console ezeket fogadja:

``` text
RX 17 Vocal
RX 18 Guitar
RX 19 Bass
RX 20 Keys
```

Egy megfelelő unicast flow ezeket a csatornákat együtt is szállíthatja,
a Dante eszköz és a flow konfigurációjának megfelelően.

Ezért nem feltétlenül igaz:

``` text
4 channel = 4 külön hálózati flow
```

------------------------------------------------------------------------

# 5.8 Unicast routing

A Dante alapértelmezett routingja unicast.

Ez azt jelenti, hogy egy transmitter és egy receiver között külön flow
jöhet létre.

``` text
TX
 │
 ├────→ RX 1
 │
 └────→ RX 2
```

Ha ugyanazt az audioforrást több külön vevőnek küldöd unicasttal, több
flow jöhet létre.

Az Audinate Dante Controller dokumentációja szerint a unicast routing
külön flow-t hoz létre az egyes receiving device-okhoz.
citeturn0search3

------------------------------------------------------------------------

# 5.9 Miért jó az unicast?

Az unicast előnye, hogy a hálózat csak azok felé továbbítja az
audioforgalmat, akiknek ténylegesen szükségük van rá.

Például:

``` text
Stage Box
   │
   ├────→ Console
   │
   └────→ DSP
```

Ha csak két receiver van, ez egyszerű és jól kezelhető.

------------------------------------------------------------------------

# 5.10 Fanout

Mi történik, ha ugyanazt az egy csatornát húsz receivernek akarod
küldeni?

Unicast esetben:

``` text
             ┌──→ RX1
             ├──→ RX2
TX ──────────┼──→ RX3
             ├──→ ...
             └──→ RX20
```

A transmitternek sok külön flow-t kell kezelnie.

Ezt nevezzük fanout helyzetnek.

A Dante Controller jelzi a fanout konfigurációt, ha ugyanazon
transmitter csatornáját több receiver használja. citeturn0search1

------------------------------------------------------------------------

# 5.11 Multicast routing

Multicast esetén:

``` text
             ┌──→ RX1
             ├──→ RX2
TX ──────────┼──→ RX3
             ├──→ RX4
             └──→ RX20
```

A transmitter egy multicast flow-t küld.

A hálózatban több receiver feliratkozhat ugyanarra a multicast flow-ra.

Az Audinate szerint multicast flow-k használhatók arra, hogy ugyanazt a
médiát több receiver kapja meg, miközben a transmitter oldalán nem kell
minden receiverhez külön unicast flow-t létrehozni. citeturn0search1

------------------------------------------------------------------------

# 5.12 Unicast vagy multicast?

Egyszerű szabály:

### Kevés receiver

``` text
Unicast
```

általában egyszerű.

### Sok receiver ugyanarra a tartalomra

``` text
Multicast
```

hatékonyabb lehet.

De:

> **A multicast nem automatikusan jobb.**

Multicast esetén a hálózati infrastruktúrának megfelelően kell kezelnie
a forgalmat.

Az IGMP Snooping ezért fontos lehet.

------------------------------------------------------------------------

# 5.13 Mi történik a hálózattal multicast esetén?

Nem megfelelő multicast-kezelés esetén a multicast forgalom sokkal több
hálózati linkre juthat el, mint amennyire szükség lenne.

``` text
TX
 │
 ▼
Switch
 ├──→ port 1
 ├──→ port 2
 ├──→ port 3
 ├──→ port 4
 └──→ ...
```

Ezért:

``` text
Multicast
+
IGMP Snooping
```

fontos kombináció lehet nagyobb rendszereknél.

Az Audinate szerint IGMP használata különösen vegyes hálózatokban vagy
jelentős multicast audio használatakor indokolt; Dante-only hálózatban
kevés vagy nulla multicast flow esetén nem feltétlenül követelmény.
citeturn0search18

------------------------------------------------------------------------

# 5.14 Dante Controller

A Dante Controller a Dante hálózat konfigurációjának egyik központi
eszköze.

Segítségével többek között:

-   Dante eszközöket fedezhetsz fel;
-   routingot állíthatsz be;
-   subscriptionöket kezelhetsz;
-   clock státuszt nézhetsz;
-   eszközbeállításokat kezelhetsz;
-   multicast flow-kat konfigurálhatsz;
-   hibakeresési információkat nézhetsz.

Nagyon fontos:

> **A Dante Controller nem maga a Dante audiohálózat.**

A Controller egy vezérlőeszköz.

A tényleges audioforgalom a Dante-eszközök között folyik.

------------------------------------------------------------------------

# 5.15 Dante Controller és discovery

Amikor megnyitod a Dante Controllert, az első kérdés:

> „Honnan tudja, hogy milyen Dante eszközök vannak a hálózaton?"

A discovery mechanizmusok segítségével.

A Dante hálózatban mDNS / DNS-SD alapú discovery is használatos; az
Audinate dokumentációja a Dante discovery forgalmat 224.0.0.251:5353/UDP
címmel és porttal dokumentálja. citeturn0search19

Ezért lehet olyan helyzet:

``` text
Ping működik
        ↓
de Dante Controller nem látja az eszközt
```

Ez nem feltétlenül ellentmondás.

------------------------------------------------------------------------

# 5.16 Dante Controller nem „audio router"

A Controllerben látod:

``` text
TX → RX
```

és kiválasztod:

``` text
subscription
```

De a Controller nem úgy működik, mint egy digitális audio console, amely
minden audio packetet magán keresztül továbbít.

A logika inkább:

``` text
Controller
    │
    │ configuration
    ▼
Dante Devices
    │
    │ actual audio flow
    ▼
Network
```

------------------------------------------------------------------------

# 5.17 Clocking -- miért kell közös idő?

Most érkezünk a Dante architektúra egyik legfontosabb részéhez.

Képzeld el, hogy két eszköz van:

``` text
Stage Box
Clock A

Console
Clock B
```

Ha a két óra nem azonos ütemben jár, idővel eltérés keletkezik.

Digitális audióban ez komoly problémát okozhat.

Ezért a Dante hálózatnak közös időalapra van szüksége.

------------------------------------------------------------------------

# 5.18 PTP

A Dante Precision Time Protocolt használ a hálózati időzítéshez.

Egyszerűsítve:

``` text
Clock Leader
      │
      ├────→ Device 1
      ├────→ Device 2
      ├────→ Device 3
      └────→ Device 4
```

A Dante dokumentációja szerint egy Dante eszköz kerülhet clock leader
szerepbe egy broadcast domainben, a többi eszköz pedig followerként
szinkronizálhat hozzá. citeturn0search20

------------------------------------------------------------------------

# 5.19 Clock Leader és Follower

A legegyszerűbb modell:

``` text
        Clock Leader
             │
      ┌──────┼──────┐
      ▼      ▼      ▼
     RX1    RX2    RX3
```

A leader adja a hálózati időreferenciát.

A followerek ehhez igazítják saját működésüket.

A Dante Controllerben a clock státusz ellenőrizhető. citeturn0search5

------------------------------------------------------------------------

# 5.20 Mi történik, ha a clock rossz?

A rendszer audiohibákat produkálhat:

-   dropouts;
-   kattogás;
-   instabil működés;
-   synchronization problémák.

Ezért egy Dante hibakeresésnél:

``` text
Audio nincs
```

esetén nem csak az audio subscriptiont nézzük.

Meg kell nézni:

``` text
Clock
```

is.

------------------------------------------------------------------------

# 5.21 Clock és audio -- két külön dolog

Nagyon fontos:

``` text
Audio flow
≠
Clock flow
```

A hangadat és a szinkronizációs információ külön funkció.

Mindkettő hálózaton megy.

Ezért:

``` text
Audio működik?
Clock működik?
```

két külön kérdés.

------------------------------------------------------------------------

# 5.22 Latency

A Dante egyik fontos paramétere a latency.

A latency azt az időt jelenti, amely alatt a rendszer a hálózaton
keresztül továbbítja és feldolgozza az audioadatot.

Egyszerűsített modell:

``` text
TX
 │
 │ network
 ▼
RX
```

A teljes rendszer késleltetését több tényező befolyásolja:

-   Dante latency beállítás;
-   hálózati hopok;
-   switch-ek;
-   eszközfeldolgozás;
-   egyéb rendszerkomponensek.

------------------------------------------------------------------------

# 5.23 Miért nem mindig a legalacsonyabb latency a legjobb?

Mert a kisebb latency kevesebb időt hagy a hálózati jitter és egyéb
változások kezelésére.

Egyszerűen:

``` text
Alacsony latency
→ kisebb tartalék

Magasabb latency
→ nagyobb tartalék
```

Ezért a latencyt nem „minél kisebb, annál jobb" szabályként kell
kezelni.

A megfelelő értéket a topológia és a rendszer követelményei alapján kell
kiválasztani.

------------------------------------------------------------------------

# 5.24 A Dante hálózati architektúra

Most rakjuk össze:

``` text
                 Dante Network
                      │
       ┌──────────────┼──────────────┐
       │              │              │
     Audio           Clock        Control
       │              │              │
       ▼              ▼              ▼
   UDP flows          PTP        Discovery /
                                 Routing
       │              │              │
       └──────────────┼──────────────┘
                      │
                  Ethernet
                      │
                    Switch
```

Ez a fejezet kulcsábrája.

------------------------------------------------------------------------

# 5.25 Egy teljes Dante-rendszer

Példa:

``` text
                  Aruba Switch
                /      |       \
               /       |        \
              /        |         \
       Stage Box     Console      DSP
          TX            RX        RX
           \             |        /
            \            |       /
             └──────── Dante ────┘
```

És a háttérben:

``` text
Audio
Clock
Discovery
Control
```

------------------------------------------------------------------------

# 5.26 Primary és Secondary

Dante-eszközökön találkozhatsz:

``` text
Primary
Secondary
```

interfészekkel.

A Secondary interfész redundáns vagy második hálózati kapcsolatot
biztosító architektúrák része lehet.

Fontos:

> **A Secondary nem egyszerűen „második switchport".**

A tényleges működés az adott Dante eszköz redundancia módjától és
konfigurációjától függ.

------------------------------------------------------------------------

# 5.27 Dante Redundancy

Redundáns Dante-rendszerben:

``` text
             Stage Box
             /       \
            /         \
     Primary         Secondary
        │                 │
        ▼                 ▼
   Switch A           Switch B
        │                 │
        └───────┬─────────┘
                ▼
             Console
```

A cél az, hogy egy hálózati infrastruktúra- vagy linkhiba ne okozzon
audio-kiesést.

A redundáns Dante hálózat megtervezésénél fontos, hogy a két hálózati
útvonal megfelelően legyen elkülönítve.

------------------------------------------------------------------------

# 5.28 Switched és Redundant mód

Nem minden Dante-eszköz ugyanazt a redundancia-funkciót támogatja.

Egyes eszközök több hálózati interfésszel rendelkeznek, mások nem.

Ezért:

> **Mindig az adott Dante eszköz dokumentációját kell ellenőrizni.**

A könyvben a „Dante-eszköz" nem jelent automatikusan kétportos redundáns
eszközt.

------------------------------------------------------------------------

# 5.29 Dante Domain Manager -- miért van rá szükség?

Egy kis rendszerben lehet:

``` text
10 Dante devices
1 PC
1 Dante Controller
```

Egy nagyobb vállalati vagy több helyszínes rendszerben viszont szükség
lehet:

-   központi felhasználókezelésre;
-   hálózati szegmentációra;
-   biztonsági szabályokra;
-   központi menedzsmentre;
-   több subnet közötti Dante-routingra.

Itt jelenik meg a Dante Domain Manager, vagy DDM.

Az Audinate a DDM-et olyan menedzsment- és hálózati képességekkel írja
le, amelyek nagyobb, IT-környezetbe integrált Dante rendszerek kezelését
teszik lehetővé. citeturn0search19turn0search4

------------------------------------------------------------------------

# 5.30 Dante Domain

A DDM-ben a Dante eszközök domainekbe szervezhetők.

Egyszerű modell:

``` text
DDM
 │
 ├── Domain A
 │     ├── Stage Box
 │     ├── Console
 │     └── DSP
 │
 └── Domain B
       ├── Studio
       └── OB Truck
```

Ez már nem egy egyszerű „kis Dante hálózat" modell.

------------------------------------------------------------------------

# 5.31 Dante és több subnet

Kezdő Dante-rendszerben célszerű úgy gondolkodni:

``` text
Dante devices
      ↓
same IP subnet
```

Ez egyszerű.

A fejlettebb Dante/Domain Manager architektúrákban azonban a Dante
hálózat több IP subneten keresztül is kezelhető.

A DDM dokumentációja külön kezeli a routed hálózatokat és a domain
clocking szerepeit; modern eszközök esetén a domain leader/follower
szerepek több subnetet is átfoghatnak. citeturn0search5turn0search19

Ezért:

> **„Dante nem működik routeren keresztül" általános állításként nem
> helyes.**

A helyes állítás:

> **A Dante hálózati architektúrája és támogatott routed működése az
> adott Dante generációtól, firmware-től, DDM használatától és
> konfigurációtól függ.**

------------------------------------------------------------------------

# 5.32 Dante Domain Manager és biztonság

Egy nagyobb Dante rendszerben nem szeretnénk, hogy bárki korlátlanul
módosíthassa a routingot.

A DDM lehetőséget ad központi menedzsmentre és jogosultságkezelésre.

Ez már egy fontos szemléletváltás:

``` text
Kis rendszer:
„Látom az eszközöket → routingolok”

Nagy rendszer:
„Azonosítás → jogosultság → domain → routing”
```

------------------------------------------------------------------------

# 5.33 AES67 és Dante

A Dante nem feltétlenül csak Dante-eszközökkel létezik.

Bizonyos Dante-eszközök támogatják az AES67 interoperabilitást.

Fontos:

> **A Dante és az AES67 nem ugyanaz a protokoll.**

Az Audinate dokumentációja szerint AES67 módban a Dante-eszközök RTP
multicast audiofolyamokkal tudnak együttműködni nem-Dante AES67
eszközökkel. Ugyanakkor két Dante-eszköz között AES67 mód mellett is a
natív Dante audio transport használatos.
citeturn0search0turn0search6

------------------------------------------------------------------------

# 5.34 SMPTE ST 2110

Bizonyos modern Dante-eszközök SMPTE ST 2110-30 audio interoperabilitást
is támogathatnak.

Ez már broadcast környezethez közelít.

Egyszerű modell:

``` text
Dante
  │
  ├── Dante native audio
  │
  ├── AES67
  │
  └── SMPTE ST 2110-30
```

De:

> **Nem minden Dante-eszköz támogat minden interoperabilitási módot.**

Mindig az adott eszköz és firmware képességeit kell ellenőrizni.
citeturn0search0

------------------------------------------------------------------------

# 5.35 Egy Dante-rendszer három nagy síkja

A fejezet végére használjuk ezt a modellt:

``` text
┌───────────────────────────────┐
│           CONTROL             │
│ Discovery / Controller / DDM  │
└───────────────┬───────────────┘
                │
┌───────────────▼───────────────┐
│            CLOCK              │
│             PTP               │
└───────────────┬───────────────┘
                │
┌───────────────▼───────────────┐
│             AUDIO             │
│       Dante media flows       │
└───────────────┬───────────────┘
                │
┌───────────────▼───────────────┐
│           NETWORK             │
│ Ethernet / IP / Switch / QoS │
└───────────────────────────────┘
```

Ez nagyon hasznos hibakeresési modell.

------------------------------------------------------------------------

# 5.36 Hibakeresési példa

A Console látszik a Dante Controllerben.

De nincs audio.

Mit tudunk?

``` text
Discovery működik
```

De még nem tudjuk:

``` text
Subscription?
Flow?
Clock?
Network?
Audio?
```

Ellenőrzési sorrend:

``` text
Device visible
       ↓
Subscription
       ↓
Flow
       ↓
Clock
       ↓
Network counters
       ↓
Audio
```

------------------------------------------------------------------------

# 5.37 Másik hibakeresési példa

A Console egyáltalán nem látszik.

Első gondolat:

``` text
„Dante hiba.”
```

Helyesebb:

``` text
Link?
 ↓
VLAN?
 ↓
IP?
 ↓
Discovery?
 ↓
mDNS?
 ↓
Dante Controller interface?
```

Ezért a jó Dante szakember **rétegekben gondolkodik**.

------------------------------------------------------------------------

# 5.38 Egy Dante-rendszer felépítése lépésről lépésre

## 1. Fizikai hálózat

``` text
Cat kábel
Switch
NIC
```

## 2. Ethernet

``` text
MAC
VLAN
Switch
```

## 3. IP

``` text
IP
Subnet
ARP
Gateway
```

## 4. Dante discovery/control

``` text
Dante Controller
Discovery
```

## 5. Clock

``` text
PTP
Leader
Follower
```

## 6. Routing

``` text
TX
 ↓
Subscription
 ↓
RX
```

## 7. Media

``` text
Flow
 ↓
Audio
```

Ez a könyv eddigi anyagának egyik legfontosabb összefoglalása.

------------------------------------------------------------------------

# 5.39 Gyakorlati labor -- első Dante-rendszer

Topológia:

``` text
                 Aruba Switch
              /       |       \
             /        |        \
      Stage Box    Console      DSP
```

IP:

``` text
Stage Box:
192.168.50.10

Console:
192.168.50.20

DSP:
192.168.50.30

PC:
192.168.50.100
```

## Feladat

1.  Ellenőrizd a linkeket.
2.  Ellenőrizd az IP-címeket.
3.  Ellenőrizd a subnetet.
4.  Ellenőrizd a VLAN-t.
5.  Indítsd el a Dante Controllert.
6.  Ellenőrizd, hogy minden eszköz látható.
7.  Ellenőrizd a clock státuszt.
8.  Hozz létre egy subscriptiont.
9.  Ellenőrizd az audio flow-t.
10. Dokumentáld az eredményt.

------------------------------------------------------------------------

# 5.40 Labor -- unicast

Állítsd be:

``` text
Stage Box TX 1
        ↓
Console RX 1
```

Ellenőrizd:

``` text
Subscription:
OK

Clock:
Locked / megfelelő állapot

Audio:
OK
```

Ezután hozz létre még egy receivert:

``` text
Stage Box TX 1
   ├──→ Console RX 1
   └──→ DSP RX 1
```

Figyeld meg a fanout helyzetet.

------------------------------------------------------------------------

# 5.41 Labor -- multicast

Hozz létre multicast flow-t egy olyan audioforrásból, amelyet több
receivernek kell megkapnia.

``` text
Stage Box
    │
    │ multicast
    ▼
Aruba Switch
 ├──→ Console
 ├──→ DSP
 └──→ Recorder
```

Ellenőrizd:

-   multicast flow létrejött-e;
-   a receiverek kapják-e;
-   switch oldalon megfelelő-e az IGMP kezelés;
-   nincs-e szükségtelen multicast flooding.

Az Audinate dokumentációja szerint a multicast flow-k használata több
receiver esetén hatékony lehet, de a multicast forgalom megfelelő
hálózati kezelést igényel. citeturn0search1turn0search18

------------------------------------------------------------------------

# 5.42 Labor -- clock hibakeresés

Szándékosan hozz létre egy olyan helyzetet, amelyben a clock állapot
problémássá válik, majd figyeld meg:

``` text
Dante Controller
      ↓
Clock Status
```

Jegyezd fel:

``` text
Leader:
?

Follower:
?

Latency:
?

Audio:
?
```

A cél nem a rendszer „elrontása", hanem annak megértése, hogy a clock
státusz külön vizsgálati pont.

------------------------------------------------------------------------

# 5.43 Aruba switch -- Dante ellenőrzési lista

Egy Aruba switch konfigurációjának vizsgálatakor legalább ezeket
ellenőrizd:

### Port

``` text
Link up?
Speed?
Duplex?
Errors?
```

### VLAN

``` text
Correct VLAN?
Access / trunk?
```

### QoS

``` text
DSCP preserved?
Priority queues?
```

### Multicast

``` text
IGMP Snooping?
Querier?
Correct VLAN?
```

### EEE

``` text
Disabled on Dante ports?
```

### Redundancy

``` text
Primary / Secondary
physically and logically separated?
```

A konkrét Aruba modellhez tartozó parancsokat mindig az adott
ArubaOS/Aruba CX dokumentáció alapján kell kiválasztani.

------------------------------------------------------------------------

# 5.44 Deep Dive -- subscription → flow

A routing logikája:

``` text
TX Channel
     │
     │ subscription
     ▼
RX Channel
```

A subscription létrehozásakor a Dante rendszer kialakítja a szükséges
médiafolyamot.

Unicast esetben:

``` text
TX
 │
 └────→ RX
```

Több receiver esetén:

``` text
TX
 ├────→ RX1
 ├────→ RX2
 └────→ RX3
```

Multicast esetben:

``` text
TX
 │
 ▼
Multicast Flow
 ├────→ RX1
 ├────→ RX2
 └────→ RX3
```

A különbség nem pusztán „egy cím több gépnek".

A flow-allokáció és a hálózati forwarding viselkedése is eltér.

------------------------------------------------------------------------

# 5.45 Deep Dive -- miért nem ugyanaz a multicast és a broadcast?

Broadcast:

``` text
„Mindenki ugyanabban a broadcast domainben.”
```

Multicast:

``` text
„Azok kapják, akik a csoport forgalmára jogosultak / feliratkoztak.”
```

IGMP Snooping segítségével a switch a multicast tagság alapján
hatékonyabban tudja kezelni a forgalmat.

Ezért:

``` text
Broadcast ≠ Multicast
```

------------------------------------------------------------------------

# 5.46 Deep Dive -- clock leader választás

A Dante hálózatban a clock leader szerep automatikus választási
mechanizmus része.

A Dante Controllerben láthatod:

``` text
Leader
Follower
```

A konkrét eszközök clock-prioritása és konfigurációja befolyásolhatja a
választást.

A részletes PTP election mechanizmust külön fejezetben fogjuk
feldolgozni.

Most a lényeg:

> **A Dante hálózatban nem kell minden eszközt kézzel „master clocknak"
> kijelölni.**

------------------------------------------------------------------------

# 5.47 Deep Dive -- miért fontos a latency?

A Dante latency beállítása egy pufferelési időt is jelent.

Túl alacsony:

``` text
kevés jitter-tartalék
```

Túl magas:

``` text
nagyobb rendszerkésleltetés
```

Ezért a megfelelő latency kompromisszum:

``` text
Network stability
        +
System scale
        +
Required responsiveness
```

alapján választandó.

------------------------------------------------------------------------

# 5.48 Deep Dive -- Dante nem „hangot küld a switchen"

Ez egy nagyon fontos nyelvi különbség.

Pontatlan:

> „A switch átviszi a Dante hangot."

Pontosabb:

> **A Dante-eszközök hálózati csomagokban továbbítják a médiaadatot, az
> Ethernet switch pedig az adott Layer 2/Layer 3 infrastruktúra
> szabályai szerint továbbítja a hálózati forgalmat.**

Ez azért fontos, mert hibakeresésnél más kérdéseket teszünk fel:

Nem:

> „Miért rossz a Dante hang?"

Hanem:

``` text
Van link?
Van Ethernet?
Van IP?
Van discovery?
Van clock?
Van subscription?
Van flow?
Van audio?
```

------------------------------------------------------------------------

# 5.49 Deep Dive -- Dante mint elosztott rendszer

Egy nagy Dante-rendszerben nincs egyetlen központi „Dante gép", amely
minden audioadatot kezel.

``` text
Stage Box
   ↕
Console
   ↕
DSP
   ↕
Recorder
```

Az eszközök egymással kommunikálnak a hálózaton.

A Dante Controller és adott esetben a DDM a konfiguráció és menedzsment
oldalát támogatja.

Ez az elosztott architektúra teszi lehetővé, hogy a rendszer modulárisan
bővíthető legyen.

------------------------------------------------------------------------

# 5.50 Channel és Flow -- kézzelfogható példa

A `channel` és a `flow` közötti különbséget érdemes egy konkrét példán
megérteni.

Tegyük fel, hogy a Stage Box négy csatornát küld:

``` text
Stage Box

TX 1 = Vocal
TX 2 = Guitar
TX 3 = Bass
TX 4 = Keys
```

A hálózaton ezek egy médiafolyam részeként is továbbíthatók:

``` text
              FLOW
        ┌──────────────┐
        │ Vocal        │
        │ Guitar       │
        │ Bass         │
        │ Keys         │
        └──────────────┘
```

A lényeg:

``` text
Channel
→ egy audiócsatorna

Flow
→ hálózati médiafolyam, amely több csatorna adatát is hordozhat
```

Ezért ne így gondolkodj:

``` text
4 channel = 4 flow
```

hanem:

``` text
több channel
     ↓
egy vagy több flow
     ↓
network
```

A tényleges flow-kapacitás és az egy flow-ban szállítható csatornák
száma az adott Dante implementációtól és eszköztől függ.

> **Tanulási szabály:** először a fogalmi különbséget jegyezd meg; a
> konkrét flow-kapacitásokat csak akkor kell megtanulnod, amikor egy
> adott Dante eszköz vagy rendszer tervezésével foglalkozol.

------------------------------------------------------------------------

# 5.51 Miért kell egyáltalán clock?

A PTP megértése előtt először azt kell megérteni, **miért van szükség
szinkronizált időalapra**.

Képzeljünk el két digitális audióeszközt:

``` text
Eszköz A
48 000 sample/sec

Eszköz B
48 001 sample/sec
```

Első pillantásra ez szinte ugyanaz.

De a két eszköz órája nem pontosan ugyanazzal az ütemmel jár.

Ha az egyik eszköz küld, a másik pedig fogad, hosszabb idő alatt az
eltérés felhalmozódhat.

Ez digitális audiórendszerben problémát okozhat.

Ezért szükséges egy közös, pontos időalap:

``` text
          Közös időalap
                │
        ┌───────┼───────┐
        ▼       ▼       ▼
       TX      RX      DSP
```

A Dante ezt hálózati időzítéssel, PTP-alapú clockinggal oldja meg.

## Egyszerű mentális modell

``` text
PTP
 ↓
közös időalap
 ↓
szinkronizált Dante eszközök
 ↓
stabil digitális audiórendszer
```

A részletes PTP működést későbbi fejezetben fogjuk megtanulni.

> **Most még nem kell PTP-szakértőnek lenned.** Azt kell értened, hogy
> az audióadat továbbítása és az eszközök időalapjának szinkronizálása
> két külön, de egymással összefüggő feladat.

------------------------------------------------------------------------

# 5.52 Unicast, multicast és IGMP -- egyetlen ábrán

Ezt a három fogalmat együtt érdemes megtanulni.

## Unicast

Egy transmitter egy receivernek küld:

``` text
TX ─────────────→ RX1
```

Ha ugyanazt az audioforrást három receivernek kell elküldeni:

``` text
              ┌──→ RX1
TX ───────────┼──→ RX2
              └──→ RX3
```

A Dante unicast routingja esetén több külön flow jöhet létre.

------------------------------------------------------------------------

## Multicast

Multicastnál egy multicast flow-t több receiver is fogadhat:

``` text
              ┌──→ RX1
              │
TX ───────────┼──→ RX2
              │
              └──→ RX3
```

A lényeg itt nem pusztán az, hogy „három eszköz kapja".

Hanem az, hogy:

``` text
TX
 ↓
multicast group / flow
 ↓
az érdeklődő receiverek
```

------------------------------------------------------------------------

## IGMP

A switchnek tudnia kell, hogy mely portokon vannak olyan eszközök,
amelyek az adott multicast forgalmat kérik.

Egyszerűsített modell:

``` text
Receiver
   │
   │ „ezt a multicastot kérem”
   ▼
 IGMP
   │
   ▼
 Switch
   │
   ├────→ érdeklődő port
   │
   ├────→ érdeklődő port
   │
   └─X──→ nem érdeklődő port
```

Ez az oka annak, hogy nagyobb multicast Dante-rendszerekben az IGMP és
az IGMP Snooping fontos.

> **Ne úgy jegyezd meg, hogy „Dante = multicast = IGMP".**
>
> A helyes modell:
>
> ``` text
> Dante multicast
>       ↓
> multicast network traffic
>       ↓
> megfelelő multicast-kezelés
>       ↓
> szükség esetén IGMP / IGMP Snooping
> ```

A szükséges multicast-kezelés a rendszer méretétől,
multicast-forgalmától és hálózati topológiájától függ.

------------------------------------------------------------------------

# 5.53 Haladó kitekintés -- DDM, AES67 és SMPTE ST 2110

Most már látjuk a Dante alaparchitektúráját.

Ezek a fogalmak azonban **nem az első napi Dante-tudás részei**.

## Dante Domain Manager

``` text
Kis rendszer

Dante Controller
      ↓
Dante devices
```

Nagyobb, menedzselt környezetben:

``` text
Dante Domain Manager
          ↓
   Domains / Users
          ↓
    Dante devices
```

A DDM olyan rendszerekben válik különösen érdekessé, ahol több hálózati
zóna, több felhasználó, jogosultságkezelés és központi menedzsment
szükséges.

**Ezt most még nem kell konfigurálnod.**

------------------------------------------------------------------------

## AES67

Az AES67 interoperabilitás lehetővé teheti, hogy megfelelő
Dante-eszközök más, AES67-kompatibilis rendszerekkel is
együttműködjenek.

``` text
Dante
   │
   │ interoperabilitás
   ▼
AES67
   │
   ▼
más IP audio rendszer
```

De:

``` text
Dante ≠ AES67
```

A két technológia nem ugyanaz.

------------------------------------------------------------------------

## SMPTE ST 2110

Broadcast környezetben további interoperabilitási követelmények
jelenhetnek meg:

``` text
Dante
   │
   └── ST 2110-30
```

Ez már haladó terület.

> **A könyvben azért találkozol ezekkel a nevekkel, hogy amikor később
> egy broadcast vagy nagyvállalati AV-rendszer dokumentációjában látod
> őket, ne legyenek ismeretlenek.**

A konkrét támogatás mindig az adott Dante eszköz, firmware és
rendszerarchitektúra képességeitől függ.

------------------------------------------------------------------------

# 5.54 Összefoglalás

A Dante architektúrájának megértéséhez ezt a modellt tartsd fejben:

``` text
                 DANTE
                   │
        ┌──────────┼──────────┐
        │          │          │
      AUDIO      CLOCK      CONTROL
        │          │          │
      Flow        PTP      Discovery
        │          │       Controller
        │          │          │
        └──────────┼──────────┘
                   │
                NETWORK
                   │
            Ethernet / IP
                   │
                 Switch
```

A legfontosabb fogalmak:

### Dante Device

A hálózaton részt vevő Dante-képes végpont.

### TX

Transmit -- küldő csatorna.

### RX

Receive -- fogadó csatorna.

### Subscription

Egy RX channel hozzárendelése egy TX channelhez.

### Flow

A hálózaton továbbított médiafolyam.

### Unicast

Egy transmitter → egy receiver flow.

### Multicast

Egy transmitter → több receiver által használható flow.

### Fanout

Egy transmitter tartalmának több receiverhez történő továbbítása, amely
unicast esetén több flow-t igényelhet.

### Dante Controller

Dante hálózat konfigurációs és felügyeleti eszköze.

### PTP / Clock

A Dante hálózat időzítési és szinkronizációs alapja.

### DDM

Dante Domain Manager -- nagyobb, menedzselt Dante hálózatok központi
menedzsmentjének eszköze.

------------------------------------------------------------------------

# 5.55 A teljes Dante mentális modell

``` text
┌─────────────────────────────────────┐
│              USER                   │
│       „Vocal → Console”             │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│          DANTE CONTROLLER            │
│          Subscription                │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│          DANTE DEVICES               │
│                                     │
│       TX ───────→ RX                │
│          \        /                 │
│           \ Flow /                  │
└──────────────────┬──────────────────┘
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
     CLOCK                  NETWORK
      PTP              Ethernet / IP
        │                     │
        └──────────┬──────────┘
                   ▼
               SWITCH
                   │
                   ▼
              AUDIO DATA
```

Ha ezt az ábrát érted, akkor **érted a Dante architektúra alapját**.

------------------------------------------------------------------------

# 5.56 Ellenőrző kérdések

## Alap

1.  Mi a Dante?
2.  Mi a különbség a Dante és az Ethernet között?
3.  Mi a TX?
4.  Mi az RX?
5.  Mi a subscription?
6.  Mi a flow?
7.  Mi a különbség a channel és a flow között?
8.  Mi az unicast?
9.  Mi a multicast?
10. Mi a fanout?
11. Mi a Dante Controller?
12. Mi a PTP?
13. Mi a clock leader?
14. Mi a clock follower?
15. Mi a latency?

## Dante-rendszer

16. Mi történik egy subscription létrehozásakor?
17. Miért hozhat létre több flow-t a fanout?
18. Mikor lehet célszerű multicastot használni?
19. Miért fontos az IGMP?
20. Miért nem bizonyítja a ping, hogy a Dante-rendszer működik?
21. Miért kell a clock státuszt ellenőrizni?
22. Mi a különbség a Controller és az audiohálózat között?
23. Mire való a Primary és Secondary interfész?
24. Miért lehet szükség DDM-re?
25. Mi a különbség Dante native audio és AES67 interoperabilitás között?

------------------------------------------------------------------------

# 5.57 Vizsgafeladat -- egy audioforrás eljuttatása több helyre

Adott:

``` text
Stage Box
TX 1 = Vocal
```

A cél:

``` text
Console RX 1
DSP RX 1
Recorder RX 1
```

### Kérdések

1.  Mi történik, ha mindhárom receiver unicastot használ?
2.  Mi a fanout?
3.  Mikor lehet multicast jobb választás?
4.  Mit kell ellenőrizni a switchen?
5.  Miért fontos az IGMP?
6.  Mit kell ellenőrizni a Dante Controllerben?
7.  Mit kell ellenőrizni a clock státusznál?
8.  Mi történik, ha a subscription OK, de nincs audio?

### Elvárt gondolkodás

``` text
TX
 ↓
Routing
 ↓
Subscription
 ↓
Flow
 ↓
Network
 ↓
Clock
 ↓
RX
```

------------------------------------------------------------------------

# 5.58 Laborprojekt -- Dante-rendszer dokumentálása

Készíts dokumentációt az alábbi rendszerről:

``` text
                 Aruba Switch
              /       |       \
             /        |        \
       Stage Box    Console      DSP
```

Dokumentáld:

  Eszköz      IP   MAC   TX   RX   Switch   Port
  ----------- ---- ----- ---- ---- -------- ------
  Stage Box                        Aruba    
  Console                          Aruba    
  DSP                              Aruba    

Ezután dokumentáld:

``` text
Clock Leader:
__________

Latency:
__________

Subscriptions:
__________

Multicast:
__________

VLAN:
__________

IGMP:
__________
```

A cél nem a papírmunka.

A cél az, hogy egy hibakereső technikus **öt perc alatt megértse a
rendszer topológiáját**.

------------------------------------------------------------------------

# 5.59 Laborprojekt -- hibakeresési szimuláció

Szándékosan hozz létre egy hibát.

### Hiba 1

``` text
Device látszik
Subscription:
OK
Audio:
NINCS
```

Mit vizsgálsz?

``` text
Clock
Flow
Network
RX state
```

### Hiba 2

``` text
Device nem látszik
```

Mit vizsgálsz?

``` text
Link
IP
VLAN
Discovery
mDNS
Controller interface
```

### Hiba 3

``` text
Audio működik
De időnként dropouts
```

Mit vizsgálsz?

``` text
Link errors
Bandwidth
QoS
EEE
Multicast
Clock
Latency
```

Ez már a valódi rendszerintegrátori gondolkodás.

------------------------------------------------------------------------

# 5.60 Források és műszaki ellenőrzés

A Dante-specifikus állításokat elsődlegesen Audinate dokumentációval
ellenőriztük.

Különösen:

-   Dante routing és subscription;
-   unicast és multicast flow-k;
-   fanout;
-   Dante Controller;
-   discovery;
-   PTP és clock status;
-   multicast és IGMP;
-   DDM;
-   AES67 és SMPTE ST 2110 interoperabilitás.

Az Audinate aktuális dokumentációja szerint a Dante routing
alapértelmezésben unicast, multicast flow-k pedig több receiver
kiszolgálására használhatók. citeturn0search3turn0search1

Az Audinate hálózati adminisztrátori dokumentációja szerint az IGMP
különösen jelentős multicast-forgalom vagy vegyes hálózat esetén fontos,
és Dante-eszközök PTP-t használnak a hálózati időzítéshez.
citeturn0search18turn0search20

A DDM és az interoperabilitási funkciók esetében mindig az adott Dante
firmware, eszköztípus és aktuális Audinate dokumentáció az irányadó.
citeturn0search19turn0search0

------------------------------------------------------------------------

# 5.61 Fejezeti állapot

**Állapot: COMPLETE -- pedagógiai átdolgozott változat**

A fejezet tartalmaz:

-   Dante architektúra alapmodell;
-   Dante eszközök;
-   TX/RX;
-   subscription;
-   flow;
-   channel vs flow;
-   unicast;
-   multicast;
-   fanout;
-   Dante Controller;
-   discovery;
-   PTP;
-   clock leader/follower;
-   latency;
-   Primary/Secondary;
-   redundancy;
-   Dante Domain Manager;
-   routed Dante alapok;
-   AES67;
-   SMPTE ST 2110;
-   Dante hibakeresési modellek;
-   Aruba switch ellenőrzési lista;
-   unicast labor;
-   multicast labor;
-   clock labor;
-   hibakeresési szimuláció;
-   vizsgafeladatok;
-   műszaki ellenőrzési megjegyzések.
