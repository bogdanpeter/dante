# 10. Dante hálózati topológiák és redundancia

> **A fejezet célja:** megérteni, hogyan épül fel egy Dante hálózat fizikailag és logikailag, hogyan különbözik a normál és a redundáns Dante működés, mit jelent a Primary és Secondary hálózat, milyen hibák ellen véd a redundancia, és hogyan kell a hibát úgy keresni, hogy ne keverjük össze az audio-, clock-, switch- és vezetékezési problémákat.

Az előző fejezetekben megtanultuk:

```text
Dante
 ├── Audio
 ├── PTP / clock
 ├── Control
 └── Discovery
```

majd:

```text
QoS
DSCP
Multicast
IGMP
Switch
```

Most ezekből építünk komplett hálózati topológiát.

A kulcskérdés:

> **Hogyan lesz egy működő Dante hálózatból jól megtervezett Dante infrastruktúra?**

---

# 10.1 Mi az a hálózati topológia?

A topológia a hálózati elemek és kapcsolataik felépítése.

Egyszerű Dante rendszer:

```text
Dante A
   │
   ▼
Switch
   │
   ├── Dante B
   ├── Dante C
   └── Dante D
```

Ez egy tipikus **csillagtopológia**.

A központi switchhez csatlakozik minden végpont.

A Dante szempontjából ez fontos, mert a switch:

- továbbítja az Ethernet-forgalmat;
- kezeli a VLAN-okat;
- szükség esetén QoS-t alkalmaz;
- multicast esetén IGMP Snoopingot használhat;
- és a teljes rendszer központi kapcsolódási pontja lehet.

---

# 10.2 Miért jó a csillagtopológia?

A csillag egyik nagy előnye a hibák lokalizálhatósága.

Ha:

```text
Dante A
   │
   X
   │
Switch
```

akkor elsősorban A és a switch közötti linket kell vizsgálni.

Ha viszont egy hosszú láncban:

```text
A ─ B ─ C ─ D ─ E
```

egy közbenső kapcsolat hibásodik meg, az utána következő eszközök is kieshetnek.

Ezért a professzionális hálózatokban a központi, jól menedzselhető switch-alapú topológia általában előnyös.

---

# 10.3 Egyetlen switch

A legegyszerűbb Dante rendszer:

```text
              ┌── Dante 1
              │
              ├── Dante 2
              │
Dante switch ─┼── Dante 3
              │
              ├── Dante 4
              │
              └── Dante 5
```

Előnye:

- egyszerű;
- olcsó;
- könnyen konfigurálható;
- könnyen hibakereshető.

Hátránya:

> **A switch egyetlen hibapont.**

Ha a switch teljesen meghibásodik:

```text
Switch OFF
   ↓
minden hozzá kapcsolódó Dante kapcsolat megszakad
```

---

# 10.4 Single Point of Failure

A **single point of failure**, röviden SPOF, olyan komponens, amelynek egyetlen hibája az egész rendszer vagy egy jelentős része kiesését okozhatja.

Egy egyszerű rendszerben:

```text
Dante devices
      │
      ▼
   SWITCH
      │
      ▼
   mindenki
```

A switch SPOF.

De SPOF lehet:

```text
switch
uplink
fiber
power supply
network interface
server
```

is.

A redundáns tervezés célja nem az, hogy minden komponensből kettő legyen.

A cél:

> **A kritikus egyetlen hibapontok eltávolítása.**

---

# 10.5 Redundancia ≠ „mindenből kettő”

Ez nagyon fontos.

A redundancia nem pusztán:

```text
2 switch
```

hanem:

```text
2 független útvonal
+
2 megfelelően kialakított hálózati oldal
+
redundanciát támogató Dante eszköz
```

Ha két switch van, de:

```text
mindkettő ugyanarról az egyetlen tápegységről
```

működik, akkor a tápellátás továbbra is SPOF lehet.

Ha két hálózati kábel ugyanabban a kábelcsatornában fut, egy fizikai sérülés mindkettőt elvághatja.

Ezért a redundancia **rendszerszintű tervezési kérdés**.

---

# 10.6 Dante Primary és Secondary

A redundáns Dante eszközök két hálózati interfészt használhatnak:

```text
Primary
Secondary
```

Egyszerű modell:

```text
Dante Device
 ┌───────────────┐
 │               │
 │ Primary       │──── Primary Network
 │               │
 │ Secondary     │──── Secondary Network
 │               │
 └───────────────┘
```

Redundáns konfigurációban a Primary és Secondary hálózatok külön hálózati utak. Az Audinate Dante Controller dokumentációja külön Primary és Secondary hálózatot mutat, a két switch közötti közvetlen összeköttetés nélkül.

---

# 10.7 A klasszikus redundáns Dante topológia

