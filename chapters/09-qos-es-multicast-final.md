---
title: "DANTE – A professzionális Audio over IP rendszerek kézikönyve"
chapter: 9
chapter_title: "QoS és Multicast"
version: "1.0"
status: "draft-review"
---

# 9. QoS és Multicast

> **A fejezet célja:** megérteni, hogy a Dante miért használ QoS-t és multicastot, hogyan működnek ezek Ethernet/IP hálózaton, mikor van rájuk szükség, és hogyan kell egy Dante hálózatot úgy megtervezni, hogy a valós idejű audio- és clock-forgalom akkor is megfelelően működjön, amikor más hálózati forgalommal osztozik az infrastruktúrán.

A korábbi fejezetekben már láttuk:

```text
Dante
  │
  ├── Audio
  ├── PTP / Clock
  ├── Control
  └── Discovery
```

A 7. fejezetben megismertük:

```text
Unicast Flow
Multicast Flow
```

A 8. fejezetben pedig megtanultuk ezeket a Dante Controllerben megfigyelni.

Most a kérdés:

> **Mi történik akkor, ha ugyanazon az Ethernet hálózaton sokféle forgalom versenyez ugyanazért a linkért?**

Erre ad választ többek között:

```text
QoS
```

A multicast pedig arra a kérdésre ad választ:

> **Hogyan küldjünk ugyanazt az adatfolyamot hatékonyan több receivernek?**

---

# 9.1 Mi a QoS?

A QoS, vagyis **Quality of Service**, olyan hálózati mechanizmusok összessége, amelyekkel különböző forgalomtípusok eltérő kezelést kaphatnak.

Egy egyszerű Ethernet hálózatban:

```text
Audio
Video
Backup
Web
File copy
Control
```

mind ugyanazért a hálózati kapacitásért versenyezhet.

Ha egy kimeneti linken torlódás alakul ki:

```text
                  ┌── Audio
                  │
Switch port ──────┼── Video
                  │
                  ├── Backup
                  │
                  └── Other traffic
```

akkor a switchnek döntenie kell:

> **Melyik packetet továbbítsa előbb?**

A QoS erre ad szabályozott választ.

---

# 9.2 Miért fontos ez Dante esetén?

A Dante audio valós idejű médiaforgalom.

Egy fájlmásolásnál:

```text
1 másodperc késés
```

általában nem jelent hallható problémát.

Audio streamingnél viszont:

```text
packet késik
        ↓
jitter / late packet
        ↓
buffer elfogyhat
        ↓
audio glitch
```

A PTP esetén még szigorúbb a helyzet:

```text
PTP packet késik
        ↓
clock timing romolhat
        ↓
szinkronizációs probléma
```

Fontos pontosítás: a **standard Dante hálózatok alapértelmezésben PTPv1-et használnak**. PTPv2 támogatott Dante környezetekben is megjelenhet, például RTP/AES67 vagy SMPTE/ST 2110 alapú működésnél. Ezért a „PTP” általános kifejezésként használható, de konkrét QoS- vagy clock-konfigurációnál mindig jelezni kell, hogy PTPv1-ről vagy PTPv2-ről van szó.

Ezért a Dante hálózatban nem minden forgalom egyformán időkritikus.

---

# 9.3 A Dante QoS-modell

A Dante a Differentiated Services, vagyis **DiffServ** alapú QoS-t tudja használni.

A Dante packetek DSCP értékekkel jelölhetők, amelyeket a hálózati infrastruktúra felhasználhat a prioritás meghatározására.

Az Audinate dokumentációja szerint a tipikus Dante QoS-osztályok:

| Prioritás | Forgalom | DSCP | Decimális |
|---|---|---:|---:|
| High | időkritikus PTP események | CS7 | 56 |
| Medium | audio + PTPv1 / PTPv2, a konfigurációtól függően | EF | 46 |
| Low | fenntartott | CS1 | 8 |
| None | egyéb forgalom | Best Effort | 0 |

citeturn0search29turn0search30

A legfontosabb tanulság:

```text
PTP critical
     ↓
High priority

Audio + PTP
     ↓
Medium priority

Other
     ↓
Best Effort
```

Ezek az Audinate által dokumentált tipikus Dante QoS-osztályok. A tényleges queue-kezelést és a DSCP → queue mappinget a switch QoS-konfigurációja határozza meg.

---

# 9.4 Mi az a DSCP?

A **DSCP – Differentiated Services Code Point** az IP fejlécben található jelölés, amely alapján a hálózati eszközök osztályozhatják a packeteket.

Egyszerű modell:

```text
Dante packet
     ↓
DSCP value
     ↓
Switch classification
     ↓
Queue
     ↓
Transmission priority
```

A DSCP önmagában nem gyorsítja fel a packetet.

Azt mondja meg:

> **„Ezt a packetet ilyen forgalmi osztályba sorold.”**

A tényleges prioritást a switch QoS-konfigurációja valósítja meg.

---

# 9.5 A DSCP nem egyenlő a prioritással

Ez kezdőknek nagyon fontos.

Nem elég az, hogy:

```text
packet DSCP = 56
```

Ha a switch:

```text
nem ismeri fel
vagy
nem megfelelő queue-ba helyezi
```

akkor a kívánt prioritás nem valósul meg.

Tehát:

```text
DSCP marking
      +
Switch QoS policy
      ↓
valós prioritás
```

---

# 9.6 Mi történik torlódáskor?

Tegyük fel, hogy egy 1 Gbit/s switch uplinkre egyszerre több forgalom érkezik:

```text
900 Mbps video
200 Mbps backup
100 Mbps Dante
```

A link kapacitása:

```text
1000 Mbps
```

A beérkező forgalom:

```text
1200 Mbps
```

Tehát:

```text
1200 Mbps
   ↓
1000 Mbps link
   ↓
CONGESTION
```

