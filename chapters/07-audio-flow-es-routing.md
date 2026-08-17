---
title: "DANTE – A professzionális Audio over IP rendszerek kézikönyve"
chapter: 7
chapter_title: "Audio Flow és Routing"
version: "1.0"
status: "draft-review"
---

# 7. Audio Flow és Routing

> **A fejezet célja:** megérteni, hogyan jut el a Dante audio egy transmitter eszköztől egy vagy több receiverhez, mit jelent a subscription, mi az audio flow, mikor használ a Dante unicast és multicast forgalmat, hogyan függ össze a csatornaszám és a flow-k száma, valamint hogyan kell a routing hibákat módszeresen diagnosztizálni.

## Előismeretek

Ehhez a fejezethez már érdemes érteni:

- digitális audió és sample rate;
- Ethernet;
- IP-címzés;
- multicast alapfogalma;
- Dante clock és PTP;
- Dante Controller alapjai;
- Leader / Follower modell.

A korábbi fejezetekben azt tanultuk meg, hogy a Dante hálózatnak van közös időalapja. Most azt vizsgáljuk meg, **hogyan kerül ténylegesen az audio egyik Dante-eszközről a másikra.**

---

# 7.1 A Dante routing alapmodell

A legegyszerűbb Dante kapcsolat:

```text
TRANSMITTER
   │
   │ Audio
   ▼
 NETWORK
   │
   ▼
RECEIVER
```

Például:

```text
Stage Box
   │
   │ Mic 1
   ▼
Aruba Switch
   │
   ▼
Mixing Console
   │
   ▼
FOH Input 1
```

A Dante Controllerben ezt egy **subscription** létrehozásával állítjuk be.

```text
Stage Box / Ch 1
        │
        │ subscription
        ▼
Console / Ch 1
```

---

# 7.2 Transmitter és Receiver

A Dante terminológiában:

### Transmitter

Az az eszköz vagy csatorna, amely Dante audioadatot küld.

```text
Stage Box
Ch 1 ──→
Ch 2 ──→
Ch 3 ──→
```

### Receiver

Az az eszköz vagy csatorna, amely Dante audioadatot fogad.

```text
→ Ch 1
→ Ch 2
→ Ch 3
Console
```

Egy fizikai Dante-eszköz gyakran **egyszerre transmitter és receiver**.

```text
          Dante Device
         /            \
     TX channels    RX channels
        │               │
        ▼               ▲
      Network         Network
```

---

# 7.3 Mi az a subscription?

A subscription a Dante routing konfigurációjában azt mondja meg:

> **Melyik transmitter csatorna kerüljön melyik receiver csatornára?**

Például:

```text
Stage Box / Input 1
        │
        ▼
Console / Input 1
```

A Dante Controllerben ezt a transmitter és receiver metszéspontjának kiválasztásával hozhatjuk létre.

---

# 7.4 A subscription nem maga az audio

Nagyon fontos különbség:

```text
Subscription
     │
     ▼
Routing konfiguráció
     │
     ▼
Audio Flow
     │
     ▼
Audio packetek
```

A subscription **routing konfiguráció**: meghatározza, hogy egy adott transmitter audioforrás mely receiverhez jusson el. A Dante-rendszer ennek alapján hozza létre és kezeli a szükséges media flow-kat.

Ha egy subscription megszűnik, az adott transmitter–receiver audioútvonal megszűnik vagy unresolved állapotba kerül. **Ebből nem következik automatikusan, hogy maga a flow is megszűnik**, mert ugyanazt a flow-t más aktív subscriptionök is használhatják.

---

# 7.5 Egy egyszerű példa

Tegyük fel:

```text
Stage Box
Input 1 = Vocal
Input 2 = Guitar
Input 3 = Bass
```

és:

```text
Console
Input 1
Input 2
Input 3
```

A routing:

```text
Vocal
  │
  └────────→ Console Input 1

Guitar
  │
  └────────→ Console Input 2

Bass
  │
  └────────→ Console Input 3
```

---

# 7.6 Mi történik a háttérben?

Egyszerűsítve:

```text
Dante Controller
      │
      │ routing configuration
      ▼
Dante devices
      │
      │ establish media flows
      ▼
Network packets
      │
      ▼
Receiver
      │
      ▼
Audio output
```

A Controller nem úgy működik, mint egy hagyományos audio patchbay, amely minden egyes audio packetet továbbít.

A Controller elsősorban **konfigurációs és felügyeleti eszköz**.

---

# 7.7 Subscription állapotok

A Dante Controllerben a subscription státusza fontos diagnosztikai információ.

Egyszerűen:

```text
OK
 ↓
Subscription működik

Error
 ↓
Subscription nem oldódott fel megfelelően
```

Egy subscription hibájának több oka lehet:

- transmitter offline;
- receiver offline;
- inkompatibilis sample rate;
- clock-domain probléma;
- hálózati kommunikációs probléma;
- audio flow elvesztése.

A subscription error tehát **nem automatikusan routing-hiba**.

