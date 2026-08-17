# 12. Dante haladó hibakeresés és diagnosztika

> **A fejezet célja:** megtanulni, hogyan kell egy Dante rendszer hibáját módszeresen feltárni akkor is, amikor a tünet nem egyértelmű, és több lehetséges hibaforrás is szóba jöhet.

Az előző fejezetben megtanultuk a rendszerüzemeltetés alapjait:

```text
eszközleltár
firmware
dokumentáció
baseline
backup
change control
recovery
```

Most a következő lépés következik:

> **Mi történik, amikor valami elromlik?**

A kezdő gyakran így gondolkodik:

```text
Nincs audio
    ↓
routing?
```

A tapasztalt üzemeltető viszont:

```text
TÜNET
  ↓
BIZONYÍTÉK
  ↓
HIPOTÉZIS
  ↓
TESZT
  ↓
IZOLÁCIÓ
  ↓
ROOT CAUSE
  ↓
JAVÍTÁS
  ↓
VERIFIKÁCIÓ
```

A fejezet egyik legfontosabb gondolata:

> **Ne azt keresd elsőként, hogy mi lehet a hiba. Azt keresd, milyen megfigyelés választja szét a lehetséges hibákat.**

---

# 12.1 Miért nehéz a Dante hibakeresés?

Mert egy audiohiba több különböző rétegből származhat.

Például:

```text
Nincs audio
    ↓
kábel
switch
VLAN
IP
Dante discovery
subscription
clock
sample rate
latency
multicast
QoS
firmware
device
```

A tünet tehát nem azonos a hibaforrással.

```text
symptom ≠ root cause
```

---

# 12.2 A hibakeresés négy alapvető kérdése

Minden hibánál először válaszolj:

1. **Mi nem működik?** — device, audio, clock, network vagy redundancy?
2. **Hol nem működik?** — egy eszközön, egy flow-n, egy VLAN-on vagy az egész hálózaton?
3. **Mikor kezdődött?** — mindig, most, firmware- vagy hálózati változás után?
4. **Mi változott?** — cable, switch, firmware, routing, VLAN, IP, sample rate?

Ez a négy kérdés gyakran többet ér, mint tíz találomra végzett konfigurációs módosítás.

---

# 12.3 A hiba hatóköre

Az egyik legerősebb diagnosztikai eszköz:

> **A hiba hatókörének meghatározása.**

```text
A = OK
B = OK
C = FAIL
D = OK
```

Ilyenkor valószínűbb:

```text
C device
C cable
C port
C configuration
```

mint az egész hálózat hibája.

Ha viszont:

```text
A = FAIL
B = FAIL
C = FAIL
D = FAIL
```

akkor valószínűbb:

```text
switch
uplink
VLAN
clock
power
```

A hatókör tehát csökkenti a keresési teret.

---

# 12.4 Single device vs multiple devices

### Egyetlen eszköz hibás

Első gyanúsítottak:

```text
power
cable
port
device
configuration
```

### Több eszköz ugyanazon az ágon hibás

Gyanús lehet:

```text
common switch
common uplink
common VLAN
common power
```

### Minden eszköz hibás

Vizsgáld:

```text
core
switch
VLAN
network
clock
power
```

---

# 12.5 A „last known good” állapot

A 11. fejezetben megtanultuk a Known Good Configuration fogalmát.

Hibakeresésnél ez különösen fontos:

```text
Known Good
    ↓
változás
    ↓
hiba
```

Ha például:

```text
14:00 = OK
14:15 = firmware update
14:20 = audio drops
```

akkor a firmware-frissítés releváns hipotézis.

De:

> **Időbeli egyezés még nem bizonyít ok-okozati kapcsolatot.**

Ezt tesztelni kell.

---

# 12.6 A bizonyíték és a feltételezés

### Bizonyíték

```text
Primary link = DOWN
```

### Feltételezés

```text
Primary cable is bad
```

A második még nem bizonyított.

Lehet:

```text
cable
switch port
Dante port
power
```

Ezért a jó hibakereső különválasztja:

```text
FACT
vs
HYPOTHESIS
```

---

# 12.7 A diagnosztikai napló

Érdemes minden komolyabb hibát így dokumentálni:

| Idő | Tünet | Hatókör | Megfigyelés | Hipotézis | Teszt | Eredmény |
|---|---|---|---|---|---|---|
| 14:20 | audio drop | 1 device | packet errors | cable | cable swap | PASS |
| 14:25 | audio drop | 1 device | errors remain | switch port | port swap | FAIL |

A napló célja:

> **Ne ugyanazt a hipotézist teszteld újra és újra.**

---

# 12.8 Első diagnosztikai lépés: ne változtass

Ha a rendszer hibás:

```text
STOP
```

Először:

```text
observe
capture
document
```

Ne kezdd azonnal:

```text
change routing
change clock
change VLAN
reboot everything
```

Egy változtatás ugyanis:

```text
eredeti hiba
      +
új változás
      ↓
összetettebb állapot
```

hozhat létre.

---

# 12.9 Reboot nem diagnosztika

A „kapcsold ki-be” néha megold egy hibát.

De:

```text
problem
 ↓
reboot
 ↓
works
```

után még nem tudod:

> **Mi okozta a hibát?**

A reboot lehet:

```text
workaround
```

de nem feltétlenül:

```text
root cause analysis
```

---

# 12.10 Hibakeresési sorrend

Hasznos általános sorrend:

```text
1. Symptom
2. Scope
3. Physical
4. Link
5. Network
6. Dante discovery/control
7. Clock
8. Subscription
9. Performance
10. Device / firmware
11. Root cause
12. Fix
13. Verification
```

