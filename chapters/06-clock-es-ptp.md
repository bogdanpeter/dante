---
chapter: 6
chapter_title: Clock és PTP
status: complete
title: DANTE -- A professzionális Audio over IP rendszerek kézikönyve
version: 1.0
---

# Clock és PTP

> **A fejezet célja:** megérteni, miért kell közös időalap egy
> Dante-rendszerben, hogyan működik a Dante clock modellje, mit jelent a
> PTP, hogyan választódik ki a Leader Clock, hogyan működik a Follower,
> és hogyan lehet a clock problémákat a Dante Controllerben felismerni
> és hibakeresni.

## Előismeretek

Ehhez a fejezethez már érdemes érteni: - digitális audió és sample
rate; - PCM; - Ethernet; - IP és UDP; - Dante TX / RX / subscription /
flow; - switch, VLAN és QoS.

## Hogyan tanuld ezt a fejezetet?

### 🟢 1. szint -- ezt mindenképpen értsd

``` text
Digitális audió
      ↓
közös időalap
      ↓
PTP
      ↓
Leader Clock
      ↓
Follower Clocks
```

### 🟡 2. szint -- rendszerintegrátori tudás

-   clock election;
-   Preferred Leader;
-   külső Word Clock;
-   PTPv1 és PTPv2;
-   Primary / Secondary clocking;
-   clock domain;
-   frequency offset;
-   clock monitoring;
-   hálózati hibák hatása a clockra.

### 🔴 3. szint -- Deep Dive

-   PTP üzenetek;
-   Sync / Follow Up;
-   Delay Request / Delay Response;
-   offset és path delay;
-   boundary clock;
-   routed DDM clocking;
-   AES67 / SMPTE PTPv2.

> **Első olvasáskor nem a PTP packetmezőket kell megjegyezned. Azt kell
> megértened, hogyan lesz több külön Dante-eszközből egy közös időalapon
> működő digitális audiórendszer.**

------------------------------------------------------------------------

# 6.1 Miért kell egyáltalán clock?

Képzeljünk el két digitális audióeszközt:

``` text
Eszköz A
48 000 sample/sec

Eszköz B
48 001 sample/sec
```

A példa nem azt jelenti, hogy az egyik eszköz konfiguráció szerint
48 kHz-en, a másik pedig 48,001 kHz-en működik. Azt szemlélteti, hogy
két, névlegesen azonos sample rate-re beállított fizikai clock között is
lehet kis frekvenciaeltérés. Ha ezt az eltérést nem kezeljük, az idővel
felhalmozódhat.

Ezért a Dante-eszközöknek nem csak adatot kell cserélniük: **az
időalapjukat is össze kell hangolniuk.**

Az Audinate szerint a Dante-eszközök IEEE 1588 Precision Time Protocolt
használnak arra, hogy helyi óráikat a hálózati leader clockhoz
szinkronizálják. citeturn0search1

------------------------------------------------------------------------

# 6.2 Sample rate és clock

``` text
48 kHz
=
48 000 sample / second
```

Nem elég, hogy két eszköz kijelzőjén ugyanaz a sample rate szerepel. A
helyi clockoknak is megfelelően kell járniuk.

``` text
Sample rate
    +
Clock stability
```

A PTP a hálózaton keresztül segít az eszközök időalapjának
összehangolásában.

------------------------------------------------------------------------

# 6.3 Mi történik közös clock nélkül?

``` text
Stage Box ─────→ Console
  Clock A          Clock B
```

Ha a két óra eltér, a két rendszer idővel eltávolodik egymástól.

A helyes modell:

``` text
        Leader Clock
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
      A      B      C
```

------------------------------------------------------------------------

# 6.4 A Dante clock modellje

Standard Dante hálózatban egy Dante-eszköz lesz a PTP Leader Clock. A
többi eszköz Follower Clockként működik.

``` text
                 Leader
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Follower   Follower   Follower
```

Az Audinate szerint sok Dante-eszköz képes lehet leader szerepre, de egy
clock domainben egy eszköz nyeri meg a választást. citeturn0search1

