# 11. Dante eszközök, firmware és rendszerüzemeltetés

> **A fejezet célja:** megtanulni, hogyan kell egy Dante rendszert nemcsak felépíteni, hanem hosszú távon biztonságosan, dokumentáltan és reprodukálható módon üzemeltetni.

Az előző fejezetben a hálózati topológiákat és a redundanciát vizsgáltuk.

Most egy másik, nagyon fontos kérdés következik:

> **Mi történik a rendszerrel az első sikeres üzembe helyezés után?**

Egy Dante hálózat ugyanis nem egyszeri konfiguráció.

A rendszer életciklusa:

```text
tervezés
   ↓
telepítés
   ↓
konfiguráció
   ↓
teszt
   ↓
üzemeltetés
   ↓
monitoring
   ↓
karbantartás
   ↓
firmware-frissítés
   ↓
változáskezelés
   ↓
hibaelhárítás
   ↓
visszaállítás
```

A professzionális üzemeltetés célja nem az, hogy soha ne legyen hiba.

A cél:

> **A hiba valószínűségének csökkentése, a hibák gyors felismerése, a változások kontrollálása és a rendszer reprodukálható visszaállítása.**

A fejezetet érdemes három nagy területként fejben tartani:

```text
                 ÜZEMELTETÉS
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
   MANAGEMENT     MAINTENANCE      RECOVERY
       │              │              │
   naming          firmware        backup
   routing         updates         restore
   inventory       testing         rollback
   lock            baseline        failsafe
```

Ez a három terület összekapcsolódik, de nem azonos:

- **Management:** mit és hogyan konfiguráltunk;
- **Maintenance:** hogyan tartjuk a rendszert stabil és támogatott állapotban;
- **Recovery:** hogyan térünk vissza ismert, működő állapotba hiba után.

---

# 11.1 Miért külön fejezet a rendszerüzemeltetés?

Egy Dante rendszer lehet technikailag helyesen konfigurálva, mégis rosszul üzemeltetett.

Például:

```text
minden működik
      ↓
nincs dokumentáció
      ↓
firmware frissítés történik
      ↓
egy eszköz nem működik
      ↓
senki nem tudja, mi volt korábban
```

A probléma nem feltétlenül maga a firmware.

Lehet, hogy:

- nem volt változásnapló;
- nem volt mentett konfiguráció;
- nem volt kompatibilitási ellenőrzés;
- nem volt tesztterv;
- nem volt visszaállítási terv.

Ezért a Dante-rendszer üzemeltetése ugyanúgy **folyamat**, mint a hálózat konfigurációja.

---

# 11.2 A Dante eszköz nem egyetlen szoftververzió

Kezdőként könnyű azt mondani:

> „A készülék firmware-je 4.x.”

A valóság ennél összetettebb lehet.

A Dante Controller Device View / Status nézete különböző verzióinformációkat is megjeleníthet, például:

```text
Manufacturer
Product Type
Product Version
Software Version
Firmware Version
Dante Firmware Version
Hardware Version
ROM / Boot version
```

Az Audinate dokumentációja ezeket külön mezőkként kezeli.

Ezért amikor egy hibát vagy frissítést dokumentálsz, ne csak annyit írj:

```text
firmware = 4.x
```

hanem lehetőleg a releváns verziókat is rögzítsd.

---

# 11.3 Dante firmware és gyártói firmware

A Dante eszközök firmware-kezelése **készülék- és gyártófüggő**.

Egy adott terméknél találkozhatsz:

```text
Dante firmware / platform
        +
gyártói termékfirmware / software
```

de ezt nem szabad általános szabályként kezelni. Nem minden eszközön ugyanaz a firmware-struktúra, és nem minden frissítés ugyanúgy történik.

Ezért nem szabad automatikusan azt feltételezni, hogy:

> „A Dante firmware frissítése mindig a teljes készülék firmware-ének frissítését jelenti.”

A helyes forrás az adott termék gyártói dokumentációja és az adott firmware-frissítési eljárás.

---

# 11.4 Miért kell firmware-t frissíteni?

Firmware-frissítés több okból történhet:

- hibajavítás;
- stabilitásjavítás;
- teljesítményjavítás;
- új funkciók;
- biztonsági javítások;
- új Dante-funkciók támogatása;
- kompatibilitási okok.

Az Audinate Firmware Update Manager dokumentációja szerint a firmware-frissítések új funkciókat adhatnak, javíthatják a teljesítményt és hibákat javíthatnak.

De ebből nem következik:

> **„Mindig azonnal frissítsünk minden eszközt a legújabb verzióra.”**

