---
title: "DANTE – A professzionális Audio over IP rendszerek kézikönyve"
chapter: 8
chapter_title: "Dante Controller"
version: "1.0"
status: "draft-review"
---

# 8. Dante Controller

> **A fejezet célja:** megtanulni a Dante Controller használatát úgy, hogy ne csak routingot tudjunk létrehozni, hanem egy Dante hálózat állapotát is értelmezni, konfigurálni és módszeresen diagnosztizálni tudjuk.

A 7. fejezetben azt tanultuk meg, **mi történik a Dante hálózaton belül**:

```text
Transmitter
     ↓
Subscription
     ↓
Flow
     ↓
Network
     ↓
Receiver
```

Most azt tanuljuk meg, hogyan látjuk és kezeljük mindezt a **Dante Controllerben**.

---

# 8.1 Mi a Dante Controller?

A Dante Controller az Audinate Dante hálózatok konfigurálására és felügyeletére szolgáló alkalmazás.

A Controller segítségével többek között:

- felfedezhetjük a Dante eszközöket;
- audio routingot hozhatunk létre;
- eszköz- és csatornaneveket kezelhetünk;
- megtekinthetjük a clock állapotot;
- ellenőrizhetjük a hálózati státuszt;
- megtekinthetjük a flow-információkat;
- bizonyos eszközök konfigurációját módosíthatjuk;
- latency adatokat figyelhetünk;
- preseteket menthetünk és tölthetünk be;
- támogatott eszközöknél multicast flow-kat konfigurálhatunk;
- támogatott rendszereknél eszközlockot használhatunk.

Az Audinate jelenlegi dokumentációja a Dante Controller két fő nézetét különbözteti meg:

```text
NETWORK VIEW
     +
DEVICE VIEW
```

citeturn1search0turn1search1

---

# 8.2 A két fő nézet

A teljes Controller megértésének legfontosabb alapja:

```text
Dante Controller
       │
       ├───────────────┐
       ▼               ▼
Network View       Device View
       │               │
       │               │
  hálózat egésze   egy eszköz
```

### Network View

A teljes Dante hálózat áttekintésére szolgál.

### Device View

Egy konkrét Dante eszköz részletes vizsgálatára és konfigurálására szolgál.

Egyszerű szabály:

> **Network View = rendszer**
>
> **Device View = egy eszköz**

---

# 8.3 A Network View

A Dante Controller indításakor alapértelmezés szerint a Network View jelenik meg, benne a Routing nézettel.

A jelenlegi Controller Network View-ja több fontos fület tartalmaz:

```text
Routing
Device Info
Clock Status
Network Status
Events
```

citeturn1search0

Gondolkodj így:

```text
                 NETWORK VIEW
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Routing       Clock Status   Network Status
        │              │              │
      Audio          PTP / clock     Network
      paths           state          state
```

---

# 8.4 Routing tab

A Routing tab a Dante rendszer logikai patching felülete.

Egyszerűsítve:

```text
                    RECEIVERS
               Console    DSP    Recorder

TX 1              ✓
TX 2                       ✓
TX 3                               ✓
TX 4              ✓        ✓
```

A metszéspontban létrehozott subscription azt mondja meg:

```text
Melyik TX csatorna
        ↓
melyik RX csatornára
```

A Routing nézet tehát elsősorban a **routing topológiáját** mutatja.

---

# 8.5 Device Info tab

A Device Info nézetben az eszközök fontos azonosító és hálózati adatai tekinthetők át.

A jelenlegi Dante Controllerben többek között ilyen információk jelenhetnek meg:

- Device Name;
- Model Name;
- Product Version;
- Dante Version;
- Device Lock állapot;
- Primary Address;
- Primary Link Speed;
- redundáns eszközöknél Secondary Address;
- Secondary Link Speed.

A pontosan megjelenő adatok eszköz- és firmwarefüggők.

A Device Info ezért különösen hasznos például:

```text
Melyik eszköz ez?
        ↓
Milyen IP-címen érhető el?
        ↓
Milyen linksebességen működik?
        ↓
Milyen Dante / product verziót használ?
```

Ez nem csak azonosításra, hanem hibakeresésre is alkalmas. citeturn1search5

---

# 8.6 Clock Status tab

A Clock Status hálózati szintű clock-áttekintést ad.

A nézetben többek között látható:

- Sync állapot;
- Mute állapot;
- Clock Source;
- clock szerepek;
- Preferred Leader;
- PTP állapotok;
- támogatott rendszereknél további PTPv2 információk.

citeturn0search6

A kezdő számára a legfontosabb:

```text
Clock Status
     │
     ├── Sync
     ├── Mute
     ├── Clock Source
     └── Leader / Follower
```

---

# 8.7 Network Status tab

A Network Status a Dante-eszközök hálózati kapcsolatának és az audiohálózat működésének fontos diagnosztikai nézete.

Támogatott környezetben többek között ellenőrizhetők:

- a hálózati kapcsolat és linksebesség;
- a latency;
- latency errorok;
- packet errorok;
- az érintett Dante hálózati interfész állapota.