---

# 7.8 Kompatibilis sample rate

A Dante eszközök között a routingnak kompatibilis sample rate mellett kell működnie.

```text
Transmitter
48 kHz
      │
      ▼
Receiver
48 kHz

          ✓
```

Ha a kompatibilitás nem teljesül:

```text
Transmitter
48 kHz
      │
      ▼
Receiver
96 kHz

          ✕
```

A Dante Controller a subscription hibáján keresztül ezt is jelezheti.

> **A „látom az eszközt” nem jelenti azt, hogy az eszközök között audio routing is létrehozható.**

---

# 7.9 Egy transmitter több receivernek

A Dante egyik fontos képessége, hogy egy transmitter audioja több receiverhez is eljuthat.

```text
                 ┌──→ Console
                 │
Stage Box Ch 1 ──┼──→ Recorder
                 │
                 └──→ Broadcast
```

A kérdés ezután:

> **Hogyan történik a hálózaton az egy-forrás → több-célpont továbbítás?**

Itt jutunk el az audio flow fogalmához.

---

# 7.10 Mi az audio flow?

A flow-t egyszerűsítve úgy képzelhetjük el, mint egy logikai audioadat-folyamot a hálózaton.

Unicast esetén egy transmitter több receiverhez több külön unicast flow-t is használhat:

```text
Transmitter
    │
    ├──→ Unicast Flow 1 ──→ Receiver A
    ├──→ Unicast Flow 2 ──→ Receiver B
    └──→ Unicast Flow 3 ──→ Receiver C
```

Multicast esetén ezzel szemben egy multicast flow több receiverhez is eljuthat:

```text
Transmitter
    │
    ▼
Multicast Flow
    │
    ├──→ Receiver A
    ├──→ Receiver B
    └──→ Receiver C
```

A pontos flow-szerkezet az alkalmazott routingtól és az adott Dante-eszköz képességeitől függ.

A tényleges flow-k száma nem azonos automatikusan azzal, hogy hány csatornát látunk a Dante Controllerben.

Ez később fontos lesz a kapacitástervezésnél.

---

# 7.11 Channel és Flow nem ugyanaz

Ez az egyik legfontosabb fogalom a fejezetben.

```text
Channel
=
egy audio csatorna

Flow
=
egy hálózati media stream / adatfolyam
```

Például:

```text
Transmitter:
8 audio channel

Receiver:
8 audio channel
```

nem feltétlenül jelenti azt, hogy:

```text
8 külön hálózati flow
```

A Dante több audio csatornát is kezelhet közös flow-kon belül.

---

# 7.12 Miért fontos a flow?

Mert a Dante-eszköznek lehetnek flow-kapacitásai.

Egy eszköz például rendelkezhet:

```text
32 transmit flows
32 receive flows
```

miközben ennél több audio csatornát kezel.

A Dante Controller Flow Information nézete az adott eszköz audio-, video- és ancillary flow-it képes megjeleníteni.

> **Ezért a „hány csatornát küldünk?” és a „hány flow-t használunk?” két külön tervezési kérdés.**

A konkrét transmit- és receive-flow kapacitás **eszközfüggő**, ezért tervezéskor mindig az adott Dante-eszköz dokumentációját és specifikációját kell ellenőrizni.**

---

# 7.13 Unicast audio flow

A legegyszerűbb elképzelés:

```text
Transmitter
     │
     │ unicast
     ▼
Receiver
```

A hálózatban a forgalom egy konkrét célpont, tipikusan egy receiver felé tart. Ha ugyanazt az audioforrást több receivernek kell továbbítani, több unicast flow is létrejöhet.

---

# 7.14 Több receiver és unicast

Ha egy forrásnak több külön receiverhez kell eljutnia:

```text
             ┌──→ Console
             │
Stage Box ───┼──→ Recorder
             │
             └──→ Broadcast
```

unicast esetben a transmitternek több külön célpont felé kell forgalmat küldenie.

Ez növelheti a transmitter és a hálózat terhelését.

---

# 7.15 Multicast audio flow

Multicast esetén:

```text
Stage Box
    │
    ▼
Multicast group
    │
 ┌──┼──┬──┐
 ▼  ▼  ▼  ▼
 A  B  C  D
```

A receiver eszközök csatlakoznak az adott multicast group-hoz.

---

# 7.16 Miért hasznos a multicast?

Nagyobb rendszerekben, ahol egy audioforrás sok receiverhez jut el, a multicast hatékonyabb lehet.

```text
1 transmitter
      │
      ▼
 multicast flow
      │
 ┌────┼────┬────┐
 ▼    ▼    ▼    ▼
 A    B    C    D
```

A hálózati infrastruktúra feladata ilyenkor, hogy a multicastot a szükséges portok felé továbbítsa.

---

# 7.17 Multicast nem egyenlő broadcast

Nagyon fontos:

```text
Unicast
1 → 1

Multicast
1 → meghatározott csoport

Broadcast
1 → mindenki
```