Ez nem merev szabály. A cél:

> **A lehető legkisebb számú, leginformatívabb teszttel szűkíteni a hibát.**

---

# 12.11 Fizikai réteg

Elsőként:

```text
Power
Cable
Connector
Link LED
Port
```

Ellenőrizd:

```text
device powered?
switch port up?
link speed?
errors?
```

Sok „Dante hiba” valójában:

```text
bad cable
```

---

# 12.12 Link speed

A link státusza:

```text
UP
```

nem jelenti azt, hogy minden rendben.

Vizsgáld:

```text
1 Gbps
100 Mbps
10 Mbps
Down
```

Ha egy link váratlanul lassabb:

```text
cable
termination
switch port
NIC
```

lehet érintett.

A link speed tehát diagnosztikai adat.

---

# 12.13 Ethernet hibák

Vizsgáld a switch oldalon:

```text
CRC errors
input errors
output errors
drops
link flaps
```

A pontos megnevezés switchgyártónként eltérhet.

Ha egy porton:

```text
errors ↑
```

az fontos nyom.

De:

> **A hiba nem automatikusan bizonyítja az okot.**

---

# 12.14 Link flap

Link flap:

```text
UP
 ↓
DOWN
 ↓
UP
 ↓
DOWN
```

fizikai vagy interfészszintű problémára utalhat.

Lehetséges ok:

```text
bad cable
loose connector
port problem
NIC problem
power instability
```

A switch logja sokszor nagyon hasznos.

---

# 12.15 VLAN hibák

Ha az eszközök:

```text
physically connected
```

de nem látják egymást:

```text
VLAN
```

gyanús lehet.

Ellenőrizd:

```text
access VLAN
trunk VLAN
native VLAN
tagging
allowed VLAN
```

A pontos konfiguráció switchplatform-függő.

---

# 12.16 „Látom az eszközt” vs „működik az audio”

Nagyon fontos különbség:

```text
Device visible
```

nem jelenti:

```text
Audio working
```

Lehet:

```text
Controller látja
      ↓
subscription nincs
      ↓
audio nincs
```

Ezért a discovery/control és az audio routing külön diagnosztikai kérdés.

---

# 12.17 Dante discovery probléma

Ha egy Dante eszköz nem jelenik meg a Controllerben:

```text
Power
 ↓
Link
 ↓
VLAN / hálózati elérés
 ↓
Network interface
 ↓
Discovery / control
```

ellenőrzendő.

Ha csak bizonyos eszközök hiányoznak, keresd a közös fizikai vagy hálózati hibapontot. Kábel- és switchproblémák miatt Dante eszközök eltűnhetnek vagy megjelenhetnek a Controllerben.

Ne az audio subscriptionnel kezdj, ha a készüléket sem látod.

---

# 12.18 Egy eszköz nem látszik

Példa:

```text
A = visible
B = visible
C = invisible
D = visible
```

Első vizsgálat:

```text
C power
C link
C switch port
C VLAN
C network interface
```

Ha:

```text
C port = DOWN
```

akkor nem érdemes routingot keresni.

---

# 12.19 Több eszköz nem látszik

Példa:

```text
A = visible
B = visible
C = invisible
D = invisible
E = invisible
```

Ha C/D/E ugyanazon switch mögött vannak:

```text
common switch
common uplink
common VLAN
```

erősen gyanús.

Ez a **common denominator** elv.

---

# 12.20 Common denominator

Keresd:

> **Mi közös a hibás eszközökben?**

Például:

```text
same switch
same VLAN
same uplink
same rack
same power
same firmware
```

Ha C/D/E mind ugyanahhoz az uplinkhez tartozik:

```text
uplink
```

sokkal érdekesebb, mint C egyedi konfigurációja.

---

# 12.21 Clock probléma

Ha az audio hibás, a clock állapotát is ellenőrizni kell.

A Dante Controllerben különösen fontos:

```text
Sync
Mute
Clock Source
Primary / Secondary állapot
```

A `Listening` lehet átmeneti állapot, de ha tartósan fennáll, az arra utalhat, hogy az eszköz leaderre vár.

A Clock Status Monitor további bizonyítékot adhat:

```text
Clock Sync Warning
Clock Sync Unlocked
Clock Sync Locked
Audio Mute
Audio UnMute
```

A `Clock Sync Unlocked` tényleges sync-vesztést jelez; az érintett eszköz ilyenkor automatikusan mute-olódhat a szinkron visszaállásáig.

Ezért ne csak azt kérdezd:

```text
Who is the leader?
```

hanem azt is:

```text
Is the device synced?
Was sync lost?
Did it mute?
When did it happen?
```

---

# 12.22 „Nincs audio” – subscription ellenőrzés

Ha:

```text
TX device = visible
RX device = visible
```

de nincs audio, a subscription állapotát célzottan ellenőrizd.

Lehetséges állapotok például:

```text
In progress
Subscribed
Warning
Error
Pending
```

A `Warning` fel nem oldott subscriptiont vagy fizikai hálózati problémát jelezhet; az `Error` például elégtelen bandwidthre is utalhat.

Nézd meg:

```text
TX channel
RX channel
subscription state
tooltip / hibaüzenet
```

A tooltip konkrétabb okot is jelezhet, például eltérő sample rate-et, clock-domain eltérést, flow-kapacitási problémát vagy latency-beállítási hibát.

---

# 12.23 Subscription vs flow

Egy subscription:

```text
TX channel
      ↓
RX channel
```

kapcsolat.

A mögötte lévő Dante flow lehet:

```text
unicast
```

vagy:

```text
multicast
```

Ezért nagyobb rendszernél érdemes megérteni:

```text
channel
   ↓
flow
   ↓
network traffic
```

---

# 12.24 Multicast probléma

Ha egy flow több receiverhez megy:

```text
TX
 │
 ├── RX1
 ├── RX2
 ├── RX3
 └── RX4
```

multicast lehet hatékony megoldás.

Hibakereséskor különítsd el:

```text
multicast flow
        ↓
IGMP membership / snooping
        ↓
switch forwarding
        ↓
receiver
```

A cél annak megállapítása, hogy a multicast forgalom:

```text
létrejön-e?
megfelelő VLAN-on halad-e?
a switch megfelelő portokra továbbítja-e?
a receiver ténylegesen megkapja-e?
```

Ezért pontosabb a **multicast forwarding / IGMP kezelés** fogalma; a „multicast routing” csak akkor indokolt, ha ténylegesen routing is része a hálózatnak.

Vizsgáld:

```text
IGMP snooping
IGMP querier
VLAN
multicast membership
switch forwarding state
```

---

# 12.25 Multicast storm gyanú

Tünetek:

```text
high bandwidth
switch CPU high
packet loss
many devices affected
```

Lehetséges ok:

```text
multicast control problem
```

De:

> **A tünet nem bizonyíték.**

Switch oldali statisztikával és konfigurációval kell megerősíteni.

---

# 12.26 QoS probléma

Ha a hálózat terhelt:

```text
audio packets
control packets
clock packets
other traffic
```

versenyezhetnek.

A QoS célja a megfelelő forgalom prioritásának biztosítása.

Ha QoS hibás:

```text
packet delay
jitter
clock instability
audio issues
```

jelentkezhetnek.

Ellenőrizd:

```text
DSCP
queue
priority
switch QoS
```

---

# 12.27 Bandwidth probléma

Vizsgáld:

```text
Tx
Rx
utilization
peak
average
```

Ha egy link:

```text
near saturation
```

akkor packet loss vagy késleltetés kockázata nőhet.

Ne csak azt kérdezd:

> „Van kapcsolat?”

Hanem:

> **„Van elég kapacitás?”**

---

# 12.28 Latency probléma

A Dante receiver latency-beállítása a hálózaton érkező audio packetek számára biztosított időtartalék. A cél, hogy a packetek a receiverhez a lejátszásuk előtt megérkezzenek. A tényleges flow latency a transmitter és receiver képességeitől és beállításaitól is függ; a Dante a subscription létrehozásakor egyezteti a használható értéket.

Ha a hálózat:

```text
congested
```

vagy az útvonal:

```text
complex
```

és a latency túl alacsony:

```text
late packets
      ↓
packet drop
      ↓
audio glitch
```

A Dante Controller Network Status nézetében a latency állapota külön is megjelenik: a figyelmeztető állapot a limithez közeli packeteket, a hibás állapot későn érkező packeteket jelez.

Vizsgáld:

```text
latency setting
network path
number of switch hops
traffic
QoS
bandwidth
```

> **A latency növelése nem javítja meg a rosszul megtervezett hálózatot.**

---

# 12.29 Packet error

Packet error esetén vizsgáld:

```text
device
switch
cable
link
traffic
```

Ha egy eszköznél:

```text
packet errors ↑
```

de másoknál nincs:

```text
device path
```

lehet gyanús.

Ha minden eszköznél:

```text
packet errors ↑
```

akkor közös hálózati ok valószínűbb.

---

# 12.30 Redundancia diagnosztika

A 10. fejezet alapján:

```text
Primary
Secondary
```

állapotát külön vizsgáld.

Példa:

```text
Primary = DOWN
Secondary = UP
Audio = OK
```

Ez nem teljesen egészséges állapot.

Keresd:

```text
cable
switch
port
device interface
```

---

# 12.31 Redundancia – egyik oldal hibás

Ha:

```text
P = DOWN
S = UP
```

ne állítsd vissza automatikusan azonnal.

Előbb dokumentáld:

```text
time
device
port
link
switch
```

majd teszteld.

A cél:

> **Megtalálni, mi okozta az első hibát.**

---

# 12.32 Redundancia – mindkét oldal hibás

Ha:

```text
P = DOWN
S = DOWN
```

akkor lehet:

```text
device power
device failure
common power
common physical path
```

is.

Itt különösen fontos a közös hibapont keresése.

---

# 12.33 Firmware mint hipotézis

Firmware lehet hibaforrás.

De csak egy a sok közül.

Jó bizonyíték:

```text
Known Good firmware
      ↓
update
      ↓
specific reproducible failure
      ↓
rollback
      ↓
failure disappears
```

Ez már erős ok-okozati bizonyíték.

---

# 12.34 Firmware rollback mint diagnosztikai teszt

Rollback lehet:

```text
diagnostic experiment
```

Ha:

```text
Firmware A
  ↓
problem

Firmware B
  ↓
no problem
```

akkor erősödik a firmware-hipotézis.

De csak akkor értelmes, ha:

- a rollback támogatott;
- a korábbi verzió kompatibilis;
- van mentés;
- kontrollált környezetben történik.

---

# 12.35 Sample rate probléma

A Dante routingnál a transmitter és receiver kompatibilis sample-rate beállítása alapvető.

Például:

```text
Device A = 48 kHz
Device B = 96 kHz
```

esetén a subscription létrehozása hibába futhat, ha a két oldal nem kompatibilis. A Controller ilyenkor konkrét subscription hibaüzenetet is adhat, például `Incorrect channel format` jellegű üzenetet.

Ellenőrizd:

```text
sample rate
pull-up / pull-down, ha releváns
device clock
```