Egyszerű hibakeresési modell:

```text
Device látható?
        ↓
Network Status
        ↓
Link / speed / errors / latency
        ↓
Switch oldali ellenőrzés
```

A Network Status nem helyettesíti a switch CLI-jét vagy webes menedzsmentjét, de fontos Dante-oldali diagnosztikai információt ad.

> **Ha a Controllerben packet vagy latency hibát látsz, a következő lépés gyakran a switch portstatisztikájának ellenőrzése.**

---

# 8.8 Events tab

Az Events nézet a Dante Controller által észlelt eseményeket és állapotváltozásokat jeleníti meg.

Az események kategóriákba sorolhatók:

```text
Information
Warning
Error
```

Például találkozhatsz:

```text
Device offline
Clock change
Network change
Subscription-related event
```

Az Events nézet különösen hasznos akkor, amikor nem csak az aktuális állapotot akarjuk látni, hanem azt is, **mi történt korábban**.

Az eseménylista naplófájlba is menthető, ami hibakeresésnél hasznos lehet.

> **A „most rossz” és a „mikor romlott el?” két külön diagnosztikai kérdés.**

---

# 8.9 Device View megnyitása

Device View-t több módon megnyithatsz.

A legegyszerűbb:

```text
Network View
     ↓
dupla kattintás az eszközre
     ↓
Device View
```

A jelenlegi Dante Controllerben a `Ctrl + D` is használható Windows alatt.

citeturn1search1

Több Device View ablak is megnyitható, ezért összehasonlíthatsz például:

```text
Stage Box
     +
FOH Console
```

---

# 8.10 A Device View fő fülei

Az elérhető fülek eszközfüggők.

A jelenlegi Audinate dokumentáció alapján tipikusan találkozhatsz ezekkel:

```text
Receive
Transmit
Status
Latency
Device Config
Network Config
Issues
```

Bizonyos eszközök további gyártó- vagy termékspecifikus füleket is kínálhatnak.

citeturn1search1

Nagyon fontos:

> **Nem minden Dante eszközön jelenik meg minden fül vagy minden beállítás.**

---

# 8.11 Receive tab

A Receive tab az adott eszköz receiver oldalát mutatja.

Itt látható többek között:

```text
Receive Channel
Connected To
Subscription status
Signal
```

Az Audinate dokumentáció szerint a Receive tab két fő része:

```text
Receive Channels
        +
Available Channels
```

citeturn1search8

---

# 8.12 Available Channels

Az Available Channels területen láthatók azok a transmitter csatornák, amelyekhez subscription hozható létre.

Egyszerűsítve:

```text
AVAILABLE TRANSMITTERS
        │
        ▼
Receive Channel
```

A subscription létrehozása történhet például drag-and-drop módszerrel.

---

# 8.13 Subscription státusz

A Receive tab egyik legfontosabb része a subscription állapota.

Egyszerűsített modell:

```text
OK
↓
Subscription működik

Unresolved
↓
A kapcsolat nem oldódott fel

Error / No subscription
↓
Nincs érvényes audio subscription
```

A Dante Controller külön jelölésekkel tudja mutatni például az unicast és multicast subscriptionöket is. citeturn1search8

---

# 8.14 Signal oszlop

Támogatott eszközöknél a Receive tab a jel jelenlétét is képes jelezni.

Ez nagyon hasznos:

```text
Subscription = OK
Signal = present
```

mert így nem csak azt látjuk, hogy a routing kapcsolat létrejött, hanem azt is, hogy a receiver oldalon tényleges audiojel érkezik.

A signal kijelzés azonban nem helyettesíti a fizikai audiojel útjának teljes ellenőrzését.

---

# 8.15 Receive és Transmit irány

A kezdők gyakran felcserélik a nézőpontot.

Emlékezz:

```text
TRANSMIT
= innen küldöm

RECEIVE
= ide érkezik
```

Példa:

```text
StageBox
TX 1 Vocal
     │
     ▼
FOH Console
RX 1 Vocal
```

A StageBox oldalán:

```text
Transmit
```

A Console oldalán:

```text
Receive
```

---

# 8.16 Transmit tab

A Transmit tab az adott Dante eszköz által továbbított csatornákat és bizonyos esetekben multicast flow-kat kezeli.

A fontos kérdés:

> **Mit küld ez az eszköz a hálózatba?**

Például:

```text
TX 1 Vocal
TX 2 Guitar
TX 3 Bass
TX 4 Kick
```

---

# 8.17 Channel naming

A Dante routing név alapján működik.

Az Audinate dokumentációja szerint:

- a device name a hálózaton egyedi kell legyen;
- a channel name az adott eszközön egyedi;
- a routing ezekre a nevekre támaszkodik.

citeturn1search9

Ezért:

```text
StageBox-A
```

sokkal jobb, mint:

```text
Device-001
```

és:

```text
Vocal
Guitar
Bass
Kick
```

jobb, mint:

```text
Ch 1
Ch 2
Ch 3
Ch 4
```