A multicast receivernek a megfelelő multicast grouphoz kell csatlakoznia; ennek kezelésében a hálózati multicast-mechanizmusok, például az IGMP is szerepet játszhatnak.

Ezért a multicast kontrolláltabb, mint a broadcast.

---

# 7.18 IGMP szerepe

Megfelelő multicast-kezeléssel a switch képes korlátozni a multicast forgalmat azokra a portokra, amelyeknek szükségük van rá.

Ebben az IGMP, illetve a switch IGMP snooping funkciója fontos szerepet játszhat. Az, hogy pontosan milyen IGMP-konfiguráció szükséges, a hálózati architektúrától és a switch működésétől függ.

Egyszerűsített modell:

```text
Receiver
   │
   │ IGMP membership
   ▼
Switch
   │
   │ multicast forwarding
   ▼
Receiver port
```

A megfelelő multicast switch-konfiguráció később külön fejezetben kerül részletesen tárgyalásra.

> **Ebben a fejezetben egyelőre azt kell megérteni, hogy a multicast routinghoz a hálózatnak tudnia kell, mely receiver portok kérik az adott multicast forgalmat.**

---

# 7.19 Mi történik rosszul, ha nincs megfelelő multicast kezelés?

Egyszerűsített hibakép:

```text
Multicast source
      │
      ▼
Switch
      │
      ├──→ szükséges port ✓
      ├──→ szükséges port ✓
      ├──→ minden port ✕
      └──→ minden port ✕
```

Ha a multicastot a switch nem megfelelően kezeli, a forgalom szükségtelen portokra is eljuthat.

Ez:

- felesleges hálózati terhelést;
- rosszabb skálázhatóságot;
- potenciális packet loss / congestion problémát

okozhat.

---

# 7.20 Dante flow és multicast – fontos pontosítás

Nem minden Dante routing multicast.

A rendszer a routingtól és a konfigurációtól függően unicast és multicast flow-kat is használhat.

Ezért hibakeresésnél ne feltételezd:

> „Dante = multicast.”

A helyes gondolkodás:

```text
Milyen subscription?
        ↓
Milyen flow?
        ↓
Unicast vagy multicast?
        ↓
Milyen switch forwarding szükséges?
```

---

# 7.21 Dante Controller és routing matrix

A Network View Routing nézete lényegében egy patch matrix.

```text
                    RECEIVERS
              Console   DSP   Recorder
TX 1             ✓
TX 2                       ✓
TX 3                               ✓
TX 4             ✓        ✓
```

A `✓` azt jelenti:

```text
subscription exists
```

Nem azt jelenti, hogy:

```text
audio guaranteed
```

Ez nagyon fontos különbség.

---

# 7.22 Subscription létrehozása

A tipikus folyamat:

```text
1. Open Dante Controller
          ↓
2. Network View
          ↓
3. Routing
          ↓
4. Find transmitter
          ↓
5. Find receiver
          ↓
6. Select intersection
          ↓
7. Wait for subscription to resolve
          ↓
8. Verify audio
```

A routing beállítása után **mindig ellenőrizni kell a státuszt és az audiojelet.**

---

# 7.23 Egy csatorna routingja

Példa:

```text
Stage Box / Ch 1
       │
       ▼
Console / Ch 1
```

Ellenőrzés:

```text
Subscription = OK
Clock = OK
Signal = present
```

---

# 7.24 Több csatorna routingja

```text
Stage Box
Ch 1 ─────────→ Console Ch 1
Ch 2 ─────────→ Console Ch 2
Ch 3 ─────────→ Console Ch 3
Ch 4 ─────────→ Console Ch 4
```

Itt már érdemes csatornanév alapján dolgozni:

```text
Vocal
Guitar
Bass
Kick
```

nem pedig:

```text
Ch 1
Ch 2
Ch 3
Ch 4
```

---

# 7.25 Channel labels és routing

A Dante eszközök a csatornákhoz neveket is hirdethetnek.

Például:

```text
TX:
01 Vocal
02 Guitar
03 Bass
04 Kick
```

Ez jelentősen csökkenti a routing hibák lehetőségét.

A Dante Controller a felfedezés során a csatornaneveket és a csatornaszámokat is meg tudja jeleníteni.

> **Egy nagy Dante rendszerben a jó channel naming nem esztétikai kérdés, hanem hibamegelőzési eszköz.**

---

# 7.26 Több receiver ugyanarról a forrásról

```text
Stage Box / Vocal
        │
        ├──→ FOH Console
        ├──→ Monitor Console
        ├──→ Recorder
        └──→ Broadcast
```

Ilyenkor fontos megérteni:

```text
subscription count
       ≠
flow count
       ≠
channel count
```

---

# 7.27 Flow Information

A Dante Controllerben a Device View → View → View Flow Information nézetben meg lehet tekinteni az adott eszköz audio-, video- és ancillary flow-információit.

Ez különösen akkor hasznos, amikor a routing matrix már túl absztrakt.