A pull-up/pull-down külön clock domain használatát is eredményezheti, ezért nem elegendő csak a névleges sample rate-et összehasonlítani.

A pontos lehetőségek készüléktípus-függők.

---

# 12.36 Clock leader változás

Ha a clock leader váratlanul megváltozik:

```text
Leader A
   ↓
Leader B
```

vizsgáld:

```text
device availability
clock settings
PTP network
network changes
```

Redundáns rendszernél azt is:

```text
Primary
Secondary
```

oldalon.

---

# 12.37 Network change után hiba

Ha:

```text
network change
   ↓
audio problem
```

akkor vizsgáld:

```text
VLAN
QoS
IGMP
ports
trunks
routing
firewall / filtering
```

A hálózati változás sokszor több dolgot módosít egyszerre.

Ezért a változásnapló fontos.

---

# 12.38 „Csak néha hibázik”

Intermittáló hibánál különösen veszélyes a találgatás.

Mérj:

```text
time
frequency
duration
affected devices
network state
clock state
errors
```

Például:

```text
every 30 min
```

más hipotéziseket támogat, mint:

```text
only under high traffic
```

vagy:

```text
only after 2 hours
```

Az időbeli minta bizonyíték.

---

# 12.39 Intermittáló hiba – logolás

Használj:

```text
timestamp
```

minden eseményhez.

Példa:

```text
14:02:11 audio drop
14:02:12 packet error
14:02:13 clock warning
```

Ha switch logban:

```text
14:02:10 port flap
```

akkor már van kapcsolat a két esemény között.

---

# 12.40 Root cause vs symptom

Tegyük fel:

```text
Audio drops
```

és:

```text
reboot device
```

megoldja.

A root cause lehet:

```text
firmware state
memory leak
network condition
```

A reboot csak:

```text
symptom cleared
```

Nem bizonyította a root cause-t.

---

# 12.41 A hibakeresés célja

Nem az:

> „Hogy újra szóljon.”

Hanem:

> **„Hogy tudjuk, miért nem szólt, és megakadályozzuk az ismétlődést.”**

Ez a különbség:

```text
troubleshooting
```

és:

```text
root cause analysis
```

között.

---

# 12.42 Hypothesis tree

Példa:

```text
NO AUDIO
   │
   ├── Physical
   ├── Network
   ├── Clock
   ├── Subscription
   ├── Performance
   └── Device
```

Majd:

```text
Network
   │
   ├── VLAN
   ├── QoS
   ├── Multicast
   ├── Bandwidth
   └── Link
```

A fa célja:

> **Ne felejts el fontos kategóriát, de ne vizsgálj mindent egyszerre.**

---

# 12.43 Binary search hibakeresés

Ha nagy rendszerben:

```text
100 devices
```

van, ne egyesével vizsgáld.

Oszd ketté:

```text
50 / 50
```

Ha az egyik oldalon van a hiba:

```text
25 / 25
```

majd:

```text
12 / 13
```

Ez a binary search logika.

Nagy hálózatban nagyon hatékony lehet.

---

# 12.44 Isolation

Az egyik legerősebb diagnosztikai módszer:

> **Izoláld a hibás részt.**

```text
large network
      ↓
small test network
      ↓
device
```

Ha a hiba eltűnik:

```text
network environment
```

gyanúsabb.

Ha megmarad:

```text
device
```

gyanúsabb.

---

# 12.45 A/B teszt

Példa:

```text
Device A
Cable A
Port A
```

hiba.

Cseréld csak a kábelt:

```text
Device A
Cable B
Port A
```

Ha a hiba eltűnik:

```text
Cable A
```

erősen gyanús.

A szabály:

> **Egy teszt során lehetőleg egy változót változtass.**

---

# 12.46 Component swap

Ha van ismert jó:

```text
known-good cable
known-good switch port
known-good Dante device
```

használd összehasonlításként.

De dokumentáld:

```text
what swapped?
when?
result?
```

Különben a végén nem tudod, melyik változtatás oldotta meg.

---

# 12.47 Loopback gondolkodás

Bizonyos eszközöknél és laborhelyzetekben hasznos lehet:

```text
TX
 ↓
known path
 ↓
RX
```

Ha:

```text
local path = OK
network path = FAIL
```

akkor a hálózati út erősen gyanús.

A pontos loopback tesztet mindig az adott eszköz és rendszer lehetőségei szerint kell kialakítani.

---

# 12.48 Kontrollált teszt

Jó teszt:

```text
hypothesis
   ↓
prediction
   ↓
test
   ↓
result
```

Példa:

**Hipotézis:**

> A kábel hibás.

**Predikció:**

> Ismert jó kábellel a hiba megszűnik.

**Teszt:**

```text
replace cable
```

**Eredmény:**

```text
PASS → cable likely cause
FAIL → cable hypothesis weakened
```

Ez már valódi diagnosztika.

---

# 12.49 Mit jelent a „FAIL”?

Nagyon fontos.

Ha a teszt:

```text
FAIL
```

nem feltétlenül rossz eredmény.

Ha:

```text
Cable hypothesis
   ↓
test
   ↓
FAIL
```

akkor:

> **Értékes információt kaptunk.**

A hibakeresés célja a bizonytalanság csökkentése.

---

# 12.50 Információs érték

Két teszt közül azt válaszd, amelyik több lehetséges okot választ szét.

Például:

```text
reboot device
```

kevés információt ad.

Míg:

```text
swap cable
```

jobban megkülönbözteti:

```text
cable
vs
device
```

hipotéziseket.

Ez a haladó hibakereső egyik legfontosabb gondolkodási módja.

---

# 12.51 Dante Controller mint diagnosztikai központ