Ilyenkor queue alakulhat ki.

QoS nélkül a Dante packetek ugyanabba az általános torlódási mechanizmusba kerülhetnek, mint más forgalom.

QoS-szal:

```text
PTP
 ↓
High priority queue

Audio
 ↓
High/medium priority queue

Other
 ↓
Best effort queue
```

---

# 9.7 Strict Priority

Az Audinate Dante hálózati ajánlása szerint ha QoS-t használunk, **strict priority queueinget** kell alkalmazni.

A modell:

```text
Queue 1 – PTP critical
        ↓
Queue 2 – Audio / PTP
        ↓
Queue 3 – Other
```

A switch először a magasabb prioritású queue-t szolgálja ki.

citeturn0search29

Ez nem azt jelenti, hogy az alacsonyabb prioritású forgalom mindig éhezik.

A megfelelő QoS-tervezés célja:

```text
kritikus forgalom
     ↓
minimális várakozás

nem kritikus forgalom
     ↓
maradék kapacitás
```

---

# 9.8 Mikor szükséges a QoS?

Az Audinate jelenlegi hálózati adminisztrátori útmutatója szerint QoS:

- szükséges lehet 100 Mbps infrastruktúrán;
- szükséges lehet vegyes 1 Gbit/s / 100 Mbps infrastruktúrán;
- hasznos lehet mixed-use hálózatokon;
- dedikált, teljes egészében gigabites Dante-audio hálózaton általában nem szükséges.

citeturn0search29turn0search30

Ezért nem helyes azt tanítani:

> „Dante esetén mindig kötelező a QoS.”

Helyesebb:

> **A QoS szükségessége a hálózat kapacitásától, terhelésétől és felépítésétől függ.**

---

# 9.9 Dedikált gigabites Dante hálózat

Egy egyszerű rendszer:

```text
Dante device
      │
      ▼
Gigabit switch
      │
      ├── Dante device
      ├── Dante device
      └── Dante device
```

Ha:

```text
1 Gbit/s
+
Dante audio only
+
nincs túlterhelés
```

akkor a QoS tipikusan nem szükséges.

Ez nem azt jelenti, hogy a QoS „rossz”.

Hanem azt:

> **Nincs feltétlenül szükség prioritási mechanizmusra, ha nincs releváns verseny a kapacitásért.**

---

# 9.10 Mixed-use hálózat

Más a helyzet:

```text
                Core
                 │
       ┌─────────┼─────────┐
       │         │         │
     Dante     Video     IT data
       │         │         │
     Audio     Cameras   Servers
```

Itt már valódi verseny lehet.

Például:

```text
Dante audio
+
IP video
+
backup
+
user traffic
```

egy közös uplinken.

Ilyenkor a QoS sokkal fontosabb.

---

# 9.11 A QoS fő problémája

A QoS nem növeli meg a fizikai link kapacitását.

Ha egy link:

```text
1 Gbit/s
```

akkor QoS után is:

```text
1 Gbit/s
```

marad.

A QoS csak azt szabályozza:

> **ki kapja meg előbb a rendelkezésre álló kapacitást torlódás esetén.**

Ezért:

```text
rosszul méretezett hálózat
       +
QoS
```

nem feltétlenül lesz jó hálózat.

---

# 9.12 Over-subscription

Tegyük fel:

```text
4 × 1 Gbit/s access link
          │
          ▼
     1 Gbit/s uplink
```

Elméletileg:

```text
4 Gbit/s
   ↓
1 Gbit/s
```

Ez **4:1 oversubscription**.

Ha minden port egyszerre maximális terhelést küld:

```text
4 Gbit/s
```

érkezne egy:

```text
1 Gbit/s
```

uplinkre.

Itt már nem az a kérdés, hogy van-e QoS.

Hanem:

> **Helyes-e egyáltalán ez a hálózati méretezés?**

---

# 9.13 QoS és hálózattervezés

A helyes sorrend:

```text
1. Kapacitástervezés
       ↓
2. Linkek méretezése
       ↓
3. Congestion pontok azonosítása
       ↓
4. QoS
       ↓
5. Monitoring
```

Ne:

```text
rossz hálózat
   ↓
QoS
   ↓
„kész”
```

---

# 9.14 DSCP remarking

A Dante packetek DSCP jelölése integrálható meglévő IT QoS-rendszerbe.

Az Audinate szerint a DSCP értékek szükség esetén átjelölhetők, **feltéve, hogy a PTP packetek továbbra is magas prioritást kapnak**.

citeturn0search29

Ez mixed-use enterprise hálózatban lehet fontos.

Például:

```text
Dante DSCP
      ↓
Enterprise QoS policy
      ↓
belső DSCP osztály
```

De csak akkor biztonságos, ha a Dante időkritikus forgalmának prioritása megmarad.

---

# 9.15 Multicast – mi az?

A multicast egy olyan IP kommunikációs modell, amelyben egy sender egy multicast group címre küldi az adatot, és több receiver csatlakozhat ehhez a csoporthoz.

Egyszerű modell:

```text
             ┌── Receiver A
             │
Transmitter ─┼── Receiver B
             │
             └── Receiver C
```

Ahelyett, hogy:

```text
TX → RX A
TX → RX B
TX → RX C
```

külön unicast flow-kat kellene létrehozni, egy multicast flow használható.

---

# 9.16 Unicast vs Multicast

## Unicast

```text
TX
 │
 ├── Flow → RX A
 ├── Flow → RX B
 └── Flow → RX C
```

A transmitter több külön receiver felé küld.

## Multicast

```text
TX
 │
 ▼
Multicast Group
 │
 ├── RX A
 ├── RX B
 └── RX C
```

A transmitter egy multicast streamet küld, amelyhez több receiver csatlakozhat.

---

# 9.17 Mikor előnyös a multicast?