---

# 8.18 Mi történik, ha átnevezünk egy eszközt?

Ez nagyon fontos Dante-sajátosság.

A routing névhez kötött.

Ha például:

```text
Old StageBox
```

átneveződik:

```text
StageBox-A
```

akkor a Dante routing szempontjából ez a korábbi névhez képest más eszközazonosításnak számít.

Az Audinate dokumentációja szerint ha egy régi eszköz kiesik, majd egy új eszközt ugyanazzal a régi device névvel helyettesítünk, a korábbi receiver subscriptionök automatikusan helyreállhatnak az új eszközre.

citeturn1search9

Ezért a device naming **rendszertervezési kérdés** is.

---

# 8.19 Device Config

A Device Config fül eszközfüggő konfigurációs lehetőségeket tartalmaz.

Például:

```text
Device name
Sample rate
Pull-up/down
```

Bizonyos egyéb paraméterek is megjelenhetnek az adott eszköz képességeitől függően.

citeturn1search2

Fontos:

> **A Dante Controllerben látható konfigurációs lehetőségek nem univerzálisak minden Dante eszközön.**

---

# 8.20 Sample Rate beállítása

A Device Config felületen támogatott eszközöknél módosítható a sample rate.

Például:

```text
48 kHz
44.1 kHz
96 kHz
```

de kizárólag az adott eszköz által támogatott értékek közül lehet választani.

Egy módosítás bizonyos eszközöknél újraindítást is igényelhet. citeturn1search2

---

# 8.21 Network Config

Támogatott eszközöknél a Network Config fülön hálózati konfiguráció is megjelenhet.

A pontos lehetőségek eszköz- és firmwarefüggők.

Ne feltételezd:

```text
„Dante Controller = teljes switch management”
```

Ez nem igaz.

A Dante Controller az **endpoint Dante konfigurációját** kezeli; a switch konfigurációját továbbra is a switch saját management felületén kell végezni.

---

# 8.22 Status tab

A Status tab egy eszköz állapotának részletesebb vizsgálatára használható.

Tipikusan:

```text
Software / firmware
Clock
Network
Device status
```

Ez különösen hasznos akkor, ha:

```text
„Ez az egy eszköz miért viselkedik másképp?”
```

---

# 8.23 Latency tab

Támogatott eszközöknél a Latency tab az audio packet latency eloszlását képes megjeleníteni.

Az Audinate szerint a histogram:

- a packet latency eloszlását mutatja;
- átlagos latencyt mutat;
- peak latencyt mutat;
- late packet eseményeket is jelezhet.

citeturn0search0

---

# 8.24 Latency histogram értelmezése

Egyszerűen:

```text
Packetek messze a limit előtt
        ↓
jó tartalék

Packetek a limit közelében
        ↓
kockázat

Late packetek
        ↓
audio loss / glitch
```

A túl alacsony receiver latency miatt a packetek nem biztos, hogy időben megérkeznek.

citeturn0search0

---

# 8.25 Miért a receiveren van a latency?

A Dante receiver latency beállítása azt az időt határozza meg, amelyet a receiver a packetek megfelelő érkezésének biztosítására használ.

Az Audinate szerint a latency a receiveren van beállítva, de subscription létrehozásakor a transmitter és receiver között automatikus egyeztetés történik annak érdekében, hogy a választott érték elegendő legyen a packet loss elkerüléséhez.

citeturn1search16

---

# 8.26 Multicast latency

A Dante multicast flow-k automatikusan legalább 1 ms latencyt használnak.

Ha multicast és unicast subscriptionök is vannak, a Latency nézetben emiatt eltérő histogramok is megjelenhetnek.

citeturn0search0

---

# 8.27 Flow Information

A Device View:

```text
View
 ↓
View Flow Information
```

menüpontjából megtekinthető az adott eszköz audio-, video- és ancillary flow-információja.

Az Audinate példája szerint egy eszköz például:

```text
32 audio transmit flows
32 audio receive flows
```

kapacitást támogathat.

Ez **nem univerzális Dante-érték**, hanem az adott eszköz képessége.

citeturn0search3

---

# 8.28 Device Lock

Támogatott Dante eszközöknél a Device Lock segítségével 4 számjegyű PIN-nel zárolható a konfiguráció.

Lock után:

```text
Media továbbra is folyhat
        ↓
Monitoring lehetséges
        ↓
Konfiguráció módosítása tiltott
```

A meglévő subscriptionök tovább működhetnek, miközben a konfiguráció read-only állapotba kerül.

citeturn1search11

Ez különösen hasznos lehet olyan rendszereknél, ahol:

```text
Live production
+
több kezelő
+
stabil konfiguráció
```

áll fenn.

---

# 8.29 Advanced Filter

Nagy Dante hálózatnál a Routing matrix gyorsan hatalmas lehet.

Ilyenkor az Advanced Filter nagyon hasznos.

Szűrhetünk például:

```text
Device name
Channel name
Sample rate
State
Configuration
```

A filtercsoportok egymással AND logikával kombinálhatók.