------------------------------------------------------------------------

# 6.5 Leader és Follower

A Leader biztosítja a hálózati időreferenciát. A Follower saját helyi
óráját ehhez igazítja.

Nem arról van szó, hogy a follower egyszerűen „lemásolja" a leader
óráját: a saját clockja továbbra is létezik, de folyamatosan
korrekciókat alkalmaz.

------------------------------------------------------------------------

# 6.6 Hogyan választódik ki a Leader?

A Dante clock leader választása automatikus.

Az Audinate dokumentációja szerint a választásnál többek között szerepet
játszhat: - külső clock inputtal rendelkező eszköz; - gigabites
kapcsolat; - MAC-cím alapú döntés; - Preferred Leader beállítás.
citeturn0search1

> **Nem véletlenszerűen választódik ki a Leader.**

------------------------------------------------------------------------

# 6.7 Preferred Leader

A Dante Controllerben egy eszközön beállítható:

``` text
Preferred Leader
```

Ez elsőbbséget ad az adott eszköznek a leader election során.

``` text
Console
Preferred Leader = ON

Stage Box
Preferred Leader = OFF

DSP
Preferred Leader = OFF
```

Az Audinate szerint több Preferred Leader esetén is automatikus
választás történik. citeturn0search1turn0search2

> **A Preferred Leader tervezési döntés, nem kötelező beállítás.**

------------------------------------------------------------------------

# 6.8 Külső Word Clock

Támogatott eszköz külső referenciaórát is használhat:

``` text
House Clock
    │
    ▼
Dante Device
    │
    ▼
Dante Network
```

Az `Enable Sync To External` beállítás külső Word Clockhoz igazíthatja
az eszköz clockját. Ez a leader-választásra is hatással lehet.
citeturn0search1turn0search2

------------------------------------------------------------------------

# 6.9 A külső clocknak pontosnak kell lennie

A külső clock nem automatikusan jobb.

Az Audinate Clock Status Monitor dokumentációja szerint pontatlan külső
Word Clockból származó leader clock instabilitást vagy túl nagy
frequency offsetet okozhat. citeturn0search0

------------------------------------------------------------------------

# 6.10 Mi az a PTP?

**PTP = Precision Time Protocol.**

A Dante ennek segítségével hangolja össze az eszközök clockját.

``` text
Leader
  │
  │ PTP
  ├────────→ Device A
  ├────────→ Device B
  └────────→ Device C
```

A cél a közös időalap, nem az audioadat továbbítása.

``` text
Dante Audio
≠
PTP Clock
```

------------------------------------------------------------------------

# 6.11 PTPv1 és PTPv2

Az Audinate dokumentációja szerint: - a standard Dante alapértelmezésben
PTPv1-et használ; - AES67 és SMPTE ST 2110-30 interoperabilitásnál PTPv2
is használatos; - bizonyos eszközökön PTPv2 külön konfigurálható.
citeturn0search1turn0search3

``` text
Standard Dante
      ↓
    PTPv1

AES67 / ST 2110
      ↓
    PTPv2
```

A konkrét eszköz- és firmware-képességeket mindig ellenőrizni kell.

------------------------------------------------------------------------

# 6.12 PTP üzenetek -- alapmodell

A PTP nem egyszerűen „Leader elküldi az időt".

Egyszerűsített modell:

``` text
Leader
  │
  │ Sync
  ▼
Follower
  │
  │ Delay Request
  ▼
Leader
  │
  │ Delay Response
  ▼
Follower
```

A cél az időviszonyok és a hálózati út késleltetésének meghatározása.

------------------------------------------------------------------------

# 6.13 Sync

A Leader Sync üzeneteket küld:

``` text
Leader
   │
   │ Sync
   ▼
Follower
```

A follower ezekből információt kap a leader időalapjáról. A hálózati
késleltetést azonban külön is figyelembe kell venni.

### Follow Up

Bizonyos PTP működési módokban a Sync üzenethez kapcsolódó pontos
időbélyeg külön **Follow Up** üzenetben érkezik.