```text
                 PRIMARY
                    │
             ┌──────┴──────┐
             │ Primary SW  │
             └──────┬──────┘
                    │
       ┌────────────┼────────────┐
       │            │            │
      P1           P2           P3


                 SECONDARY
                    │
             ┌──────┴──────┐
             │ Secondary SW│
             └──────┬──────┘
                    │
       ┌────────────┼────────────┐
       │            │            │
      S1           S2           S3
```

A Dante eszközök:

```text
P → Primary switch
S → Secondary switch
```

kapcsolatot kapnak.

A két hálózatot nem úgy kell elképzelni, mint két egymással összekötött switch hálózatot.

A redundancia lényege éppen az, hogy **két külön út áll rendelkezésre**.

---

# 10.8 Mi történik normál működésben?

Redundáns Dante eszköz esetén a Dante médiaforgalom mindkét hálózati interfészen továbbítódik.

Redundant módban a készülék a Dante médiaforgalmat mindkét Ethernet porton továbbítja, ezzel biztosítva a Secondary hálózatot is.

Fontos: ez nem azt jelenti, hogy a receiver ugyanazt az audioadatot kétszer hallja. A redundancia hálózati szinten működik; a redundáns packetekből a receiver a szükséges médiát állítja elő.

Egyszerű modell:

```text
Audio packet
    │
    ├──── Primary
    │
    └──── Secondary
```

---

# 10.9 Mi történik egy link hibájánál?

Tegyük fel:

```text
Primary
   X
```

A Secondary továbbra is él:

```text
Secondary
   │
   ▼
Dante
```

A redundáns eszköz így továbbra is képes hálózati kapcsolatot fenntartani a másik oldalon.

Redundáns Dante hálózatban a clock synchronization is mindkét hálózaton működik; egy hálózati oldal hibája esetén a redundáns eszköz a másik hálózaton továbbra is kaphat clock synchronization információt.

Ez a redundancia egyik legfontosabb tulajdonsága:

```text
Primary failure
      ↓
Secondary remains
      ↓
Audio + clock continuity
```

---

# 10.10 Redundancia és failover

A két fogalom összefügg, de nem azonos.

**Redundancia:**

> Van egy második működő út.

**Failover:**

> Hiba esetén a rendszer a rendelkezésre álló másik útra támaszkodik.

Tehát:

```text
Redundancy
   ↓
van alternatív út

Failure
   ↓
az első út kiesik

Failover behavior
   ↓
a második út biztosítja a működést
```

Dante esetén ezért nem elég azt mondani:

> „Van két kábel.”

Azt kell megérteni:

> **A két kábel két valóban külön hálózati út része-e, és az adott Dante eszköz támogatja-e a redundáns működést?**

---

# 10.11 A két switch összekötésének veszélye

A klasszikus Dante redundáns topológiában a Primary és Secondary hálózat két külön hálózati út:

```text
Primary switch       Secondary switch
      │                     │
      │                     │
 Primary                Secondary
 network                  network
```

Ezeket **nem szabad egyszerűen összekötni** csak azért, hogy „legyen kapcsolat”.

A Dante redundáns modellben az izolált Primary és Secondary hálózat a kívánt működés része. Az Audinate tipikus redundáns konfigurációja külön Primary és Secondary hálózatot mutat, a két switch közötti közvetlen összeköttetés nélkül.

Egy hálózati mérnök természetesen kialakíthat összetettebb, redundáns switch-fabricet, de annak működését a konkrét switch-platform, VLAN- és loop-prevention mechanizmusok alapján kell megtervezni. Ez **nem ugyanaz**, mint a két Dante redundáns hálózat egyszerű összekötése.

Kezdő Dante-rendszerben ezért az alap szabály:

> **Primary és Secondary: két külön hálózati út, nem egy összekötött hurok.**

---

# 10.12 Redundáns és Switched mód

Nem minden kétportos Dante eszköz redundáns.

Az Audinate Dante Controller Network Config dokumentációja szerint támogatott eszközökön az Ethernet portok között választható lehet:

```text
Redundant
```

vagy:

```text
Switched
```

Redundant módban a két Ethernet interfész a redundáns Dante hálózathoz használható.

Switched módban a második Ethernet port standard switch portként viselkedhet, így például daisy-chain kialakítás is lehetséges.

Ezért:

> **Két Ethernet port ≠ automatikusan redundancia.**

Mindig ellenőrizni kell az adott készülék dokumentációját és konfigurációját.

---

# 10.13 Redundancia vs daisy-chain

Két teljesen különböző modell:

## Redundáns

```text
Device
 ├── Primary → Switch A
 └── Secondary → Switch B
```

## Switched / daisy-chain

```text
Switch
   │
Device A
   │
Device B
   │
Device C
```

A második esetben a készülék második portja switchként működhet.

Ez nem ugyanaz a hibatűrés.

Ha:

```text
A ─ X ─ B
```

akkor B és C is kieshet a hálózatról.

---

# 10.14 Mikor érdemes redundáns Dante hálózatot használni?

Nem minden rendszernek kell.