Multicast akkor lehet különösen előnyös, ha:

```text
1 source
+
sok receiver
```

van.

Például:

```text
Announcement mic
      ↓
   Multicast
      ↓
 ┌────┼────┬────┐
 ▼    ▼    ▼    ▼
Room1 Room2 Room3 Lobby
```

Ilyenkor nem feltétlenül érdemes:

```text
1 source
+
20 külön unicast flow
```

megoldást használni.

---

# 9.18 Multicast bandwidth

Tegyük fel példaként, hogy egy audio stream:

```text
6 Mbps
```

és 10 receivernek kell.

> **A 6 Mbps csak szemléltető érték; a tényleges Dante flow sávszélessége a csatornaszámtól, mintavételi frekvenciától, frames-per-packet beállítástól és a használt media formátumtól függhet.**

Unicast modellben a transmitter oldalán közel:

```text
10 × 6 Mbps
=
60 Mbps
```

lehet a továbbított forgalom.

Multicast esetén:

```text
1 × 6 Mbps
```

stream indul a forrásból, és a hálózat multicast továbbítási mechanizmusa juttatja el az érintett szegmensekhez.

Ezért multicastnál a hálózat feladata különösen fontos.

---

# 9.19 Multicast nem broadcast

Nagyon fontos különbség:

```text
Broadcast
= mindenkihez

Multicast
= az adott csoport tagjaihoz

Unicast
= egy receiverhez
```

Tehát:

```text
Broadcast
      ↓
minden host

Multicast
      ↓
group members
```

A multicast kontrolláltabb, mint a broadcast.

---

# 9.20 Mi az IGMP?

Az **IGMP – Internet Group Management Protocol** az IPv4 multicast csoporttagság kezelésére szolgál.

Egyszerűen:

```text
Receiver
   │
   │ „Érdekel ez a multicast group.”
   ▼
IGMP
   │
   ▼
Switch
```

A switch így megtudhatja, melyik porton van szükség az adott multicast forgalomra.

---

# 9.21 IGMP Snooping

Az IGMP Snooping a switch olyan funkciója, amely figyeli az IGMP üzeneteket, és ezek alapján multicast forwarding táblát épít.

Egyszerű modell:

```text
Receiver A
   │
IGMP Join
   │
   ▼
Switch
   │
   ├── Port A → group 239.x.x.x
   ├── Port B → group 239.x.x.x
   └── Port C → nincs group
```

Ha a multicast stream beérkezik:

```text
Multicast
   ↓
Switch
   ↓
csak releváns portok
```

Ez sokkal hatékonyabb, mint minden portra továbbítani.

---

# 9.22 IGMP Snooping nélkül

Ha a switch nem kezeli megfelelően a multicastot, multicast flooding alakulhat ki.

Egyszerű modell:

```text
Multicast stream
      ↓
Switch
      ↓
minden releváns / akár sok port
```

Ennek következménye lehet:

```text
felesleges bandwidth
+
nem érintett endpointok terhelése
+
nagyobb hálózati zaj
```

Ez különösen problémás lehet nagy multicast rendszerekben.

---

# 9.23 Az IGMP Snooping önmagában nem minden

Az egyik leggyakoribb kezdő hiba:

> „Bekapcsoltam az IGMP Snoopingot, tehát kész.”

Nem feltétlenül.

Az IGMP Snoopingnak szüksége lehet egy **IGMP Querierre**, amely a VLAN multicast csoporttagságát karbantartja.

Az Audinate útmutatója szerint VLAN-onként egy IGMP Querier legyen kijelölve.

citeturn0search29

---

# 9.24 Mi az IGMP Querier?

A Querier időszakosan lekérdezi:

> „Melyik eszköz akar továbbra is tagja lenni ennek a multicast groupnak?”

Egyszerűen:

```text
IGMP Querier
      │
      │ Query
      ▼
Receivers
      │
      │ Membership report
      ▼
Switch
```

A switch ez alapján frissíti a multicast forwarding információit.

---

# 9.25 Miért legyen egy Querier VLAN-onként?

Az Audinate ajánlása:

> **One IGMP Querier per VLAN**

citeturn0search29

Ha több, egymással versengő querier van:

```text
Querier A
+
Querier B
+
Querier C
```

akkor a multicast management viselkedése a hálózati implementációtól függően bonyolultabbá válhat.

A tervezési cél:

```text
1 VLAN
   ↓
1 aktív Querier
```

---

# 9.26 IGMP v2 és v3

Az Audinate dokumentáció szerint Dante IGMPv2/v3 használatát támogatja multicast managementhez.

citeturn0search29turn0search30

A tanuló számára a fontosabb kérdés nem az, hogy:

> „Melyik IGMP verziót tudom fejből?”

hanem:

> **„A switch és a Dante hálózat multicast managementje kompatibilis és következetesen konfigurált?”**

---

# 9.27 Mikor kell IGMP?

Az Audinate szerint IGMP:

- vegyes hálózatokban hasznos;
- IP video mellett különösen fontos lehet;
- jelentős multicast audio használat esetén ajánlott;
- kevés vagy nulla multicast audio flow-t használó, Dante-only audio hálózatban nem feltétlenül szükséges.

citeturn0search29

Ezért itt is kerülendő a túl egyszerű szabály:

> „Dante = mindig IGMP.”

Helyesebb:

> **A multicast mennyisége és a hálózat többi forgalma határozza meg, mennyire fontos az IGMP-alapú multicast management.**

---

# 9.28 Multicast és VLAN

A VLAN logikai broadcast domain.

Ha egy Dante hálózatot VLAN-okkal szegmentálunk:

```text
VLAN 10
Dante Audio

VLAN 20
Control

VLAN 30
Other AV
```

akkor figyelni kell:

```text
Multicast scope
+
IGMP Querier
+
Routing
+
Dante discovery
```