Egyszerűsített modell:

``` text
Leader
   │
   │ Sync
   ▼
Follower
   │
   │ Follow Up
   ▼
Follower
```

A Follow Up tehát nem egy újabb „óra”, hanem a PTP időzítési információ
pontosításának része. Kezdőként elég azt megjegyezni, hogy a PTP
szinkronizáció több, egymással összefüggő üzenetből áll.

------------------------------------------------------------------------

# 6.14 Delay Request / Delay Response

``` text
Follower
    │
    │ Delay Request
    ▼
Leader
    │
    │ Delay Response
    ▼
Follower
```

A folyamat a hálózati út késleltetésének meghatározását segíti.

> **A PTP nem pusztán „óraátadás": a hálózati útvonalat is figyelembe
> kell venni.**

------------------------------------------------------------------------

# 6.15 Clock offset

A clock offset leegyszerűsítve azt mutatja, hogy a follower helyi
időalapja mennyire tér el a referencia clocktól.

A cél nem feltétlenül az, hogy minden pillanatban pontosan nulla legyen,
hanem hogy a szinkron stabil legyen.

------------------------------------------------------------------------

# 6.16 Frequency offset

A hardveres clockok nem tökéletesek. A quartz kristályok között kis
különbségek vannak.

Ezért a follower clockját finoman „húzni" kell a leaderhez.

``` text
Leader:
0 ppm referencia

Follower:
+2 ppm
```

Az Audinate Clock Status Monitor ezt a frequency offsetet ppm-ben is
megjeleníti. citeturn0search0

------------------------------------------------------------------------

# 6.17 Stabil és instabil clock

### Stabil

``` text
+1.8 ppm
+2.0 ppm
+2.1 ppm
+1.9 ppm
+2.0 ppm
```

### Instabil

``` text
-1 ppm
+7 ppm
-4 ppm
+11 ppm
-8 ppm
```

Az Audinate szerint a szűk offset-eloszlás általában stabilabb clockot,
a szélesebb eloszlás instabilitást jelezhet. Az alábbi számok
szemléltető értékek; nem univerzális Dante-határértékek. citeturn0search0

------------------------------------------------------------------------

# 6.18 Mi okozhat clock instabilityt?

Az Audinate több lehetséges okot emel ki: - túlterhelt hálózati link; -
rosszul implementált EEE; - nem megfelelő switch-konfiguráció; -
problémás külső Word Clock; - 100 Mbit/s-os switch vagy link ott, ahol
gigabites kapcsolat szükséges. citeturn0search0turn0search4

> **A clock hiba nem feltétlenül „audio clock hiba". Lehet hálózati
> probléma következménye.**

------------------------------------------------------------------------

# 6.19 EEE -- miért lehet problémás?

Az Energy Efficient Ethernet energiát takarít meg, de bizonyos
implementációk nem kívánt link- és késleltetési viselkedést okozhatnak
Dante clocking mellett.

Az Audinate kifejezetten felsorolja a nem megfelelő EEE-t a clock
instability lehetséges okai között. citeturn0search0turn0search4

Dante switch konfigurációnál ezért:

``` text
EEE / Green Ethernet
```

külön ellenőrzendő.

------------------------------------------------------------------------

# 6.20 Miért számít a Gigabit?

Az Audinate szerint a gigabites kapcsolattal rendelkező eszköz előnyt
élvezhet a 100 Mbit/s-os eszközzel szemben a leader választás során. A
100 Mb kapcsolat bizonyos rendszerekben clock instabilityt is okozhat.
citeturn0search1turn0search4

Ezért nem elég:

``` text
„A kábel linkel.”
```

Azt is tudni kell:

``` text
100 Mb?
1 Gb?
10 Gb?
```

------------------------------------------------------------------------

# 6.21 Clock Status a Dante Controllerben

A Clock Status nézetben többek között megjelenhet:

``` text
Leader
Follower
Preferred Leader
External
```

