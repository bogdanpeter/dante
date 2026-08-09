---
author: Peter Bogdan
chapter: 2
chapter_title: Digitális audió
status: complete
title: DANTE -- A professzionális Audio over IP rendszerek kézikönyve
version: 1
---

# 2. Digitális audió

## A fejezet célja

Az előző fejezetben azt vizsgáltuk meg, miért vált szükségessé az Audio
over IP, és milyen problémákat old meg a Dante.

Most egy lépéssel visszalépünk.

Mielőtt egy hangjelet Etherneten, IP-n és Dante-flow-kban továbbítanánk,
meg kell értenünk, **mi az a digitális adat, amelyet egyáltalán
továbbítunk**.

A fejezet végére képes leszel:

-   megmagyarázni, hogyan lesz egy folyamatos analóg jelből digitális
    adat;
-   megkülönböztetni a mintavételezést és a kvantálást;
-   értelmezni a sample rate és bit depth fogalmát;
-   megérteni a Nyquist--Shannon mintavételi tétel lényegét;
-   felismerni az aliasing okát és megelőzésének módját;
-   megérteni a kvantálási hibát és a kvantálási zaj fogalmát;
-   elmagyarázni, miért használunk dithert bitmélység-csökkentéskor;
-   megkülönböztetni az ADC, DAC, sample rate converter és DSP szerepét;
-   kiszámítani alapvető digitális audió-adatmennyiségeket;
-   megérteni, hogyan kapcsolódik mindez a későbbi Dante-sávszélesség-
    és hálózattervezéshez.

------------------------------------------------------------------------

## 2.1 Az analóg hangból digitális adat

A természetes hangjel folyamatos.

Egy mikrofon membránja folyamatosan mozog, és ebből egy analóg
elektromos jel keletkezik.

Leegyszerűsítve:

``` text
Hangnyomás
    │
    ▼
Mikrofon
    │
    ▼
Analóg feszültség
    │
    ▼
ADC
    │
    ▼
Digitális minták
```

A digitális rendszer már nem egy folyamatos feszültséggel dolgozik.

A jelből számszerű értékek sorozata lesz.

Például:

``` text
0.12
0.18
0.27
0.31
0.24
0.10
-0.08
...
```

Ezeket a mintákat lehet:

-   tárolni;
-   feldolgozni;
-   keverni;
-   effektezni;
-   tömöríteni;
-   hálózaton továbbítani;
-   majd DAC segítségével ismét analóg jellé alakítani.

Az egész folyamatot két alapvető lépésre érdemes bontani:

1.  **mintavételezés** -- mikor mérjük meg a jelet;
2.  **kvantálás** -- milyen pontossággal írjuk le a mért értéket.

Az Analog Devices ADC-áttekintése is ezt a két alapvető műveletet
különíti el: az ADC időben mintavételezi a jelet, majd véges számú
digitális kódra kvantálja. citeturn0search0turn0search3

------------------------------------------------------------------------

## 2.2 Mintavételezés

A mintavételezés azt jelenti, hogy a folyamatos analóg jelből
meghatározott időközönként mintát veszünk.

Egy egyszerű szemléltetés:

``` text
Analóg jel:

       /\          /\
      /  \        /  \
-----/----\------/----\-----

Minták:

       •           •
     •   •       •   •
----•-----•-----•-----•------
```

A minták időben diszkrét pontok.

A mintavételi frekvencia, vagy **sample rate**, azt mondja meg, hogy
másodpercenként hány mintát veszünk.

Például:

-   44,1 kHz = 44 100 minta/s;
-   48 kHz = 48 000 minta/s;
-   96 kHz = 96 000 minta/s;
-   192 kHz = 192 000 minta/s.

A kHz itt nem „minőségi kategória".

Egyszerűen azt jelenti, hogy egy másodperc alatt hány mintát veszünk.

------------------------------------------------------------------------

## 2.3 Sample rate és időfelbontás

A sample rate-ből kiszámítható a két egymást követő minta közötti idő:

\[ T_s = `\frac{1}{f_s}`{=tex} \]

48 kHz esetén:

\[ T_s = `\frac{1}{48000}`{=tex} `\approx 20.833`{=tex}  `\mu`{=tex}s \]

Tehát 48 kHz-es mintavételnél körülbelül 20,83 mikrosekundum telik el
két minta között.

96 kHz esetén:

\[ T_s `\approx 10.417`{=tex}  `\mu`{=tex}s \]

A magasabb sample rate tehát sűrűbb időbeli mintavételezést jelent.

Fontos azonban:

> **A magasabb sample rate nem egyszerűen azt jelenti, hogy „jobb
> hang".**

A sample rate meghatározza a reprezentálható frekvenciatartományt és a
digitális feldolgozás bizonyos feltételeit, miközben növeli a
feldolgozandó adatmennyiséget.

Az Analog Devices összefoglalója szerint a mintavételi frekvencia fele a
Nyquist-frekvencia, és a gyakorlatban a mintavételi frekvenciát a kívánt
sávszélességnél magasabbra választják, hogy az analóg szűrőknek legyen
megfelelő átmeneti tartománya. citeturn0search1turn0search2

------------------------------------------------------------------------

## 2.4 A Nyquist--Shannon mintavételi tétel

A digitális audió egyik legfontosabb fogalma.

A leegyszerűsített szabály:

> Egy sávkorlátozott jel veszteségmentes rekonstrukciójához a
> mintavételi frekvenciának nagyobbnak kell lennie a jel legmagasabb
> frekvenciájának kétszeresénél.

Ha a legmagasabb megőrzendő frekvencia:

\[ f\_{max} \]

akkor:

\[ f_s \> 2f\_{max} \]

A sample rate fele a **Nyquist frequency**:

\[ f_N = `\frac{f_s}{2}`{=tex} \]

### Példa

Ha egy jel legfeljebb 20 kHz-ig tartalmaz hasznos információt,
elméletileg legalább 40 kHz feletti mintavétel szükséges.

Ezért a 44,1 kHz-es mintavétel alkalmas a 20 kHz körüli audiósáv
ábrázolására.

A valós ADC-kben azonban nem ideális, függőleges falú szűrőket
használunk. Az analóg anti-aliasing szűrőnek átmeneti tartományra van
szüksége, ezért a gyakorlatban a sample rate és a hasznos audiósáv
között megfelelő tartalékot alkalmazunk.
citeturn0search0turn0search1

------------------------------------------------------------------------

## 2.5 Mi történik, ha túl alacsony a sample rate?

Ha a jel olyan frekvenciakomponenst tartalmaz, amely a
Nyquist-frekvencia fölé kerül, az mintavételezés után **aliasingként**
jelenhet meg.

Ez nem egyszerűen „gyengébb hang".

Az eredeti frekvenciából egy másik, alacsonyabb frekvenciájú komponens
jelenhet meg.

Például ha:

-   sample rate = 48 kHz;
-   Nyquist-frekvencia = 24 kHz;

és a bemeneten 30 kHz-es komponens van, akkor az a mintavételezés után
18 kHz-es alias komponensként jelenhet meg.

Általános esetben az aliasing frekvenciája a mintavételi frekvenciához
képest „visszahajló" komponensként értelmezhető.

### Miért veszélyes?

Az aliasing azért különösen kellemetlen, mert a hibás komponens már a
digitális adatban jelenik meg.

Nem egyszerűen arról van szó, hogy a DAC „nem tudja jól visszaadni".

A mintavételezés során keletkezett alias komponens a digitális jel része
lett.

Az Analog Devices anyagai szerint az aliasing elkerülésének alapvető
módszere az, hogy a mintavétel előtt megfelelően szűrjük azokat a
bemeneti komponenseket, amelyek a kívánt Nyquist-sávon kívülre esnek.
citeturn0search4turn0search6

------------------------------------------------------------------------

## 2.6 Anti-aliasing filter

Az ADC bemenetén ezért jellemzően analóg aluláteresztő szűrésre van
szükség.

``` mermaid
flowchart LR
    IN["Analóg jel"] --> LP["Anti-aliasing filter"]
    LP --> ADC["ADC"]
    ADC --> DATA["Digitális minták"]
```

A szűrő feladata:

> Ne engedje be az ADC-be azokat a frekvenciakomponenseket, amelyek a
> mintavétel után nem reprezentálhatók egyértelműen.

Ezért a valós rendszerben nem egyszerűen:

``` text
20 kHz jel → 40 kHz sample rate
```

a teljes történet.

A tervezés része:

``` text
hasznos audiósáv
       │
       ▼
anti-aliasing filter
       │
       ▼
Nyquist-határ
       │
       ▼
sample rate
```

A modern konvertereknél az analóg szűrés, oversampling és digitális
szűrés együtt alkothatja a teljes konverziós láncot.

------------------------------------------------------------------------

## 2.7 Oversampling

Az oversampling azt jelenti, hogy a konverter vagy feldolgozási lánc a
minimálisan szükséges mintavételnél jóval magasabb belső sample rate-et
használ.

Ez több szempontból hasznos lehet.

A magasabb belső sample rate:

-   távolabbra helyezi a Nyquist-határt;
-   nagyobb átmeneti tartományt adhat a szűrőknek;
-   lehetővé teszi a digitális szűrés nagyobb részének használatát;
-   bizonyos ADC-architektúrákban javíthatja a teljesítményt.

A sigma-delta konverterek például tipikusan oversamplinget és noise
shapinget alkalmaznak. Az Analog Devices dokumentációja szerint az
oversampling a kvantálási zajt szélesebb sávra terítheti, amelyből a
digitális szűrés a kívánt sávon kívüli részt eltávolíthatja.
citeturn0search8turn0search14

Fontos megkülönböztetés:

> **Az ADC belső oversamplingje és a fájl vagy Dante-rendszer által
> használt végső sample rate nem feltétlenül ugyanaz.**

------------------------------------------------------------------------

# 2.8 Kvantálás

A mintavételezés az időt diszkretizálja.

A kvantálás az amplitúdót.

Egy analóg jel elvileg végtelen sok amplitúdóértéket vehet fel.

Egy N bites digitális rendszer viszont csak véges számú kódot tud
reprezentálni.

Ideális esetben:

\[ N\_{codes}=2\^N \]

Például:

-   8 bit → 256 kód;
-   16 bit → 65 536 kód;
-   24 bit → 16 777 216 kód.

Az ADC tehát nem tud minden lehetséges analóg feszültséghez külön
digitális értéket rendelni.

A legközelebbi kvantálási szintre kell kerekítenie.

Az Analog Devices leírása szerint ez elkerülhetetlen kvantálási hibát
eredményez, mert a folytonos amplitúdót véges számú digitális kódra kell
leképezni. citeturn0search0turn0search3

------------------------------------------------------------------------

# 2.9 Kvantálási hiba

Tegyük fel, hogy egy adott analóg minta valódi értéke:

``` text
2.37
```

A rendszer azonban csak bizonyos lépcsőkben tud reprezentálni:

``` text
2.0
2.5
3.0
...
```

A 2.37 érték például 2.5-re kerülhet.

A különbség:

\[ e_q=x-x_q \]

ahol:

-   \(x\) az eredeti érték;
-   (x_q) a kvantált érték;
-   (e_q) a kvantálási hiba.

Ez az eltérés minden mintánál változhat.