Professzionális környezetben a helyes kérdés:

> **A frissítés milyen problémát old meg, milyen új funkciót ad, és milyen kockázatot jelent a jelenlegi rendszerre?**

---

# 11.5 A „latest = best” tévedés

A legújabb firmware önmagában nem garantálja, hogy egy adott telepítéshez ez a legjobb választás.

Egy rendszerben lehet:

```text
Device A
Firmware 4.2
```

és:

```text
Device B
Firmware 4.1
```

Ha minden stabil, nem biztos, hogy azonnal szükséges mindkettőt frissíteni.

Másik eset:

```text
Manufacturer
   ↓
recommends firmware X
   ↓
specific product compatibility
```

Ilyenkor viszont a gyártói ajánlás lehet meghatározó.

A helyes üzemeltetési elv:

> **Ne verziószámot frissíts; problémát vagy követelményt kezelj.**

---

# 11.6 Firmware-frissítés előtt: leltár

Mielőtt bármit frissítesz, készíts eszközleltárt.

Minimum:

| Mező | Példa |
|---|---|
| Device name | Stagebox-01 |
| Manufacturer | gyártó |
| Model | modell |
| Serial | sorozatszám |
| Product version | gyártói verzió |
| Dante version | Dante verzió |
| IP address | Primary IP |
| Secondary IP | ha van |
| Location | rack / terem |
| Role | stagebox / console / DSP |
| Current status | működő / hibás |

Ez a későbbi hibakeresés alapja.

---

# 11.7 Dante Controller mint üzemeltetési eszköz

A Dante Controller nem pusztán routing felület.

Az Audinate dokumentációja szerint használható többek között:

- Dante eszközök és csatornák megjelenítésére;
- clock- és hálózati állapot vizsgálatára;
- audio routingra;
- routing presetek mentésére és alkalmazására;
- eszközkonfigurációra;
- Device Lock használatára;
- hálózati és teljesítményinformációk vizsgálatára;
- latency és packet error információk ellenőrzésére.

Ezért a Dante Controller az üzemeltető egyik elsődleges diagnosztikai eszköze.

---

# 11.8 Device Info mint „rendszerleltár”

A Dante Controller Device Info nézete hasznos hálózati és eszközszintű áttekintést ad.

Megjeleníthet többek között:

```text
Device Name
Model Name
Product Version
Dante Version
Primary Address
Primary Link Speed
Secondary Address
Secondary Link Speed
```

A Secondary állapot értelmezésénél mindig azt a konkrét mezőt és nézetet kell figyelembe venni, amelyet a Dante Controller megjelenít. Például a dokumentációban a Secondary kapcsolat hiánya és a nem támogatott Secondary interfész külön állapotként jelenhet meg.

Ezért a Device Info nem csak „információs képernyő”.

> **Egy jól dokumentált rendszerben ez az egyik fontos leltárforrás.**

---

# 11.9 A név nem kozmetika

Egy Dante eszköz neve lehet:

```text
Dante-01
```

vagy:

```text
Stagebox_Left
```

A második sokkal hasznosabb.

Nagy rendszerben:

```text
Dante-01
Dante-02
Dante-03
Dante-04
```

gyorsan értelmezhetetlenné válhat.

Jobb:

```text
FOH-Console
FOH-DSP
STAGE-BOX-A
STAGE-BOX-B
MON-DSP
REC-01
```

A névnek azt kell megmutatnia:

```text
mi?
hol?
milyen szerep?
```

---

# 11.10 Névkonvenció

Egy professzionális rendszerben érdemes előre definiálni:

```text
SITE-AREA-ROLE-NUMBER
```

Például:

```text
THEATER-FOH-DSP-01
THEATER-STAGE-BOX-01
THEATER-STAGE-BOX-02
THEATER-REC-01
```

A szabály fontosabb, mint maga a formátum.

A lényeg:

> **Minden üzemeltető ugyanabból a névből ugyanazt értse.**

---

# 11.11 Channel naming

Az eszköz neve mellett a csatornák neve is fontos.

Rossz:

```text
Input 1
Input 2
Input 3
Input 4
```

Jobb:

```text
VOX_LEAD
KICK
SNARE
GTR_1
```

A routing során így:

```text
KICK → FOH_KICK
```

sokkal könnyebben értelmezhető.

A Dante Controller lehetőséget ad a csatornák számozásának nevére történő cseréjére.

---

# 11.12 Dokumentáció nélkül a rendszer gyorsan elveszti az értékét