és redundáns rendszerekben Primary / Secondary státusz is.

Az Audinate dokumentációja külön kezeli a Primary és Secondary PTP v1/v2
állapotokat, a Preferred Leader és external sync információkat.
citeturn0search2

------------------------------------------------------------------------

# 6.22 Clock Status Monitor

A Clock Status Monitor fontos eseményeket naplózhat:

``` text
Clock Sync Warning
Clock Sync Unlocked
Clock Sync Locked
Audio Mute
Audio UnMute
```

Az Audinate szerint clock sync elvesztése esetén egy eszköz
automatikusan némulhat, amíg vissza nem szerzi a szinkront.
citeturn0search0

------------------------------------------------------------------------

# 6.23 Clock History

A History a frequency offset eloszlását mutatja.

``` text
Frequency offset
        │
        │      █
        │    ████
        │   █████
        │    ███
        │      █
        └────────────────
             ppm
```

A stabil clock szűkebb, az instabilabb clock szélesebb eloszlást
mutathat. A mérés körülbelül másodpercenként frissül.
citeturn0search0

------------------------------------------------------------------------

# 6.24 Miért lehet egy clock stabil, mégis veszélyes?

Például:

``` text
+9 ppm
+9.2 ppm
+9.1 ppm
+9.0 ppm
```

Ez lehet stabil, de kevés tartalékot hagyhat.

Az Audinate szerint ha a follower eléri a hardveres clock pull range-ét,
elveszítheti a szinkront és némulhat. citeturn0search0

------------------------------------------------------------------------

# 6.25 Clock és audio -- két külön kérdés

``` text
Audio flow
     ≠
Clock synchronization
```

Lehet:

``` text
Subscription = OK
Clock = ERROR
Audio = nincs
```

vagy:

``` text
Clock = OK
Subscription = nincs
Audio = nincs
```

Ezért hibakeresésnél külön kérdezd:

``` text
Van audio flow?
Van clock?
```

------------------------------------------------------------------------

# 6.26 Clock és hálózati hibák

Egy túlterhelt vagy hibás hálózati útvonal okozhat:

``` text
Clock instability
      ↓
Clock Sync Warning
      ↓
Clock Sync Unlocked
      ↓
Audio Mute
```

A Dante Controller ezt eseménynaplóban is képes jelezni.
citeturn0search0

------------------------------------------------------------------------

# 6.27 PTP és QoS

A PTP-csomagok időérzékeny szerepet töltenek be. A QoS nem a PTP
része, hanem a hálózati infrastruktúra forgalomkezelési mechanizmusa,
amely megfelelő konfiguráció esetén segíti az időérzékeny PTP-forgalom
prioritását.

``` text
PTP
 ↓
DSCP
 ↓
QoS
 ↓
Switch queue
```

Az Audinate PTPv2 konfigurációjában DSCP érték használható a PTPv2
forgalom osztályozására és prioritására. citeturn0search3

A konkrét értékeket mindig az adott Dante-eszköz és switch
dokumentációja alapján kell kialakítani.

------------------------------------------------------------------------

# 6.28 PTP és VLAN

Egyszerű rendszer:

``` text
Dante VLAN
   │
   ├── Stage Box
   ├── Console
   └── DSP
```

A kérdés:

> **A PTP-forgalom eljut minden szükséges Dante-eszközhöz?**

Routed DDM-rendszerben már boundary clockok és PTPv2 unicast clocking is
megjelenhet. citeturn0search1

------------------------------------------------------------------------

# 6.29 Redundant Dante és PTP

``` text
             Device
             /                /             Primary    Secondary
          │           │
          ▼           ▼
      Switch A     Switch B
```

Az Audinate szerint redundáns hálózatban a clock synchronization mindkét
hálózaton működik, és az egyik hálózat kiesésekor a redundáns eszköz a
másikon keresztül továbbra is kaphat clockinformációt.
citeturn0search1

A redundancy tehát:

``` text
audio
+
clock
+
network path
```

együttes kérdése.

------------------------------------------------------------------------