```text
Routing:
„Van subscription.”

Flow Information:
„Milyen flow-k vannak ténylegesen?”
```

---

# 7.28 Flow capacity

A Dante eszközöknek lehet meghatározott transmit és receive flow kapacitásuk.

Például:

```text
Device
32 TX flows
32 RX flows
```

Ha egy rendszerben sok különálló receiverre kell ugyanazt a tartalmat küldeni, a flow-kapacitás tervezési korláttá válhat.

Ezért nagy rendszerben:

```text
Csatornaszám
+
Receiver szám
+
Flow típus
+
Device flow capacity
```

együtt vizsgálandó.

---

# 7.29 Miért nem elég a csatornákat megszámolni?

Tegyük fel:

```text
64 audio channel
```

Ez önmagában nem mondja meg:

- hány transmitter van;
- hány receiver van;
- hány külön flow keletkezik;
- unicast vagy multicast routing történik-e;
- milyen eszközökön jelentkezik a flow-limit;
- milyen switch kapacitás szükséges.

A valódi hálózati tervezéshez a teljes routing topológiát kell látni.

---

# 7.30 Audio latency

A transmitter és receiver között a packeteknek időben meg kell érkezniük.

```text
Transmitter
     │
     │ packet
     ▼
  Switch 1
     │
     ▼
  Switch 2
     │
     ▼
  Receiver
```

A receiver latency beállítása azt az időt is meghatározza, amely alatt az audio packetnek meg kell érkeznie a lejátszás előtt.

---

# 7.31 Mi történik túl alacsony latency esetén?

Ha a receiver latency beállítása túl alacsony:

```text
Packet
  │
  ├──→ megérkezik időben ✓
  │
  └──→ megérkezik későn ✕
```

A későn érkező packetet a receiver eldobhatja.

Következmény:

```text
Packet loss
    ↓
Audio glitch
```

Az Audinate Dante Controller dokumentációja szerint a receiver latency értékének elegendőnek kell lennie ahhoz, hogy a packetek a lejátszás előtt megérkezzenek; késői packetek esetén audio loss következhet be.

---

# 7.32 Multicast latency

Az Audinate dokumentációja szerint a multicast flow-k automatikusan legalább 1 ms latencyt használnak.

A konkrét eszközök támogatott latency értékeit mindig ellenőrizni kell.

---

# 7.33 Latency monitoring

Támogatott Dante eszközöknél a Dante Controller Latency nézete histogramot is képes mutatni.

Ez megmutathatja például:

- average latency;
- peak latency;
- late packet eseményeket.

Egyszerű értelmezés:

```text
Minden packet jóval a limit előtt
        ↓
stabil

Packetek a limit közelében
        ↓
kockázatos

Late packets
        ↓
audio loss
```

---

# 7.34 Routing és clock együtt

```text
PTP clock
    │
    ▼
Network timing
    │
    ▼
Audio flow
    │
    ▼
Receiver
```

Ha a clock nincs rendben:

```text
Routing OK
Clock ERROR
     ↓
Audio ERROR
```

Ezért a routing hibakeresésnél is ellenőrizni kell a clockot.

---

# 7.35 Routing és network discovery

```text
Ethernet
   ↓
IP configuration
   ↓
Discovery
   ↓
Dante Controller
   ↓
Routing
```

Ha egy transmitter nem látható:

```text
nem tudsz rá normálisan subscriptiont létrehozni
```

Ezért a routing problémát nem szabad automatikusan routing-problémának tekinteni.

---

# 7.36 A routing hibakeresési modell

```text
Nincs audio
    │
    ├── Device látható?
    │       │
    │       ├── NEM → Discovery / Network
    │       └── IGEN
    │
    ├── Subscription létrejött?
    │       │
    │       ├── NEM → Compatibility / Clock / Network
    │       └── IGEN
    │
    ├── Flow működik?
    │       │
    │       ├── NEM → Flow / Network
    │       └── IGEN
    │
    ├── Clock stabil?
    │       │
    │       ├── NEM → PTP / Network / External Clock
    │       └── IGEN
    │
    └── Audio signal?
            │
            ├── NEM → Source / Device / Routing
            └── IGEN → OK
```

---

# 7.37 Gyakori hiba: rossz transmitter

Nagy rendszerben:

```text
Vocal Mic
    │
    ├── StageBox-A / Ch 1
    │
    └── StageBox-B / Ch 1
```

A két csatorna neve akár hasonló is lehet.

Ezért mindig ellenőrizd:

```text
Device name
+
Channel name
+
Subscription destination
```

---

# 7.38 Gyakori hiba: rossz receiver

Például:

```text
Stage Box Vocal
       │
       ▼
Monitor Console Ch 12
```

miközben a mérnök ezt várta:

```text
FOH Console Ch 12
```

A subscription technikailag működik, mégis:

```text
„nincs hang ott, ahol keresem”
```

Ezért a működő subscription sem bizonyítja, hogy a **helyes** routingot állítottuk be.