### Ideális kvantáló modell

Ideális esetben a kvantálási hiba nagysága egy LSB környezetében marad.

A tényleges ADC-k ennél sokkal összetettebbek, mert a kvantálási hibán
kívül például:

-   termikus zaj;
-   referenciazaj;
-   órajelhibák;
-   nemlinearitás;
-   torzítás

is jelen lehet.

Ezért a „24 bit = pontosan 144,5 dB valós dinamika" típusú állítás
helytelen.

A 24 bit elsősorban a **digitális kódolás elméleti felbontását** írja
le.

A valós konverter teljesítményét külön mérőszámokkal kell jellemezni.

------------------------------------------------------------------------

# 2.10 Bit depth

A bit depth, vagy bitmélység az egyes minták amplitúdófelbontását
határozza meg.

Minél több bit áll rendelkezésre, annál több kvantálási szintet tudunk
használni.

``` text
8 bit
│
├── 256 szint
│
└── durvább amplitúdófelbontás

16 bit
│
├── 65 536 szint
│
└── finomabb felbontás

24 bit
│
├── 16 777 216 szint
│
└── még finomabb felbontás
```

A bitmélység tehát nem ugyanaz, mint a sample rate.

### Sample rate

Az időbeli felbontást határozza meg.

### Bit depth

Az amplitúdó kvantálási felbontását határozza meg.

Ezt a kettőt mindig külön kezeld.

------------------------------------------------------------------------

# 2.11 Bitmélység és elméleti dinamikatartomány

Ideális PCM-rendszerben gyakran használjuk a következő közelítést:

\[ DR `\approx`{=tex}6.02N + 1.76 `\text{ dB}`{=tex} \]

ahol (N) a bitmélység.

Ez alapján:

### 16 bit

\[ 6.02`\times16`{=tex}+1.76`\approx98.1`{=tex}`\text{ dB}`{=tex} \]

### 24 bit

\[ 6.02`\times24`{=tex}+1.76`\approx146.2`{=tex}`\text{ dB}`{=tex} \]

Ez **elméleti ideális érték**, nem egy valódi audiointerface garantált
analóg dinamikaértéke.

A valós ADC-k és DAC-ok teljesítményét az effektív bitszám (ENOB), SNR,
THD+N, SINAD és egyéb specifikációk jellemzik.

Az Analog Devices külön is hangsúlyozza, hogy az ideális kvantálási
zajból számított érték csak elméleti határ; a valódi konverterekben
további zaj- és torzításforrások vannak.
citeturn0search4turn0search6

------------------------------------------------------------------------

# 2.12 Miért használunk 24 bitet professzionális audióban?

A professzionális munkafolyamatban a 24 bitnek nem az a fő előnye, hogy
„hangosabb" vagy „jobb minőségű" lenne.

A lényeg a rendelkezésre álló headroom és a kvantálási zajhoz képesti
nagy felbontás.

Például egy felvételt nem kell minden egyes mintánál a digitális full
scale közelébe vezérelni.

Ez lehetővé teszi, hogy biztonságosabb headroom mellett dolgozzunk.

Ez különösen fontos:

-   élő felvételnél;
-   több sávos felvételnél;
-   keverésnél;
-   DSP-láncokban;
-   gain stagingnél.

A bitmélységet ezért nem úgy érdemes elképzelni, mint egy
„hangminőség-kapcsolót".

Inkább úgy:

> **Mekkora amplitúdófelbontással reprezentáljuk a mintákat?**

------------------------------------------------------------------------

# 2.13 PCM

A PCM a Pulse Code Modulation rövidítése.

A digitális audió egyik alapvető reprezentációs formája.

Egyszerűen:

``` text
analóg jel
    │
    ▼
mintavételezés
    │
    ▼
kvantálás
    │
    ▼
PCM mintasorozat
```

Egy PCM audiófolyamnak legalább a következő jellemzőit kell ismernünk:

-   sample rate;
-   bit depth;
-   csatornaszám;
-   mintaformátum / kódolási forma;
-   signed/unsigned reprezentáció az adott formátumtól függően;
-   csatornák sorrendje.

A Dante szempontjából különösen fontos, hogy a hálózaton nem „hang" nevű
absztrakt adatot küldünk.

Digitális audióadatot küldünk.

------------------------------------------------------------------------

# 2.14 Mono, stereo és csatornaszám

A hálózati terheléshez nem elég tudni a sample rate-et és bit depth-et.

A csatornaszám is számít.

Egy mono PCM stream:

``` text
CH1
CH1
CH1
CH1
...
```

Stereo:

``` text
L R
L R
L R
L R
...
```

8 csatorna:

``` text
CH1 CH2 CH3 CH4 CH5 CH6 CH7 CH8
...
```

A nyers PCM adatsebesség egyszerű közelítése:

\[ R = f_s `\times`{=tex}N `\times`{=tex}C \]

ahol:

-   (f_s) = sample rate;
-   \(N\) = bitmélység;
-   \(C\) = csatornák száma.

------------------------------------------------------------------------

# 2.15 Példa: 48 kHz / 24 bit / 1 csatorna

\[ 48,000 `\times 24`{=tex} = 1,152,000 \]

Ez:

\[ 1.152`\text{ Mbit/s}`{=tex} \]

nyers PCM adat.

### 8 csatorna

\[ 48,000 `\times 24`{=tex} `\times 8`{=tex} = 9,216,000 \]

Tehát:

\[ 9.216`\text{ Mbit/s}`{=tex} \]

### 32 csatorna

\[ 48,000 `\times 24`{=tex} `\times 32`{=tex} = 36,864,000 \]

Tehát:

\[ 36.864`\text{ Mbit/s}`{=tex} \]