Érdemes megfontolni, ha:

- a hangrendszer kiesése üzletileg vagy biztonságtechnikailag kritikus;
- nagy rendezvényről van szó;
- broadcast / production környezetben dolgozunk;
- színházi infrastruktúra kritikus;
- voice alarm vagy más magas rendelkezésre állású rendszer része;
- a rendszer leállása jelentős következménnyel jár.

Egy kis próbateremben:

```text
2 Dante endpoint
+
1 switch
```

esetén a teljes redundáns infrastruktúra lehet indokolatlanul drága.

A helyes kérdés:

> **Mennyibe kerül a redundancia, és mennyibe kerül a kiesés?**

---

# 10.15 Redundancia és költség

Egy redundáns hálózat több erőforrást igényel:

```text
+ switch
+ kábelezés
+ portok
+ rack space
+ power
+ konfiguráció
+ monitoring
```

Ezért:

```text
availability
      ↕
cost
```

között kompromisszum van.

A cél nem a maximális redundancia mindenáron.

A cél:

> **A szükséges rendelkezésre állás gazdaságosan megvalósítva.**

---

# 10.16 Redundancia és fizikai infrastruktúra

A logikai redundancia csak akkor ér valamit, ha fizikailag is külön úton halad.

Rossz:

```text
Primary cable ─┐
               ├── same cable tray
Secondary ─────┘
```

Ha a kábelcsatorna sérül:

```text
Primary X
Secondary X
```

Mindkettő kiesik.

Jobb:

```text
Primary → út A
Secondary → út B
```

akár külön racken, kábelnyomvonalon vagy más fizikai útvonalon.

---

# 10.17 Tápellátási redundancia

Ugyanez igaz a switchre.

Két hálózat:

```text
Switch A
Switch B
```

de:

```text
Power
  │
  └── one UPS
```

esetén az UPS továbbra is közös hibapont.

Magas rendelkezésre állású rendszerben:

```text
Power A
   ↓
Switch A

Power B
   ↓
Switch B
```

lehet a cél.

A tényleges kialakítást az adott helyszín és üzleti követelmény határozza meg.

---

# 10.18 Redundáns hálózat és PTP

A redundanciánál nem csak az audio számít.

A clock is kritikus.

Redundáns Dante hálózatban a clock synchronization protocol mind a Primary, mind a Secondary hálózaton működik. Mindkét hálózaton van PTP leader clock; normál esetben ez ugyanaz az eszköz. citeturn0search0

Mentális modell:

```text
Primary PTP
    +
Secondary PTP
    ↓
redundáns clock path
```

Ha az egyik hálózat kiesik:

```text
PTP path A → X
PTP path B → OK
```

akkor a clock synchronization fennmaradhat a másik hálózaton.

---

# 10.19 Miért fontos ez?

Audio és clock összefügg:

```text
Audio packets
      +
stable clock
      ↓
reliable playback
```

Ha az audio út fennmaradna, de a clock teljesen eltűnne, a rendszer továbbra sem lenne egészséges.

Ezért redundáns Dante rendszer ellenőrzésekor mindig kérdezd:

```text
Audio path?
Clock path?
Control path?
```

---

# 10.20 Dante Controller és redundancia

A Dante Controller külön képes megjeleníteni a Primary és Secondary hálózat állapotát.

A Network Status nézetben többek között látható:

```text
Primary status
Primary link speed
Secondary status
Secondary link speed
Primary bandwidth
Secondary bandwidth
```

Ez rendkívül hasznos hibakereséshez.

Például:

```text
Primary = 1 Gbps
Secondary = Link Down
```

nem feltétlenül audiohiba, ha a rendszer redundanciája nincs használatban.

De egy redundáns rendszerben ez azt jelenti:

> **A redundáns út jelenleg nem áll rendelkezésre.**

---

# 10.21 Egy hálózati oldal kiesése

Normál állapot:

```text
Primary    = UP
Secondary  = UP
```

Hiba után:

```text
Primary    = DOWN
Secondary  = UP
```

A cél:

```text
Audio = tovább működik
Clock = tovább működik
```

A Controllerben viszont hibajelzés jelenhet meg.

Ez fontos különbség:

> **A rendszer működése és a rendszer egészséges redundáns állapota nem ugyanaz.**

---

# 10.22 Ezért veszélyes a „még szól” gondolkodás

Ha:

```text
Primary = DOWN
Secondary = UP
Audio = OK
```

az nem feltétlenül jelent teljesen egészséges rendszert.

Valójában:

```text
redundancy margin
      ↓
csökkent
```

Ha most a Secondary is kiesik:

```text
Primary X
Secondary X
```

akkor már nincs tartalék.

Ezért a hibát **az első hiba után** kell kijavítani, nem csak akkor, amikor már elnémult a rendszer.

---

# 10.23 Redundancia és monitoring

Egy jó monitoring rendszer nem csak azt kérdezi:

> „Van audio?”

Hanem:

```text
Primary OK?
Secondary OK?
Clock OK?
Link speed OK?
Errors?
Drops?
Bandwidth?
```

Ez a különbség a:

```text
reactive
```

és:

```text
proactive
```

üzemeltetés között.

---

# 10.24 Redundancia és link speed

A redundáns kapcsolat nem mentesít a sebesség ellenőrzése alól.

Például:

```text
Primary = 1 Gbps
Secondary = 100 Mbps
```

Lehet, hogy a kapcsolat fizikailag működik, de a Secondary oldal nem az elvárt kapacitással üzemel.

A Dante Controller Device Info és Network Status nézetei képesek megjeleníteni a Primary és Secondary link speed értékeket.

Ezért:

> **UP ≠ megfelelő.**

---

# 10.25 Redundancia és bandwidth

Ha a redundáns eszköz a media trafficet mindkét hálózaton továbbítja:

```text
Primary traffic
+
Secondary traffic
```

akkor **mindkét hálózati oldalnak** megfelelő kapacitással kell rendelkeznie.

Nem szabad azt gondolni:

> „A Secondary csak tartalék, ezért elég egy nagyon lassú hálózat.”

A redundáns útvonalnak képesnek kell lennie a szükséges Dante-forgalom kezelésére.

A redundáns hálózatot tehát nem úgy kell méretezni, hogy a Secondary „csak vészhelyzetben legyen használható”. A hibatűréshez az alternatív útnak ténylegesen alkalmasnak kell lennie a szükséges forgalom továbbítására.

---

# 10.26 Redundáns hálózat és multicast

Multicast esetén a redundancia különösen gondos tervezést igényel.

A két hálózatnak:

```text
Primary multicast
+
Secondary multicast
```

oldalon is megfelelően kell kezelnie a forgalmat.

A Dante redundancia nem jelenti azt, hogy a hálózati multicast problémák automatikusan megoldódnak.

Továbbra is vizsgálni kell:

```text
IGMP
Snooping
Querier
VLAN
switch behavior
```

A redundáns hálózat nem helyettesíti a megfelelő multicast-tervezést.

---

# 10.27 DDM és redundancia

Fontos különválasztani:

```text
Dante device redundancy
```

és:

```text
Dante Domain Manager High Availability
```

fogalmát.

A DDM High Availability a DDM szerver működésének redundanciája.

Az Audinate dokumentációja kifejezetten jelzi, hogy a DDM High Availability **nem azonos** a Dante eszközök Primary/Secondary redundanciájával.

Tehát:

```text
Dante Redundancy
=
media/network redundancy

DDM HA
=
server/control-plane availability
```

Ne keverd össze a kettőt.

---

# 10.28 DDM és több subnet

A DDM-es redundáns hálózat már haladó téma.

Az Audinate jelenlegi DDM dokumentációja szerint a redundáns media **nem támogatott különböző subnetekben lévő eszközök között**. A redundancia subneten belül támogatott, és a Primary/Secondary subneteknek megfelelően kell tükrözniük egymást.

Ezért az alapmodell:

```text
Primary subnet
     +
Secondary subnet
```

ahol a redundáns eszköz Primary és Secondary interfészei a megfelelő, egymáshoz tartozó hálózatokra csatlakoznak.

Fontos: ez nem azt jelenti, hogy egy DDM-rendszerben egyáltalán nem lehet több Primary subnet. Az Audinate támogat olyan DDM-architektúrát, ahol több Primary subnet között media routing működik, miközben a redundáns Secondary hálózatok subnetenként elkülönülnek.

A több subnetes DDM architektúrát ezért külön, haladó témaként kell kezelni.

---

# 10.29 Tipikus topológia – kis rendszer

```text
             ┌───────────┐
             │  Switch   │
             └─────┬─────┘
                   │
       ┌───────────┼───────────┐
       │           │           │
      DSP         Console     Stagebox
```

Ez:

- egyszerű;
- olcsó;
- könnyen hibakereshető.

Ha nincs kritikus rendelkezésre állási követelmény, sok esetben teljesen megfelelő.

---

# 10.30 Tipikus topológia – redundáns kis/közepes rendszer

```text
             PRIMARY
                │
          ┌─────┴─────┐
          │  Switch A │
          └─────┬─────┘
                │
          ┌─────┴─────┐
          │  Devices  │
          └─────┬─────┘
                │
          ┌─────┴─────┐
          │  Switch B │
          └─────┬─────┘
                │
           SECONDARY
```

Fontos: a készülékek Primary és Secondary portjai külön hálózatra mennek.

A két switch hálózata nem egyszerűen egyetlen közös switchként működik.

---

# 10.31 Core / Access topológia

Nagyobb rendszer:

```text
             CORE
          ┌────┴────┐
          │         │
       Access A   Access B
        / | \\      / | \
       D  D  D    D  D  D
```

