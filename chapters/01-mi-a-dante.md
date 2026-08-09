---
author: Peter Bogdan
chapter: 1
chapter_title: Mi a Dante?
status: complete
title: DANTE -- A professzionális Audio over IP rendszerek kézikönyve
version: 1.0
---

# 1. Mi a Dante?

> **A fejezet célja:** megérteni, milyen problémára született a Dante,
> hogyan illeszkedik az Audio over IP világába, és milyen mérnöki
> alapfogalmakra épül a későbbi működésének megértése.

## Mit fogsz megtanulni?

A fejezet végére képes leszel:

-   elmagyarázni, miért vált nehezen kezelhetővé a nagy analóg
    audiórendszerek fizikai routingja;
-   megkülönböztetni az analóg audiót, a digitális audiót és az Audio
    over IP-t;
-   megérteni, miért más probléma az audió hálózati továbbítása, mint
    egy fájl elküldése;
-   értelmezni a latency, jitter, packet loss és clock synchronization
    fogalmát;
-   elmagyarázni, milyen szerepet játszik az Ethernet és az IP egy
    Dante-rendszerben;
-   magas szinten megérteni a Dante discovery, clocking és routing
    működését;
-   megérteni, miért fontos a hálózati tervezés;
-   felkészülni az Ethernetről, IP-ről, PTP-ről, QoS-ról és multicast
    forgalomról szóló következő fejezetekre.

------------------------------------------------------------------------

# 1.1 Kezdjük a problémával, ne a technológiával

Amikor valaki először találkozik a Dante névvel, könnyű beleesni abba a
hibába, hogy a technológiát egy újabb audióinterfészként próbálja
megérteni.

Ez félrevezető.

A Dante megértéséhez jobb kérdéssel kezdeni:

> **Miért volt egyáltalán szükség hálózati audióra?**

A válaszhoz képzeljünk el egy nagyobb élő produkciót.

Legyen benne:

-   32 mikrofon;
-   16 vezeték nélküli mikrofon;
-   két keverőpult;
-   külön monitorrendszer;
-   több DSP;
-   felvevő számítógép;
-   broadcast feed;
-   több színpadi egység;
-   erősítők;
-   vezérlőrendszer.

A rendszernek nem egyszerűen hangot kell továbbítania.

Ugyanazt a hangforrást több különböző rendszernek kell eljuttatni.

Egy mikrofon jele például egyszerre lehet:

``` text
mikrofon
   │
   ├──► FOH
   ├──► Monitor
   ├──► Recorder
   └──► Broadcast
```

Az analóg világban ezek a kapcsolatok alapvetően fizikai kapcsolatok.

A hálózati audió egyik legfontosabb újítása éppen az, hogy a **fizikai
infrastruktúrát és a logikai routingot szétválasztja**.

------------------------------------------------------------------------

# 1.2 Az analóg gondolkodás

Az analóg audiórendszer alapmodellje egyszerű:

``` text
FORRÁS ───────────────► CÉL
```

A mikrofonból kijövő jelnek fizikailag el kell jutnia a keverőig.

Ha ugyanazt a jelet több helyre kell eljuttatni, a jelút fizikai
elágaztatására van szükség.

## Splitter

A legegyszerűbb megoldás egy splitter:

``` mermaid
flowchart LR
    MIC["Mikrofon"] --> SPLIT["Splitter"]
    SPLIT --> FOH["FOH"]
    SPLIT --> MON["Monitor"]
    SPLIT --> REC["Recorder"]
```

Ez teljesen működőképes megoldás.

Egy nagy produkcióban azonban sok ilyen kapcsolat lehet.

A rendszerhez hozzáadódik:

-   kábelezés;
-   patch panel;
-   multicore;
-   stage box;
-   splitter;
-   csatlakozók;
-   tartalék kábelek;
-   dokumentáció.

A probléma tehát nem az, hogy az analóg technológia nem tudja
továbbítani a hangot.

A probléma a **méretezés**.

------------------------------------------------------------------------

# 1.3 Mi történik, amikor a rendszer nagyobb lesz?

Képzeljünk el egy 8 csatornás rendszert.

``` text
8 bemenet
   │
   ▼
keverő
```

Ez könnyen kezelhető.

Most legyen 64 bemenet, két keverő, monitorrendszer és felvétel.

``` text
             ┌──► FOH
             │
Bemenetek ───┼──► Monitor
             │
             ├──► Recorder
             │
             └──► Broadcast
```

A fizikai kapcsolatok száma növekedni kezd.

A rendszer üzemeltetője egy idő után már nemcsak azt tartja fejben,
hogy:

> „A 17-es csatorna a lábdob."

Hanem azt is, hogy:

> „A 17-es jel melyik splitből jön, melyik patchen megy át, melyik
> multicore melyik végén van, melyik keverő melyik bemenetére érkezik,
> és melyik másik rendszer kap még belőle."

Ez már routingprobléma.

------------------------------------------------------------------------

# 1.4 A digitális átmenet

A következő nagy lépés a digitális audió volt.

A hang analóg jelként érkezik:

``` text
Mikrofon
   │
   ▼
Analóg elektromos jel
```

Az ADC ezt mintákká alakítja:

``` text
Analóg jel
   │
   ▼
ADC
   │
   ▼
Digitális minták
```

A digitális jel feldolgozható, tárolható és továbbítható.

Ez óriási előrelépés volt.

De itt fontos egy különbséget megérteni:

> **A digitális audió nem feltétlenül hálózati audió.**

Egy digitális interfész továbbra is lehet pont--pont kapcsolat.

------------------------------------------------------------------------

# 1.5 Digitális interfész ≠ Audio over IP

A professzionális audió történetében több digitális interfész jelent
meg.

Például:

-   AES3;
-   ADAT;
-   MADI;
-   S/PDIF.

Ezeket nem kell „elavult" technológiáknak tekinteni.

Mindegyiknek megvan a saját alkalmazási területe.

A lényeg inkább az architektúra.

## Pont--pont digitális kapcsolat

``` text
Eszköz A ═════════════ Eszköz B
```

## Hálózati audió

``` mermaid
flowchart LR
    A["Eszköz A"] --> SW["Ethernet hálózat"]
    B["Eszköz B"] --> SW
    C["Eszköz C"] --> SW
    D["Eszköz D"] --> SW
```

A második modellben több végpont ugyanazt a hálózati infrastruktúrát
használhatja.

Ez a Dante egyik alapvető gondolata.

------------------------------------------------------------------------

# 1.6 Miért pont Ethernet?

Itt jutunk el a Dante egyik legfontosabb alapjához.

Az Ethernet eredetileg nem audiótechnológia.

Számítógépes hálózatokhoz készült.

A professzionális Audio over IP egyik nagy felismerése az volt, hogy a
már létező hálózati technológiákat audiótovábbításra is lehet használni.

Ez azért erős koncepció, mert az Ethernet ökoszisztémája már
rendelkezik:

-   szabványos fizikai rétegekkel;
-   switchekkel;
-   optikai kapcsolatokkal;
-   réz Ethernet-kábelekkel;
-   VLAN-okkal;
-   QoS-mechanizmusokkal;
-   multicast-kezeléssel;
-   redundanciával;
-   hálózatfelügyeleti eszközökkel.

A Dante ezekre a hálózati alapokra épít.

Ez azonban rögtön felvet egy problémát:

> Egy irodai hálózat és egy professzionális élő audióhálózat ugyanazokat
> a technológiákat használhatja, de **nem feltétlenül ugyanúgy kell
> megtervezni őket**.

------------------------------------------------------------------------

# 1.7 Miért nem elég az, hogy „van elég sávszélesség"?

Ez az egyik leggyakoribb kezdő félreértés.

Tegyük fel, hogy egy switch gigabites portokkal rendelkezik.

Első gondolat:

> „A gigabit bőven elég, tehát a Dante működni fog."

A sávszélesség valóban fontos.

De nem az egyetlen tényező.

Egy valós idejű audiórendszerben számít:

-   a késleltetés;
-   a jitter;
-   a packet loss;
-   a csomagok sorrendje;
-   a pufferelés;
-   az időszinkronizáció;
-   a multicast kezelés;
-   a QoS;
-   a hálózati torlódás;
-   a redundancia.

Ezért a Dante-hálózat tervezésekor nem azt kérdezzük:

> „Elég gyors a switch?"

hanem:

> **„Megfelelően viselkedik-e a teljes hálózat az időkritikus
> audióforgalom alatt?"**

------------------------------------------------------------------------

# 1.8 Az élő hang és az adatfájl közötti különbség

Vegyünk két példát.

## 1. példa -- PDF

Egy PDF letöltése közben egy adatcsomag elveszik.

A TCP újraküldheti.

A felhasználó ebből valószínűleg semmit sem vesz észre.

## 2. példa -- koncert

Egy énekes hangjának egy részlete ugyanabban a pillanatban kell eljutnia
a feldolgozási lánc következő eleméhez.

Ha az adat túl későn érkezik, már nem ugyanazt a problémát oldja meg.