A multicast nem „automatikusan megy át” tetszőleges VLAN-határon.

---

# 9.29 Multicast routing

A multicast routing más probléma, mint az egyszerű L2 multicast switching.

Egyetlen VLAN-on:

```text
Source
  ↓
L2 switch
  ↓
Receivers
```

IGMP Snooping lehet elegendő a multicast terítés kezeléséhez.

Ha multicastnak L3 határon kell átmennie:

```text
VLAN A
   ↓
Router / L3
   ↓
VLAN B
```

akkor már multicast routing / routed-Dante architektúrával kapcsolatos kérdések merülnek fel.

Ez nem azonos az egyszerű:

```text
L2 multicast
+
IGMP Snooping
```

problémával. A több subnetet érintő Dante architektúra külön clocking- és routing-mechanizmusokat használhat.

> **Ebben a könyvben most az alap L2 multicast + IGMP modellt tanuljuk meg; a több subnetes / DDM-es Dante architektúrát külön, haladó témaként kezeljük.**

---

# 9.30 Multicast címek Dante esetén

A Dante multicast audio a jelenlegi Audinate hálózati dokumentáció szerint a:

```text
239.255/16
```

tartományt használja, UDP 4321 porttal.

A PTP külön multicast címeket használ:

```text
224.0.1.129 – 224.0.1.132
UDP 319 / 320
```

citeturn0search31

A tanulónak nem kell ezeket minden esetben fejből konfigurálnia, de diagnosztikánál hasznos felismerni:

```text
Multicast audio
≠
PTP multicast
```

---

# 9.31 Dante audio és PTP nem ugyanaz

Ezt különösen fontos megtanulni.

```text
Dante audio
     ↓
media traffic

PTP
     ↓
clock traffic
```

Mindkettő időérzékeny, de különböző funkciót lát el.

```text
Audio
= mit hallunk?

PTP
= mikor játsszuk le?
```

Ezért a QoS-ban is eltérő prioritási szintek jelenhetnek meg.

---

# 9.32 PTP és QoS

Az Audinate által megadott DSCP prioritási modellben:

```text
PTP critical
    ↓
CS7 / 56
```

míg:

```text
Audio + PTP v2
    ↓
EF / 46
```

citeturn0search29

Ezért ha valaki úgy konfigurál QoS-t, hogy:

```text
Audio = high
PTP = normal
```

az veszélyes lehet.

A clock forgalomnak megfelelő prioritást kell kapnia.

---

# 9.33 Mi történik QoS nélkül torlódáskor?

Például:

```text
Video traffic
████████████████████

Backup
████████

Dante
██
```

A kimeneti queue megtelik.

Ha a Dante packet várakozik:

```text
packet
 ↓
queue
 ↓
delay
 ↓
late
```

A packet akár még meg is érkezhet.

De lehet, hogy:

```text
túl későn
```

Ez a real-time audio szempontjából már hiba lehet.

---

# 9.34 Packet loss vs late packet

Nem ugyanaz.

### Packet loss

```text
packet
   ↓
ELVESZETT
```

### Late packet

```text
packet
   ↓
MEGÉRKEZIK
   ↓
DE TÚL KÉSŐN
```

Audio szempontból mindkettő okozhat problémát.

A Dante Controller latency nézete képes late packet eseményeket is megjeleníteni.

citeturn0search4

---

# 9.35 QoS nem javítja a fizikai hibát

Ha a switch port:

```text
CRC error
+
duplex / negotiation problem
+
packet drop
```

akkor nem az a megoldás:

```text
QoS = ON
```

A helyes diagnosztikai sorrend:

```text
Physical
   ↓
Link
   ↓
Switch errors
   ↓
Capacity
   ↓
QoS
   ↓
Application
```

---

# 9.36 EEE – Energy Efficient Ethernet

Az **EEE / IEEE 802.3az**, vagy „Green Ethernet” energia-megtakarítást célzó Ethernet funkció.

Az Audinate kifejezetten azt javasolja, hogy a Dante real-time forgalmát hordozó portokon az EEE legyen kikapcsolva.

Az EEE bizonyos hálózati környezetekben szinkronizációs problémákat és időszakos audio dropokat okozhat. citeturn0search29turn0search30

Egyszerű szabály:

```text
Dante real-time port
        ↓
EEE = Disabled
```

---

# 9.37 EEE és QoS nem ugyanaz

Ne keverd össze:

```text
EEE
=
energiahatékonyság

QoS
=
forgalmi prioritás

IGMP Snooping
=
multicast forwarding management
```

Három külön funkció:

```text
EEE
QoS
IGMP
```

---

# 9.38 Dante és a 70%-os szabály

Az Audinate korábbi hálózati útmutatója azt javasolja, hogy az audioforgalom ne használja egy hálózati link kapacitásának több mint körülbelül 70%-át.

citeturn0search30

Ez **tervezési ökölszabály**, nem Dante Controller által kikényszerített hard limit.

A jelenlegi Dante Controller Status nézete ugyanakkor külön megjegyzi, hogy a jó clock-szinkronizáció érdekében a Rx és Tx utilization lehetőleg ne haladja meg a linksebesség körülbelül 85%-át. A két szám nem ugyanazt jelenti: a 70% konzervatív hálózattervezési ökölszabály, a 85% pedig a Controller clock-szinkronizációs megjegyzése.

A gondolat:

```text
100% utilization
      ↓
nincs tartalék
      ↓
burst / congestion
      ↓
probléma
```

A cél:

```text
használt kapacitás
        <
rendelkezésre álló kapacitás
```

megfelelő biztonsági tartalékkal.

---

# 9.39 Multicast flow-k és flow capacity

A multicast nem „ingyenes”.

A multicast flow:

```text
audio bandwidth
+
switch forwarding state
+
multicast management
```

erőforrást igényel.