Ez már közvetlenül kapcsolódik a Dante hálózattervezéséhez.

------------------------------------------------------------------------

# 2.16 Miért nem azonos a PCM bitrate és a hálózati forgalom?

Nagyon fontos.

A fenti számítás csak a **nyers audióadat** mennyiségét adja meg.

Egy hálózati rendszerben további adatok is vannak:

``` text
PCM audió
+
protokollfejlécek
+
Ethernet overhead
+
IP overhead
+
UDP overhead
+
flow / hálózati működés
```

Ezért:

> **36,864 Mbit/s PCM-adat nem azt jelenti, hogy a switch portján
> pontosan 36,864 Mbit/s Ethernet-forgalom fog megjelenni.**

A későbbi Dante-fejezetben pontosabban fogjuk tárgyalni a flow-k és
hálózati csomagok viselkedését.

------------------------------------------------------------------------

# 2.17 Sample rate összehasonlítás

    Sample rate   Minta/s   Nyquist-frekvencia
  ------------- --------- --------------------
       44,1 kHz    44 100            22,05 kHz
         48 kHz    48 000               24 kHz
       88,2 kHz    88 200             44,1 kHz
         96 kHz    96 000               48 kHz
      176,4 kHz   176 400             88,2 kHz
        192 kHz   192 000               96 kHz

A táblázatból látszik, hogy a sample rate növelésével a
Nyquist-frekvencia is nő.

Ez azonban nem jelenti automatikusan azt, hogy egy adott rendszerben a
magasabb sample rate hallhatóan jobb eredményt ad.

A választás függhet:

-   az alkalmazástól;
-   a konvertertől;
-   a DSP-kapacitástól;
-   a fájlmérettől;
-   a hálózati terheléstől;
-   a kompatibilitástól.

------------------------------------------------------------------------

# 2.18 44,1 kHz vagy 48 kHz?

Mindkettő professzionális használatban elterjedt.

A történeti háttér eltérő.

A 44,1 kHz erősen kötődik a CD-audióhoz.

A 48 kHz pedig különösen fontos a professzionális videó- és
broadcast-környezetekben.

A könyv szempontjából a fontos tanulság:

> **A sample rate-et nem „jobb/rosszabb" tengelyen kell kiválasztani,
> hanem a teljes rendszer követelményei alapján.**

Dante-rendszerben ráadásul az összes résztvevő kompatibilis sample
rate-je és clockingja a routing működésére is hatással lehet.

------------------------------------------------------------------------

# 2.19 Bit depth és headroom

Tegyük fel, hogy egy felvételnél a legnagyobb pillanatnyi csúcs:

``` text
-3 dBFS
```

Ez nem azt jelenti, hogy a rendszer rosszul van beállítva.

Sőt, digitális felvételnél a full scale közelébe történő kényszeres
vezérlés veszélyes lehet, mert a 0 dBFS fölötti értékeket a klasszikus
PCM reprezentáció nem tudja egyszerűen továbbvinni.

A digitális túlvezérlés:

``` text
0 dBFS
   │
   └──► clipping
```

nem ugyanaz, mint az analóg áramköri túlvezérlés karaktere.

Ezért fontos a gain staging.

------------------------------------------------------------------------

# 2.20 dBFS

A digitális audióban a dBFS a **decibels relative to full scale**
rövidítése.

A 0 dBFS a digitális full-scale referencia.

A hagyományos PCM-rendszerekben:

``` text
0 dBFS
  │
  ├── legnagyobb pozitív tartomány
  │
  └── nincs fölötte további lineáris headroom
```

Ezért a digitális clipping határát nem úgy kell elképzelni, mint egy
analóg előerősítő „kellemes túlvezérlését".

A digitális rendszerben a 0 dBFS feletti érték reprezentációja
korlátozott.

------------------------------------------------------------------------

# 2.21 Clipping

Ha a digitális jel eléri a reprezentálható maximumot és azon túl kellene
mennie, a rendszernek le kell vágnia vagy más módon kell kezelnie a
túlcsordulást.

A legegyszerűbb clipping:

``` text
Eredeti:

       /\
      /  \
_____/    \_____

Clipping után:

       ┌──┐
      /    \
_____/      \____
```

A levágott hullámforma további harmonikus komponenseket hoz létre.

Ezért a digitális túlvezérlés hallható torzítást okozhat.

------------------------------------------------------------------------

# 2.22 Quantization noise

Az ideális kvantálási folyamatban az analóg érték és a kvantált érték
közötti különbség kvantálási hibát eredményez.

Megfelelő körülmények között ez zajszerű komponensként modellezhető.

Az elméleti kvantálási zaj szintje a bitmélységgel javul.

Ezért nő a potenciális dinamikatartomány a bitmélység növelésével.

De itt is fontos:

> **A valós ADC nem ideális kvantáló.**

A valós konverter teljesítményét a teljes analóg és digitális jelút
együttesen határozza meg.

------------------------------------------------------------------------

# 2.23 Mi az a dither?

A dither kis, szándékosan hozzáadott zaj.

Elsőre furcsán hangzik:

> Miért adnánk zajt egy jó minőségű audióhoz?

Azért, mert bitmélység-csökkentéskor a kvantálási hiba nem mindig
viselkedik kellemesen zajszerűen.

A dither segíthet a kvantálási torzítás linearizálásában, és
kiszámíthatóbb zajkomponenst eredményez.

Különösen fontos akkor, amikor például:

``` text
24 bit
  ↓
16 bit
```

átalakítást végzünk.

Az iZotope dokumentációja szerint a dithering a kisebb bitmélységre
történő átalakítás során csökkenti a kvantálási torzítás jellegzetes
artefaktumait, és segít megőrizni a jel használható dinamikáját.
citeturn0search5turn0search11