---

# 7.39 Gyakori hiba: channel naming

Ha minden csatorna:

```text
Ch 1
Ch 2
Ch 3
...
```

akkor a hibakeresés nehezebb.

Jobb:

```text
Vocal
Guitar
Bass
Kick
Snare
OH-L
OH-R
```

---

# 7.40 Gyakori hiba: sample rate

```text
Transmitter
48 kHz

Receiver
96 kHz
```

A routing matrixban láthatod a két eszközt, de a subscription nem feltétlenül lesz érvényes.

Ezért:

```text
Visible
≠
Compatible
```

---

# 7.41 Gyakori hiba: clock domain

Például:

```text
TX
Clock Domain A

RX
Clock Domain B
```

A két eszköz ugyanazon fizikai hálózaton lehet, de a clock-domain eltérés miatt a natív Dante media routing nem feltétlenül lesz kompatibilis.

---

# 7.42 Gyakori hiba: multicast switch configuration

Ha multicast routingot használunk:

```text
Source
  ↓
Switch
  ↓
Receiver group
```

akkor a switch multicast kezelése is része a rendszernek.

A későbbi switch-fejezetben részletesen foglalkozunk:

- IGMP;
- IGMP snooping;
- querier;
- multicast forwarding;
- VLAN;
- QoS.

> **A Dante routing nem csak Dante Controller konfiguráció. A switch is része az audioútvonalnak.**

---

# 7.43 Gyakori hiba: túl sok receiver

Egy transmitter sok receiverhez történő továbbításánál a hálózati és eszközoldali terhelés is nőhet.

```text
1 source
   │
   ├──→ 1 receiver
   ├──→ 2 receiver
   ├──→ 8 receiver
   └──→ 20 receiver
```

Ez már nem ugyanaz a hálózati terhelés.

A rendszertervezőnek figyelnie kell:

```text
Flow capacity
+
Bandwidth
+
Multicast requirements
+
Switch capacity
```

---

# 7.44 Routing dokumentáció

Professzionális rendszernél érdemes routing táblát vezetni.

| Source Device | Source Channel | Destination Device | Destination Channel | Purpose |
|---|---|---|---|---|
| StageBox-A | Vocal 1 | FOH-Console | Ch 1 | FOH |
| StageBox-A | Vocal 1 | Monitor-Console | Ch 1 | Monitor |
| StageBox-A | Vocal 1 | Recorder | Ch 1 | Recording |
| StageBox-A | Guitar 1 | FOH-Console | Ch 2 | FOH |

A routing matrix mellett legyen dokumentált rendszerterv is.

---

# 7.45 Labor 1 – Egy csatorna routingja

Topológia:

```text
Stage Box
    │
    ▼
Aruba Switch
    │
    ▼
Console
```

Feladat:

1. Ellenőrizd, hogy mindkét Dante-eszköz látható.
2. Ellenőrizd a sample rate-et.
3. Ellenőrizd a clock státuszt.
4. Hozz létre egy subscriptiont.
5. Ellenőrizd a subscription státuszát.
6. Adj jelet a transmitterre.
7. Ellenőrizd a receiver inputot.

Elvárt állapot:

```text
Device = visible
Clock = OK
Subscription = OK
Signal = present
```

---

# 7.46 Labor 2 – Egy forrás több receiverre

Topológia:

```text
                 ┌──→ FOH
                 │
Stage Box Ch 1 ──┼──→ Monitor
                 │
                 └──→ Recorder
```

Feladat:

1. Hozd létre mindhárom subscriptiont.
2. Ellenőrizd a három receiveren a jelet.
3. Figyeld meg a flow-információt.
4. Jegyezd fel, hány transmit/receive flow látható.

Cél:

> Megérteni, hogy a routing matrixban látható subscriptionök és a tényleges media flow-k nem feltétlenül ugyanazt jelentik.

---

# 7.47 Labor 3 – Multicast routing

Csak olyan laborhálózaton végezd, ahol a multicast konfigurációt ellenőrizni tudod.

Feladat:

```text
Transmitter
     │
     ▼
Multicast flow
     │
 ┌───┼───┐
 ▼   ▼   ▼
 A   B   C
```

Ellenőrizd:

- melyik flow multicast;
- mely receiverhez jut el;
- az Aruba switchen milyen multicast forwarding történik;
- van-e IGMP snooping / querier konfiguráció;
- változik-e a forgalom, ha egy receivert leiratkoztatsz.

---

# 7.48 Labor 4 – Rossz sample rate

Állítsd a laborban lévő kompatibilis eszközöket úgy, hogy:

```text
Transmitter = 48 kHz
Receiver = eltérő sample rate
```

Próbálj subscriptiont létrehozni.

Vizsgáld:

```text
Subscription status
Error message / tooltip
Clock status
```

Ezután állítsd vissza a kompatibilis sample rate-et.

Cél:

> Megtanulni különbséget tenni a „device visible” és a „device compatible” között.

---