Például:

```text
Name contains:
FOH

AND

Sample Rate:
48 kHz
```

eredménye:

```text
csak a FOH nevű
+
48 kHz-es
eszközök
```

citeturn1search3

---

# 8.30 Egyszerű Filter példa

Tegyük fel, hogy van:

```text
FOH-Console
FOH-DSP
FOH-Recorder
Monitor-Console
Monitor-DSP
StageBox-A
StageBox-B
```

Ha ezt keresed:

```text
FOH
```

akkor csak a releváns eszközök maradnak a nézetben.

Nagy rendszerben ez nem kényelmi funkció.

> **A filter a hibakeresés sebességét jelentősen növelheti.**

---

# 8.31 Dante Controller hálózati interfész

A számítógépen több hálózati interfész is lehet:

```text
Wi-Fi
Ethernet
USB Ethernet
VPN adapter
Virtual adapter
```

A Dante Controllernek a megfelelő hálózati interfészt kell használnia.

Ha rossz interfészt választ:

```text
Dante devices
      ↓
nem látszanak
```

Az Audinate kifejezetten említi ezt gyakori okként.

citeturn1search5

---

# 8.31.1 Refresh – amikor a Controller nézete nem frissült

A Dante Controller nézetei nem minden helyzetben azonnal tükrözik a hálózaton történt változást.

Ilyenkor használd a megfelelő **Refresh / frissítés** funkciót, mielőtt egy régi képernyőállapotból vonnál le következtetést.

Egyszerű szabály:

```text
Hálózati / konfigurációs változás
        ↓
Refresh
        ↓
aktuális állapot ellenőrzése
        ↓
diagnózis
```

Ez különösen hasznos akkor, ha:

- egy eszköz nemrég csatlakozott;
- routingot módosítottál;
- egy eszköz újraindult;
- IP- vagy hálózati változás történt.

> **Először győződj meg arról, hogy az általad látott Controller-állapot aktuális.**

# 8.32 Ezért nincs mindig Dante Controller hiba

Ha:

```text
Dante Controller
       │
       ▼
0 devices
```

ne azt feltételezd elsőként:

> „A Dante Controller rossz.”

Ellenőrizd:

```text
1. megfelelő Ethernet interface?
2. fizikai link?
3. IP configuration?
4. VLAN?
5. switch?
6. Dante devices online?
```

Ez a 3–4. fejezet hálózati ismereteinek közvetlen alkalmazása.

---

# 8.33 DHCP és Link Local

A Dante eszközök automatikusan IP-címet kaphatnak.

Ha a hálózaton van DHCP-szerver:

```text
DHCP server
    ↓
Dante device
    ↓
DHCP address
```

Ha nincs elérhető DHCP-szerver:

```text
Dante device
    ↓
Automatic Link Local
    ↓
169.254.x.x
```

Az Audinate dokumentációja szerint a Dante eszközök DHCP használatára vannak felkészítve, és DHCP hiányában automatikus Link Local címet használhatnak.

A fontos gyakorlati szabály:

> **A Dante Controllerben az IP-címek megtekintésekor mindig ellenőrizd, hogy a címzés megfelel-e a tervezett hálózati architektúrának.**

A DHCP és a Link Local nem két kézzel kiválasztandó, egymást kizáró „üzemmód”; a Link Local tipikusan akkor jelenik meg, amikor DHCP-cím nem áll rendelkezésre.

citeturn1search5

---

# 8.34 Network View mint diagnosztikai panel

Ha nincs hang, ne csak a Routing tabot nézd.

Használd ezt a sorrendet:

```text
Routing
   ↓
Device Info
   ↓
Clock Status
   ↓
Network Status
   ↓
Events
   ↓
Device View
```

Ez sokkal jobb módszer, mint:

```text
Routing → újra kattintgatás
```

---

# 8.35 Hibakeresési példa 1

Helyzet:

```text
StageBox
látható

Console
látható

Subscription
ERROR
```

Első kérdések:

```text
Sample rate?
Clock domain?
Transmitter online?
Receiver online?
Flow?
```

Ne állítsd át találomra a routingot.

---

# 8.36 Hibakeresési példa 2

Helyzet:

```text
Subscription = OK
Signal = nincs
```

Ellenőrzési sorrend:

```text
Source
 ↓
TX channel
 ↓
Flow
 ↓
Network
 ↓
RX channel
 ↓
Signal
```

Ha a subscription OK, a következő kérdés már nem feltétlenül:

> „Miért nem routingol?”

hanem:

> **„Van-e tényleges audiojel a transmitter oldalon, és eljut-e a receiverig?”**

---

# 8.37 Hibakeresési példa 3

Helyzet:

```text
Minden device látható
Clock = OK
Subscription = OK
Nincs audio
```

Ellenőrizd:

```text
Device View → Receive
        ↓
Signal column
        ↓
Flow Information
        ↓
Latency
        ↓
Source device
```

Ha a Receive oldalon nincs jel, menj vissza a transmitterhez.

---