Egy jól működő Dante rendszerhez tartozzon legalább:

```text
topology diagram
device inventory
IP plan
VLAN plan
routing map
clock design
firmware matrix
change log
backup / preset files
test results
```

Ha ezek nincsenek:

```text
rendszer
  ↓
ember tudása
  ↓
ember kiesik
  ↓
tudás kiesik
```

A dokumentáció célja, hogy a rendszer ne egyetlen ember fejében létezzen.

---

# 11.13 Routing preset

A Dante Controller képes routing preseteket menteni és alkalmazni.

Ez különösen hasznos:

```text
Show A
Show B
Show C
```

vagy:

```text
Normal
Rehearsal
Recording
Backup
```

konfigurációk esetén.

A preset előnye:

> **A konfiguráció reprodukálható.**

---

# 11.14 Preset ≠ teljes rendszermentés

Nagyon fontos különbség.

Egy routing preset nem feltétlenül tartalmazza a teljes infrastruktúra minden beállítását.

Nem szabad úgy gondolkodni:

> „Van presetem, tehát van backupom.”

A teljes backup lehet például:

```text
routing preset
+
device configuration
+
switch configuration
+
IP/VLAN documentation
+
firmware versions
+
manufacturer configuration files
```

---

# 11.15 Firmware matrix

Nagyobb rendszernél készíts:

| Device | Model | Product FW | Dante FW | Date | Approved |
|---|---|---|---|---|---|
| FOH-DSP | X | 5.x | 4.x | 2026-08 | YES |
| Stagebox-01 | Y | 3.x | 4.x | 2026-08 | YES |
| REC-01 | Z | 2.x | 4.x | 2026-08 | YES |

A cél nem az adminisztráció kedvéért végzett adminisztráció.

A cél:

> **Tudd, hogy a működő rendszer pontosan milyen állapotban van.**

---

# 11.16 Változáskezelés

Minden jelentős változás legyen dokumentálva.

Például:

```text
2026-08-17
Stagebox-01 firmware update
4.1 → 4.2
Reason: manufacturer bug fix
Result: PASS
```

Ez később aranyat ér.

Ha másnap probléma jelentkezik:

```text
Mi változott?
```

Az első kérdések egyike lehet:

> „Mi változott a legutóbbi működő állapot óta?”

---

# 11.17 A „working state” fogalma

Minden professzionális rendszernek legyen egy ismert működő állapota.

Például:

```text
Baseline 2026-08-17
```

Ehhez tartozik:

```text
device list
firmware
routing
network config
switch config
```

Ha egy változtatás után probléma van:

```text
Current state
     ↓
compare
     ↓
Baseline
```

Ez nagyságrendekkel gyorsabb hibakeresést tesz lehetővé.

---

# 11.18 Firmware-frissítési stratégia

Javasolt folyamat:

```text
1. Identify
2. Read release notes
3. Check manufacturer recommendation
4. Check compatibility
5. Backup
6. Schedule maintenance window
7. Update one device / controlled group
8. Verify
9. Test
10. Document
```

Ne:

```text
Download
   ↓
Update everything
   ↓
hope
```

---

# 11.19 Firmware-frissítés és kompatibilitás

A kompatibilitás több szinten értelmezhető:

```text
Dante firmware
      +
product firmware
      +
Dante Controller
      +
DDM / management
      +
switch environment
```

Ezért frissítés előtt mindig ellenőrizd:

```text
manufacturer release notes
supported firmware
required software version
known limitations
```

A firmware-verziók közötti kompatibilitást nem lehet pusztán a verziószámokból levezetni.

---

# 11.20 Mikor ne frissíts?

Például ha:

- éles show fut;
- nincs karbantartási ablak;
- nincs visszaállítási terv;
- nincs dokumentált jelenlegi állapot;
- a gyártó nem ajánlja az adott verziót;
- nem tudod, mit javít a frissítés;
- nincs mód a tesztelésre.

A „latest” nem sürgősségi kategória.

A „critical security fix” vagy bizonyított stabilitási hiba viszont más prioritás lehet.

---

# 11.21 Dante Updater

A Dante Updater a Dante Controller telepítésével együtt elérhető alkalmazás.

Az Audinate dokumentációja szerint:

- felismeri a hálózaton található Dante eszközöket;
- lekérdezi a firmware-verziókat;
- az online adatbázisban elérhető firmware-frissítéseket jelzi;
- lehetővé teszi az update telepítését;
- a firmware fájl helyi letöltése offline használatra is lehetséges;
- több eszköz frissítése is kezelhető.

Ez nagyon kényelmes.