A hangnak nemcsak **meg kell érkeznie**.

Megfelelő időben kell megérkeznie.

Ez az időkritikus működés az Audio over IP egyik központi problémája.

------------------------------------------------------------------------

# 1.9 Latency

A latency késleltetést jelent.

Egy audiórendszerben a teljes késleltetés több részből állhat:

``` text
Mikrofon
   │
   ▼
ADC
   │
   ▼
DSP
   │
   ▼
Network
   │
   ▼
Receive Buffer
   │
   ▼
DSP
   │
   ▼
DAC
   │
   ▼
Hangszóró
```

A hálózati latency tehát csak egy része az end-to-end latencynek.

Ezért nagyon fontos elkerülni az olyan leegyszerűsítést, hogy:

> „A Dante késleltetése X ms."

A valós érték függhet:

-   az eszköztől;
-   a hálózattól;
-   a konfigurációtól;
-   a receiver latency-beállításától;
-   a használt flow típusától;
-   a hálózati kapcsolatok sebességétől.

Az Audinate dokumentációja szerint például a multicast flow-k minimális
latency-beállítása 1 ms, miközben egyes eszközök ennél magasabb
minimumot igényelhetnek. Ezért konkrét számot mindig az adott eszköz és
konfiguráció dokumentációjával együtt kell értelmezni.

------------------------------------------------------------------------

# 1.10 Jitter

A jitter az időzítés változása.

Ha az adatcsomagok ideálisan érkeznek:

``` text
|----|----|----|----|----|
```

akkor egy ingadozó érkezési idő így nézhet ki:

``` text
|------|-|-------|---|---|
```

A vevőoldali rendszer pufferrel képes bizonyos eltéréseket kezelni.

De itt kompromisszum jelenik meg:

``` text
Nagyobb buffer
     │
     ├──► nagyobb tolerancia
     │
     └──► nagyobb késleltetés
```

Ezért a valós idejű audióban a stabil hálózat és a kiszámítható működés
fontosabb, mint az, hogy egyetlen pillanatnyi mérésben mekkora maximális
throughputot látunk.

------------------------------------------------------------------------

# 1.11 Packet loss

A hálózat csomagokkal továbbítja az adatot.

Ha egy audióhoz tartozó csomag elveszik, a vevőnek nem áll
rendelkezésére az adott adat.

Ez hallható hibát okozhat.

A packet loss lehetséges okai között lehet:

-   fizikai hibás kapcsolat;
-   túlterhelés;
-   hibás konfiguráció;
-   elégtelen hálózati erőforrás;
-   hibás multicast-kezelés;
-   problémás uplink;
-   hálózati hurok;
-   eszközhiba.

A hibakeresésnél ezért nem elég azt látni, hogy:

> „A Dante Controllerben nincs zöld pipa."

Meg kell határozni, hogy **hol keletkezik a probléma**.

------------------------------------------------------------------------

# 1.12 Clock: miért kell egyáltalán közös idő?

Ez az egyik legfontosabb Dante-fogalom.

A digitális audió minták időben egymáshoz kötöttek.

Ha egy rendszerben több különálló digitális eszköz dolgozik, nem elég,
hogy mindegyik „nagyjából ugyanakkor" dolgozik.

Hosszú távon az órák eltérnének egymástól.

Ezért közös időalapra van szükség.

A standard Dante-hálózatokban az eszközök IEEE 1588 Precision Time
Protocolt (PTP) használnak a helyi órák szinkronizálására.

Az Audinate dokumentációja szerint a Dante-hálózatban egy eszköz kerül
kiválasztásra PTP Leader Clockként, a többi pedig ehhez igazítja a helyi
óráját.

``` mermaid
flowchart TB
    L["PTP Leader Clock"]
    A["Dante eszköz A"]
    B["Dante eszköz B"]
    C["Dante eszköz C"]
    D["Dante eszköz D"]

    L --> A
    L --> B
    L --> C
    L --> D
```

A későbbi PTP-fejezetben ezt a mechanizmust nagyon részletesen fogjuk
tárgyalni.

------------------------------------------------------------------------

# 1.13 A Dante története

A Dante történetét érdemes pontosan megismerni, mert sok rövid
összefoglaló pontatlanul kezeli az éveket.

Az Audinate hivatalos története szerint a történet 2003-ban indult,
amikor egy Sydneyben dolgozó mérnöki csapat egy hálózati problémaként
kezdett gondolkodni az audiókapcsolatokról.

A gondolat lényegében ez volt:

> Ha a hangtechnikai eszközök között rengeteg különálló kapcsolatot
> építünk, miért ne lehetne ezeket egy közös hálózaton kezelni?

2006-ban a NICTA úgy döntött, hogy a technológia kereskedelmi
hasznosítására kiválik a projektből az Audinate.

2008-ban a Dolby Lake Processor lett az első Dante-kompatibilis
professzionális audióeszköz. Az Audinate történeti leírása szerint a
technológia egy Barbra Streisand-koncerten debütált Washingtonban.

Ezért három évszámot érdemes megjegyezni:

``` text
2003 ──► fejlesztési irány
2006 ──► Audinate spin-out
2008 ──► első Dante-kompatibilis professzionális eszköz
```

A „Dante 2006-ban jelent meg" állítás tehát leegyszerűsítő.

------------------------------------------------------------------------

# 1.14 Mit próbált megoldani a Dante?

A Dante mögötti probléma több részből áll.

## 1. Routing

Hogyan kapcsoljuk össze rugalmasan az audióforrásokat és a vevőket?

## 2. Synchronization

Hogyan biztosítjuk, hogy az eszközök közös időalapon működjenek?

## 3. Transport

Hogyan juttatjuk el az audiót a hálózaton?

## 4. Discovery

Hogyan találják meg egymást az eszközök?

## 5. Configuration

Hogyan konfiguráljuk a rendszert anélkül, hogy minden kapcsolatot
fizikailag újra kellene kábelezni?

Ezért a Dante-t nem érdemes egyszerűen „hang Etherneten" technológiaként
kezelni.

Inkább egy olyan platformként érdemes gondolni rá, amely az
audióroutinghoz, az időszinkronizációhoz és a hálózati működéshez
szükséges mechanizmusokat egy egységes rendszerben kezeli.

------------------------------------------------------------------------

# 1.15 Mi történik, amikor csatlakoztatunk egy Dante-eszközt?

Ez az első olyan pont, ahol érdemes bepillantani a háttérfolyamatokba.

Tegyük fel, hogy egy Dante stage boxot csatlakoztatunk a switchhez.

A folyamat leegyszerűsítve:

``` text
1. Fizikai kapcsolat létrejön
             │
             ▼
2. Hálózati konfiguráció
             │
             ▼
3. Eszközfelderítés
             │
             ▼
4. Dante Controller megjeleníti
             │
             ▼
5. Routing konfigurálható
             │
             ▼
6. Clock szinkronizáció
             │
             ▼
7. Audióforgalom
```

Az Audinate dokumentációja szerint egy Dante-eszköz hálózatra
csatlakoztatáskor automatikusan konfigurálja IP-címét és hirdeti magát a
hálózaton.

Ha van DHCP-szerver, az eszköz DHCP-n keresztül kaphat IP-konfigurációt.

Ha nincs DHCP, link-local címzést használhat.

A Dante Controller ezután automatikusan felderítheti és megjelenítheti
az eszközt.

Ez nagyon kényelmes.

De fontos:

> **Az automatikus discovery nem jelenti azt, hogy a hálózat
> automatikusan jól van megtervezve.**

------------------------------------------------------------------------

# 1.16 IP-címzés a Dante világában

Kezdőként könnyű azt gondolni, hogy a Dante „nem is igazi hálózat", mert
az eszközök sokszor szinte azonnal megjelennek.

Valójában éppen ellenkezőleg.

A Dante IP-alapú hálózati technológiát használ.

Egy tipikus egyszerű rendszerben:

``` text
Dante eszköz ──┐
Dante eszköz ──┼──► Switch
Dante eszköz ──┤
PC / Dante Controller ──┘
```

Minden résztvevőnek működő hálózati kapcsolat és megfelelő
IP-konfiguráció szükséges.

Az Audinate dokumentációja külön is felhívja a figyelmet arra, hogy a
Dante-eszközöknek megfelelő IP-címzési környezetben kell működniük, és a
hibás IP-konfigurációk a Dante Controllerben is megjelenhetnek.

Ez később az IP-fejezetben lesz igazán fontos.

------------------------------------------------------------------------

# 1.17 Discovery

A discovery azt jelenti, hogy a rendszer képes megtalálni és azonosítani
az eszközöket.

A Dante Controllerben például megjelenhet:

``` text
Stagebox-01
Monitor-Console
FOH-Console
Dante Virtual Soundcard
DSP-01
```

Az Audinate dokumentációja szerint az eszközök olyan információkat is
hirdethetnek, mint:

-   eszköznév;
-   csatornanevek;
-   csatornaszám;
-   sample rate;
-   bit depth.