Itt már az uplinkek a kritikus pontok.

Ha:

```text
Access A
   │
   X
   │
Core
```

akkor az Access A mögötti eszközök kieshetnek.

Ezért nagy rendszerben az uplink redundanciája is fontos.

---

# 10.32 Két core switch

Magasabb rendelkezésre állású infrastruktúra például:

```text
        Core A       Core B
          │           │
          └─────┬─────┘
                │
           Access layer
```

Itt azonban már **általános Ethernet hálózati redundanciáról** beszélünk.

A konkrét megoldás lehet például:

```text
LACP
MLAG
stacking
VSX
STP
```

de ezek nem felcserélhető technológiák, és támogatásuk, szerepük, konfigurációjuk **switch-platformfüggő**.

Különösen fontos:

> **A switch-fabric redundanciája nem automatikusan Dante Redundancy.**

Egy switchgyártó saját redundancia-mechanizmusa az Ethernet infrastruktúra rendelkezésre állását javíthatja, miközben a Dante eszköz Primary/Secondary működését külön kell megtervezni.

---

# 10.33 Dante redundancia vs switch redundancy

Nagyon fontos különbség.

### Dante redundancy

```text
Dante device
 ├── Primary
 └── Secondary
```

### Switch redundancy

```text
Switch A
   +
Switch B
   +
redundant uplinks / control
```

Ezek kombinálhatók.

Egy nagy rendszerben például:

```text
Dante device
 ├── Primary → Switch fabric A
 └── Secondary → Switch fabric B
```

Ez sokkal nagyobb hibatűrést adhat, mint egyetlen redundáns Dante link.

---

# 10.34 Redundáns rendszer tervezési elve

A jó redundáns tervnél előre meg kell határozni, **milyen hibákat kell túlélnie a rendszernek**.

Például:

```text
Primary cable failure
       ↓
Secondary path remains

Primary switch failure
       ↓
Secondary path remains
```

Egy magasabb rendelkezésre állású infrastruktúrában további cél lehet:

```text
uplink failure
power-path failure
rack failure
```

túlélése is.

De egyik sem automatikus.

A valós rendszer csak addig redundáns, amíg az adott hibából valóban megmarad egy működő, megfelelő kapacitású és megfelelően konfigurált alternatív útvonal.

Ezért a redundanciát mindig **hibamódonként** kell megtervezni.

---

# 10.35 Hibaelemzés – mi esett ki?

Ha redundáns Dante rendszerben probléma van, először ne állíts át semmit.

Kérdezd:

```text
Mi esett ki?
```

Lehet:

```text
device
cable
port
switch
uplink
VLAN
Primary network
Secondary network
clock
```

---

# 10.36 Hibaelemzés – Primary kiesés

Tünet:

```text
Primary = Link Down
Secondary = Up
Audio = OK
```

Első következtetés:

> A redundancia valószínűleg működött.

De még nem kész a diagnózis.

Vizsgáld:

```text
Primary cable
Primary switch port
Primary switch
Dante device Primary interface
```

---

# 10.37 Hibaelemzés – Secondary kiesés

Tünet:

```text
Primary = Up
Secondary = Link Down
Audio = OK
```

Ez lehet:

- Secondary kábelhiba;
- switch porthiba;
- Secondary switch probléma;
- Dante Secondary interfész problémája;
- konfigurációs probléma.

A rendszer még működhet, de a redundancia sérült.

---

# 10.38 Hibaelemzés – mindkét hálózat jó, mégis nincs audio

Ha:

```text
Primary = Up
Secondary = Up
```

de nincs audio, akkor ne automatikusan a redundanciát hibáztasd.

Vizsgáld:

```text
Subscription
Clock
Flow
VLAN
QoS
Multicast
Device state
```

A redundancia csak egy lehetséges tényező.

---

# 10.39 Hibaelemzés – csak egy eszköz hibás

Ha:

```text
Dante A = OK
Dante B = OK
Dante C = FAIL
Dante D = OK
```

akkor valószínűbb:

```text
C device
C cable
C port
C configuration
```

mint egy teljes hálózati topológiai hiba.

A hibakeresés egyik alapelve:

> **A hiba hatóköréből következtess a hibaforrásra.**

---

# 10.40 Hibaelemzés – minden eszköz hibás

Ha:

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

szintű probléma.

Ez sokkal hatékonyabb gondolkodás, mint egyesével újrakonfigurálni minden Dante eszközt.

---

# 10.41 Dante Controller – Network Status

A Network Status nézetben vizsgáld:

```text
Primary Status
Secondary Status
Primary Link Speed
Secondary Link Speed
Primary Tx
Primary Rx
Secondary Tx
Secondary Rx
```

Egy redundáns rendszerben ez az egyik legfontosabb első diagnosztikai képernyő.

---

# 10.42 Dante Controller – Device Info

A Device Info nézetben látható többek között:

```text
Primary Address
Primary Link Speed
Secondary Address
Secondary Link Speed
```