------------------------------------------------------------------------

# 2.24 Dither és noise shaping

A dither nem feltétlenül teljesen „lapos" zajként kerül alkalmazásra.

Noise shaping segítségével a zaj spektrális eloszlása módosítható.

A cél lehet:

``` text
Hallható sávban
      ↓
kevesebb érzékelhető zaj

Kevésbé érzékeny tartományban
      ↓
több zaj
```

A pszichoakusztikai megközelítések ezt kihasználhatják.

Az iZotope dokumentációja például kifejezetten leírja a noise shaping
használatát a ditherzaj kevésbé hallható tartományok felé történő
elosztására. citeturn0search5

------------------------------------------------------------------------

# 2.25 Mikor kell dither?

A klasszikus mérnöki szabály:

> **Akkor érdekes a dither, amikor ténylegesen csökkentjük a
> bitmélységet.**

Például:

``` text
24 bit master
      ↓
16 bit export
      ↓
Dither
```

Nem minden digitális feldolgozási lépés után kell ditherelni.

Ha egy 24 bites munkafolyamaton belül maradunk, nem az a cél, hogy
minden plug-in után dither kerüljön a jelbe.

A dither alkalmazását a végső word-length reduction folyamatához kell
kötni.

Az Avid dokumentációja is a bitmélység-csökkentéshez kapcsolja a dither
használatát. citeturn0search13turn0search15

------------------------------------------------------------------------

# 2.26 ADC -- Analog-to-Digital Converter

Az ADC az analóg jelet digitális adattá alakítja.

Leegyszerűsített blokkdiagram:

``` mermaid
flowchart LR
    A["Analóg bemenet"] --> F["Analóg előszűrés"]
    F --> S["Sampling"]
    S --> Q["Quantization"]
    Q --> C["Digitális kód"]
```

A valós ADC-k ennél sokkal összetettebbek.

Lehetnek például:

-   SAR;
-   pipeline;
-   sigma-delta;
-   más architektúrák.

A professzionális audióban különösen fontos a konverter teljes analóg és
digitális lánca.

------------------------------------------------------------------------

# 2.27 DAC -- Digital-to-Analog Converter

A DAC az ellenkező irányt végzi.

``` mermaid
flowchart LR
    D["Digitális audió"] --> C["DAC"]
    C --> F["Reconstruction filter"]
    F --> A["Analóg kimenet"]
```

A DAC feladata, hogy a digitális reprezentációból olyan analóg jelet
hozzon létre, amelyet az analóg rendszer fel tud használni.

A rekonstrukciós szűrés azért fontos, mert a mintavételezett jel
spektrális képe nem csak a kívánt alapsávból áll.

A digitális-analóg átalakítás részletes matematikáját ebben a könyvben
nem fogjuk teljes konvertertervezési mélységben tárgyalni, de a
rendszerintegrációhoz szükséges alapokat meg fogjuk tartani.

------------------------------------------------------------------------

# 2.28 ADC → DSP → DAC

Egy professzionális audiójelút gyakran így néz ki:

``` text
Mikrofon
   │
   ▼
Analóg előfok
   │
   ▼
ADC
   │
   ▼
Digitális audió
   │
   ├──► DSP
   │
   ├──► Mixer
   │
   ├──► Recorder
   │
   └──► Dante
           │
           ▼
       Ethernet
```

A Dante tehát nem az ADC helyett van.

A Dante azt teszi lehetővé, hogy a már digitális audióadatot hálózati
infrastruktúrán továbbítsuk.

Ez az egyik legfontosabb kapcsolat az előző és a jelenlegi fejezet
között.

------------------------------------------------------------------------

# 2.29 Sample Rate Conversion

Mi történik, ha két digitális rendszer eltérő sample rate-et használ?

Például:

``` text
Rendszer A
48 kHz

       ↓

Rendszer B
96 kHz
```

A két rendszer között sample rate conversion, azaz **SRC** szükséges
lehet.

Az SRC nem egyszerűen „kihagy minden második mintát".

A valós folyamat digitális szűrést és újramintavételezést alkalmaz.

A megfelelő SRC célja, hogy a jel új sample rate-en is megfelelően
reprezentálható legyen.

Ez különösen fontos olyan összetett rendszerekben, ahol több digitális
eszköz találkozik.

------------------------------------------------------------------------

# 2.30 Miért fontos a sample rate a Dante számára?

A Dante-rendszerben az audió végpontoknak kompatibilis konfigurációban
kell működniük.

Ha egy adó és vevő nem azonos vagy kompatibilis sample rate-en működik,
subscription-probléma jelentkezhet.

Ezért egy Dante-rendszer hibakeresésénél a következő kérdés teljesen
legitim:

> „Milyen sample rate-en működik az adó és a vevő?"

Ez nem pusztán stúdiótechnikai részlet.

A hálózati audió működésének egyik alapfeltétele.

------------------------------------------------------------------------

# 2.31 A digitális audió és a clock kapcsolata

A sample rate valójában időalaphoz kötött.

A 48 kHz azt jelenti, hogy:

\[ 48,000 \]

mintát szeretnénk egy másodperc alatt.

Ehhez a rendszernek stabil időalapra van szüksége.

Ezért kapcsolódik össze a jelenlegi fejezet a későbbi PTP-fejezettel.

A digitális audióban:

``` text
Clock
  │
  ▼
Sample timing
  │
  ▼
PCM samples
```

A Dante-ban pedig:

``` text
PTP
  │
  ▼
Clock synchronization
  │
  ▼
Audio sample timing
```

Ezért a PTP nem „valami hálózati extra".

A digitális audió időalapjához kapcsolódik.

------------------------------------------------------------------------