De:

> **A kényelem nem helyettesíti a változáskezelést.**

---

# 11.22 Dante Updater használata – ajánlott workflow

```text
Dante Updater
      ↓
Scan
      ↓
Review available updates
      ↓
Identify target devices
      ↓
Check manufacturer notes
      ↓
Backup
      ↓
Update
      ↓
Reboot / wait as required
      ↓
Verify
```

A frissítés alatt ne módosíts párhuzamosan más konfigurációkat.

---

# 11.23 Mi történik, ha megszakad a firmware-frissítés?

Ez ritka, de fontos.

Az Audinate dokumentációja szerint firmware-frissítés megszakadása például áramkimaradás vagy hálózati hiba miatt firmware-korrupcióhoz és failsafe mode-hoz vezethet.

Ezért firmware-frissítés közben:

```text
power stable
network stable
no unnecessary changes
```

legyen az alap.

---

# 11.24 Failsafe mode

A failsafe mode nem normál firmware-frissítési állapot.

Akkor fordulhat elő, ha a készüléken tárolt firmware image sérült.

Az Audinate dokumentáció szerint ilyen esetben a Firmware Update Manager Failsafe Recovery funkciója használható, ha az adott eszköz támogatja; más esetben a készülék gyártójának támogatása szükséges. A recovery eljárásnak az adott eszköz dokumentációjában meghatározott hálózati és működési feltételei lehetnek.

Ezért:

> **A failsafe recovery nem az elsődleges frissítési módszer.**

Normál esetben:

```text
Firmware Update
```

használatos.

---

# 11.25 Firmware Update vs Failsafe Recovery

Ne keverd össze:

### Firmware Update

```text
működő eszköz
      ↓
új firmware
```

### Failsafe Recovery

```text
firmware probléma
      ↓
failsafe állapot
      ↓
recovery
```

Az Audinate Firmware Update Manager dokumentációja külön kezeli ezt a két működési módot.

---

# 11.26 Miért kell karbantartási ablak?

Egy firmware-frissítés:

```text
reboot
+
audio interruption
+
temporary network state
```

hatással lehet a rendszerre.

Ezért:

```text
Show
  ↓
NO
```

hanem:

```text
Maintenance window
  ↓
YES
```

A pontos hatás készülék- és gyártófüggő, ezért a gyártói dokumentációt mindig ellenőrizni kell.

---

# 11.27 Daisy-chain és firmware-frissítés

Daisy-chainelt eszközöknél a frissítésnek további következménye lehet.

Az Audinate dokumentációja szerint bizonyos, belső Ethernet switch-et tartalmazó eszközök frissítés közben upgrade mode-ba kerülhetnek, és a reboot időzítése befolyásolhatja a mögöttük lévő daisy-chain eszközök media- és firmware-forgalmát.

Ezért:

> **Daisy-chain esetén a frissítési sorrend és reboot időzítése különösen fontos.**

---

# 11.28 Frissítés után nem azonnal „kész”

A firmware update sikeres befejezése nem azonos a rendszer sikeres tesztjével.

Update után:

```text
Device visible?
   ↓
Clock locked?
   ↓
Network healthy?
   ↓
Subscriptions?
   ↓
Audio?
   ↓
Latency?
   ↓
Redundancy?
```

csak ezután mondjuk:

> **A frissítés sikeres.**

---

# 11.29 Post-update ellenőrzőlista

```text
□ Device appears
□ Correct device name
□ Correct firmware
□ Primary link UP
□ Secondary link UP if applicable
□ Clock locked
□ Subscriptions present
□ Audio passing
□ No unexpected errors
□ Latency normal
□ Redundancy healthy
□ Preset still valid
□ Test completed
```

---

# 11.30 Device Lock

A Dante Controller támogatja a Dante eszközök Device Lock funkcióját.

A Device Lock célja, hogy megakadályozza a készülék konfigurációjának nem kívánt módosítását. Zárolt állapotban a meglévő media subscriptionök tovább működhetnek, miközben a készülék konfigurációja korlátozott / read-only állapotba kerül.

Ez különösen hasznos lehet:

```text
production
broadcast
theater
permanent installation
```

környezetben.

De:

> **A Device Lock nem helyettesíti a teljes hálózati és hozzáférés-biztonsági modellt.**

---

# 11.31 Miért veszélyes a véletlen konfiguráció?

Dante Controllerből nagyon könnyű lehet olyan változtatást végrehajtani, amely azonnal hat a rendszerre.

Például:

```text
subscription change
sample rate change
clock setting
device name
latency
```