A Dante Controller különböző nézetei különböző diagnosztikai kérdésekre valók.

Hasznos nézetek:

```text
Network View
Routing
Device View
Clock Status
Network Status
Events
```

A Network Status például subscription státuszt, primary link speedet, latency-beállítást, latency hibákat és packet error állapotot is mutathat.

A Device View az eszköztől függően Receive, Transmit, Status, Latency, Device Config és Network Config információkat is tartalmazhat.

A cél:

```text
observe
```

nem pedig:

```text
randomly change
```

Először gyűjts bizonyítékot, és csak ezután változtass konfigurációt.

---

# 12.52 Network View

Keresd:

```text
device visibility
device count
names
status
```

Kérdés:

> **Minden várt eszköz látható?**

Ha nem:

```text
discovery/network
```

irányba menj.

---

# 12.53 Network Status

Keresd:

```text
Primary
Secondary
link speed
bandwidth
errors
```

Redundáns rendszerben különösen fontos.

---

# 12.54 Clock Status

Keresd:

```text
Leader
Follower
Listening
```

és a releváns clock állapotokat.

Ha:

```text
unexpected leader
```

van, keresd az okát.

---

# 12.55 Routing View

Kérdezd:

```text
TX exists?
RX exists?
subscription?
```

Ha:

```text
device visible
clock OK
routing absent
```

akkor a probléma már szűkebb.

---

# 12.56 Switch diagnosztika

A Dante Controller nem lát mindent.

A switch CLI / web interface sokszor szükséges.

Vizsgáld:

```text
port status
errors
counters
VLAN
QoS
IGMP
CPU
uplinks
```

A Dante hibakeresés ezért gyakran:

```text
Dante Controller
+
switch management
```

kombinációja.

---

# 12.57 Packet capture

Haladó környezetben packet capture is használható.

Például:

```text
Wireshark
```

segítségével.

Vizsgálható:

```text
Ethernet
IP
UDP
multicast
PTP
DSCP
```

De:

> **A packet capture nem kezdő diagnosztikai eszköz.**

Először értsd meg a rendszer állapotát a Dante Controllerben és a switchben.

---

# 12.58 DSCP ellenőrzés

Ha QoS problémára gyanakszol:

```text
packet
   ↓
DSCP marking
   ↓
switch queue
```

útvonalat kell ellenőrizni.

Nem elég azt mondani:

> „QoS be van kapcsolva.”

Azt kell tudni:

```text
marking correct?
queue correct?
priority correct?
```

---

# 12.59 PTP packet capture

Haladó vizsgálatnál a PTP forgalom is megfigyelhető.

Cél:

```text
Who is leader?
Are PTP packets present?
Are they reaching the expected segment?
```

Ez különösen hasznos lehet összetett hálózatokban.

---

# 12.60 Multicast packet capture

Ha multicast problémára gyanakszol:

```text
Is multicast traffic present?
Who sends?
Who receives?
Which VLAN?
```

A packet capture segíthet elválasztani:

```text
sender problem
network problem
receiver problem
```

---

# 12.61 Diagnosztikai eszközök

Alap:

```text
Dante Controller
switch management
cable tester
```

Haladó:

```text
packet capture
Wireshark
managed switch counters
network monitoring
```

Speciális:

```text
optical tester
fiber analyzer
manufacturer diagnostics
```

A megfelelő eszközt a hiba szintjéhez válaszd.

---

# 12.62 Hibakeresési döntési fa – nincs audio

```text
NO AUDIO
   │
   ▼
Device visible?
   │
 ┌─┴─┐
NO   YES
│      │
▼      ▼
Network  Clock OK?
          │
        ┌─┴─┐
       NO   YES
       │      │
       ▼      ▼
      Clock  Subscription?
                │
              ┌─┴─┐
             NO   YES
             │      │
             ▼      ▼
          Routing  Performance
```

Ez nem minden lehetséges hibát tartalmaz.

A cél a gondolkodás struktúrája.

---

# 12.63 Hibakeresési döntési fa – intermittent audio

```text
INTERMITTENT
     │
     ▼
Time pattern?
     │
     ├── periodic
     ├── traffic dependent
     └── random
```

Majd:

```text
packet errors?
clock events?
link flap?
bandwidth?
switch logs?
firmware change?
```

Az időbeli mintázat gyakran kulcsfontosságú.

---

# 12.64 Hibakeresési döntési fa – egy eszköz

```text
ONE DEVICE FAIL
       │
       ▼
Power?
       │
       ▼
Link?
       │
       ▼
Network?
       │
       ▼
Dante visible?
       │
       ▼
Clock?
       │
       ▼
Routing?
       │
       ▼
Device / firmware?
```

---

# 12.66 Valós Dante Controller hibajelenségek

A hibakeresésben nagy előnyt jelent, ha nem csak azt látod, hogy „valami piros”, hanem érted is, mit jelent az adott állapot.

## 12.65.1 Device nem látható

**Tünet:**

```text
A = visible
B = visible
C = invisible
D = visible
```

**Első kérdések:**

```text
C powered?
C link up?
C switch port up?
C VLAN correct?
C hálózati kapcsolat működik?
```

Ha csak egy vagy néhány eszköz hiányzik, keresd a közös fizikai vagy hálózati hibapontot.

**Ne ezt tedd elsőként:**

```text
routing changes
clock changes
firmware update
```

---

## 12.65.2 Unresolved subscription

**Tünet:**

```text
TX = previously existed
RX = still exists
subscription = unresolved
```

Ez például akkor történhet, ha a transmitter eltűnt a hálózatról vagy kikapcsolt, illetve más fizikai hálózati probléma akadályozza a media signal útját.