# 2.32 Bit rate számítás -- gyakorlati labor

Számítsd ki a nyers PCM-adatsebességet.

### Feladat A

48 kHz / 24 bit / mono

\[ 48,000`\times24`{=tex}=1.152`\text{ Mbit/s}`{=tex} \]

### Feladat B

48 kHz / 24 bit / 8 csatorna

\[ 48,000`\times24`{=tex}`\times8`{=tex}=9.216`\text{ Mbit/s}`{=tex} \]

### Feladat C

48 kHz / 24 bit / 32 csatorna

\[ 48,000`\times24`{=tex}`\times32`{=tex}=36.864`\text{ Mbit/s}`{=tex}
\]

### Feladat D

96 kHz / 24 bit / 64 csatorna

\[ 96,000`\times24`{=tex}`\times64`{=tex}=147.456`\text{ Mbit/s}`{=tex}
\]

Ez már jól mutatja, hogy a sample rate és csatornaszám növelése hogyan
növeli a nyers adatforgalmat.

De ne feledd:

> Ezek **nyers PCM-számítások**, nem Dante Ethernet-forgalmi értékek.

------------------------------------------------------------------------

# 2.33 Gyakorlati példa -- miért számít a sample rate?

Képzeljünk el két rendszert.

### Rendszer A

``` text
48 kHz
24 bit
64 csatorna
```

### Rendszer B

``` text
96 kHz
24 bit
64 csatorna
```

A B rendszer nyers PCM-adatsebessége kétszerese A-nak.

Ez nem jelenti automatikusan azt, hogy a Dante-hálózat kétszer akkora
switch-kapacitást igényel.

A tényleges hálózati forgalom függ a Dante-implementációtól és a flow-k
felépítésétől.

De a rendszertervező számára fontos jelzés:

> **A magasabb sample rate növeli a továbbítandó digitális audió
> mennyiségét.**

------------------------------------------------------------------------

# 2.34 Gyakorlati példa -- miért fontos a bit depth?

Hasonlítsunk össze:

``` text
48 kHz / 16 bit
48 kHz / 24 bit
```

A sample rate azonos.

A 24 bites rendszer egy mintához több digitális adatot használ.

Nyers PCM esetén a bitrate arány:

\[ `\frac{24}{16}`{=tex}=1.5 \]

Tehát ugyanannyi csatornán és azonos sample rate mellett a 24 bites
nyers adat 50%-kal nagyobb.

Ez ismét közvetlen kapcsolat a digitális audió és a későbbi
hálózattervezés között.

------------------------------------------------------------------------

# 2.35 Miért nem csak a bit rate számít?

Egy digitális audiórendszer minőségét nem lehet egyetlen számmal
jellemezni.

A sample rate és bit depth mellett számít:

-   ADC/DAC minőség;
-   órajel;
-   analóg előfok;
-   analóg szűrés;
-   zaj;
-   torzítás;
-   jitter;
-   gain staging;
-   DSP-feldolgozás;
-   rendszerarchitektúra.

Ezért hibás lenne azt mondani:

> „A 24 bit automatikusan jobb hangot jelent, mint minden 16 bites
> rendszer."

A specifikációkat mindig a teljes jelút részeként kell értelmezni.

------------------------------------------------------------------------

# 2.36 A digitális audió teljes jelútja

Most rakjuk össze az eddig tanultakat:

``` mermaid
flowchart LR
    S["Hangforrás"] --> M["Mikrofon"]
    M --> A["Analóg jel"]
    A --> AF["Analóg előszűrés"]
    AF --> ADC["ADC"]
    ADC --> PCM["PCM audió"]
    PCM --> DSP["Digitális feldolgozás"]
    DSP --> NET["Dante / hálózat"]
    NET --> RX["Digitális vevő"]
    RX --> DAC["DAC"]
    DAC --> RF["Rekonstrukciós szűrés"]
    RF --> AMP["Erősítő"]
    AMP --> SPK["Hangszóró"]
```

Ez a könyv egyik alapdiagramja.

Később, amikor Ethernetről beszélünk, a diagram középső részét fogjuk
kibontani.

Amikor Dante-ról beszélünk, a hálózati részt.

Amikor PTP-ről beszélünk, az időalapot.

------------------------------------------------------------------------

# 2.37 Ellenőrző kérdések

## Alapfogalmak

1.  Mi a mintavételezés?
2.  Mi a sample rate?
3.  Mi a bit depth?
4.  Mi a kvantálás?
5.  Mi a kvantálási hiba?
6.  Mi a Nyquist-frekvencia?
7.  Mi az aliasing?
8.  Mi az anti-aliasing filter?
9.  Mi az ADC?
10. Mi a DAC?

## Összefüggések

11. Miért nem ugyanazt jelenti a sample rate és a bit depth?
12. Miért növeli a sample rate a nyers adatsebességet?
13. Miért növeli a bit depth a nyers adatsebességet?
14. Miért nem azonos a PCM bitrate és az Ethernet-forgalom?
15. Miért fontos a clock a digitális audióban?
16. Miért nem lehet a 24 bitet egyszerűen „146 dB valódi dinamikának"
    tekinteni?
17. Miért használunk dithert bitmélység-csökkentéskor?
18. Miért szükséges anti-aliasing szűrés?
19. Mi a különbség az ADC és DAC feladata között?
20. Miért lehet szükség sample rate conversionre?

## Számítás

21. Mennyi a nyers PCM bitrate 48 kHz / 24 bit / 16 csatornán?
22. Mennyi 96 kHz / 24 bit / 32 csatornán?
23. Mennyi idő telik el két minta között 48 kHz-en?
24. Mennyi a Nyquist-frekvencia 96 kHz-es sample rate esetén?