Ezért nagy rendszernél a jogosultságok és a változtatási folyamat fontos.

---

# 11.32 Operátori és mérnöki szerepek

Érdemes megkülönböztetni:

```text
Operator
Engineer
Administrator
```

Például:

### Operator

```text
monitor
identify
basic routing
```

### Engineer

```text
configuration
routing
clock
network troubleshooting
firmware
```

### Administrator

```text
DDM
security
user management
domain
infrastructure
```

A konkrét jogosultsági modell a rendszer és a használt menedzsmentkörnyezet függvénye.

---

# 11.33 Hibakeresés – ne változtass azonnal

Ha valami nem működik:

```text
Audio = intermittent
```

a kezdő reakció:

> „Állítsunk át valamit.”

A jó reakció:

```text
Observe
   ↓
Document
   ↓
Hypothesis
   ↓
Test
   ↓
Change
   ↓
Verify
```

Ez az egyik legfontosabb szakmai szokás.

---

# 11.34 „Mi változott?”

Hiba esetén első kérdések:

```text
Mikor kezdődött?
Mi változott?
Melyik eszköz?
Minden flow?
Egy flow?
Mindkét hálózat?
Clock?
Network?
```

A hibakeresés nem találgatás.

A cél:

> **A lehetséges okok számának gyors csökkentése.**

---

# 11.35 Hibakeresési rétegek

Hasznos **gyakorlati hibakeresési sorrend**:

```text
Physical
   ↓
Ethernet / VLAN
   ↓
IP / routing, ha releváns
   ↓
Dante discovery / control
   ↓
Clock
   ↓
Audio routing
   ↓
Application / device
```

Ez **nem egy új OSI-modell és nem azt jelenti, hogy a Dante egy külön OSI-réteg**. Ez egy diagnosztikai gondolkodási sorrend: az alsóbb infrastruktúra problémáit érdemes kizárni, mielőtt magasabb szintű konfigurációs okokat keresünk.

Nem minden Dante-hiba IP-routing probléma.

És nem minden audiohiba audio-routing probléma.

---

# 11.36 Egy eszköz nem látszik

Ha:

```text
Device X
```

nem jelenik meg:

Először:

```text
Power
Cable
Link LED
Switch port
VLAN
IP
```

majd:

```text
Dante Controller
Network interface
```

és csak utána:

```text
device configuration
```

Ne a routinggal kezdd, ha maga az eszköz sem látszik.

---

# 11.37 Eszköz látszik, de nincs audio

Most más a helyzet.

```text
Device = visible
```

de:

```text
Audio = no
```

Vizsgáld:

```text
Subscription
TX channel
RX channel
Clock
Sample rate
Latency
Flow
Multicast
```

A fizikai réteg valószínűleg működik, de ezt továbbra is ellenőrizni kell.

---

# 11.38 Audio szakadozik

Intermittent hiba esetén különösen fontos:

```text
packet errors
clock events
link errors
bandwidth
QoS
multicast
switch port
cable
```

A Dante Controller képes hálózati teljesítmény- és hibainformációkat megjeleníteni, beleértve latency és packet error adatokat is.

---

# 11.39 Firmware-hiba vagy hálózati hiba?

A tünet önmagában nem dönt.

Például:

```text
audio drop
```

lehet:

```text
firmware bug
network congestion
packet loss
clock issue
switch issue
bad cable
```

Ezért a firmware-t nem szabad automatikusan hibaforrásnak tekinteni.

---

# 11.40 Recovery terv

Minden kritikus Dante rendszerhez legyen:

```text
Recovery plan
```

Legalább:

```text
known-good firmware
known-good configuration
routing preset
device inventory
switch backup
network diagram
contact list
```

Ha a rendszer leáll:

```text
panic
```

helyett:

```text
procedure
```

kell.

---

# 11.41 Rollback

Ha egy változtatás után a rendszer rosszabb lett:

```text
Current
  ↓
known bad
```

vissza kell tudni térni:

```text
Known Good
```

állapotba.

Rollback lehet:

```text
firmware rollback
configuration rollback
routing preset restore
switch config restore
```

A pontos lehetőség mindig az adott eszköztől és gyártótól függ.

---

# 11.42 Backup stratégia

Legalább három szintet érdemes megkülönböztetni:

### 1. Configuration backup

```text
routing
device settings
presets
```

### 2. Infrastructure backup

```text
switch config
VLAN
IP plan
network topology
```

### 3. Documentation backup

```text
inventory
firmware matrix
change log
test reports
```

A backup akkor ér valamit, ha:

> **vissza is tudod állítani.**

---

# 11.43 Restore teszt

Egy backupot időnként ellenőrizni kell.

Például:

```text
Backup exists?
      ↓
Can open?
      ↓
Correct version?
      ↓
Can restore?
      ↓
System works?
```

A nem tesztelt backup csak feltételezett backup.

---

# 11.44 Üzemeltetési rutin

Az ellenőrzések gyakoriságát a rendszer kritikalitása, rendelkezésre állási követelménye és üzemeltetési szabályzata határozza meg.

### Eseményenként / üzembe helyezéskor

```text
Network status
Clock
critical devices
redundancy
```

### Rendszeres ellenőrzésként

```text
error review
device status
redundancy
```

### Karbantartási ablakban

```text
firmware
configuration review
backup
test
```

### Változás után

```text
document
verify
baseline update
```

A fenti kategóriák **példák**, nem univerzális kötelező időintervallumok.

---

# 11.45 Labor 1 – Eszközleltár

Készíts táblázatot:

```text
Device
Model
Location
Role
Firmware
IP
Primary
Secondary
```

Cél:

> A rendszer legyen reprodukálhatóan dokumentált.

---

# 11.46 Labor 2 – Dante Controller audit

Nyisd meg a Device Info nézetet.

Jegyezd fel:

```text
Device Name
Model
Product Version
Dante Version
Primary
Secondary
Link Speed
```

Hasonlítsd össze a saját leltáraddal.

---

# 11.47 Labor 3 – Firmware matrix

Készíts:

```text
Device
Current version
Approved version
Target version
Reason
Date
Result
```

Ne frissíts még.

A cél a változás megtervezése.

---

# 11.48 Labor 4 – Routing preset

Hozz létre működő routingot.

Mentsd el.

Majd:

```text
change routing
```

és állítsd vissza a mentett presetből.

Cél:

> Megérteni a reprodukálható konfiguráció értékét.

---

# 11.49 Labor 5 – Device naming

Hozz létre névkonvenciót.

Például:

```text
LAB-FOH-DSP-01
LAB-STAGE-BOX-01
LAB-STAGE-BOX-02
LAB-REC-01
```

A neveket következetesen alkalmazd.

---

# 11.50 Labor 6 – Controlled firmware update

Csak laborban, megfelelő gyártói firmware-rel.

Lépések:

```text
inventory
backup
release notes
update
reboot
verify
document
```

Ne hagyd ki a verification lépést.

---

# 11.51 Labor 7 – Firmware update megszakításának elméleti vizsgálata

Ne éles rendszeren!

Vizsgáld meg az Audinate dokumentáció alapján:

```text
Mi történik firmware-korrupció esetén?
Mikor kell Failsafe Recovery?
Mikor kell gyártói támogatás?
```

Cél:

> Megérteni a recovery fogalmát anélkül, hogy felesleges kockázatot vállalnánk.

---

# 11.52 Labor 8 – Hibakeresési napló

Szimulálj:

```text
Audio intermittent
```

Dokumentáld:

```text
Time
Symptom
Affected devices
Clock
Network
Hypothesis
Test
Result
Fix
```

Ezután értékeld:

> Melyik adat rövidítette volna legjobban a hibakeresést?

---

# 11.53 Labor 9 – Baseline

Mentsd el:

```text
Baseline-01
```

majd változtass:

```text
device name
routing
```

Ezután hasonlítsd össze a jelenlegi és baseline állapotot.

---

# 11.54 Labor 10 – Recovery gyakorlat

Állíts elő kontrollált hibát:

```text
routing wrong
```

majd:

```text
restore preset
verify
document
```

Cél:

> A hibából történő visszatérés legyen eljárás, ne improvizáció.

---

# 11.55 Vizsgafeladat – Miért kell firmware-frissítést tervezni?

**Válasz:**

Mert a firmware-változás hatással lehet a készülék működésére, kompatibilitására és a rendszer rendelkezésre állására. A frissítés előtt ezért ellenőrizni kell a gyártói dokumentációt, kompatibilitást, mentést és a visszaállítási lehetőséget.

---

# 11.56 Vizsgafeladat – Dante Updater

**Kérdés:**

Mire való a Dante Updater?

**Válasz:**

A Dante eszközök firmware-frissítésének kezelésére. A hálózaton felismeri az eszközöket, összeveti a firmware-információkat az elérhető frissítésekkel, és támogatott esetben lehetővé teszi a firmware telepítését.

---

# 11.57 Vizsgafeladat – Failsafe

**Kérdés:**