**Vizsgáld:**

```text
TX device visible?
TX powered?
TX link?
network path?
subscription tooltip?
```

---

## 12.65.3 Subscription Error

Ha a subscription `Error` állapotú, ne csak töröld és hozd létre újra.

Előbb nézd meg a tooltipet.

Lehetséges okok között szerepelhet:

```text
insufficient bandwidth
incompatible sample rate
clock-domain mismatch
flow capacity
latency configuration
locked device
```

A konkrét okot a Controller által jelzett hibaüzenet alapján azonosítsd.

---

## 12.65.4 Clock Sync Warning

**Tünet:**

```text
Clock Sync Warning
```

Ez nem feltétlenül azt jelenti, hogy az audio már azonnal megszakadt.

Azt jelzi, hogy a clock instabilitást mutat, és fennáll a sync elvesztésének kockázata.

Vizsgáld:

```text
network
switch configuration
link speed
clock source
external word clock, ha van
```

---

## 12.65.5 Clock Sync Unlocked

**Tünet:**

```text
Clock Sync Unlocked
```

Ez tényleges sync-vesztést jelent.

Az érintett eszköz ilyenkor automatikusan mute-olódhat, amíg a szinkron vissza nem áll.

Ilyenkor a kérdés:

> **Miért vesztette el a készülék a clock sync-et?**

Nem pusztán az:

> „Hogyan kapcsoljuk vissza az audio-t?”

---

## 12.65.6 Late packets / latency error

**Tünet:**

```text
Latency Errors = warning / error
```

A figyelmeztető állapot azt jelzi, hogy egyes packetek a latency limit közelében érkeznek; a hibás állapot későn érkező packeteket jelez, amelyek audio glitchhez vezethetnek.

Vizsgáld:

```text
receiver latency
network path
traffic
switch hops
QoS
bandwidth
```

A latency növelése lehet megoldás, de ha a hálózat túlterhelt vagy rosszul tervezett, az infrastruktúrát is javítani kell.

---

## 12.65.7 Packet Errors

**Tünet:**

```text
Packet Errors = red
```

A Dante Controller ezt olyan media packetekhez kapcsolja, amelyek a switch és a receiver között megsérültek; ennek gyakori oka hibás Ethernet-kábel.

Első vizsgálatok:

```text
cable
connector
switch port
port counters
CRC / input errors
```

Itt különösen hasznos lehet egy ismert jó kábellel végzett A/B teszt.

---

## 12.65.8 Primary link down

**Tünet:**

```text
Primary = DOWN
Secondary = UP
```

A rendszer még működhet, de a redundancia már sérült.

Ne tekintsd ezt „megoldott problémának” csak azért, mert továbbra is van audio.

Vizsgáld:

```text
primary cable
primary switch port
primary path
link speed
switch logs
```

A cél a tartalékút szükségtelen használatának megszüntetése, mielőtt egy második hiba teljes kiesést okozna.

---

## 12.65.9 Több hibajelzés egyszerre

Például:

```text
Clock Sync Warning
+
Packet Errors
+
Audio drop
```

Ne kezeld automatikusan három külön hibaként.

Lehet közös ok:

```text
switch problem
cable problem
network instability
```

Ezért:

> **A több tünet mögött lehet egyetlen root cause.**

---

## 12.65.10 A tooltip mint bizonyíték

A Dante Controllerben a subscription ikon fölé vitt egérrel részletesebb állapotinformáció kérhető.

Ezért diagnosztikánál:

```text
icon
   ↓
tooltip
   ↓
exact message
   ↓
hypothesis
```

legyen az alapfolyamat.

Ne csak ezt dokumentáld:

```text
subscription = red
```

hanem lehetőleg ezt is:

```text
exact tooltip / error text
```

Ez sokkal értékesebb adat.

---

# 12.67 Labor 1 – Scope analysis


Építs legalább négy Dante végpontot.

Szimulálj:

```text
one device fail
```

majd:

```text
three devices fail
```

Írd le:

```text
common denominator
```

Cél:

> Megtanulni a hiba hatóköréből következtetni.

---

# 12.66 Labor 2 – Known Good baseline

Hozz létre:

```text
Known Good
```

állapotot.

Jegyezd fel:

```text
devices
clock
routing
network
firmware
```

Ezután szándékosan változtass egy dolgot.

Dokumentáld a különbséget.

---

# 12.68 Labor 3 – Cable A/B test

Szimulálj linkproblémát.

Használj:

```text
Cable A
Cable B = known good
```

Cseréld csak a kábelt.

Dokumentáld:

```text
before
test
after
```

---

# 12.69 Labor 4 – Switch port A/B test

Használj két ismert portot:

```text
Port A
Port B
```

Mozgasd át a kapcsolatot.

Figyeld:

```text
link
errors
speed
audio
```

Cél:

> Megkülönböztetni a kábel- és portproblémát.

---

# 12.70 Labor 5 – Device scope

Szimulálj:

```text
Device A = FAIL
B = OK
C = OK
D = OK
```

Majd:

```text
A = FAIL
B = FAIL
C = FAIL
D = OK
```

Mindkét esetben keresd:

```text
common denominator
```

---

# 12.71 Labor 6 – Clock investigation

Szimulálj clock problémát, ha a labor eszközei ezt biztonságosan lehetővé teszik.

Dokumentáld:

```text
Leader
Follower
Listening
sample rate
```

Cél:

> Megérteni, hogyan különül el a clock probléma az audio-routing problémától.

---

# 12.72 Labor 7 – Subscription investigation

Hozz létre:

```text
TX
RX
```

eszközöket.