------------------------------------------------------------------------

# 2.38 Labor -- Digitális audió számítások

Számítsd ki az alábbiakat:

  Rendszer     Sample rate   Bit depth   Csatorna   Nyers PCM bitrate
  ---------- ------------- ----------- ---------- -------------------
  A                 48 kHz      24 bit          8                   ?
  B                 48 kHz      24 bit         32                   ?
  C                 96 kHz      24 bit         32                   ?
  D                 96 kHz      24 bit         64                   ?
  E                 48 kHz      16 bit         64                   ?

Majd válaszolj:

1.  Melyik rendszer igényli a legnagyobb nyers adatsebességet?
2.  Mi történik az adatsebességgel, ha ugyanannyi csatornát 48 kHz
    helyett 96 kHz-en továbbítunk?
3.  Mi történik, ha 16 bitről 24 bitre váltunk?
4.  Miért nem szabad a kapott értéket közvetlenül switch-port
    forgalomként kezelni?

------------------------------------------------------------------------

# 2.39 Labor -- Alias kísérlet

Használj egy audiószerkesztőt vagy DAW-ot.

1.  Hozz létre egy 48 kHz-es projektet.
2.  Generálj 18 kHz-es szinuszt.
3.  Vizsgáld meg a spektrumát.
4.  Generálj 30 kHz-es tesztjelet olyan környezetben, ahol a
    mintavételezés lehetővé teszi annak vizsgálatát.
5.  Vizsgáld meg, hogyan jelenik meg egy nem megfelelően szűrt, Nyquist
    fölötti komponens alacsonyabb frekvenciaként.
6.  Kapcsold be az anti-aliasing / megfelelő resampling beállításokat.
7.  Hasonlítsd össze az eredményt.

### A labor célja

Nem az, hogy „meghallgasd a Nyquist-tételt".

A cél annak megértése, hogy:

> **Az aliasing a mintavételezésből származó spektrális leképezési
> probléma, amelyet utólag nem lehet egyszerű EQ-val eredeti
> információként visszaállítani.**

------------------------------------------------------------------------

# 2.40 Labor -- Bit depth és dither

Készíts egy nagyon halk, lassan elhalkuló szinuszjelet.

Ezután:

``` text
24 bit
   ↓
16 bit truncation
```

majd:

``` text
24 bit
   ↓
16 bit + dither
```

Vizsgáld:

-   a spektrumot;
-   a fade végét;
-   a nagyon halk részek viselkedését.

A cél annak megértése, hogy a dither nem „jobb hangú zaj hozzáadása",
hanem a kvantálási viselkedés kontrollált kezelése word-length reduction
során.

------------------------------------------------------------------------

# 2.41 Deep Dive -- Miért nem „pixelek" a hangminták?

Gyakori hasonlat:

> „A digitális hang olyan, mint egy sok apró pixelből álló kép."

A hasonlat hasznos lehet kezdőknek, de műszakilag félrevezető, ha túl
messzire visszük.

A minták nem egyszerűen egyes „pontok", amelyeket összekötünk egyenes
vonallal.

A mintavételi tétel feltételei mellett egy sávkorlátozott jel folytonos
időbeli alakja a mintákból elméletileg egyértelműen rekonstruálható.

Ezért a digitális audió nem szükségszerűen „lépcsős hang".

A DAC és a rekonstrukciós szűrés feladata éppen az, hogy a diszkrét
mintákból a megfelelő analóg jel előálljon.

------------------------------------------------------------------------

# 2.42 Deep Dive -- Miért nem „minél több bit, annál hangosabb"?

A bitmélység nem a maximális hangerőt határozza meg.

A PCM-rendszer full-scale értéke a reprezentálható digitális
tartományhoz kötődik.

A több bit:

-   finomabb amplitúdófelbontást;
-   kisebb elméleti kvantálási zajt;
-   nagyobb potenciális dinamikatartományt

jelent.

Nem azt jelenti, hogy:

``` text
16 bit = halk
24 bit = hangos
```

A két rendszer full-scale referenciaértéke eltérő bitmélységnél is
ugyanahhoz a digitális referenciafogalomhoz köthető.

------------------------------------------------------------------------

# 2.43 Deep Dive -- Miért nem egyenlő az ENOB a bit depth értékkel?

Egy ADC lehet 24 bites névleges felbontású, miközben a valós analóg
teljesítménye kevesebb effektív bitnek felel meg.

Az ENOB, vagy Effective Number of Bits, a valós konverter
teljesítményéből származtatott mérőszám.

A különbség:

``` text
Névleges bit depth
        │
        ▼
digitális kódok száma

ENOB
        │
        ▼
valós konverter teljesítménye
```

Ezért professzionális konverter összehasonlításakor nem elég csak azt
látni, hogy:

> „24 bit".

Meg kell nézni a tényleges műszaki specifikációkat is.

------------------------------------------------------------------------

# 2.44 Kapcsolat a Dante-hálózattal

Most már össze tudjuk kötni az első két fejezetet.

### 1. fejezet

``` text
Miért kell hálózati audió?
```

### 2. fejezet

``` text
Milyen digitális adatot küldünk?
```

### Következő lépés

``` text
Hogyan továbbítjuk ezt az adatot?
```

Ez vezet el az Ethernethez.

A következő fejezetben ezért:

-   Ethernet frame;
-   MAC-cím;
-   switch;
-   full duplex;
-   link speed;
-   VLAN;
-   broadcast;
-   multicast;
-   unicast;
-   duplex;
-   MTU

kerül sorra.

------------------------------------------------------------------------

# 2.45 Összefoglalás

A digitális audió alapmodellje:

``` text
Analóg jel
   │
   ▼
Sampling
   │
   ▼
Quantization
   │
   ▼
PCM
   │
   ▼
DSP / Storage / Network
   │
   ▼
DAC
   │
   ▼
Analóg jel
```

A legfontosabb fogalmak:

-   **sample rate** -- másodpercenkénti mintaszám;
-   **bit depth** -- amplitúdófelbontás;
-   **Nyquist frequency** -- a sample rate fele;
-   **aliasing** -- nem megfelelő mintavételezésből származó spektrális
    torzulás;
-   **quantization error** -- az analóg érték és a kvantált digitális
    érték különbsége;
-   **dither** -- kontrollált zaj alkalmazása a kvantálási torzítás
    kezelésére word-length reduction során;
-   **ADC** -- analóg → digitális;
-   **DAC** -- digitális → analóg;
-   **SRC** -- sample rate conversion.

A Dante szempontjából különösen fontos felismerés:

> **A hálózaton digitális audióadatot továbbítunk. A hálózati terhelés
> megértéséhez ezért előbb az audió mintavételezését, bitmélységét és
> csatornaszámát kell értenünk.**

------------------------------------------------------------------------

# 2.46 Következő fejezet

## 3. Ethernet

A következő fejezetben azt vizsgáljuk meg, hogyan működik az a hálózat,
amelyen a digitális audió közlekedik.

Megtanuljuk:

-   mi az Ethernet frame;
-   hogyan működik egy switch;
-   mi a MAC-cím;
-   hogyan tanulja meg a switch a hálózatot;
-   mi a broadcast;
-   mi a multicast;
-   mi az unicast;
-   mi a VLAN;
-   hogyan működik a full duplex;
-   mit jelent a 100 Mbit/s, 1 Gbit/s és 10 Gbit/s;
-   és miért lesz a switch a Dante-rendszer egyik legfontosabb
    komponense.

------------------------------------------------------------------------

# 2.47 Források

1.  Analog Devices -- ADC\
    https://www.analog.com/en/resources/glossary/adc.html

2.  Analog Devices -- Sampling Rate\
    https://www.analog.com/en/resources/glossary/sampling-rate.html

3.  Analog Devices -- Nyquist\
    https://www.analog.com/en/resources/glossary/nyquist.html

4.  Analog Devices -- Chapter 20: Analog to Digital Conversion\
    https://wiki.analog.com/university/courses/electronics/text/chapter-20

5.  Analog Devices -- Types of ADCs and DACs\
    https://www.analog.com/en/resources/technical-articles/types-of-adcs-and-dacs.html

6.  Analog Devices -- Understanding AC Behaviors of High Speed ADCs\
    https://www.analog.com/en/resources/technical-articles/understanding-ac-behaviors-of-high-speed-adcs.html

7.  Analog Devices -- Fundamental Principles Behind the Sigma-Delta ADC
    Topology\
    https://www.analog.com/en/resources/technical-articles/behind-the-sigma-delta-adc-topology.html

8.  iZotope -- Dither Documentation\
    https://downloads.izotope.com/docs/rx6/39-dither/index.html

9.  Avid -- Dither\
    https://apps.avid.com/proToolsFirstHelp/version12.0/enu/Pro_Tools_First_Help/digirack-Dither.46.1.html

------------------------------------------------------------------------

# 2.48 Fejezeti állapot

**Állapot: COMPLETE**

A fejezet tartalmaz:

-   mintavételezési alapokat;
-   Nyquist--Shannon tételt;
-   aliasingot;
-   anti-aliasing szűrést;
-   oversamplinget;
-   kvantálást;
-   kvantálási hibát;
-   bitmélységet;
-   elméleti dinamika-tartományt;
-   PCM-et;
-   dBFS-t;
-   clippinget;
-   dithert;
-   noise shapinget;
-   ADC/DAC működést;
-   sample rate conversiont;
-   digitális audió bitrate-számításokat;
-   Dante-kapcsolatot;
-   három Deep Dive részt;
-   három gyakorlati labort;
-   ellenőrző kérdéseket;
-   forrásokat.

------------------------------------------------------------------------

# 2.49 Műszaki ellenőrzési megjegyzés

A fejezet kritikus állításait elsődleges vagy gyártói műszaki
forrásokkal ellenőriztük. Különösen:

-   a mintavételi frekvencia, Nyquist-határ és aliasing leírását az
    Analog Devices mintavételi és ADC-anyagai alapján;
-   az anti-aliasing szűrés szerepét az Analog Devices szűrőtervezési
    dokumentációja alapján;
-   a dither és noise shaping bitmélység-csökkentésben betöltött
    szerepét Avid dokumentáció alapján;
-   a Dante subscription, sample-rate és clock-domain feltételeit az
    Audinate dokumentációja alapján.

Források:

-   Analog Devices -- Sampling Rate:
    https://www.analog.com/en/resources/glossary/sampling-rate.html
-   Analog Devices -- ADC:
    https://www.analog.com/en/resources/glossary/adc.html
-   Analog Devices -- Aliasing:
    https://www.analog.com/en/resources/glossary/aliasing.html
-   Analog Devices -- Anti-Aliasing Filter Basics:
    https://www.analog.com/en/resources/technical-articles/guide-to-antialiasing-filter-basics.html
-   Analog Devices -- Chapter 20: Analog to Digital Conversion:
    https://wiki.analog.com/university/courses/electronics/text/chapter-20
-   Audinate -- Subscription Tooltips:
    https://dev.audinate.com/GA/dante-controller/userguide/webhelp/content/subscription_tooltips.htm
-   Avid -- Pro Tools Reference Guide:
    https://resources.avid.com/SupportFiles/attach/Pro_Tools/12.5/Pro_Tools_12_5_Reference_Guide.pdf