# 7.49 Labor 5 – Flow Information

Válassz ki egy Dante-eszközt.

Nyisd meg:

```text
Device View
   ↓
View
   ↓
View Flow Information
```

Vizsgáld:

- transmit flows;
- receive flows;
- audio flows;
- támogatott flow-kapacitás.

Cél:

> A routing matrix mögött meglátni a tényleges media-flow modellt.

---

# 7.50 Labor 6 – Latency

Támogatott eszközökkel nyisd meg a latency nézetet.

Figyeld:

```text
Average latency
Peak latency
Late packets
Histogram
```

Kérdés:

> Mi történhet, ha a receiver latency túl alacsony?

Elvárt gondolkodás:

```text
Network delay
      ↓
Packet arrives late
      ↓
Packet dropped
      ↓
Audio glitch
```

---

# 7.51 Labor 7 – Routing hibakeresés

Helyzet:

```text
Device:
VISIBLE

Subscription:
ERROR

Clock:
OK
```

Vizsgáld:

1. transmitter online?
2. receiver online?
3. sample rate kompatibilis?
4. clock domain azonos?
5. van-e hálózati kommunikáció?
6. van-e flow?
7. van-e eszközoldali korlátozás?

Ne a switch újraindításával kezdd.

---

# 7.52 Labor 8 – A működő, de rossz routing

Helyzet:

```text
Subscription = OK
Audio = OK
```

de a hang a Monitor Console-on jelenik meg, miközben a mérnök a FOH Console-on keresi.

Feladat:

> Találd meg a routing táblában a hibát.

Tanulság:

```text
Subscription OK
       ≠
Design OK
```

---

# 7.53 Vizsgafeladat – Mi a subscription?

Magyarázd el saját szavaiddal:

> Mi a különbség a subscription, a flow és az audio channel között?

Jó válaszban szerepeljen:

```text
Subscription
→ routing kapcsolat / konfiguráció

Channel
→ audio csatorna

Flow
→ hálózati media adatfolyam
```

---

# 7.54 Vizsgafeladat – Unicast vagy multicast?

Adott:

```text
Egy transmitter
20 receiver
```

Melyik kérdéseket tennéd fel?

```text
1. Hány receivernek kell ugyanaz az audio?
2. Milyen flow-t használunk?
3. Mekkora a transmitter flow-capacitása?
4. Mekkora a hálózati terhelés?
5. Van multicast infrastruktúra?
6. Hogyan kezeli az Aruba switch az IGMP-t?
```

A helyes válasz nem az, hogy:

> „20 receiver = mindig multicast.”

A helyes válasz:

> **A routing igénye és a rendszer architektúrája alapján kell megválasztani a megfelelő flow-modellt.**

---

# 7.55 Vizsgafeladat – Nincs hang

Adott:

```text
Device = visible
Subscription = OK
Clock = OK
```

Még sincs audio.

Mit vizsgálsz?

Javasolt sorrend:

```text
1. Source signal
2. Transmitter channel
3. Receiver channel
4. Flow information
5. Latency
6. Packet loss
7. Network path
8. Device configuration
```

---

# 7.56 Vizsgafeladat – Subscription error

Adott:

```text
Device = visible
Subscription = ERROR
Clock = OK
```

Vizsgálandó:

```text
Sample rate
Clock domain
Transmitter availability
Receiver availability
Network communication
Flow capacity
Device compatibility
```

---

# 7.57 Hibakeresési döntési fa

```text
Nincs audio
      │
      ▼
Device visible?
      │
 ┌────┴────┐
 NEM      IGEN
 │          │
Network    Subscription?
             │
        ┌────┴────┐
       NEM       IGEN
        │          │
 Compatibility   Flow?
 Clock           │
 Network     ┌───┴───┐
             NEM    IGEN
              │       │
          Network /  Clock?
          capacity     │
                  ┌────┴────┐
                 NEM       IGEN
                  │          │
                PTP /      Signal?
                network      │
                         ┌────┴────┐
                        NEM       IGEN
                         │          │
                    Source /      OK
                    device
```

---

# 7.58 A routing hibakeresés 8 kérdése

```text
1. Látom az eszközt?
2. A megfelelő transmitter?
3. A megfelelő receiver?
4. Létrejött a subscription?
5. Kompatibilis a sample rate?
6. Azonos a clock domain?
7. Van működő flow?
8. Van tényleges audio signal?
```

Ha ezekre egymás után válaszolsz, a legtöbb alap routing hibát nem találgatással, hanem bizonyíték alapján tudod lokalizálni.

---

# 7.59 Deep Dive – Mi történik egy subscription után?

Egyszerűsített modell:

```text
User creates subscription
          ↓
Controller communicates configuration
          ↓
Transmitter / receiver establish routing
          ↓
Media flow is created
          ↓
Audio packets traverse network
          ↓
Receiver buffers packets
          ↓
Receiver clock schedules playback
          ↓
Audio output
```