# 8.38 Hibakeresési példa 4 – csak egy gépen nincs Dante

Helyzet:

```text
FOH PC → Dante devices láthatók
Laptop → Dante devices nem láthatók
```

Első gyanú:

```text
Network interface
```

Nem feltétlenül a Dante hálózat hibás.

Ellenőrizd:

```text
Dante Controller interface selection
Ethernet link
IP address
Firewall / network policy
VLAN
```

---

# 8.39 Presets

A Dante Controller képes konfigurációs és routing preseteket menteni.

A preset tartalmazhat:

- routingot;
- konfigurációs paramétereket;
- eszközszerepeket;
- csatornaneveket;
- egyéb támogatott beállításokat.

A preset fájl XML formátumú.

citeturn1search10

---

# 8.40 Mire jó a preset?

Például:

```text
Production A
     ↓
Save Preset
     ↓
Production B
     ↓
Load Preset
```

Használható:

- backupra;
- restore-ra;
- ismétlődő eseményekhez;
- laborhoz;
- több konfiguráció közötti váltáshoz.

---

# 8.41 Preset ≠ teljes rendszermentés

Nagyon fontos:

> **A Dante Controller preset nem az egész Dante hálózati infrastruktúra teljes mentése.**

A preset a támogatott Dante-eszközök routingját és konfigurációs elemeit képes menteni, de nem szabad automatikusan teljes rendszermentésként kezelni.

Külön dokumentációban / backupban szükség lehet például:

```text
Külső hálózati switch konfigurációja
+
VLAN / IP terv
+
Fizikai topológia
+
Eszköz- és firmware-információk
+
Külső clock infrastruktúra
```

Fontos különbség:

```text
Dante endpoint konfiguráció
        ≠
teljes Ethernet infrastruktúra konfiguráció
```

Ezért professzionális dokumentációban a Dante Controller preset csak **egyik része** a teljes backupnak.

---

# 8.42 Preset és másik eszköz

A presetben szereplő device role másik kompatibilis eszközre is alkalmazható lehet.

De:

```text
Device A
32 ch
96 kHz

Device B
2 ch
48 kHz
```

nem lesz automatikusan kompatibilis.

Az Audinate szerint más típusú eszközre alkalmazva a preset csak az adott eszköz képességeinek megfelelően tud érvényesülni.

citeturn1search14

---

# 8.43 Biztonságos konfigurációs munkafolyamat

Érdemes ezt követni:

```text
1. Dokumentáld az aktuális állapotot
          ↓
2. Ments presetet
          ↓
3. Módosíts
          ↓
4. Ellenőrizd
          ↓
5. Tesztelj
          ↓
6. Dokumentáld az új állapotot
```

Ne így:

```text
kattintgatás
↓
„majd lesz valami”
```

---

# 8.44 A Dante Controller mint mérőműszer

Egy kezdő gyakran így gondolkodik:

> „A Dante Controllerrel routingolunk.”

Ez igaz, de kevés.

Haladóbb gondolkodás:

```text
Dante Controller
=
Routing
+
Clock monitoring
+
Network diagnostics
+
Device diagnostics
+
Flow analysis
+
Configuration
```

Ezért a Controller a Dante mérnök egyik legfontosabb eszköze.

---

# 8.45 Labor 1 – Device discovery

Topológia:

```text
PC
 │
 ▼
Aruba Switch
 │
 ├── Dante Stage Box
 └── Dante Console
```

Feladat:

1. Indítsd el a Dante Controllert.
2. Ellenőrizd a Network View-t.
3. Keresd meg a két Dante eszközt.
4. Nyisd meg a Device Info fület.
5. Jegyezd fel az IP-címeket.
6. Ellenőrizd a sample rate-et.

Elvárt eredmény:

```text
PC → Controller → devices visible
```

---

# 8.46 Labor 2 – Device View

Nyisd meg a Stage Box Device View-ját.

Vizsgáld:

```text
Receive
Transmit
Status
Device Config
```

Jegyezd fel:

- device name;
- sample rate;
- firmware / software version;
- TX channel count;
- RX channel count.

---

# 8.47 Labor 3 – Channel naming

Nevezd át a laborcsatornákat:

```text
TX 1 = Vocal
TX 2 = Guitar
TX 3 = Bass
TX 4 = Kick
```

Ezután a Routing View-ban ellenőrizd:

```text
A nevek megjelentek?
```

Tanulság:

> A jó névadás közvetlenül javítja a routing áttekinthetőségét.

---

# 8.48 Labor 4 – Routing Device View-ból

A Receive tab Available Channels listájából hozz létre subscriptiont.

```text
StageBox / Vocal
        ↓
Console / Input 1
```

Ellenőrizd:

```text
Subscription = OK
Signal = present
```

---

# 8.49 Labor 5 – Clock Status

Nyisd meg:

```text
Network View
     ↓
Clock Status
```

Keresd:

```text
Primary Leader Clock
Followers
Sync
Mute
Clock Source
Preferred Leader
```

Kérdés:

> Van-e olyan eszköz, amely nincs szinkronban?