Nagy multicast rendszernél tehát nem csak azt kell nézni:

```text
hány receiver?
```

hanem:

```text
hány multicast group?
hány flow?
milyen bandwidth?
melyik VLAN?
melyik uplink?
```

---

# 9.40 Multicast és switch CPU

A multicast forwarding kezelését a switch hardveres vagy szoftveres mechanizmusokkal végezheti, típustól függően.

Ezért nem helyes általánosan azt mondani:

> „A multicast mindig terheli a switch CPU-ját.”

A helyes megközelítés:

> **Ismerd az adott switch multicast forwarding és IGMP Snooping implementációját.**

Ezért lesz fontos a későbbi Aruba switch fejezet.

---

# 9.41 Aruba switch – miért fontos?

A Dante hálózatban az Aruba switch nem csak „kábelosztó”.

Egy managed switchen többek között kezelhetjük:

```text
VLAN
QoS
DSCP
IGMP Snooping
IGMP Querier
EEE
Port statistics
Link speed
Errors
```

Ezért:

```text
Dante Controller
      +
Aruba switch
```

együtt ad teljesebb diagnosztikai képet.

---

# 9.42 Aruba – QoS gondolkodási modell

A konkrét CLI parancsokat a későbbi switch-fejezetben tanuljuk.

Most a logika a fontos:

```text
Dante packet
     ↓
DSCP
     ↓
Aruba classification
     ↓
Queue
     ↓
Priority
```

Ha a Dante DSCP értéke:

```text
56
```

akkor a switch QoS policy-jának ezt az értéket a megfelelő, magas prioritású forgalmi osztályhoz kell rendelnie.

Ugyanez az audio:

```text
DSCP 46
```

esetén a switch konfigurációjától függően a megfelelő audio/PTP queue-ba kerülhet.

A konkrét queue mapping **switch- és konfigurációfüggő**.

---

# 9.43 Aruba – IGMP gondolkodási modell

Multicast esetén:

```text
Dante TX
   ↓
Multicast group
   ↓
Aruba switch
   ↓
IGMP Snooping
   ↓
csak szükséges portok
```

A Querier szerepe:

```text
IGMP Querier
      ↓
membership maintenance
```

A konkrét Aruba konfigurációt később tanuljuk.

---

# 9.44 Hibakeresés – multicast flooding

Tünet:

```text
Multicast audio
+
sok porton látható
```

Gyanú:

```text
IGMP Snooping
```

Ellenőrizd:

```text
1. IGMP Snooping enabled?
2. Querier exists?
3. Correct VLAN?
4. Receiver joined?
5. Multicast forwarding table?
```

---

# 9.45 Hibakeresés – multicast audio nem érkezik

Tünet:

```text
TX
  ↓
Multicast
  ↓
RX = nincs audio
```

Ellenőrzés:

```text
1. Subscription
2. Multicast flow
3. IGMP membership
4. Switch multicast table
5. VLAN
6. Link
7. Signal
```

Ne az IGMP-t állítsd át találomra.

---

# 9.46 Hibakeresés – audio jó, de clock problémás

Tünet:

```text
Audio
= többnyire OK

Clock
= instabil
```

Első gondolat:

```text
PTP
+
QoS
+
switch configuration
```

Ellenőrizd:

```text
Dante Controller
   ↓
Clock Status
```

majd:

```text
Aruba
   ↓
QoS / DSCP
   ↓
Port statistics
```

---

# 9.47 Hibakeresés – minden forgalom jó, amikor nincs backup

Tünet:

```text
Normal:
Dante = OK

Backup running:
Dante = glitches
```

Ez klasszikus congestion/QoS jellegű probléma lehet.

Ellenőrizd:

```text
Backup traffic
      ↓
shared uplink?
      ↓
congestion?
      ↓
QoS?
```

Ha a Dante hálózat csak akkor hibázik, amikor egy másik alkalmazás nagy forgalmat generál, az nagyon erős jel arra, hogy a közös hálózati erőforrásokat kell vizsgálni.

---

# 9.48 Hibakeresés – EEE

Tünet:

```text
alkalmi dropouts
+
furcsa sync problémák
+
nincs nyilvánvaló congestion
```

Ellenőrizd:

```text
EEE
```

A Dante real-time portokon:

```text
EEE = Disabled
```

citeturn0search29turn0search30

---

# 9.49 Diagnosztikai döntési fa

```text
Dante probléma
      │
      ▼
Csak nagy terhelésnél?
      │
 ┌────┴────┐
 IGEN      NEM
 │           │
Congestion  Physical / Clock
QoS         / Network
 │
 ▼
Shared link?
 │
 ▼
QoS / DSCP
```

Multicast problémánál:

```text
Multicast hiba
      ↓
IGMP?
      ↓
Snooping?
      ↓
Querier?
      ↓
VLAN?
      ↓
Forwarding table?
```

---

# 9.50 Labor 1 – QoS nélkül / QoS-szal

Topológia:

```text
PC / Traffic generator
          │
          ▼
     Aruba Switch
       │       │
       ▼       ▼
     Dante   Dante
```

Feladat:

1. Hozz létre kontrollált háttérforgalmat.
2. Figyeld a Dante működését.
3. Ellenőrizd a switch portstatisztikákat.
4. Vizsgáld meg a Dante Controller latency/packet állapotait.
5. Alkalmazd a megfelelő QoS policy-t.
6. Hasonlítsd össze a viselkedést.

Cél:

> Megérteni, hogy a QoS torlódás esetén prioritási mechanizmus, nem kapacitásnövelés.

---

# 9.51 Labor 2 – DSCP megfigyelése

Feladat:

```text
Dante traffic
     ↓
packet capture
```

Vizsgáld meg:

```text
PTP DSCP
Audio DSCP
Other traffic
```

Elvárt modell:

```text
PTP critical → CS7 / 56
Audio + PTP → EF / 46
```

citeturn0search29

---

# 9.52 Labor 3 – Multicast subscription

Topológia:

```text
Dante TX
   │
   ▼
Aruba Switch
   │
   ├── RX 1
   ├── RX 2
   └── RX 3
```

Feladat:

1. Hozz létre multicast flow-t.
2. Csatlakoztass több receivert.
3. Ellenőrizd a multicast subscriptionöket.
4. Figyeld a switch multicast forwarding állapotát.

Cél:

> Megérteni a multicast group működését.

---

# 9.53 Labor 4 – IGMP Snooping

Az Aruba switchen vizsgáld:

```text
IGMP Snooping
```

Majd:

```text
multicast group
```

ellenőrizd, hogy mely portok érdekeltek.

A cél:

```text
Multicast
   ↓
nem minden port
   ↓
csak group members
```

---

# 9.54 Labor 5 – IGMP Querier

Vizsgáld meg:

```text
VLAN
 ↓
IGMP Querier
```

Kérdés:

> Van-e pontosan egy aktív Querier az adott VLAN-ban?

Az Audinate ajánlása szerint VLAN-onként egy IGMP Querier legyen.

citeturn0search29

---

# 9.55 Labor 6 – Multicast flooding

Kontrollált laborban hasonlítsd össze:

```text
IGMP Snooping
OFF
```

és:

```text
IGMP Snooping
ON
```

Figyeld:

```text
port traffic
multicast forwarding
Dante behavior
```

Ne éles hálózaton végezd a kikapcsolást.

---

# 9.56 Labor 7 – QoS + multicast együtt

Topológia:

```text
Dante TX
   │
   ▼
Aruba
   │
   ├── Dante multicast receivers
   └── Other network traffic
```

Vizsgáld:

```text
QoS
+
IGMP Snooping
+
Querier
```

Cél:

> Megérteni, hogy a QoS és a multicast management külön problémát old meg.

---

# 9.57 Labor 8 – EEE

Az Aruba switchen keress egy Dante portot.

Ellenőrizd:

```text
EEE
```

Állapot:

```text
Disabled
```

Jegyezd fel:

```text
Port
EEE state
Link speed
Errors
```

---

# 9.58 Labor 9 – Shared uplink

Topológia:

```text
Access Switch
     │
     │ 1 Gbit/s
     ▼
Core
```

Ugyanezen uplinken:

```text
Dante
+
backup
+
other traffic
```

Feladat:

1. Mérd a normál állapotot.
2. Indíts nagy háttérforgalmat.
3. Figyeld a switch portot.
4. Figyeld Dante Controllerben a latency/packet állapotot.
5. Vizsgáld a QoS hatását.

---

# 9.59 Labor 10 – DSCP remarking

Csak kontrollált laborban.

Feladat:

```text
Dante DSCP
   ↓
Aruba QoS policy
   ↓
classification
```

Vizsgáld meg, hogy a remarking után:

```text
PTP
```

továbbra is megfelelő prioritást kap-e.

Cél:

> Megérteni, hogy a DSCP értékek és a tényleges switch queue-kapcsolat külön konfigurációs réteg.

---

# 9.60 Labor 11 – Packet capture

Wireshark segítségével keress:

```text
PTP
Dante multicast
Dante unicast
IGMP
```

Figyeld:

```text
source
destination
UDP port
DSCP
```

Használd összevetésként az Audinate által dokumentált címeket és portokat.

---

# 9.61 Labor 12 – Teljes diagnosztikai labor

Helyzet:

```text
Dante network
+
Aruba switch
+
shared IT traffic
+
multicast audio
```

Szimulált hiba:

```text
Audio intermittently glitches
```

A tanulónak kell meghatároznia:

```text
Physical?
Congestion?
QoS?
Multicast?
IGMP?
EEE?
Clock?
```

Elvárt munkafolyamat:

```text
Dante Controller
       ↓
Clock / Network / Latency
       ↓
Aruba
       ↓
Port statistics
       ↓
QoS
       ↓
IGMP
       ↓
EEE
       ↓
Root cause
```

---

# 9.62 Vizsgafeladat – QoS

Miért nem helyes azt mondani:

> „Dante-hoz mindig kell QoS”?

**Válasz:**

Mert dedikált, teljesen gigabites Dante-audio hálózaton, megfelelő kapacitással és terheléssel a QoS általában nem szükséges. 100 Mbps vagy vegyes sebességű, illetve mixed-use hálózatokban viszont szükségessé vagy hasznossá válhat.

citeturn0search29turn0search30

---

# 9.63 Vizsgafeladat – DSCP

Melyik érték tartozik az időkritikus PTP eseményekhez?

```text
A) 0
B) 8
C) 46
D) 56
```

**Helyes válasz:**

```text
D) 56
CS7
```

---

# 9.64 Vizsgafeladat – Audio DSCP

Melyik a Dante tipikus audio/PTP v2 DSCP értéke?

```text
A) 0
B) 8
C) 46
D) 56
```

**Helyes válasz:**

```text
C) 46
EF
```

citeturn0search29

---

# 9.65 Vizsgafeladat – Multicast

Mi a fő különbség?

```text
Unicast
Multicast
Broadcast
```

**Válasz:**

```text
Unicast
→ egy sender → egy receiver

Multicast
→ egy sender → egy multicast group → több receiver

Broadcast
→ egy sender → minden érintett host
```

---

# 9.66 Vizsgafeladat – IGMP Snooping

Mi a feladata?

**Válasz:**

A switch az IGMP membership információ alapján meghatározza, mely portokon vannak multicast group tagok, és ezek alapján célzottan továbbíthatja a multicast forgalmat.

---

# 9.67 Vizsgafeladat – Querier

Miért kell?

**Válasz:**