Ez azért fontos, mert a routingnál nemcsak azt kell tudni, hogy „van egy
eszköz".

Azt is tudni kell, hogy:

> Milyen csatornái vannak, milyen formátumban működik, és kompatibilis-e
> a másik végponttal?

------------------------------------------------------------------------

# 1.18 Routing és subscription

A Dante Controller egyik legfontosabb funkciója az audiórouting
konfigurálása.

Például:

``` text
Stagebox-01
    TX 01
      │
      ▼
FOH-Console
    RX 01
```

Ezt a Dante terminológiában subscriptionként kezeljük.

A felhasználói felületen ez egyszerűnek tűnik.

A háttérben azonban a rendszernek több dolgot kell összehangolnia.

Például:

-   az adócsatornát;
-   a vevőcsatornát;
-   a kompatibilis formátumot;
-   a clock domaint;
-   a flow-t;
-   az adott eszköz kapacitását.

Az Audinate dokumentációja szerint a subscription hibák között például
előfordulhat eltérő sample rate, eltérő clock domain vagy flow-kapacitás
elérése.

Ez nagyon fontos tanulság:

> **Ha egy subscription nem működik, nem biztos, hogy a hálózati kábel a
> hibás.**

------------------------------------------------------------------------

# 1.19 Unicast és multicast

Két alapvető hálózati továbbítási modell:

## Unicast

Egy adó → egy konkrét vevő.

``` text
TX ─────────► RX
```

## Multicast

Egy adó → egy multicast csoport → több vevő.

``` text
              ┌──► RX1
TX ──► Group ─┼──► RX2
              └──► RX3
```

A multicast különösen akkor érdekes, amikor ugyanazt az audióforrást sok
vevőnek kell eljuttatni.

Ez viszont már közvetlenül összekapcsolja a Dante-t a hálózati
multicast-mechanizmusokkal.

Később ezért részletesen foglalkozunk majd:

-   multicasttal;
-   IGMP-vel;
-   IGMP Snoopinggal;
-   Querierrel;
-   switch multicast táblákkal.

------------------------------------------------------------------------

# 1.20 Miért fontos a QoS?

A hálózatban többféle forgalom jelenhet meg.

Például:

``` text
Audió
PTP
Vezérlés
Egyéb IP-forgalom
```

Nem minden csomag azonos jelentőségű.

Egy időkritikus audió- vagy clocking-forgalmat nem feltétlenül ugyanúgy
kell kezelni, mint egy nagy fájl másolását.

Ezért fontos a Quality of Service.

A QoS segítségével a hálózati eszközök különböző prioritásokat
alkalmazhatnak.

A későbbi QoS-fejezetben megvizsgáljuk:

-   DSCP;
-   802.1p;
-   queue;
-   priority;
-   congestion;
-   buffer.

------------------------------------------------------------------------

# 1.21 A Dante és az „irodai hálózat" problémája

Egy Dante-rendszer lehet fizikailag nagyon hasonló egy irodai
Ethernet-hálózathoz.

Mindkettőben lehet:

-   Cat kábel;
-   optika;
-   Ethernet switch;
-   IP-cím;
-   VLAN;
-   router.

A különbség az alkalmazás követelményeiben van.

Egy irodai hálózatban egy rövid megszakadás gyakran csak annyit jelent,
hogy egy weboldal egy pillanattal később töltődik be.

Egy élő audiórendszerben ugyanez hallható hibát okozhat.

Ezért a hálózati mérnök és az audiómérnök szemléletét össze kell
kapcsolni.

------------------------------------------------------------------------

# 1.22 A hibakeresési szemlélet

Képzeld el, hogy a Dante Controllerben egy vevő nem kap jelet.

A kezdő reakció:

> „Valami baj van a Dante-tal."

A mérnöki reakció:

> „Melyik rétegben van a hiba?"

Egy lehetséges hibakeresési fa:

``` text
Nincs hang
   │
   ├── Fizikai kapcsolat?
   │       └── nincs → kábel / port / eszköz
   │
   ├── IP-kapcsolat?
   │       └── nincs → címzés / interfész / VLAN
   │
   ├── Dante discovery?
   │       └── nincs → discovery / hálózat
   │
   ├── Clock rendben?
   │       └── nincs → PTP / clock domain
   │
   ├── Subscription rendben?
   │       └── nincs → routing / kompatibilitás
   │
   ├── Flow rendben?
   │       └── nincs → kapacitás / multicast / hálózat
   │
   └── Audió rendben?
           └── nincs → eszköz / DSP / gain / routing
```