Mikor használunk Failsafe Recovery-t?

**Válasz:**

Nem normál firmware-frissítéshez. Akkor lehet szükség rá, amikor egy eszköz firmware image-e sérült és failsafe állapotba került, feltéve hogy az adott eszköz támogatja ezt a recovery módot.

---

# 11.58 Vizsgafeladat – Device Info

**Kérdés:**

Mit jelent egy Secondary állapot?

**Válasz:**

A pontos jelentést a Dante Controller konkrét nézetében megjelenő mező és státusz alapján kell értelmezni. A nem támogatott Secondary interfész és a támogatott, de pillanatnyilag nem kapcsolódó Secondary interfész külön állapot lehet.

---

# 11.59 Vizsgafeladat – Backup

**Kérdés:**

A routing preset önmagában teljes rendszerbackup?

**Válasz:**

Nem feltétlenül. A teljes rendszer helyreállításához szükség lehet eszközkonfigurációra, switch-konfigurációra, firmware-információkra, IP/VLAN dokumentációra és egyéb gyártói konfigurációs adatokra is.

---

# 11.60 Vizsgafeladat – Hibakeresés

**Kérdés:**

Mi a helyes első reakció egy intermittáló audiohibánál?

**Válasz:**

Nem azonnali konfigurációs változtatás. Először meg kell figyelni és dokumentálni a tünetet, majd hipotézist felállítani, célzott tesztet végezni, és csak ezután változtatni.

---

# 11.61 Deep Dive – Configuration drift

A **configuration drift** azt jelenti, hogy a rendszer jelenlegi állapota fokozatosan eltér a dokumentált vagy jóváhagyott baseline-tól.

Példa:

```text
Baseline
   ↓
Device A name
Device B routing
Device C latency
```

majd hónapok alatt:

```text
A name changed
B routing changed
C latency changed
```

de senki nem dokumentálta.

Ekkor:

> **A rendszer működhet, de már nem reprodukálható.**

---

# 11.62 Deep Dive – Change control

A változáskezelés minimális modellje:

```text
Request
   ↓
Reason
   ↓
Risk
   ↓
Backup
   ↓
Change
   ↓
Test
   ↓
Approve
   ↓
Document
```

Nem minden apró változásnak kell vállalati bürokráciává válnia.

De kritikus Dante-rendszernél a nagy változás legyen visszakövethető.

---

# 11.63 Deep Dive – Known Good Configuration

A **Known Good Configuration**, röviden KGC, egy olyan állapot, amelyről tudjuk:

```text
audio OK
clock OK
network OK
routing OK
redundancy OK
```

Ha hiba történik:

```text
Current
   ↓
compare with KGC
```

A KGC ezért sokszor értékesebb, mint egy általános „backup”.

---

# 11.64 Deep Dive – Miért nem frissítünk mindent egyszerre?

Ha:

```text
20 devices
```

egyszerre frissül, és:

```text
problem
```

történik, nehéz megmondani:

```text
which device?
which change?
which firmware?
```

Controlled rollout:

```text
1 device
   ↓
test
   ↓
small group
   ↓
test
   ↓
remaining devices
```

jelentősen csökkentheti a diagnosztikai bizonytalanságot.

A konkrét rollout-stratégia a rendszer kritikalitásától és a gyártói ajánlásoktól függ.

---

# 11.65 Deep Dive – Canary device

Nagy rendszerben kijelölhető egy:

```text
Canary device
```

amelyen először teszteljük a változtatást.

Például:

```text
Stagebox-TEST
```

Ha:

```text
update
↓
test
↓
PASS
```

akkor lehet továbbhaladni.

Ez különösen hasznos, ha sok hasonló készülék van.

---

# 11.66 Deep Dive – Firmware ≠ hálózati konfiguráció

Egy firmware-frissítés után lehet, hogy:

```text
firmware = correct
```

de:

```text
network = wrong
```

vagy:

```text
routing = wrong
```

Ezért a post-update tesztnek több réteget kell vizsgálnia:

```text
device
network
clock
routing
audio
redundancy
```

---

# 11.67 Mélyebb üzemeltetési modell

Egy professzionális Dante rendszer:

```text
HARDWARE
   +
FIRMWARE
   +
NETWORK
   +
CONFIGURATION
   +
DOCUMENTATION
   +
PROCESS
```

Ha bármelyik hiányzik, a rendszer üzemeltethetősége romlik.

---

# 11.68 Gyakorlati üzemeltetési ellenőrzőlista

## Eszközök

```text
□ Device inventory
□ Serial numbers
□ Locations
□ Roles
□ Firmware versions
□ Product versions
```