Az IGMP Querier rendszeresen lekérdezi a multicast group tagságot, így a switch fenn tudja tartani az aktuális multicast forwarding állapotot.

Az Audinate ajánlása szerint VLAN-onként egy IGMP Querier legyen.

citeturn0search29

---

# 9.68 Vizsgafeladat – EEE

Miért érdemes kikapcsolni Dante real-time portokon?

**Válasz:**

Az Audinate szerint az EEE bizonyos környezetekben szinkronizációs problémákat és időszakos audio dropokat okozhat.

citeturn0search29turn0search30

---

# 9.69 Vizsgafeladat – QoS és multicast

Igaz vagy hamis?

> „Az IGMP Snooping megoldja a QoS problémát.”

**Hamis.**

```text
IGMP Snooping
=
multicast forwarding management

QoS
=
traffic prioritization
```

Két külön problémát kezelnek.

---

# 9.70 Vizsgafeladat – 1 Gbit/s uplink

Négy access switch port:

```text
4 × 1 Gbit/s
```

egy:

```text
1 Gbit/s
```

uplinkre kerül.

Mi a probléma?

**Válasz:**

Az uplink oversubscribed lehet. QoS segíthet a prioritás kezelésében, de nem teszi az 1 Gbit/s linket 4 Gbit/s kapacitásúvá.

---

# 9.71 Vizsgafeladat – late packet

Mi a különbség?

```text
Packet loss
Late packet
```

**Válasz:**

Packet loss esetén a packet nem érkezik meg. Late packet esetén megérkezik, de a real-time audio számára túl későn.

---

# 9.72 Deep Dive – Miért fontos a PTP magas prioritása?

A Dante audio megfelelő időzítéséhez a receivernek és transmitternek közös, stabil időalapra van szüksége.

Ezért:

```text
PTP
 ↓
clock synchronization
 ↓
audio scheduling
```

Ha a PTP packetek rendszeresen késnek:

```text
clock quality
 ↓
romolhat
```

Ezért a Dante QoS modellben az időkritikus PTP események a legmagasabb prioritási osztályba kerülnek.

citeturn0search29

---

# 9.73 Deep Dive – Miért nem kell minden multicastot tiltani?

A multicast nem önmagában rossz.

A probléma:

```text
uncontrolled multicast
```

A helyes modell:

```text
Multicast
+
IGMP management
+
megfelelő switch
=
kontrollált multicast
```

Ezért nem az a cél, hogy:

> „Multicast = OFF.”

Hanem:

> **„Multicast legyen tervezett és kontrollált.”**

---

# 9.74 Deep Dive – Miért különül el a QoS és IGMP?

Képzeld el:

```text
Multicast
     ↓
kinek küldjük?
```

Erre:

```text
IGMP
```

válaszol.

Másik kérdés:

```text
Ha torlódás van,
ki mehet előbb?
```

Erre:

```text
QoS
```

válaszol.

Tehát:

```text
IGMP
= WHERE / WHO

QoS
= WHEN / PRIORITY
```

Ez nagyon jó mentális modell.

---

# 9.75 Deep Dive – Miért kell Aruba oldalon is ellenőrizni?

A Dante Controller ezt mutathatja:

```text
Subscription = OK
```

de az Aruba switch oldalon lehet:

```text
port errors
+
drops
+
congestion
```

Vagy:

```text
IGMP membership
```

nem megfelelő.

Ezért a teljes diagnosztikai lánc:

```text
Dante Controller
       +
Aruba
       +
packet capture
```

---

# 9.76 Deep Dive – QoS queue nem ugyanaz, mint DSCP

A DSCP:

```text
classification mark
```

A queue:

```text
switch forwarding resource
```

A kapcsolat:

```text
DSCP
 ↓
classification
 ↓
queue
 ↓
scheduler
 ↓
egress
```

Ha a mapping hibás:

```text
DSCP = 56
```

önmagában nem garantálja, hogy a packet a kívánt queue-ban lesz.

---

# 9.77 Deep Dive – Mixed-use hálózat

Egy vállalati hálózatban lehet:

```text
Dante
CCTV
VoIP
Wi-Fi
Servers
Backup
Video
Office
```

Ekkor a Dante nem feltétlenül kap külön fizikai hálózatot.

A helyes tervezés:

```text
VLAN
+
QoS
+
Multicast management
+
Capacity planning
+
Monitoring
```

Ez az egyik oka annak, hogy a Dante jól illeszkedhet standard IT Ethernet infrastruktúrába, de csak megfelelő hálózattervezéssel.

---

# 9.78 Deep Dive – Mikor válassz multicastot?

Gondolkodj így:

```text
1 → 1
```

általában természetes unicast eset.

```text
1 → 2
```

mindkettő lehet ésszerű.

```text
1 → 20
```

már érdemes megvizsgálni a multicastot.

```text
1 → 100
```

multicast tervezése kifejezetten fontos lehet.

Ez nem merev szabály.

A tényleges döntést a:

```text
flow count
bandwidth
receiver count
network topology
switch capability
```

együttesen határozza meg.

---

# 9.79 Deep Dive – Multicast és bandwidth

Fontos megérteni, hogy multicast esetén sem „tűnik el” a bandwidth.

A forrás oldalán:

```text
1 stream
```

indulhat.

A hálózatban azonban az elágazási pontokon a stream több kimeneti linkre is továbbítódhat.

Ezért:

```text
Core
  │
  ├── Access A
  ├── Access B
  └── Access C
```

mindhárom irányban megjelenhet ugyanaz a multicast forgalom.

A multicast tehát elsősorban a **felesleges, párhuzamos forrásoldali streamelést** csökkentheti; nem jelenti azt, hogy minden hálózati linken csak egyszer kell figyelembe venni a sávszélességet.

---

# 9.80 Deep Dive – A multicast legfontosabb veszélye