# 6.30 Clock domain

Egyszerű rendszerben:

``` text
Egy Dante hálózat
      ↓
Egy clock domain
```

Fejlettebb rendszerekben több clock domain is létezhet.

Az Audinate szerint sample-rate pull-up/down esetén külön clock domainek
jöhetnek létre, saját PTP clockkal. citeturn0search2

> **Clock domain ≠ VLAN.**

A clock domain szinkronizációs logikát jelent, nem önálló fizikai
hálózatot.

------------------------------------------------------------------------

# 6.31 Miért fontos a clock domain?

Az Audinate szerint egy Dante-eszköz csak ugyanazon clock domainben lévő
eszközökkel tud médiát továbbítani és fogadni. citeturn0search2

``` text
Device A
Clock Domain A
      │
      └────→ Device B
             Clock Domain A
```

A natív Dante media routing szempontjából az eszközöknek kompatibilis
clock domainben kell működniük. A különböző clock domainek ezért nem
kezelhetők úgy, mintha ugyanahhoz a natív Dante media clockhoz tartoznának.

------------------------------------------------------------------------

# 6.32 Routed Dante és boundary clock

DDM-es, több IP subnetet átfogó Dante domainben:

``` text
Subnet A
   │
Subnet Leader
   │
Boundary Clock
   │
   ▼
Subnet B
   │
Subnet Leader
```

Az Audinate szerint a routed DDM hálózatokban subnet leader és boundary
clock szerepek jelenhetnek meg, a boundary clockok pedig PTPv2 unicast
clockingot használhatnak a subnetek között. citeturn0search1

Ez már haladó rendszertervezés.

------------------------------------------------------------------------

# 6.33 Deep Dive -- hogyan „érti meg" két eszköz ugyanazt az időt?

``` text
Local Clock
      ↓
PTP messages
      ↓
Time information
      ↓
Network delay estimation
      ↓
Clock offset estimation
      ↓
Local clock correction
      ↓
Stable follower clock
```

A PTP tehát nem egy egyszeri „óraállítás", hanem folyamatos
szinkronizációs folyamat.

------------------------------------------------------------------------

# 6.34 Deep Dive -- miért nem elég egy egyszerű NTP?

Az NTP és a PTP eltérő célokra és pontossági követelményekre készültek.
Az NTP általános rendszeróra-szinkronizációra használható, míg a Dante
a PTP-alapú clocking mechanizmust használja az audioeszközök szükséges
időalap-szinkronizációjához.

``` text
NTP
→ általános rendszeróra

PTP
→ nagy pontosságú hálózati időszinkronizáció
```

A lényeg:

> **A Dante audiohálózat clocking igénye nem azonos egy átlagos PC
> rendszerórájának szinkronizálási igényével.**

------------------------------------------------------------------------

# 6.35 Labor 1 -- Ki a Clock Leader?

Topológia:

``` text
                 Aruba Switch
              /       |                    /        |               Stage Box    Console      DSP
```

Feladat:

1.  Nyisd meg a Dante Controllert.
2.  Nyisd meg a Clock Status nézetet.
3.  Keresd meg a Leader Clockot.
4.  Írd fel az egyes eszközök státuszát.

``` text
Stage Box:
Leader / Follower

Console:
Leader / Follower

DSP:
Leader / Follower
```

------------------------------------------------------------------------

# 6.36 Labor 2 -- Preferred Leader

Válassz ki egy eszközt:

``` text
Console
Preferred Leader = ON
```

A többin:

``` text
Preferred Leader = OFF
```

Ellenőrizd:

``` text
Ki lett a Leader?
```

Ezután kapcsold ki a Preferred Leadert, és figyeld meg a választást.

------------------------------------------------------------------------

# 6.37 Labor 3 -- Clock Status Monitor

Indítsd el a Clock Status Monitort.

Figyeld:

``` text
Sync Status
Frequency Offset
Mute Status
Preferred Leader
External Word Clock
```

Hagyd futni a rendszert, hogy ne csak pillanatképet láss.