Ez a gondolkodásmód később sokkal fontosabb lesz, mint bármelyik konkrét
Dante Controller menüpont.

------------------------------------------------------------------------

# 1.23 Dante mint rendszer, nem mint „doboz"

Eddig három különálló dolgot láttunk:

### Audió

``` text
ADC → DSP → DAC
```

### Hálózat

``` text
Ethernet → Switch → IP
```

### Idő

``` text
PTP → közös időalap
```

A Dante ezeket egy működő rendszerben kapcsolja össze.

``` mermaid
flowchart TB
    AUDIO["Digitális audió"]
    NETWORK["Ethernet / IP hálózat"]
    CLOCK["PTP időszinkronizáció"]
    ROUTING["Dante routing"]

    AUDIO --> ROUTING
    NETWORK --> ROUTING
    CLOCK --> ROUTING
```

Ezért a könyv további részeiben sem egyetlen technológiaként fogjuk
vizsgálni.

------------------------------------------------------------------------

# 1.24 Mi nem Dante?

Fontos néhány dolgot elkülöníteni.

## Dante ≠ Ethernet

Az Ethernet a hálózati infrastruktúra egyik alapja.

A Dante az audió- és AV-rendszer működéséhez használja a hálózatot.

## Dante ≠ IP

Az IP a hálózati kommunikáció egyik rétege.

A Dante erre épít, de nem azonos vele.

## Dante ≠ PTP

A PTP időszinkronizációs protokoll.

A Dante használja, de a Dante ennél sokkal több.

## Dante ≠ Dante Controller

A Dante Controller egy konfigurációs és menedzsmenteszköz.

A Dante-rendszer működése nem azonos a Controller felületével.

Ez a megkülönböztetés később nagyon hasznos lesz.

------------------------------------------------------------------------

# 1.25 Egy teljes példa: kis színház

Tegyük fel, hogy egy 400 férőhelyes színház hangrendszerét tervezzük.

## Követelmények

-   24 színpadi analóg bemenet;
-   8 vezeték nélküli mikrofon;
-   FOH konzol;
-   monitorrendszer;
-   2 DSP;
-   felvétel;
-   stream;
-   erősítők.

### Hagyományos felépítés

``` mermaid
flowchart LR
    STAGE["Színpad"] --> SPLIT["Splitter / Patch"]
    SPLIT --> FOH["FOH"]
    SPLIT --> MON["Monitor"]
    FOH --> DSP["DSP"]
    DSP --> AMP["Erősítők"]
```

### Dante-alapú felépítés

``` mermaid
flowchart LR
    STAGE["Dante Stage Box"] --> SW1["Switch"]
    SW1 --> FOH["FOH"]
    SW1 --> MON["Monitor"]
    SW1 --> DSP["DSP"]
    SW1 --> REC["Recorder"]
    SW1 --> STREAM["Stream"]
    SW1 --> AMP["Dante erősítők"]
```

A második rendszerben a fizikai hálózat közös infrastruktúra.

De itt jelenik meg a könyv egyik legfontosabb mérnöki tanulsága:

> **A Dante nem szünteti meg a rendszertervezést. Áthelyezi a
> rendszertervezés egy részét a hálózatba.**

Korábban patch panelt és multicore-t terveztünk.

Most:

-   switcheket;
-   uplinkeket;
-   VLAN-okat;
-   multicast-kezelést;
-   QoS-t;
-   redundanciát;
-   PTP-t

is terveznünk kell.

------------------------------------------------------------------------

# 1.26 Egy nagyobb példa: sportcsarnok

Egy sportcsarnokban a rendszer még összetettebb lehet.

Lehet:

-   több színpad;
-   több kommentátori pozíció;
-   központi vezérlés;
-   broadcast;
-   háttérzene;
-   paging;
-   vészhangosítás;
-   több különálló keverőrendszer.

A fizikai kábelezés itt már jelentős infrastruktúra.

A hálózati architektúra lehet például:

``` mermaid
flowchart TB
    CORE["Core hálózat"]
    S1["Stage / Zone 1"]
    S2["Stage / Zone 2"]
    FOH["FOH"]
    BROAD["Broadcast"]
    DSP["DSP"]
    AMP["Amplifier"]

    S1 --> CORE
    S2 --> CORE
    FOH --> CORE
    BROAD --> CORE
    CORE --> DSP
    CORE --> AMP
```

Itt már nem elég azt mondani:

> „Dante van a hálózaton."

Meg kell tudni válaszolni:

-   milyen a topológia?
-   hol vannak a switch-ek?
-   hogyan működik a redundancia?
-   milyen multicast-forgalom van?
-   milyen a PTP?
-   milyen QoS van beállítva?
-   hol vannak a hálózati határok?
-   mi történik egy switch kiesésekor?

Ezek lesznek a későbbi rendszertervezési fejezetek témái.

------------------------------------------------------------------------

# 1.27 Az első mentális modell

Most alakítsunk ki egy egyszerű mentális modellt.

Ha Dante-rendszerre gondolsz, képzeld el ezt az öt réteget:

``` text
┌─────────────────────────────┐
│ 5. Audióalkalmazás          │
├─────────────────────────────┤
│ 4. Dante routing / flow     │
├─────────────────────────────┤
│ 3. PTP / időszinkronizáció  │
├─────────────────────────────┤
│ 2. IP / UDP / multicast     │
├─────────────────────────────┤
│ 1. Ethernet / fizikai hálózat│
└─────────────────────────────┘
```

Ha később hibát keresünk, ezt a modellt fogjuk használni.

Nem kell még minden réteget értened.

A következő fejezetekben egyenként lebontjuk őket.

------------------------------------------------------------------------

# 1.28 Ellenőrző kérdések

## Alap

1.  Miért okoz problémát a fizikai routing egy nagy analóg
    audiórendszerben?
2.  Mi a különbség egy digitális pont--pont interfész és egy
    AoIP-rendszer között?
3.  Miért nem azonos az Ethernet és a Dante?
4.  Miért nem elegendő a nagy sávszélesség?
5.  Mit jelent a latency?
6.  Mit jelent a jitter?
7.  Mi a packet loss?
8.  Miért van szükség közös clockra?
9.  Mi a PTP?
10. Mi a subscription?

## Haladóbb

11. Miért lehet előnyös az audiórouting és a fizikai hálózat
    szétválasztása?
12. Milyen előnye és hátránya van annak, ha egy audióhálózat közös
    infrastruktúrát használ?
13. Miért lehet problémás egy nem megfelelően kezelt multicast-forgalom?
14. Miért lehet egy hibás IP-konfiguráció miatt látszólag „Dante-hiba"?
15. Miért nem jó hibakeresési módszer azonnal a Dante Controllerre
    koncentrálni?

------------------------------------------------------------------------

# 1.29 Labor 1 -- Analóg vagy hálózati?

## Feladat

Tervezd meg ugyanazt a rendszert kétféleképpen.

### Követelmények

-   16 analóg bemenet;
-   8 vezeték nélküli mikrofon;
-   FOH;
-   monitor;
-   recorder;
-   stream;
-   két DSP.

## A változat

Tervezd meg hagyományos fizikai routinggal.

Rajzold fel:

-   splittereket;
-   kábeleket;
-   patch pontokat;
-   keverőket;
-   DSP-ket.

## B változat

Tervezd meg Audio over IP architektúrával.

Rajzold fel:

-   stage boxokat;
-   switcheket;
-   FOH-ot;
-   monitort;
-   DSP-t;
-   recordert;
-   stream rendszert.

## Válaszolj

1.  Melyik rendszerben több a fizikai audiókapcsolat?
2.  Melyikben egyszerűbb egy új vevő hozzáadása?
3.  Melyikben válik fontosabbá a hálózati konfiguráció?
4.  Melyik rendszerben kell különösen figyelni a hálózati hibákra?

------------------------------------------------------------------------

# 1.30 Labor 2 -- Hibakeresési gondolkodás

A FOH konzol nem kapja meg a stage box 1-es csatornáját.

A Dante Controllerben az eszközök láthatók.

A feladatod:

**Ne próbáld meg azonnal újra létrehozni a subscriptiont.**

Először készíts hibakeresési listát.

Vizsgáld meg ebben a sorrendben:

1.  fizikai link;
2.  IP-cím;
3.  discovery;
4.  clock;
5.  sample rate;
6.  subscription;
7.  flow;
8.  audióút.

A cél annak gyakorlása, hogy a hibát **rétegenként** keresd.

------------------------------------------------------------------------

# 1.31 Deep Dive -- Miért nem TCP?

A TCP megbízható és sorrendtartó adatátvitelt biztosít. Ha egy csomag
hiányzik, a protokoll képes annak újraküldésére.

Egy fájl letöltésénél ez előny.

Élő audiónál viszont a későn érkező adat problémát jelenthet.

Egyszerűsített példa:

``` text
T = 100 ms
A csomagnak ekkor kellene rendelkezésre állnia.

Csomag elveszik
      ↓
TCP újraküldi
      ↓
csomag megérkezik 140 ms-nál
```