Szándékosan szüntesd meg a subscriptiont.

Kérdés:

```text
device visible?
clock OK?
audio?
```

Cél:

> Megérteni, hogy a discovery és az audio routing külön probléma.

---

# 12.73 Labor 8 – Switch counters

Válassz egy managed switchet.

Vizsgáld:

```text
CRC
drops
errors
link flaps
```

Jegyezd fel az értékeket.

Majd szimulálj vagy keress egy kontrollált fizikai hibát.

---

# 12.74 Labor 9 – Multicast

Laborhálózaton hozz létre multicast flow-t.

Ellenőrizd:

```text
IGMP
VLAN
multicast state
receiver
```

Cél:

> Megérteni, hogy a multicast routing nem csak Dante Controller kérdés.

---

# 12.75 Labor 10 – Teljes root cause analysis

Szimulált hiba:

```text
Audio intermittent
```

A tanuló feladata:

```text
1. Observe
2. Scope
3. Collect evidence
4. Hypothesis
5. Test
6. Isolate
7. Root cause
8. Fix
9. Verify
10. Document
```

A végén legyen:

```text
Root Cause:
Evidence:
Fix:
Verification:
Prevention:
```

---

# 12.76 Vizsgafeladat – Mi a root cause?

**Kérdés:**

Mi a különbség a tünet és a root cause között?

**Válasz:**

A tünet azt írja le, amit látunk, például:

```text
audio drops
```

A root cause pedig az a tényleges ok, amely a hibát létrehozta.

---

# 12.77 Vizsgafeladat – Miért fontos a scope?

**Válasz:**

Mert a hibás eszközök közös jellemzői segíthetnek leszűkíteni a lehetséges hibaforrásokat.

---

# 12.78 Vizsgafeladat – Miért nem bizonyíték a reboot?

**Válasz:**

Mert a reboot megszüntetheti a tünetet anélkül, hogy megmutatná annak okát.

---

# 12.79 Vizsgafeladat – A/B teszt

**Kérdés:**

Mi az A/B teszt egyik alapelve?

**Válasz:**

Lehetőleg egy változót módosítsunk, és figyeljük, hogyan változik a hiba.

---

# 12.80 Vizsgafeladat – Common denominator

**Kérdés:**

Három Dante eszköz egyszerre hibás, és mind ugyanazon a switchen van.

Miért fontos ez?

**Válasz:**

Mert a közös switch, uplink, VLAN vagy tápellátás valószínűbb közös hibaforrás lehet, mint három egymástól független eszköz egyidejű meghibásodása.

---

# 12.81 Vizsgafeladat – Információs érték

**Kérdés:**

Miért jobb néha egy kábelcsere, mint egy reboot?

**Válasz:**

Mert a kábelcsere célzott hipotézist tesztel, és jobban elkülöníti a kábelhibát az eszköz- vagy hálózati hibától.

---

# 12.82 Deep Dive – Bayesian gondolkodás

A hibakeresés valószínűségi gondolkodás is.

Kezdetben például:

```text
cable      20%
switch     20%
VLAN       15%
clock      15%
routing    15%
firmware   15%
```

Ez csak szemléltető példa.

Ha megfigyeled:

```text
link = DOWN
```

akkor:

```text
cable
switch port
device port
```

hipotézisei erősödnek.

A cél:

```text
evidence
   ↓
update hypothesis
```

---

# 12.83 Deep Dive – Teszt kiválasztása

A jó teszt:

```text
cheap
safe
reversible
informative
```

Például:

```text
check link
```

olcsó és biztonságos.

Míg:

```text
firmware downgrade
```

nagyobb kockázatú.

Ezért:

> **A diagnosztikai teszteket is prioritizálni kell.**

---

# 12.84 Deep Dive – Fault domain

Egy **fault domain** egy olyan rendszerterület, amely egy közös hibától együtt érintett lehet.

Például:

```text
Rack A
```

ha minden eszköz:

```text
same power
same switch
same uplink
```

akkor Rack A lehet egy fault domain.

Nagy rendszerben ez segít a hiba lokalizálásában.

---

# 12.85 Deep Dive – Correlation

Két esemény:

```text
14:02 audio drop
14:02 switch port flap
```

korrelál.

Ez erős nyom.

De:

> **Correlation ≠ causation.**

További teszt kell.

---

# 12.86 Deep Dive – Reprodukció

A root cause megtalálásának egyik legerősebb eszköze:

> **Reprodukálhatóvá tenni a hibát.**

Ha:

```text
action A
   ↓
failure
```

és ezt többször meg tudod ismételni:

```text
A → FAIL
A → FAIL
A → FAIL
```

akkor sokkal könnyebb megtalálni az okot.

---

# 12.87 Deep Dive – Controlled reproduction

A reprodukció legyen:

```text
controlled
safe
repeatable
documented
```

Ne éles show közben kísérletezz.

Labor:

```text
YES
```

Éles rendszer:

```text
only controlled maintenance window
```

---

# 12.88 Deep Dive – Root cause chain

Egy hiba gyakran több szinten magyarázható.

```text
Audio drops
   ↓
packet loss
   ↓
switch congestion
   ↓
bad multicast design
   ↓
IGMP configuration
```

A root cause lehet:

```text
IGMP design/configuration
```

nem egyszerűen:

```text
packet loss
```

A packet loss inkább közbenső tünet vagy hatás.

---

# 12.89 Deep Dive – 5 Whys

Példa:

**Miért nincs audio?**

```text
Packet loss.
```

**Miért van packet loss?**

```text
Switch congestion.
```

**Miért van congestion?**

```text
Unexpected multicast traffic.
```