A Secondary mezőben a:

```text
N/A
```

azt jelzi, hogy az adott eszköz nem támogat másodlagos interfészt, míg:

```text
Link Down
```

arra utal, hogy támogatja, de nincs megfelelő kapcsolat.

Ez kezdőként nagyon hasznos különbség.

---

# 10.43 Dante Controller – Clock Status

Redundáns hálózatban a Clock Status különösen fontos.

A redundáns clock synchronization mindkét hálózaton működik.

Ezért ellenőrizd:

```text
Primary clock status
Secondary clock status
Leader
Follower
Listening
Link Down
```

A tartós „Listening” például nem normális stabil állapot.

---

# 10.44 Redundancia tesztelése

A redundáns rendszert **tesztelni kell**.

Nem elég azt mondani:

> „Van két kábel, tehát redundáns.”

Teszt:

```text
Normal
  ↓
Primary link
  ↓
DISCONNECT
  ↓
observe
```

Figyeld:

```text
Audio
Clock
Controller
Secondary
```

Majd:

```text
Reconnect Primary
```

és ellenőrizd a helyreállást.

---

# 10.45 Redundancia teszt – switch failure

Magas rendelkezésre állású laborban:

```text
Primary switch
      X
```

A cél:

```text
Secondary
   ↓
continues
```

Ezután:

```text
Primary switch
   ↓
restore
```

Figyeld, hogy a rendszer visszatér-e normál redundáns állapotba.

---

# 10.46 Redundancia teszt – kábelhiba

Teszt:

```text
Primary cable
      X
```

Ellenőrzés:

```text
Primary = Down
Secondary = Up
Audio = OK
```

Ezután a kábel visszakötése után:

```text
Primary = Up
Secondary = Up
```

legyen az elvárt állapot.

---

# 10.47 Redundancia teszt – Secondary oldal

Ugyanez fordítva:

```text
Secondary cable
      X
```

Ellenőrizd:

```text
Primary = Up
Secondary = Down
Audio = OK
```

A cél annak bizonyítása, hogy **mindkét irányú egyetlen hiba túlélhető**.

---

# 10.48 Labor 1 – Egyszerű Dante topológia

Építs:

```text
1 switch
4 Dante device
```

Feladat:

1. Ellenőrizd a linkeket.
2. Hozz létre audio subscriptionöket.
3. Ellenőrizd a Clock Status állapotot.
4. Dokumentáld a topológiát.

Cél:

> Megérteni az alap csillagtopológiát.

---

# 10.49 Labor 2 – SPOF azonosítása

Az előző topológián jelöld:

```text
SPOF
```

Kérdés:

> Mi történik, ha a switch meghibásodik?

Írd le:

```text
affected devices
affected flows
clock
control
```

---

# 10.50 Labor 3 – Redundáns Dante

Építs:

```text
Switch A = Primary
Switch B = Secondary
```

Csatlakoztass redundáns Dante eszközöket:

```text
Primary → A
Secondary → B
```

A két switch ne legyen egyszerűen összekötve.

---

# 10.51 Labor 4 – Primary link failure

Normál állapot:

```text
P = UP
S = UP
```

Húzd ki:

```text
Primary cable
```

Figyeld:

```text
Audio
Clock
Controller
```

Dokumentáld:

```text
before
during
after
```

---

# 10.52 Labor 5 – Secondary link failure

Ugyanezt végezd el:

```text
Secondary cable
```

hibával.

Cél:

> Megérteni, hogy a redundancia nem csak egy irányú teszt.

---

# 10.53 Labor 6 – Primary switch failure

Kontrollált körülmények között szüntesd meg a Primary switch működését.

Figyeld:

```text
Primary devices
Secondary devices
Audio
Clock
Controller
```

A cél a teljes hálózati oldal kiesésének vizsgálata.

---

# 10.54 Labor 7 – Link speed

Ellenőrizd:

```text
Primary Link Speed
Secondary Link Speed
```

Hasonlítsd össze:

```text
1 Gbps
100 Mbps
Link Down
```

állapotokat.

Cél:

> Megérteni, hogy a „link up” önmagában nem elegendő információ.

---

# 10.55 Labor 8 – Dante Controller

Nyisd meg:

```text
Network View
```

majd:

```text
Network Status
Device Info
Clock Status
```

Dokumentáld:

```text
Primary
Secondary
Clock
Bandwidth
```

---

# 10.56 Labor 9 – Fizikai redundancia

Rajzold le:

```text
Primary path
Secondary path
```

majd jelöld:

```text
shared cable tray?
shared power?
shared rack?
shared switch?
```

A cél:

> Megtalálni a látszólag redundáns, valójában közös hibapontokat.

---

# 10.57 Labor 10 – Teljes hibakeresés

Szimulált állapot:

```text
Audio occasionally drops
```

A hálózat redundáns.

A tanulónak kell meghatároznia:

```text
Primary?
Secondary?
Clock?
Switch?
Cable?
QoS?
Multicast?
Device?
```

Elvárt sorrend:

```text
Dante Controller
      ↓
Network Status
      ↓
Device Info
      ↓
Clock Status
      ↓
Switch
      ↓
Physical layer
```

---

# 10.58 Vizsgafeladat – Mi a redundancia?

**Kérdés:**

Mi a különbség a kétportos Dante eszköz és a redundáns Dante eszköz között?

**Válasz:**

A két Ethernet port önmagában nem garantál redundáns működést. Az adott eszköznek támogatnia kell a redundáns módot, és a két interfészt megfelelő Primary és Secondary hálózatra kell kötni.

---

# 10.59 Vizsgafeladat – Mi a SPOF?

**Kérdés:**

Mi az SPOF?

**Válasz:**

Olyan komponens vagy kapcsolat, amelynek egyetlen hibája a rendszer kritikus részének kiesését okozza.

Például:

```text
1 switch
```

egy egyszerű Dante hálózatban SPOF lehet.

---

# 10.60 Vizsgafeladat – Primary / Secondary

**Kérdés:**

Mi a Primary és Secondary Dante hálózat szerepe?

**Válasz:**

Két külön hálózati út biztosítása redundáns Dante eszközök számára. A cél, hogy az egyik hálózati oldal hibája esetén a másik oldal továbbra is biztosíthassa a szükséges Dante kapcsolatokat.

---

# 10.61 Vizsgafeladat – A két switch összekötése

**Kérdés:**

Miért nem kell a klasszikus Dante redundáns topológiában a Primary és Secondary switchet egyszerűen összekötni?

**Válasz:**

Mert a redundáns Dante modellben a két hálózat külön hálózati útként működik. Az összekötés szükségtelen és nem megfelelően tervezve hurkokat vagy nem kívánt forgalmi útvonalakat okozhat.

---

# 10.62 Vizsgafeladat – Audio működik, Primary down

```text
Primary = Down
Secondary = Up
Audio = OK
```

Mit jelent?

**Válasz:**

A redundáns út valószínűleg átvette a működéshez szükséges szerepet. A rendszer azonban nem tekinthető teljesen egészséges redundáns állapotúnak, mert az egyik védelmi út kiesett.

---

# 10.63 Vizsgafeladat – Miért kell azonnal javítani?

Mert:

```text
első hiba
   ↓
redundancy margin csökken
```

Ha a második oldal is meghibásodik:

```text
Primary X
Secondary X
```

akkor nincs tartalék.

Ezért a redundancia egyik legfontosabb üzemeltetési szabálya:

> **A redundanciahibát akkor kell javítani, amikor az első út kiesik, nem akkor, amikor a második is kiesik.**

---

# 10.64 Vizsgafeladat – DDM HA

**Kérdés:**

A Dante Domain Manager High Availability ugyanaz, mint a Dante Primary/Secondary redundancy?

**Válasz:**

Nem.

```text
Dante redundancy
→ hálózati/media redundancia

DDM High Availability
→ DDM szerver és control-plane rendelkezésre állása
```

---

# 10.65 Deep Dive – Miért nincs „varázslatos” redundancia?

A redundancia mindig egy konkrét hibatípus ellen véd.

Például:

```text
Primary cable failure
```

ellen védhet.

De ha:

```text
mindkét switch power failure
```

történik, a rendszer kieshet.

Ha:

```text
mindkét hálózati út ugyanabban a kábelcsatornában
```

sérül, szintén kieshet.

Ezért a helyes kérdés:

> **Melyik hibát akarjuk túlélni?**

---

# 10.66 Deep Dive – Redundanciai mátrix

Hasznos dokumentáció:

| Hiba | Primary | Secondary | Audio célállapot |
|---|---|---|---|
| Primary kábel | DOWN | UP | működik |
| Secondary kábel | UP | DOWN | működik |
| Primary switch | DOWN | UP | működik |
| Secondary switch | UP | DOWN | működik |
| Mindkét switch | DOWN | DOWN | kieshet |
| Közös táp | DOWN | DOWN | kieshet |
| Közös fizikai út | DOWN | DOWN | kieshet |

A táblázat nem garancia minden konkrét eszközre; az adott gyártói implementációt és topológiát mindig ellenőrizni kell.

---

# 10.67 Deep Dive – Redundancia és monitoring

Egy jó üzemeltetési rendszernek legalább ezeket kell figyelnie:

```text
Primary state
Secondary state
Link speed
Port errors
Bandwidth
Clock
Subscriptions
```

A Dante Controller erre több külön nézetet biztosít.

---

# 10.68 Deep Dive – Miért nem elég a hálózati rajz?

Egy rajz:

```text
Switch A
Switch B
```

még nem bizonyít redundanciát.

A dokumentációban szerepelnie kell:

```text
physical path
logical path
power path
VLAN
switch configuration
Dante redundancy mode
```

A redundancia akkor auditálható, ha tudjuk:

> **Milyen egyetlen hibát képes a rendszer túlélni?**

---

# 10.69 Gyakorlati ellenőrzőlista – topológia

```text
□ Star / hierarchy
□ Switch count
□ Access / core
□ Uplink capacity
□ SPOF
□ Physical paths
□ Power paths
□ Fiber paths
```

---

# 10.70 Gyakorlati ellenőrzőlista – Dante redundancy

```text
□ Device supports redundancy
□ Redundant mode enabled
□ Primary connected
□ Secondary connected
□ Primary switch independent
□ Secondary switch independent
□ Primary link speed
□ Secondary link speed
□ Clock status
□ Network status
□ Failure test completed
```

---

# 10.71 Gyakorlati ellenőrzőlista – üzemeltetés

```text
□ Primary failure alarm
□ Secondary failure alarm
□ Link speed monitoring
□ Port errors
□ Switch health
□ UPS status
□ Cable documentation
□ Recovery procedure
□ Periodic redundancy test
```

---

# 10.72 A fejezet mentális modellje

```text
                  DANTE SYSTEM
                       │
                       ▼
                  TOPOLOGY
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
       SIMPLE                   REDUNDANT
          │                         │
      1 switch              Primary + Secondary
          │                         │
         SPOF                 two network paths
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
                 AUDIO                            CLOCK
                    │                               │
                    └───────────────┬───────────────┘
                                    ▼
                                CONTROLLER
                                    │
                              MONITORING
                                    │
                                    ▼
                              FAILURE TEST
```

---

# 10.73 Amit ebből a fejezetből tudnod kell

### Topológia

A hálózati elemek és kapcsolatok felépítése.

### SPOF

Egyetlen hibapont, amelynek kiesése kritikus hatással van a rendszerre.

### Dante redundancy

A Dante eszköz Primary és Secondary hálózati útvonalakon képes redundáns media- és clock-kapcsolatot fenntartani, ha az adott eszköz és hálózat ezt támogatja.

### Redundant mode

A két Ethernet interfész redundáns hálózathoz használható.

### Switched mode

A második Ethernet interfész switch portként működhet, például daisy-chainhez.

### Primary

Az elsődleges Dante hálózat.

### Secondary

A redundáns Dante hálózat.

### Failover

A rendszer működése az alternatív út rendelkezésre állására támaszkodik egy hiba után.

### DDM High Availability

A DDM szerver/control-plane redundanciája, nem azonos a Dante media redundancyval.

---

# 10.74 A legfontosabb szabályok

```text
1. Két Ethernet port nem feltétlenül jelent redundanciát.
2. Mindig ellenőrizd, hogy az eszköz támogatja-e a Redundant módot.
3. Primary és Secondary hálózatot megfelelően különítsd el.
4. A két redundáns switch hálózatát ne kösd össze találomra.
5. A redundancia nem csak logikai, hanem fizikai tervezési kérdés.
6. A közös kábelút továbbra is SPOF lehet.
7. A közös tápellátás továbbra is SPOF lehet.
8. A Primary/Secondary állapotot monitorozni kell.
9. A működő audio nem bizonyítja, hogy a redundancia egészséges.
10. Az első redundanciahibát azonnal javítani kell.
11. A clock redundanciát is ellenőrizni kell.
12. A redundanciát rendszeresen tesztelni kell.
```

---

# 10.75 Következő fejezet

# 11. Dante eszközök, firmware és rendszerüzemeltetés

A 10. fejezetben megtanultuk:

```text
Topológia
SPOF
Primary
Secondary
Redundancy
Failover
Switch redundancy
Physical redundancy
Monitoring
```

A következő kérdés:

> **Hogyan tartsuk a Dante rendszert hosszú távon stabilan és karbantarthatóan?**

A következő fejezetben:

```text
Dante firmware
        ↓
Device management
        ↓
Dante Controller
        ↓
Dante Updater
        ↓
Compatibility
        ↓
Maintenance
        ↓
Recovery
```

---

# 10.76 Fejezeti állapot

**Állapot: FINAL / COMMITÁLHATÓ**

A fejezet tartalmaz:

- hálózati topológiák;
- csillagtopológia;
- single-switch rendszer;
- SPOF;
- Dante Primary / Secondary;
- Redundant / Switched mód;
- Dante redundancy;
- failover;
- PTP és redundancia;
- multicast és redundancia;
- DDM redundancy alapok;
- DDM High Availability elkülönítése;
- core/access topológia;
- switch redundancy;
- fizikai redundancia;
- tápellátási redundancia;
- Dante Controller Network Status;
- Device Info;
- Clock Status;
- hibakeresési workflow;
- 10 gyakorlati labor;
- vizsgafeladatok;
- Deep Dive részek;
- ellenőrzőlisták.

A fejezet végleges változata commitálható.