A pontos belső implementáció eszköz- és firmwarefüggő, ezért ezt a diagramot **fogalmi modellként** kezeld.

---

# 7.60 Deep Dive – Miért fontos a receiver buffer?

A hálózatban a packeteknek lehet változó érkezési idejük.

```text
Packet 1 ────────→
Packet 2 ───→
Packet 3 ───────────→
Packet 4 ─────→
```

A receiver latency/buffer mechanizmusa időt biztosít arra, hogy a packetek megfelelően megérkezzenek.

Túl kevés idő:

```text
late packet
↓
drop
↓
glitch
```

Túl nagy latency:

```text
stabilabb packet delivery
+
nagyobb end-to-end audio latency
```

A rendszertervezésben ezért kompromisszumot kell találni.

---

# 7.61 Deep Dive – Network latency és audio latency

```text
Network latency
=
mennyi idő alatt jut el a packet

Audio latency
=
a teljes audioút késleltetése
```

Az audio latency tartalmazhat:

- eszköz processinget;
- hálózati késleltetést;
- receiver bufferinget;
- további audio processinget.

Ezért a kettőt nem szabad automatikusan azonosnak tekinteni.

---

# 7.62 Deep Dive – Multicast és switch

Multicastnál a switchnek tudnia kell:

```text
Melyik multicast group?
        ↓
Mely portok kérik?
        ↓
Hová kell továbbítani?
```

Ezért válik az IGMP snooping és az IGMP querier fontossá a következő hálózati fejezetekben.

---

# 7.63 Deep Dive – Miért lehet veszélyes a floodolt multicast?

Ha a switch nem korlátozza megfelelően a multicastot:

```text
Source
  │
  ▼
Switch
  │
  ├──→ port 1
  ├──→ port 2
  ├──→ port 3
  ├──→ port 4
  └──→ ...
```

akkor a szükségesnél több port kaphatja meg a forgalmat.

Ez nagy Dante hálózatban:

```text
extra traffic
+
switch load
+
endpoint load
```

formájában jelentkezhet.

---

# 7.64 Deep Dive – Flow és hálózati kapacitás

A rendszertervezésnél:

```text
Audio channels
      ↓
Flow structure
      ↓
Packet rate / bandwidth
      ↓
Switch capacity
      ↓
Link capacity
```

A csatornaszám önmagában nem elég.

Meg kell vizsgálni:

```text
Sample rate
Bit depth
Channel count
Unicast / multicast
Number of destinations
Flow capacity
Network topology
```

---

# 7.65 Rendszertervezési példa

Adott:

```text
Stage Box
64 inputs

FOH Console
64 inputs

Monitor Console
64 inputs

Recorder
64 inputs

Broadcast
64 inputs
```

Nem azt kérdezzük elsőként:

> „64 × 4 = 256 channel, tehát ennyi hálózati kapacitás kell.”

Hanem:

```text
Mely források?
Mely célpontok?
Mely flow-k?
Unicast?
Multicast?
Milyen sample rate?
Milyen linkek?
Milyen switch?
Milyen flow-capacity?
```

Ez a Dante rendszertervezési gondolkodás alapja.

---

# 7.66 Aruba switch – routing ellenőrzés

A következő switch-fejezetben részletesen foglalkozunk a konfigurációval, de routing hibánál már most ellenőrizhető:

```text
Port link
Link speed
Errors
Drops
VLAN
Multicast
IGMP
QoS
EEE
```

Ha egy Dante receiver nem kap audio flow-t, nem szabad kizárni a switch réteget.

---

# 7.67 Routing és dokumentáció

Nagy rendszerben legyen:

```text
Device list
+
Channel list
+
Routing matrix
+
VLAN plan
+
IP plan
+
Clock plan
+
Switch port map
```

A routing matrix önmagában nem elég.

---

# 7.68 A fejezet mentális modellje

```text
              DANTE ROUTING
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
   TRANSMITTER              RECEIVER
        │                       ▲
        │                       │
        └────── FLOW ───────────┘
                    │
             UNICAST / MULTICAST
                    │
                    ▼
               ETHERNET
                    │
                    ▼
                SWITCH
                    │
                    ▼
              NETWORK PATH
                    │
                    ▼
              RECEIVER BUFFER
                    │
                    ▼
              CLOCKED PLAYBACK
                    │
                    ▼
                  AUDIO
```

---

# 7.69 Amit ebből a fejezetből tudnod kell

### Transmitter
Audioforrásként működő Dante endpoint.

### Receiver
Audiofogadó Dante endpoint.

### Subscription
A transmitter és receiver közötti routing konfiguráció.

### Flow
A hálózaton továbbított logikai media adatfolyam.

### Unicast
Egy konkrét célpont felé irányuló forgalom.

### Multicast
Egy multicast csoportba küldött forgalom, amelyet több receiver is fogadhat.

### IGMP
A multicast group membership kezelésében használt protokoll.

### Latency
A receiver számára rendelkezésre álló idő és a teljes audioút késleltetésének fontos része.