**Miért van unexpected multicast?**

```text
IGMP configuration incorrect.
```

**Miért volt incorrect?**

```text
Network template was not applied.
```

Így:

```text
symptom
 ↓
technical cause
 ↓
system cause
 ↓
process cause
```

is feltárható.

---

# 12.90 Deep Dive – A jó hibajegy

Ne:

```text
Dante nem működik.
```

Hanem:

```text
14:02:11
Stagebox-02
Input 7 → FOH-DSP
audio drop ~300 ms
Primary/Secondary = UP
Clock = stable
Switch port = no link flap
CRC = 0
Packet errors = increasing
```

Ez már diagnosztikai adat.

---

# 12.91 Deep Dive – Mit dokumentáljunk?

Legalább:

```text
Symptom
Time
Scope
Evidence
Hypothesis
Tests
Results
Root cause
Fix
Verification
Prevention
```

A jó dokumentációból a következő hiba gyorsabban diagnosztizálható.

---

# 12.92 Gyakorlati ellenőrzőlista

## Tünet

```text
□ Mi nem működik?
□ Mikor kezdődött?
□ Milyen gyakran?
□ Melyik eszköz?
```

## Hatókör

```text
□ Egy device?
□ Több device?
□ Egy VLAN?
□ Egy switch?
□ Teljes hálózat?
```

## Fizikai

```text
□ Power
□ Cable
□ Link
□ Speed
□ Errors
```

## Dante

```text
□ Device visible
□ Clock
□ Subscription
□ Routing
□ Redundancy
```

## Hálózat

```text
□ VLAN
□ QoS
□ IGMP
□ Bandwidth
□ Switch logs
```

## Haladó

```text
□ Packet capture
□ DSCP
□ PTP
□ Multicast
□ Reproduction
```

---

# 12.93 A fejezet mentális modellje

```text
                  SYMPTOM
                     │
                     ▼
                    SCOPE
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
       PHYSICAL              NETWORK
          │                     │
          └──────────┬──────────┘
                     ▼
                  DANTE
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        CLOCK    ROUTING     PERFORMANCE
          │          │          │
          └──────────┼──────────┘
                     ▼
                HYPOTHESIS
                     │
                     ▼
                    TEST
                     │
                     ▼
                 ISOLATION
                     │
                     ▼
                ROOT CAUSE
                     │
                     ▼
                    FIX
                     │
                     ▼
                VERIFICATION
                     │
                     ▼
               DOCUMENTATION
```

---

# 12.94 Amit ebből a fejezetből tudnod kell

### Scope

A hiba hatóköre.

### Evidence

Megfigyelhető tény, amely alapján hipotézist lehet értékelni.

### Hypothesis

Lehetséges magyarázat, amelyet tesztelni kell.

### A/B test

Kontrollált összehasonlítás, lehetőleg egy változó módosításával.

### Common denominator

A hibás elemek közös tulajdonsága vagy közös infrastruktúrája.

### Isolation

A hibás komponens vagy fault domain leválasztása.

### Root cause

A hiba tényleges kiváltó oka.

### Verification

A javítás után végzett ellenőrzés, amely bizonyítja, hogy a probléma megszűnt.

---

# 12.95 A legfontosabb szabályok

```text
1. A tünet nem azonos a root cause-szal.
2. Először határozd meg a hiba hatókörét.
3. Válaszd külön a tényt és a feltételezést.
4. Ne változtass azonnal.
5. Dokumentáld az eredeti állapotot.
6. Használd a Known Good állapotot összehasonlításként.
7. Egy teszt lehetőleg egy változót módosítson.
8. A FAIL teszteredmény is értékes információ.
9. Keresd a common denominatort.
10. A reboot nem root cause analysis.
11. A Dante Controller és a switch management együtt ad teljesebb képet.
12. Intermittáló hibánál időbélyeget használj.
13. A korreláció nem automatikusan ok-okozat.
14. Ha lehet, reprodukáld kontrollált környezetben.
15. A javítás után mindig verifikálj.
16. A root cause után gondolj a megelőzésre is.
```

---

# 12.96 Következő fejezet

# 13. Dante rendszertervezés – a teljes rendszer felépítése

A 12. fejezetben megtanultuk:

```text
scope
evidence
hypothesis
test
isolation
root cause
verification
```

A következő lépés:

> **Hogyan használjuk az eddig megtanult összes tudást egy teljes Dante rendszer megtervezéséhez?**

A következő fejezetben:

```text
requirements
   ↓
audio architecture
   ↓
network architecture
   ↓
clock
   ↓
redundancy
   ↓
switch design
   ↓
VLAN / QoS / multicast
   ↓
device configuration
   ↓
testing
   ↓
documentation
```

---

# 12.97 Fejezeti állapot

**Állapot: FINAL – szakmai és oktatási ellenőrzés lezárva**

A fejezet tartalmaz:

- diagnosztikai gondolkodás;
- tünet és root cause;
- scope analysis;
- evidence vs hypothesis;
- Known Good állapot;
- fizikai hibakeresés;
- link speed és Ethernet hibák;
- VLAN;
- Dante discovery;
- clock;
- subscription;
- multicast;
- QoS;
- bandwidth;
- latency;
- packet errors;
- redundancia diagnosztika;
- firmware mint hipotézis;
- sample rate;
- switch diagnosztika;
- packet capture;
- PTP és DSCP;
- hypothesis tree;
- binary search;
- isolation;
- A/B test;
- controlled testing;
- 10 gyakorlati labor;
- vizsgafeladatok;
- Deep Dive részek;
- hibakeresési ellenőrzőlista.

A fejezet **COMMITÁLHATÓ**.