---

# 8.50 Labor 6 – Network Status

Vizsgáld meg:

```text
Network Status
```

Keresd:

- linksebesség;
- network interface;
- esetleges hibákat.

Ezután ellenőrizd ugyanezt az Aruba switch oldalán.

Tanulság:

```text
Dante Controller
        +
Switch management
```

együtt sokkal több információt ad, mint bármelyik önmagában.

---

# 8.51 Labor 7 – Advanced Filter

Hozz létre egy olyan szűrést, amely csak:

```text
FOH
+
48 kHz
```

eszközöket mutat.

Ezután módosítsd:

```text
FOH
+
96 kHz
```

és figyeld meg az eredményt.

Cél:

> Megtanulni nagy hálózatban gyorsan leszűkíteni a keresési területet.

---

# 8.52 Labor 8 – Flow Information

Nyisd meg:

```text
Device View
 ↓
View
 ↓
View Flow Information
```

Vizsgáld:

```text
TX flows
RX flows
Audio flows
```

Hasonlítsd össze a routing matrixban látott subscriptionökkel.

---

# 8.53 Labor 9 – Latency histogram

Támogatott eszközzel:

```text
Device View
 ↓
Latency
```

Figyeld:

```text
Average
Peak
Late packets
Histogram
```

Kérdés:

> A packetek milyen távol vannak a latency limittől?

---

# 8.54 Labor 10 – Preset

A labor aktuális konfigurációjáról:

```text
Save Preset
```

Ezután változtass meg néhány subscriptiont.

Végül töltsd vissza a presetet.

Ellenőrizd:

```text
Routing restored?
Channel names restored?
Configuration restored?
```

---

# 8.55 Labor 11 – Network interface hiba

Ha van több Ethernet interface-szel rendelkező számítógéped, szimuláld a helytelen interface kiválasztását.

Figyeld:

```text
Dante Controller
      ↓
No devices
```

Ezután válaszd ki a megfelelő interface-t.

Tanulság:

> A Dante Controller nem tud olyan Dante hálózatot megjeleníteni, amelyhez a kiválasztott hálózati interfészen keresztül nem fér hozzá.

---

# 8.56 Labor 12 – Subscription diagnosztika

Hozz létre egy működő subscriptiont.

Ezután változtass meg egy kompatibilitási feltételt a labor eszközein.

Vizsgáld:

```text
Routing View
Receive tab
Clock Status
Events
```

Cél:

> Ugyanazt a problémát több Controller-nézetből megtanulni diagnosztizálni.

---

# 8.57 Gyakori hiba – „Nem látok semmit”

Ne kezdd újra a Controller telepítésével.

Ellenőrzési sorrend:

```text
1. Dante Controller interface
2. Ethernet link
3. IP configuration
4. VLAN
5. Switch
6. Dante device power
7. Dante network
```

---

# 8.58 Gyakori hiba – „Látom, de nem tudom routingolni”

Ellenőrizd:

```text
Sample rate
Clock domain
TX availability
RX availability
Subscription compatibility
Device Lock
```

A Device Lock különösen érdekes, mert zárolt transmitterhez új subscription nem feltétlenül hozható létre.

citeturn1search11

---

# 8.59 Gyakori hiba – „Subscription OK, nincs hang”

Ellenőrizd:

```text
Receive → Signal
        ↓
Transmit source
        ↓
Flow Information
        ↓
Latency
        ↓
Network
```

Ne állítsd át automatikusan a subscriptiont.

---

# 8.60 Gyakori hiba – „Minden device piros”

Ha egyszerre sok eszköz problémás:

```text
nem valószínű,
hogy mindegyik endpoint egyszerre romlott el
```

Gondolkodj infrastruktúrában:

```text
Switch
VLAN
Clock
Network interface
Link
Power
```

Ez az egyik legfontosabb hibakeresési szabály:

> **Ha sok eszköz egyszerre hibás, először a közös függőséget keresd.**

---

# 8.61 Gyakori hiba – „A Controller szerint minden jó”

Lehetséges:

```text
Subscription = OK
Clock = OK
```

de:

```text
audio source = nincs jel
```

Ezért a Controller állapota nem helyettesíti az audiojel tényleges ellenőrzését.

A jó diagnosztika:

```text
Configuration
+
Network
+
Clock
+
Flow
+
Actual signal
```

---

# 8.62 Diagnosztikai döntési fa

```text
Dante Controller
       │
       ▼
Látom az eszközöket?
       │
 ┌─────┴─────┐
 NEM         IGEN
 │             │
Network      Routing?
              │
         ┌────┴────┐
        NEM       IGEN
         │          │
   Compatibility  Signal?
   / Device       │
   / Network  ┌───┴───┐
              NEM    IGEN
               │       │
             Source   OK
             / Flow
```

---

# 8.63 A hét legfontosabb Controller-felülete

Kezdőként ezt a hét területet tanuld meg:

```text
1. Routing
2. Device Info
3. Clock Status
4. Network Status
5. Events
6. Device View → Receive / Transmit
7. Device View → Status / Config / Latency
```