A csomag ugyan megérkezett.

De az eredeti 100 ms-os időablak már elmúlt.

Ezért a valós idejű audió más optimalizálási problémát jelent.

A Dante audiótovábbítása UDP-alapú. Ez nem azt jelenti, hogy a rendszer
„nem megbízható".

A megbízhatóságot nem az újraküldésre épített fájlátviteli modell adja,
hanem a megfelelő:

-   hálózattervezés;
-   pufferelés;
-   időszinkronizáció;
-   prioritás;
-   forgalomkezelés;
-   hibakezelés.

A későbbi fejezetekben pontosan megvizsgáljuk, hogyan működnek ezek.

------------------------------------------------------------------------

# 1.32 Deep Dive -- Mi történik a háttérben egy subscription után?

A felhasználói nézet:

``` text
TX 01 ─────► RX 01
```

A mérnöki nézet ennél sokkal összetettebb.

A rendszernek például ellenőriznie kell:

-   van-e megfelelő adó;
-   van-e megfelelő vevő;
-   kompatibilis-e a formátum;
-   megfelelő-e a sample rate;
-   azonos clock domainben vannak-e;
-   van-e még flow-kapacitás;
-   unicast vagy multicast kapcsolat jön-e létre;
-   megfelelő-e a hálózati út.

Ezért egy egyszerű kattintás mögött több hálózati és audiómechanizmus
dolgozik.

A könyv célja, hogy ezeket a későbbi fejezetekben láthatóvá tegye.

------------------------------------------------------------------------

# 1.33 Mit kell most megjegyezned?

Ha ebből a fejezetből csak tíz dolgot jegyzel meg, ezek legyenek:

1.  **A Dante nem egyszerűen egy kábel.**
2.  **A Dante nem azonos az Ethernettel.**
3.  **A Dante nem azonos az IP-vel.**
4.  **A digitális audió nem automatikusan hálózati audió.**
5.  **Az élő audió időkritikus.**
6.  **A latency és jitter fontos.**
7.  **A packet loss hallható problémát okozhat.**
8.  **A Dante-hálózatnak közös időalapra van szüksége.**
9.  **A routing és a fizikai hálózat szétválasztható.**
10. **A Dante megértéséhez hálózati mérnöki gondolkodás kell.**

------------------------------------------------------------------------

# 1.34 A következő fejezet

Most már tudjuk, miért jelent meg az Audio over IP, és miért lett a
Dante egyik meghatározó platformja.

De még nem tudjuk pontosan, **mit küldünk át a hálózaton**.

A következő fejezet ezért visszalép a hálózat elé.

Megvizsgáljuk:

``` text
Hang
 ↓
ADC
 ↓
Mintavételezés
 ↓
PCM
 ↓
Bitmélység
 ↓
Kvantálás
 ↓
Digitális audió
```

Ezután már lesz egy pontos képünk arról, hogy milyen adatot kell
egyáltalán továbbítanunk.

Innen tudunk majd továbblépni az Ethernetre.

------------------------------------------------------------------------

# 1.35 Források

1.  **Audinate -- History**\
    https://www.audinate.com/company/history/

2.  **Audinate -- Discovery and auto-configuration**\
    https://dev.audinate.com/GA/dante-controller/userguide/webhelp/content/discovery_and_auto-configuration.htm

3.  **Audinate -- Clock Synchronization**\
    https://dev.audinate.com/GA/dante-controller/userguide/webhelp/content/clock_synchronization.htm

4.  **Audinate -- Latency Tab**\
    https://dev.audinate.com/GA/dante-controller/userguide/webhelp/content/latency_tab.htm

5.  **Audinate -- Subscription Tooltips**\
    https://dev.audinate.com/GA/dante-controller/userguide/webhelp/content/subscription_tooltips.htm

6.  **Audinate -- Dante Information for Network Administrators**\
    https://assets.audinate.com/wp-content/uploads/2022/03/dante-information-for-network-admins.pdf

------------------------------------------------------------------------

# 1.36 Fejezeti állapot

**Állapot:** `COMPLETE`

A fejezet tartalmazza:

-   elméleti bevezetést;
-   történeti hátteret;
-   rendszerpéldákat;
-   ábrákat;
-   Dante működési modellt;
-   hálózati alapfogalmakat;
-   hibakeresési gondolkodást;
-   két laborfeladatot;
-   két Deep Dive részt;
-   ellenőrző kérdéseket;
-   forrásokat.

A következő fejezet:

> **2. A digitális audió alapjai**