Az Audinate szerint a History hosszabb idejű futtatása segíthet a
hálózati clock stabilitásának megítélésében. citeturn0search0

------------------------------------------------------------------------

# 6.38 Labor 4 -- Hibakeresési szimuláció

Helyzet:

``` text
Device:
Visible

Subscription:
OK

Clock:
Warning

Audio:
Intermittent
```

Vizsgáld:

``` text
1. Clock Status
2. Clock History
3. Frequency Offset
4. Switch link
5. Link speed
6. EEE
7. Network congestion
8. External clock
```

Ne az újraindítással kezdd.

Kérdezd:

> **Melyik rétegben van bizonyítékunk a hibára?**

------------------------------------------------------------------------

# 6.39 Labor 5 -- Aruba switch ellenőrzés

Clock hiba esetén az Aruba switchen ellenőrizd:

``` text
Port status
Link speed
Errors
Drops
EEE
QoS
VLAN
Multicast
```

Az Audinate a 100 Mb kapcsolatot és a nem megfelelő EEE-t is a clock
stability problémák lehetséges okai között sorolja fel.
citeturn0search4

------------------------------------------------------------------------

# 6.40 Labor 6 -- Külső Word Clock

Csak támogatott eszközzel:

``` text
External Word Clock
        │
        ▼
Dante Device
        │
        ▼
Dante Network
```

Ellenőrizd:

``` text
External Sync:
ON

Leader:
?

Frequency Offset:
?

Clock History:
?
```

A cél annak megfigyelése, hogyan függ a Dante leader clockja a külső
referencia minőségétől.

------------------------------------------------------------------------

# 6.41 Hibakeresési döntési fa

``` text
Nincs audio
    │
    ├── Device látszik?
    │       │
    │       ├── NEM → Network / Discovery
    │       └── IGEN
    │
    ├── Subscription OK?
    │       │
    │       ├── NEM → Routing
    │       └── IGEN
    │
    ├── Clock OK?
    │       │
    │       ├── NEM → PTP / Network / External Clock
    │       └── IGEN
    │
    └── Audio?
            │
            ├── NEM → Flow / Network / Device
            └── IGEN → OK
```

------------------------------------------------------------------------

# 6.42 A legfontosabb hibakeresési szabály

Ne ezt kérdezd:

> „Miért nincs hang?"

Hanem:

``` text
1. Van link?
2. Van IP?
3. Látja a Dante Controller?
4. Van subscription?
5. Van flow?
6. Van clock?
7. Stabil a clock?
8. Van audio?
```

------------------------------------------------------------------------

# 6.43 Vizsgafeladat -- Clock Leader

Adott:

``` text
Stage Box
Console
DSP
Recorder
```

A Console:

``` text
Preferred Leader = ON
```

A Stage Box:

``` text
External Word Clock = ON
```

### Kérdések

1.  Melyik eszközt szeretnénk leadernek?
2.  Mi történik, ha a Stage Box és a Console különböző referenciaórát
    követ?
3.  Miért lehet ebből clock probléma?
4.  Mit ellenőrzöl a Dante Controllerben?
5.  Mit ellenőrzöl a Clock Status Monitorban?
6.  Mikor lenne helyes a Stage Boxot használni leaderként?
7.  Mikor lenne jobb a Console?

------------------------------------------------------------------------

# 6.44 Vizsgafeladat -- Clock instability

A Clock History:

``` text
+1 ppm
+2 ppm
+8 ppm
-5 ppm
+12 ppm
-7 ppm
```

### Kérdések

1.  Stabilnak neveznéd?
2.  Mi lehet a probléma?
3.  Mit ellenőriznél az Aruba switchen?
4.  Mit ellenőriznél az external clockon?
5.  Miért lehet érdekes az EEE?
6.  Miért érdekes a linksebesség?
7.  Mit okozhat a tartós clock instability?

Elvárt gondolkodás:

``` text
Instabil clock
      ↓
Hálózati stabilitás
      +
EEE
      +
Link speed
      +
Congestion
      +
External Word Clock
```