Ha ezeket magabiztosan használod, már nem csak routingot tudsz létrehozni, hanem alap Dante diagnosztikát is tudsz végezni.

---

# 8.64 Controller workflow

Egy professzionális munkafolyamat például:

```text
START
  │
  ▼
Select correct network interface
  │
  ▼
Network View
  │
  ├── Device Info
  ├── Clock Status
  └── Network Status
  │
  ▼
Routing
  │
  ▼
Device View
  │
  ├── Receive
  ├── Transmit
  ├── Status
  ├── Config
  └── Latency
  │
  ▼
Flow Information
  │
  ▼
Test audio
  │
  ▼
Save Preset
```

---

# 8.65 Deep Dive – Miért két nézet?

A két nézet két külön mérnöki kérdésre válaszol.

### Network View

> **„Mi történik a rendszerben?”**

### Device View

> **„Mi történik ezzel az eszközzel?”**

Például:

```text
Network View:
StageBox → Console
subscription OK
```

majd:

```text
Device View / Console:
Receive
Signal = -35 dBFS
```

A két információ együtt sokkal erősebb diagnosztikai bizonyíték.

---

# 8.66 Deep Dive – A Controller nem packet analyzer

A Dante Controller sok mindent megmutat:

```text
Routing
Clock
Device
Latency
Flow
```

de nem teljes értékű Ethernet packet analyzer.

Ha mély hálózati problémát keresel, szükség lehet:

```text
Switch counters
+
SNMP
+
Port statistics
+
Packet capture
+
Wireshark
```

A Dante Controller tehát a Dante-rendszer **alkalmazási és endpoint-szintű diagnosztikájának** központi eszköze.

---

# 8.67 Deep Dive – Controller és Aruba switch együtt

Egy valós rendszerben:

```text
Dante Controller
      │
      │ endpoint view
      ▼
Dante device
      │
      │ Ethernet
      ▼
Aruba switch
      │
      │ switch view
      ▼
Port statistics
```

Például ha a Controllerben:

```text
Late packets
```

láthatók, az Aruba oldalon érdemes vizsgálni:

```text
port errors
drops
speed
duplex / negotiation
EEE
QoS
multicast
```

A Controller tehát megmutatja **a tünetet**, a switch pedig segíthet megtalálni **az infrastruktúra okát**.

---

# 8.68 Deep Dive – Device naming mint konfigurációs identitás

A Dante device name nem egyszerű címke.

A routing névhez kötött.

Ezért egy rendszerben:

```text
StageBox-A
```

nem csak azt mondja meg:

> „Ez az A stagebox.”

hanem routing szempontból azt is:

> **„A subscriptionök ezt az identitást használják.”**

Ezért egy következetes névváltoztatás előtt mindig gondolj a routing következményeire.

---

# 8.69 Deep Dive – Miért kell várni konfiguráció után?

Az Audinate jelenlegi dokumentációja szerint routing- vagy névkonfiguráció módosítása után érdemes legalább néhány másodpercet várni az érintett eszközök lekapcsolása vagy újraindítása előtt, hogy az új információ megfelelően elmentődjön.

A dokumentáció legalább 5 másodperc várakozást javasol az ilyen változtatások után.

citeturn1search4

Ez gyakorlati szabály:

```text
Change
 ↓
Wait
 ↓
Verify
 ↓
Power down
```

ne:

```text
Change
 ↓
azonnal reboot
```

---

# 8.70 Vizsgafeladat – Network View vagy Device View?

Döntsd el, melyik nézetet választanád.

### „Melyik eszköz a Primary Leader Clock?”

→ **Network View / Clock Status**

### „Milyen sample rate-en működik ez a StageBox?”

→ **Device View / Device Config**

### „Melyik receiverre van a Vocal routolva?”

→ **Network View / Routing**

### „Van-e tényleges jel a Console RX 1-en?”

→ **Device View / Receive**

### „Hány flow-t használ ez az eszköz?”

→ **Device View / Flow Information**

---

# 8.71 Vizsgafeladat – Nincs eszköz a Controllerben

A Controller üres Network View-t mutat.

Sorold fel az első öt vizsgálatot.

Elvárt:

```text
1. Network interface
2. Ethernet link
3. IP configuration
4. VLAN / switch path
5. Dante device power / network
```

---

# 8.72 Vizsgafeladat – Subscription OK, nincs jel

Adott:

```text
Subscription = OK
Clock = OK
Signal = none
```

Mit vizsgálsz?

```text
TX source
TX channel
Flow
Network
RX signal
```

---

# 8.73 Vizsgafeladat – Late packets

Adott:

```text
Latency histogram
→ late packets
```

Mi lehet a megoldás?

```text
Increase receiver latency
        vagy
Network reconfiguration
```

A döntést a hálózat tényleges viselkedése alapján kell meghozni.

citeturn0search0

---

# 8.74 Vizsgafeladat – Nagy hálózat

100 Dante endpoint van a hálózaton.