### Flow capacity
Az eszköz által kezelhető transmit/receive flow-k korlátja.

### Routing matrix
A Dante Controllerben látható logikai patching felület.

---

# 7.70 A legfontosabb különbségek

```text
Channel
↓
Audio jel

Subscription
↓
Routing konfiguráció

Flow
↓
Network media stream

Unicast
↓
Egy konkrét célpont felé

Multicast
↓
Egy meghatározott receiver-csoport felé

Clock
↓
Időalap

Latency
↓
Packet érkezési idő + buffering követelménye
```

---

# 7.71 Fejezeti vizsgakérdések

1. Mi a különbség transmitter és receiver között?
2. Mi a subscription?
3. Mi a különbség channel és flow között?
4. Miért nem azonos automatikusan a csatornák száma és a flow-k száma?
5. Mi az unicast?
6. Mi a multicast?
7. Miért nem broadcast a multicast?
8. Mi az IGMP szerepe?
9. Mi történhet, ha multicastot nem megfelelően kezel a switch?
10. Miért fontos a receiver latency?
11. Mi történik late packet esetén?
12. Miért kell a clockot routing hibánál is ellenőrizni?
13. Mit jelent az, hogy egy subscription OK?
14. Miért lehet egy subscription OK, miközben a routing funkcionálisan rossz?
15. Miért fontos a channel naming?
16. Miért fontos a flow capacity?
17. Mit ellenőrzöl Aruba switchen routing hiba esetén?
18. Mi a különbség network latency és audio latency között?
19. Milyen esetben lehet előnyös a multicast?
20. Miért nem lehet a „Dante = multicast” állítást általános szabályként használni?

---

# 7.72 Megoldókulcs – röviden

### 1–3.
```text
Transmitter = küld
Receiver = fogad
Subscription = routing kapcsolat
```

### 4.
Mert több audio channel közös media flow-kon is továbbítható.

### 5–6.
```text
Unicast = egy konkrét célpont felé irányuló forgalom; több receiver esetén több unicast flow is lehet.

Multicast = egy meghatározott multicast csoport felé irányuló forgalom, amelyet több receiver is fogadhat.
```

### 7.
Multicastnál meghatározott group membership alapján történik a forwarding, nem minden hálózati résztvevő kapja automatikusan.

### 8.
Az IGMP a multicast group membership kezelésében segít.

### 9.
Felesleges multicast traffic, nagyobb terhelés és rosszabb skálázhatóság jelentkezhet.

### 10–11.
A packetnek a lejátszás előtt meg kell érkeznie. Ha késik, a receiver eldobhatja, ami audio glitchhez vezethet.

### 12.
Mert a routing működése és a szinkronizált audio playback a clocktól is függ.

### 13.
A konfigurált subscription feloldódott és a kapcsolat aktuálisan érvényesnek látszik.

### 14.
Például rossz destinationre routingoltuk.

### 15.
Mert csökkenti a rossz source/destination kiválasztásának kockázatát.

### 16.
Mert az eszköznek korlátozott számú transmit/receive flow kezelésére lehet kapacitása.

### 17.
Legalább:
```text
link
speed
errors
drops
VLAN
multicast
IGMP
QoS
EEE
```

### 18.
A network latency a packet útjának késleltetése; az audio latency a teljes audioút késleltetésének tágabb fogalma.

### 19.
Ha ugyanazt az audioforrást sok receivernek kell hatékonyan terjeszteni, megfelelő multicast infrastruktúrával.

### 20.
Mert a Dante alapértelmezett audioátvitele unicast, míg multicast flow-k is használhatók olyan esetekben, amikor ugyanazt az audioforrást több receivernek kell továbbítani.

---

# 7.73 Következő fejezet

# 8. Dante Controller

A most megszerzett tudásra építünk:

```text
Audio Flow
    ↓
Routing
    ↓
Dante Controller
    ↓
Network View
    ↓
Device View
    ↓
Clock Status
    ↓
Routing Matrix
    ↓
Flow Information
    ↓
Hibakeresés
```

A 8. fejezetben a Dante Controllert már nem egyszerű „programként” fogjuk kezelni, hanem **a Dante rendszer konfigurációs és diagnosztikai felületeként**.

---

# 7.74 Fejezeti állapot

**Állapot: COMPLETE – szakmai ellenőrzésre kész**

A fejezet tartalmaz:

- transmitter / receiver modell;
- subscription;
- routing matrix;
- channel vs. flow;
- unicast;
- multicast;
- IGMP alapok;
- flow capacity;
- latency;
- late packet;
- Flow Information;
- channel naming;
- routing hibakeresés;
- Aruba switch ellenőrzési pontok;
- 8 labor;
- vizsgafeladatok;
- megoldókulcs;
- Deep Dive részek;
- rendszertervezési példa.

A Dante-specifikus állításokat az Audinate dokumentációjával ellenőriztem, különösen a Flow Information, latency, discovery és clock/routing működésével kapcsolatban.