------------------------------------------------------------------------

# 6.45 Vizsgafeladat -- PTP megértése

Magyarázd el saját szavaiddal:

> **Hogyan lesz három külön Dante-eszközből egy közös időalapon működő
> digitális audiórendszer?**

A válaszban szerepeljen:

``` text
Local Clock
PTP
Leader
Follower
Sync
Delay
Clock Offset
Frequency Offset
```

------------------------------------------------------------------------

# 6.46 Laborok és vizsgafeladatok – megoldókulcs

A megoldókulcs célja nem az, hogy a feladat helyett gondolkodjon, hanem
hogy a tanuló ellenőrizni tudja, jó irányba indult-e.

### Labor 1 – Ki a Clock Leader?

A Clock Status nézetben egy eszköznek Leaderként, a többinek
Followerként kell megjelennie ugyanazon clock domainben.

### Labor 2 – Preferred Leader

Ha a kiválasztott Console jogosult a leader szerepre, a
`Preferred Leader = ON` beállítás elsőbbséget ad neki. Több Preferred
Leader esetén továbbra is a Dante clock election mechanizmusa dönt.

### Labor 3 – Clock Status Monitor

Nem egyetlen pillanatnyi értéket kell keresni. A cél a trend és a
stabilitás megfigyelése: Sync Status, Frequency Offset, mute események
és a History együtt adnak értelmezhető képet.

### Labor 4 – Clock warning

A helyes első lépés nem az újraindítás. Először el kell dönteni, hogy
a clock warning valóban PTP-szinkronizációs problémát jelez-e, és
ellenőrizni kell a hálózati útvonalat, linksebességet, EEE-t,
terhelést és az esetleges külső clockot.

### Labor 5 – Aruba switch

A cél annak ellenőrzése, hogy a switchporton nincs-e olyan fizikai vagy
konfigurációs probléma, amely a Dante-forgalom vagy a PTP-forgalom
stabilitását rontja. Különösen fontos a linksebesség, hibák/dobások,
EEE, QoS és VLAN-konfiguráció ellenőrzése.

### Labor 6 – External Word Clock

A tanulónak azt kell felismernie, hogy a külső referencia nem
automatikusan jobb. Ha a referencia instabil vagy nem megfelelő,
a Dante clocking problémája is megjelenhet.

### Vizsgafeladat – Clock Leader

A kérdésre nincs univerzális „mindig a Console” válasz. A helyes
döntés a rendszer referencia-architektúrájától függ. A Preferred Leader
tervezési döntés; külső referencia esetén azt is meg kell vizsgálni,
hogy melyik eszköz követi a referencia clockot.

### Vizsgafeladat – Clock instability

A megadott értékek széles tartományban mozognak, ezért az eloszlás
instabilitásra utaló jelként értelmezhető. A következő vizsgálati
lépések a hálózati útvonal, linksebesség, EEE, terhelés és external
Word Clock ellenőrzése.

### Vizsgafeladat – PTP megértése

A jó válasz lényege:

``` text
Leader clock
     ↓
PTP üzenetek
     ↓
időbélyegek + útvonal késleltetésének figyelembevétele
     ↓
clock offset / frequency korrekció
     ↓
Follower clocks
     ↓
közös időalap
```

------------------------------------------------------------------------

# 6.47 Fejezeti mentális modell

``` text
                    CLOCK
                      │
                 PTP Network
                      │
              ┌───────┴───────┐
              ▼               ▼
        LEADER CLOCK       NETWORK PATH
              │                 │
              │             Delay / QoS
              │                 │
              ▼                 ▼
        PTP Messages      Stable delivery
              │                 │
              └────────┬────────┘
                       ▼
                 FOLLOWER CLOCKS
                       │
                       ▼
                 COMMON TIME
                       │
                       ▼
                  AUDIO FLOW
                       │
                       ▼
                    AUDIO
```

A legfontosabb gondolat:

> **A Dante audiohálózatban nem elég, hogy az adat eljusson A pontból B
> pontba. Az eszközöknek azt is pontosan kell értelmezniük, hogyan
> viszonyul az időalapjuk egymáshoz.**

------------------------------------------------------------------------

# 6.48 Összefoglalás

A Dante clocking megértéséhez ezt a modellt tartsd fejben:

``` text
Digitális audio
      │
      ▼
Közös időalap
      │
      ▼
PTP
      │
      ▼
Leader Clock
      │
      ├────────→ Follower
      ├────────→ Follower
      └────────→ Follower
```

A legfontosabb fogalmak:

### Clock

Az audiórendszer időalapja.

### PTP

Precision Time Protocol -- a Dante hálózati clock szinkronizációjának
alapja.

### Leader Clock

Az adott clock domain referencia clockja.

### Follower Clock

A leaderhez szinkronizáló Dante-eszköz.

### Preferred Leader

Olyan konfiguráció, amely az adott eszköznek elsőbbséget ad a leader
election során.

### External Word Clock

Külső referenciaóra, amelyet támogatott Dante-eszköz használhat.

### Clock Offset

A helyi clock és a referencia közötti időeltérés.

### Frequency Offset

A follower clockjának a leaderhez történő frekvencia-korrekciója.

### Clock Domain

A PTP-alapú szinkronizációs tartomány.

### Clock Stability

A clock követésének stabilitása.

------------------------------------------------------------------------

# 6.49 Az egész fejezet egyetlen ábrán

``` text
                 DANTE NETWORK
                       │
                       ▼
                 PTP CLOCKING
                       │
              ┌────────┴────────┐
              ▼                 ▼
        LEADER CLOCK       NETWORK PATH
              │                 │
              │             Delay / QoS
              │                 │
              ▼                 ▼
        PTP Messages      Stable delivery
              │                 │
              └────────┬────────┘
                       ▼
                 FOLLOWER CLOCKS
                       │
                       ▼
                 COMMON TIME
                       │
                       ▼
                  AUDIO FLOW
                       │
                       ▼
                    AUDIO
```

------------------------------------------------------------------------

# 6.50 Források és műszaki ellenőrzés

A Dante-specifikus clocking állításokat elsődlegesen az Audinate
jelenlegi dokumentációjával ellenőriztük.

Különösen: - Dante PTP és clock synchronization; - Leader / Follower
election; - Preferred Leader; - external Word Clock; - Primary /
Secondary clocking; - PTPv1 és PTPv2; - clock domains; - Clock Status; -
Clock Status Monitor; - frequency offset; - clock instability; - EEE és
linksebesség; - DDM boundary clocking.

Az Audinate dokumentációja szerint a standard Dante hálózatok
alapértelmezésben PTPv1-et használnak, míg AES67 és SMPTE ST 2110-30
interoperabilitás esetén PTPv2 is használatos.
citeturn0search1turn0search3

A Clock Status Monitor az instabilitást, sync warning/unlocked
állapotokat, mute eseményeket és frequency offset történetet is képes
megjeleníteni. citeturn0search0turn0search4

A konkrét PTP-, DDM-, AES67- és ST 2110-beállítások mindig az adott
Dante eszköz, firmware és rendszerarchitektúra dokumentációja alapján
véglegesítendők.

------------------------------------------------------------------------

# 6.51 Fejezeti állapot

**Állapot: COMPLETE – rev. 1.1**

A fejezet tartalmaz: - clock alapfogalmak; - sample rate és clock
kapcsolat; - Dante Leader / Follower modell; - clock election; -
Preferred Leader; - external Word Clock; - PTP; - PTPv1 / PTPv2; - Sync
és Delay Request / Response; - clock offset; - frequency offset; - clock
stability; - EEE; - linksebesség; - Clock Status; - Clock Status
Monitor; - clock domain; - routed DDM clocking; - Primary / Secondary
clocking; - QoS kapcsolat; - Aruba switch hibakeresés; - hat gyakorlati
labor; - hibakeresési döntési fa; - Deep Dive részek; - vizsgafeladatok.