A rosszul kezelt multicast:

```text
Multicast
   ↓
flooding
   ↓
unnecessary traffic
   ↓
congestion
   ↓
Dante problems
```

Vagyis:

> **A multicast egyszerre lehet nagyon hatékony és nagyon veszélyes.**

A különbséget a hálózati kontroll adja.

---

# 9.81 Gyakorlati ellenőrzőlista – QoS

Ha Dante QoS-t vizsgálsz:

```text
□ Link speeds
□ Shared links
□ Congestion points
□ DSCP recognition
□ PTP priority
□ Audio priority
□ Strict priority queueing
□ Queue mapping
□ Port statistics
□ Packet drops
```

---

# 9.82 Gyakorlati ellenőrzőlista – Multicast

```text
□ Multicast flows
□ Receiver count
□ Group membership
□ IGMP Snooping
□ IGMP Querier
□ VLAN
□ Multicast forwarding
□ Uplink capacity
□ Flooding
□ Switch capability
```

---

# 9.83 Gyakorlati ellenőrzőlista – Dante + Aruba

```text
Dante Controller
□ Clock Status
□ Network Status
□ Latency
□ Flow Information
□ Subscription

Aruba
□ Link speed
□ Errors
□ Drops
□ QoS
□ DSCP
□ IGMP Snooping
□ Querier
□ EEE
□ VLAN
```

A két oldal eredményeit együtt értékeld.

---

# 9.84 A fejezet mentális modellje

```text
                    DANTE NETWORK
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
          UNICAST                   MULTICAST
             │                         │
       one-to-one              one-to-many
                                       │
                                       ▼
                                     IGMP
                                       │
                                       ▼
                              Multicast forwarding
                                       │
                                       ▼
                                     SWITCH
                                       │
                       ┌───────────────┴───────────────┐
                       ▼                               ▼
                     QoS                             VLAN
                       │
                 DSCP marking
                       │
                 Queue priority
                       │
                 Strict priority
                       │
                       ▼
                 Real-time traffic
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
           PTP                  AUDIO
        high priority         prioritized
```

---

# 9.85 Amit ebből a fejezetből tudnod kell

### QoS

Forgalompriorizálási mechanizmus, amely torlódás esetén segít az időkritikus forgalom megfelelő kezelésében.

### DSCP

Az IP-forgalom osztályozására használt jelölés.

### Dante tipikus DSCP értékek

```text
CS7 / 56
→ időkritikus PTP

EF / 46
→ audio + PTP v2

CS1 / 8
→ fenntartott

Best Effort / 0
→ egyéb
```

citeturn0search29

### Strict Priority

A magasabb prioritású queue előnyt kap a kiszolgálásnál.

### Multicast

Egy sender egy multicast group felé küldhet, amelyhez több receiver csatlakozik.

### IGMP

A multicast group tagság kezelésére szolgáló IPv4 protokoll.

### IGMP Snooping

A switch multicast forwardingját a group membership alapján optimalizálja.

### IGMP Querier

A multicast group tagság karbantartásához queryket küld.

### EEE

Energiatakarékossági funkció, amelyet Dante real-time portokon az Audinate ajánlása szerint ki kell kapcsolni.

### Aruba

A switch oldalon itt jelenik meg a:

```text
QoS
DSCP
IGMP
VLAN
EEE
Port statistics
```

gyakorlati megvalósítása.

---

# 9.86 A legfontosabb szabályok

```text
1. QoS nem növeli a link kapacitását.
2. Dante esetén nem mindig szükséges QoS.
3. Ha QoS-t használsz, a PTP megfelelő prioritása kritikus.
4. DSCP önmagában nem jelent működő QoS-t.
5. A multicast nem broadcast.
6. IGMP Snooping a multicast forwardingot kontrollálja.
7. IGMP Querier fontos a multicast membership karbantartásához.
8. VLAN-onként egy aktív Querier legyen.
9. EEE legyen kikapcsolva Dante real-time portokon.
10. A multicastot tervezni és monitorozni kell.
11. A switch kapacitása fontosabb, mint egyetlen QoS checkbox.
12. Dante Controller és Aruba switch együtt ad jó diagnosztikai képet.
```

---

# 9.87 Következő fejezet

# 10. Dante hálózati topológiák és redundancia

A 9. fejezetben megértettük:

```text
QoS
+
DSCP
+
Multicast
+
IGMP
+
Switch
```

A következő kérdés:

> **Hogyan építsünk fel egy Dante hálózatot úgy, hogy ne csak működjön, hanem megfelelően skálázható és hibatűrő is legyen?**

A következő fejezetben:

```text
Star
     ↓
Core / Access
     ↓
Redundancy
     ↓
Primary / Secondary
     ↓
Network topology
```

---

# 9.88 Fejezeti állapot

**Állapot: REVIEWED / JAVÍTOTT – commit előtti ellenőrzésre kész**

A fejezet tartalmaz:

- QoS alapfogalmak;
- DSCP;
- Dante prioritási osztályok;
- strict priority;
- QoS szükségessége;
- mixed-use hálózat;
- oversubscription;
- multicast;
- unicast vs multicast;
- multicast bandwidth;
- IGMP;
- IGMP Snooping;
- IGMP Querier;
- IGMP v2/v3;
- multicast és VLAN;
- multicast routing alapok;
- Dante multicast címzés;
- PTP és audio prioritás;
- late packet;
- EEE;
- 70%-os tervezési ökölszabály;
- Aruba switch szerepe;
- 12 labor;
- vizsgafeladatok;
- Deep Dive részek;
- hibakeresési workflow;
- Dante Controller + Aruba diagnosztika.

A Dante-specifikus QoS, multicast, IGMP és EEE állításokat az Audinate hálózati adminisztrátori dokumentációjával és kapcsolódó hivatalos dokumentációval ellenőriztem.