A Routing View nehezen kezelhető.

Mit használsz?

```text
Advanced Filter
```

Miért?

```text
Gyorsabban leszűkíthető
a releváns eszközök köre.
```

---

# 8.75 Vizsgafeladat – Preset

Miért nem elég csak egy Dante Controller presetet menteni?

Mert a teljes rendszerhez szükség lehet:

```text
Switch config
VLAN
IP plan
Physical topology
Clock architecture
Firmware
External clock
```

is.

---

# 8.76 Megoldókulcs – röviden

### Network View
A teljes Dante hálózat áttekintése.

### Device View
Egy adott Dante eszköz részletes vizsgálata és konfigurációja.

### Routing
Subscriptionök létrehozása és ellenőrzése.

### Device Info
Eszközök azonosító, IP- és konfigurációs információi.

### Clock Status
Hálózati clock állapot.

### Receive
Receiver csatornák, subscriptionök és jelállapot.

### Transmit
Transmit csatornák és támogatott multicast konfiguráció.

### Status
Eszközállapot és verzióinformációk.

### Device Config
Támogatott eszközök konfigurációja.

### Latency
Packet latency eloszlás és late packet diagnosztika.

### Flow Information
Az eszköz media flow-információi.

### Advanced Filter
Nagy hálózatban célzott keresés.

### Preset
Routing és támogatott konfiguráció mentése/visszaállítása.

---

# 8.77 A fejezet mentális modellje

```text
                  DANTE CONTROLLER
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        NETWORK VIEW           DEVICE VIEW
              │                     │
      ┌───────┼───────┐       ┌─────┼──────────┐
      ▼       ▼       ▼       ▼     ▼          ▼
   Routing  Clock   Network   Rx    Tx       Config
      │       │       │       │     │          │
      ▼       ▼       ▼       └──┬──┘          ▼
   Routes   Timing   IP/Link     Flow       Device
                                      │      settings
                                      ▼
                                  Diagnostics
```

---

# 8.78 Amit ebből a fejezetből tudnod kell

### Dante Controller

A Dante hálózat konfigurációs és felügyeleti eszköze.

### Network View

A teljes hálózat áttekintése.

### Device View

Egy konkrét Dante endpoint részletes vizsgálata.

### Routing

Subscriptionök létrehozása és ellenőrzése.

### Device Info

Az eszközök alapvető információinak áttekintése.

### Clock Status

A hálózat clock állapotának ellenőrzése.

### Receive

Receiver subscriptionök és jelállapot vizsgálata.

### Transmit

Transmit csatornák és multicast flow-k kezelése.

### Flow Information

A tényleges media flow-k áttekintése.

### Latency

A packet érkezési késleltetésének és late packeteknek a vizsgálata.

### Advanced Filter

Nagy hálózat célzott keresése.

### Preset

Routing és konfiguráció mentése.

### Device Lock

Támogatott eszközök konfigurációjának védelme.

---

# 8.79 A legfontosabb gyakorlati szabályok

```text
1. Mindig a megfelelő network interface-et használd.
2. A Network View-ban először az egész rendszert nézd.
3. Device View-ban vizsgáld az egyedi endpointot.
4. Subscription OK ≠ audiojel garantált.
5. Clock Status-t routing hibánál is ellenőrizd.
6. Nagy hálózatban használj Advanced Filtert.
7. Flow Information-t használd kapacitási problémáknál.
8. Latency histogramot használj late packet problémánál.
9. Presetet ments konfigurációváltoztatás előtt.
10. A Dante Controller nem helyettesíti a switch managementet.
```

---

# 8.80 Következő fejezet

# 9. QoS és Multicast

A 7. fejezetben megtanultuk:

```text
Audio Flow
   ↓
Unicast / Multicast
```

A 8. fejezetben megtanultuk:

```text
Dante Controller
   ↓
Routing
Clock
Network
Flow
Diagnostics
```

A következő lépés:

```text
Audio traffic
      +
PTP traffic
      +
Multicast traffic
      ↓
     QoS
      +
IGMP
```

Itt kezdjük el igazán megérteni, **miért kell a Dante hálózati infrastruktúrát megfelelően konfigurálni**.

---

# 8.81 Fejezeti állapot

**Állapot: REVIEWED / JAVÍTOTT – commit előtti ellenőrzésre kész**

A fejezet tartalmaz:

- Dante Controller szerepe;
- Network View;
- Routing;
- Device Info;
- Clock Status;
- Network Status;
- Events;
- Device View;
- Receive;
- Transmit;
- Status;
- Device Config;
- Network Config;
- Latency;
- Flow Information;
- Device Lock;
- Advanced Filter;
- network interface;
- DHCP / Link Local alapok;
- Presets;
- hibakeresési workflow;
- Aruba switch + Controller diagnosztika;
- 12 labor;
- vizsgafeladatok;
- megoldókulcs;
- Deep Dive részek;
- fejezeti mentális modell.

A Dante-specifikus állításokat az Audinate aktuális Dante Controller dokumentációjával ellenőriztem.