## Hálózat

```text
□ IP plan
□ VLAN plan
□ Switch configuration
□ Port mapping
□ Primary / Secondary
□ QoS
□ Multicast
```

## Dante

```text
□ Device names
□ Channel names
□ Routing
□ Clock
□ Latency
□ Redundancy
```

## Karbantartás

```text
□ Firmware matrix
□ Approved versions
□ Release notes
□ Backup
□ Maintenance window
□ Rollback plan
```

## Dokumentáció

```text
□ Topology
□ Baseline
□ Change log
□ Test results
□ Recovery procedure
```

---

# 11.69 A fejezet mentális modellje

```text
              DANTE SYSTEM
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       DEVICES             NETWORK
          │                   │
      firmware             switch
          │                VLAN/QoS
          │               multicast
          └─────────┬─────────┘
                    ▼
               CONFIGURATION
                    │
              routing / clock
                    │
                    ▼
                BASELINE
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      OPERATION           MAINTENANCE
          │                   │
       monitor             update
       diagnose            backup
          │                   │
          └─────────┬─────────┘
                    ▼
                 RECOVERY
```

---

# 11.70 Amit ebből a fejezetből tudnod kell

### Firmware

A Dante rendszerben a firmware a készülék működésének fontos része, de a termékfirmware és a Dante firmware fogalma készülékenként eltérhet.

### Dante Updater

Firmware-frissítések kezelésére szolgáló alkalmazás, amely a Dante Controller telepítésével együtt érhető el.

### Firmware update

Tervezett változtatás, amelyet kompatibilitás-, mentés-, teszt- és rollback-szempontból kell kezelni.

### Failsafe Recovery

Sérült firmware image esetén használható helyreállítási folyamat, ha az adott eszköz támogatja.

### Baseline

Ismert, dokumentált, működő rendszerállapot.

### Configuration drift

A tényleges konfiguráció fokozatos eltérése a dokumentált baseline-tól.

### Change control

A rendszer változásainak tervezett, tesztelt és dokumentált kezelése.

### Recovery

A rendszer ismert jó állapotba történő visszaállítása.

---

# 11.71 A legfontosabb szabályok

```text
1. Ne frissíts csak azért, mert van új firmware.
2. Frissítés előtt mindig ismerd a jelenlegi állapotot.
3. Ellenőrizd a gyártói release note-okat.
4. Tudd, melyik firmware- és szoftververzió fut.
5. Legyen ismert működő baseline.
6. Legyen visszaállítási terved.
7. A routing preset hasznos, de nem feltétlenül teljes backup.
8. Firmware-frissítés után mindig tesztelj.
9. Egy működő rendszer állapotát dokumentáld.
10. A változásokat követhetően rögzítsd.
11. Intermittáló hibánál először megfigyelj, utána változtass.
12. A failsafe recovery nem normál firmware-frissítés.
13. A legújabb firmware nem automatikusan a legjobb választás.
14. A backupot időnként visszaállítással is teszteld.
15. A jó üzemeltetés célja a reprodukálhatóság.
```

---

# 11.72 Következő fejezet

# 12. Dante haladó hibakeresés és diagnosztika

A 11. fejezetben megtanultuk:

```text
Device management
Firmware
Dante Updater
Configuration
Baseline
Change control
Backup
Recovery
Monitoring
```

A következő kérdés:

> **Mit csinálunk akkor, amikor a rendszer nem úgy működik, ahogy kellene, és a hiba nem nyilvánvaló?**

A következő fejezetben:

```text
symptom
   ↓
evidence
   ↓
hypothesis
   ↓
test
   ↓
isolation
   ↓
root cause
   ↓
fix
   ↓
verification
```

---

# 11.73 Fejezeti állapot

**Állapot: FINAL / COMMITÁLHATÓ**

A fejezet tartalmaz:

- Dante eszköz- és firmware-fogalmak;
- Dante Controller mint üzemeltetési eszköz;
- Device Info;
- eszköz- és csatornanévadás;
- dokumentáció;
- routing preset;
- firmware matrix;
- change control;
- baseline;
- firmware-frissítési stratégia;
- Dante Updater;
- firmware-frissítés előtti és utáni ellenőrzés;
- Failsafe Recovery;
- Device Lock;
- hibakeresési alapelvek;
- backup és restore;
- recovery;
- 10 gyakorlati labor;
- vizsgafeladatok;
- Deep Dive részek;
- üzemeltetési ellenőrzőlista.

A fejezet végleges változata commitálható.