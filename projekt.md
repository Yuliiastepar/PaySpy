# **PaySpy** 

Android aplikácia na automatické sledovanie výdavkov z notifikácií o platbách 

|**Položka**|**Hodnota**|
|---|---|
|**Projekt**|PaySpy – osobná mobilná aplikácia na evidenciu výdavkov|
|**Platforma**|Iba Android|
|**Autorka a vývojárka**|Jedna osoba (vlastný projekt, vývoj s pomocou AI asistenta)|
|**Verzia dokumentu**|1.0 – návrh MVP|
|**Dátum**|13. októbra 2026|



## **1. Zhrnutie** 

PaySpy je osobná Android aplikácia, ktorá automaticky zachytáva notifikácie o platbách kartou (v prvej verzii z Google Wallet). Z textu notifikácie rozpozná sumu, menu, obchodníka a čas platby, uloží ju do telefónu ako výdavok a zaradí do kategórie. Všetky sumy sa prepočítajú do jednej meny, takže aj pri kartách v eurách a v hrivnách vidím jeden prehľad: kam idú peniaze a či som v pluse, alebo v mínuse. 

Všetky údaje zostávajú iba v mojom telefóne. Aplikácia nemá účet, server ani cloud. 

**Zmena oproti pôvodnému návrhu:** namiesto čítania SMS z banky aplikácia číta notifikácie z Google Wallet. Dôvod: nie každá banka posiela SMS o platbách, ale Google Wallet ukáže notifikáciu pri každej platbe telefónom bez ohľadu na banku (Tatra banka, monobank, Sense Bank). 

## **2. Aký problém riešim** 

**Ako študentka s kartami v eurách aj v hrivnách musím každý výdavok zapisovať ručne, a preto neviem, kam odchádza najviac peňazí ani koľko voľných peňazí mi reálne zostáva.** 

### **Čo za tým stojí** 

- **Ručné zadávanie dát.** V Exceli aj v bežných aplikáciách treba zapísať každú platbu. Po pár dňoch sa zápisy prestanú robiť a prehľad je neúplný. 

- **Chýba prehľad podľa kategórií.** Bez automatického triedenia nevidím, na čo idú najväčšie sumy (jedlo, doprava, zábava…). 

- **Viac kariet, viac mien.** Mám Tatra banku (zvyčajne EUR), monobank a Sense Bank (zvyčajne UAH). Nemám jeden súhrn príjmov a výdavkov, preto neviem vypočítať reálny voľný zostatok. 

- **Cena.** Aplikácie, ktoré to robia automaticky, sú platené. 

PaySpy – popis projektu  |  Strana 1 

## **3. Kto sú moji používatelia** 

Aplikáciu bude používať jediná osoba: ja. Nie je určená na zverejnenie ani na predaj. 

|**Parameter**|**Popis**|
|---|---|
|**Používateľka**|Jediná – autorka aplikácie|
|**Profil**|21-ročná Ukrajinka, študentka, žije na Slovensku|
|**Banky a karty**|Tatra banka (SK), monobank (UA), Sense Bank (UA)|
|**Meny**|EUR a UAH (prípadne iná mena pri cestovaní)|
|**Zariadenie**|Android telefón s Google Wallet|
|**Spôsob platby**|Prevažne telefónom cez Google Wallet; občas hotovosť alebo karta mimo Wallet|
|**Technická úroveň**|Bežná používateľka; ovládanie musí byť jednoduché a bez nastavovania|



### **Typické situácie** 

**1. Platba v obchode.** Zaplatím telefónom, Google Wallet ukáže notifikáciu, PaySpy ju zachytí a výdavok sa objaví v zozname aj s kategóriou. Nič nezadávam. 

**2. Platba hotovosťou alebo kartou mimo Wallet.** Pridám výdavok ručne, zadanie má trvať do 15 sekúnd. 

**3. Príjem peňazí.** Štipendium, brigádu alebo prevod od rodiny zadám ručne ako príjem. 

**4. Koniec týždňa alebo mesiaca.** Otvorím prehľad a vidím príjmy, výdavky, bilanciu, výdavky podľa kategórií a voľné peniaze na všetkých kartách. 

### **Čo to znamená pre návrh** 

- Jeden používateľ znamená žiadne prihlasovanie, žiadny server a žiadnu synchronizáciu. 

- Len Android znamená, že môžem použiť systémový prístup k notifikáciám. 

- Viac mien znamená, že treba vyriešiť prepočet kurzov od začiatku. 

## **4. Ako chcem problém vyriešiť (MVP)** 

### **4.1 Princíp fungovania** 

**1.** Platba cez Google Wallet vyvolá notifikáciu na telefóne. 

**2.** PaySpy má povolený prístup k notifikáciám a číta iba notifikácie z Google Wallet. Ostatné ignoruje. 

**3.** Parser z textu vyberie sumu, menu a obchodníka. Dátum a čas berie z hodín telefónu. 

**4.** Podľa obchodníka aplikácia určí kategóriu (zoznam pravidiel „názov obchodníka → kategória“). Neznámy obchodník dostane kategóriu „Nezaradené“. 

**5.** Suma sa prepočíta do základnej meny kurzom z dňa platby. Použitý kurz sa uloží spolu so záznamom, takže staré údaje sa spätne nemenia. 

**6.** Aplikácia skontroluje duplicity a zapíše výdavok do lokálnej databázy. Hneď je vidieť v prehľade. 

**Obmedzenie, o ktorom treba vedieť:** zachytia sa iba platby, pri ktorých Google Wallet ukáže notifikáciu. Platby 

PaySpy – popis projektu  |  Strana 2 

fyzickou kartou bez Wallet, online platby kartou a výbery z bankomatu treba doplniť ručne. 

### **4.2 Funkcie MVP** 

|**ID**|**Funkcia**|**Čo presne robí a kedy je hotová**|
|---|---|---|
|**F1**|Zachytenie platby z<br>notifikácie|Po platbe cez Google Wallet sa výdavok uloží do pár sekúnd bez môjho zásahu, aj<br>keď je aplikácia zatvorená. Ak parser text nerozpozná, uloží sa záznam<br>„Nerozpoznané“ so surovým textom na ručnú opravu.|
|**F2**|Automatické kategórie|Približne 10 základných kategórií (zoznam nižšie). Kategória sa určí podľa pravidiel<br>obchodník → kategória. Kategóriu môžem kedykoľvek zmeniť.|
|**F3**|Jedna mena|Každá transakcia je uložená v pôvodnej mene aj v základnej mene. Základná<br>mena je predvolene EUR, v nastaveniach sa dá prepnúť na UAH. Kurz Národnej<br>banky Ukrajiny sa uloží k transakcii.|
|**F4**|Ručné pridanie a úprava|Formulár: typ (výdavok, príjem, prevod medzi mojimi kartami), suma, mena,<br>kategória, karta, dátum, poznámka. Záznam možno upraviť alebo vymazať.|
|**F5**|Príjmy|Príjmy zadávam ručne, pretože Google Wallet o prijatých peniazoch nenotifikuje.<br>Započítajú sa do bilancie a zostatkov kariet.|
|**F6**|KPI prehľad|Za zvolený mesiac ukáže príjmy, výdavky, bilanciu (plus alebo mínus) a ďalšie<br>ukazovatele podľa tabuľky v 4.3.|
|**F7**|Grafy výdavkov|Koláčový graf výdavkov podľa kategórií za mesiac. Pod ním zoznam kategórií so<br>sumou a percentom. Ťuknutie na kategóriu ukáže jej transakcie.|
|**F8**|Karty a zostatky|Zoznam mojich kariet (Tatra, monobank, Sense) s menou a počiatočným<br>zostatkom. Aktuálny zostatok = počiatočný zostatok + príjmy − výdavky ±<br>prevody.|
|**F9**|Ochrana pred duplicitami|Rovnaká suma, mena a obchodník v rozmedzí 2 minút sa uloží iba raz.|
|**F10**|Dáta iba v telefóne|Lokálna databáza. Žiadny účet, žiadny server. Do internetu ide iba dopyt na kurz.|



**Základné kategórie:** Potraviny · Reštaurácie a kaviarne · Doprava · Bývanie a účty · Štúdium · Zdravie a drogéria · Zábava · Nákupy · Predplatné a služby · Ostatné. Systémová kategória: Nezaradené. 

### **4.3 KPI, ktoré aplikácia ukazuje** 

|**KPI**|**Ako sa počíta**|**Zobrazenie**|
|---|---|---|
|**Príjmy za mesiac**|Súčet príjmov v základnej mene|Číslo|
|**Výdavky za mesiac**|Súčet výdavkov v základnej mene (prevody medzi vlastnými<br>kartami sa nerátajú)|Číslo|
|**Bilancia mesiaca**|Príjmy − výdavky|Číslo; zelená pri plus,<br>červená pri mínus|
|**Voľné peniaze celkom**|Súčet aktuálnych zostatkov všetkých kariet v základnej mene|Číslo|
|**Najväčšia kategória**|Kategória s najvyššími výdavkami a jej podiel na všetkých<br>výdavkoch|Názov a percento|
|**Priemerný denný výdavok**|Výdavky mesiaca ÷ počet uplynulých dní mesiaca|Číslo|



PaySpy – popis projektu  |  Strana 3 

Presnosť údaja „Voľné peniaze celkom“ závisí od dvoch vecí: správne zadaného počiatočného zostatku na každej karte a doplnenia platieb, ktoré Google Wallet nezachytil. 

### **4.4 Obrazovky** 

**1. Prehľad (domov):** KPI karty, koláčový graf kategórií, posledné transakcie, prepínač mesiaca. 

**2. Transakcie:** zoznam s filtrom podľa mesiaca, kategórie a karty. Ťuknutím sa otvorí úprava. 

**3. Pridať alebo upraviť transakciu:** formulár z funkcie F4. 

**4. Karty:** zoznam kariet so zostatkami a možnosťou pridať kartu. 

**5. Nastavenia:** návod na povolenie prístupu k notifikáciám, základná mena, správa kategórií a pravidiel obchodník → kategória. 

### **4.5 Dátový model** 

|**Entita**|**Polia**|
|---|---|
|**Transakcia**|id · typ (výdavok / príjem / prevod) · suma v pôvodnej mene · pôvodná mena · suma v<br>základnej mene · použitý kurz · obchodník · kategória · karta · druhá karta (pri prevode) ·<br>dátum a čas · zdroj (notifikácia / ručne) · poznámka · odtlačok na detekciu duplicít · surový<br>text notifikácie|
|**Kategória**|id · názov · farba|
|**Pravidlo kategórie**|id · kľúčové slovo v názve obchodníka · kategória|
|**Karta**|id · názov · banka · mena · počiatočný zostatok|
|**Kurz**|dátum · kód meny · kurz voči hrivne (UAH)|



### **4.6 Podporované banky** 

|**Banka**|**Typická mena**|**Ako sa platba zachytí**|**Stav**|
|---|---|---|---|
|**Tatra banka**|EUR|Notifikácia Google Wallet|Treba overiť formát textu|
|**monobank**|UAH|Notifikácia Google Wallet|Treba overiť formát textu|
|**Sense Bank**|UAH|Notifikácia Google Wallet|Treba overiť formát textu|



Pred písaním parsera zozbieram aspoň 20 reálnych notifikácií od všetkých troch bánk (etapa E1 v pláne). Z nich sa ukáže, aký presne text Google Wallet zobrazuje a či obsahuje názov alebo posledné číslice karty. 

### **4.7 Kritériá úspechu MVP** 

Návrh cieľov, ktoré chcem po 30 dňoch používania splniť: 

- Aspoň 90 % platieb cez Google Wallet sa uloží správne (suma, mena, obchodník) bez ručnej opravy. 

- Žiadna platba sa neuloží dvakrát. 

- Po doplnení platieb mimo Wallet sa mesačné výdavky v aplikácii zhodujú s bankovými výpismi s odchýlkou do 2 %. 

- Ručné pridanie transakcie trvá do 15 sekúnd. 

- Aplikáciu používam 30 dní po sebe a nevraciam sa k Excelu. 

PaySpy – popis projektu  |  Strana 4 

## **5. Čo v MVP nebude (non-goals) a prečo** 

Cieľom MVP je overiť jednu vec: či automatické zachytenie platieb z Google Wallet a prehľad v jednej mene stačia na vyriešenie problému. Všetko ostatné je vedome odložené. 

|**Čo v MVP nebude**|**Prečo**|**Kedy možno**|
|---|---|---|
|**Biometrický zámok alebo PIN**|Telefón má vlastný zámok a dáta sú iba lokálne.<br>Používam iba ja.|Po MVP, nízka priorita|
|**Čítanie SMS z banky**|Nahradené notifikáciami. SMS vyžadujú citlivé<br>oprávnenie a vlastnú šablónu pre každú banku.|Iba ak notifikácie<br>nestačia|
|**Notifikácie z aplikácií bánk**<br>**(monobank, Sense, Tatra)**|Každá aplikácia má iný formát textu a hrozia duplicity<br>s Google Wallet. MVP má jeden parser.|Prvý kandidát po MVP, ak<br>mi bude chýbať veľa<br>platieb|
|**Samoučenie kategórií**|Pravidlá a ručná zmena kategórie stačia. Učenie<br>zvyšuje zložitosť.|Verzia 1.1: zapamätať si<br>moju opravu ako nové<br>pravidlo|
|**Mesačné limity a upozornenia**|Najprv treba overiť, že údaje sú správne. Limity bez<br>správnych údajov nemajú zmysel.|Po 30 dňoch ostrej<br>prevádzky|
|**Porovnanie období a ročné**<br>**prehľady**|Na začiatku nie je dosť dát na porovnanie.|Po niekoľkých mesiacoch<br>používania|
|**Záloha, export a import (CSV,**<br>**JSON), prenos medzi telefónmi**|Mám jeden telefón. Riziko straty dát pri strate<br>telefónu prijímam vedome.|Najvyššia priorita po<br>MVP|
|**Spustenie po reštarte telefónu**|Android obnoví službu čítania notifikácií sám, keď je<br>prístup povolený.|Nie je potrebné|
|**iOS verzia**|iOS neumožňuje aplikácii čítať notifikácie z iných<br>aplikácií. Používam Android.|Nie|
|**Zverejnenie v Google Play**|Osobné použitie. Google Play prísne kontroluje<br>aplikácie s prístupom k notifikáciám a platí sa poplatok<br>za účet vývojára.|Nie, inštalácia súborom<br>APK|
|**Účty, cloud, viac používateľov**|Súkromie a jednoduchosť. Finančné údaje nechcem<br>posielať mimo telefón.|Nie|
|**Priame napojenie na banky (API,**<br>**Open Banking)**|Komplikovaná autorizácia a bezpečnosť, pre každú<br>banku iné riešenie.|Nie|
|**Skenovanie bločkov, rozpočty,**<br>**sporiace ciele**|Mimo jadra problému.|Nie|



PaySpy – popis projektu  |  Strana 5 

## **6. Aké technológie potrebujem** 

### **6.1 Odporúčaný technologický stack** 

|**Vrstva**|**Technológia**|**Prečo**|
|---|---|---|
|**Platforma a jazyk**|Natívny Android, Kotlin|Prístup k notifikáciám je systémová funkcia Androidu.<br>Kotlin je oficiálny jazyk Androidu, má najviac príkladov<br>a AI asistenti v ňom píšu najspoľahlivejšie.|
|**Používateľské**<br>**rozhranie**|Jetpack Compose (Material 3)|Moderný spôsob tvorby obrazoviek, menej kódu ako<br>staršie XML rozloženia.|
|**Zachytenie platieb**|NotificationListenerService|Systémová služba, ktorú Android spúšťa po povolení<br>„Prístupu k notifikáciám“. Beží aj pri zatvorenej<br>aplikácii.|
|**Rozpoznanie textu**|Regulárne výrazy v Kotline +<br>jednotkové testy (JUnit)|Testy na reálnych vzorkách notifikácií ochránia parser<br>pred rozbitím pri každej úprave.|
|**Databáza**|Room (SQLite)|Lokálna databáza priamo v telefóne, štandardné<br>riešenie pre Android.|
|**Nastavenia**|DataStore|Uloženie základnej meny a ďalších jednoduchých<br>nastavení.|
|**Kurzy mien**|API Národnej banky Ukrajiny<br>(bank.gov.ua), Retrofit +<br>kotlinx.serialization|Oficiálny zdroj kurzov hrivny, bezplatný, podporuje<br>kurz za konkrétny dátum.|
|**Pravidelné sťahovanie**<br>**kurzov**|WorkManager|Raz denne stiahne kurz. Pri chýbajúcom internete sa<br>použije posledný uložený kurz.|
|**Grafy**|Vico (grafy pre Compose); záloha:<br>MPAndroidChart|Hotové grafy bez písania vlastného kreslenia.|
|**Architektúra**|MVVM (ViewModel + Kotlin Flow),<br>jeden modul|Jednoduchá a štandardná štruktúra, ktorú AI asistenti<br>dobre poznajú.|
|**Vývojové nástroje**|Android Studio, Git + GitHub, fyzický<br>Android telefón|Emulátor nevie reálne notifikácie z Google Wallet,<br>preto testujem na vlastnom telefóne.|
|**AI asistencia**|AI asistent pre kód (napr. Claude<br>Code alebo Gemini v Android Studiu)|Vývoj formou „vibe codingu“ v malých krokoch.|
|**Distribúcia**|Súbor APK inštalovaný priamo do<br>telefónu|Bez Google Play, bez poplatkov a bez schvaľovania.|



### **6.2 Potrebné funkcie a povolenia telefónu** 

|**Funkcia telefónu**|**Na čo slúži**|**Poznámka**|
|---|---|---|
|**Prístup k notifikáciám**|Zachytenie notifikácie z Google Wallet hneď<br>po platbe|Špeciálne povolenie, ktoré zapnem<br>ručne v nastaveniach telefónu|
|**Internet (Wi-Fi alebo mobilné**|Sťahovanie kurzov z NBU|Posielajú sa iba kurzy, žiadne osobné|
|**dáta)**||údaje|
|**Interná pamäť telefónu**|Lokálna databáza výdavkov|—|



PaySpy – popis projektu  |  Strana 6 

|**Funkcia telefónu**|**Na čo slúži**|**Poznámka**|
|---|---|---|
|**Systémové hodiny a dátum**|Presný čas transakcie a určenie kurzu<br>platného v daný deň|—|
|**Práca na pozadí**|Spracovanie notifikácií, aj keď je aplikácia<br>zatvorená|Treba vypnúť optimalizáciu batérie pre<br>PaySpy, inak môže systém službu<br>ukončiť|
|**Notifikácie aplikácie (voliteľné)**|Potvrdenie „Uložený výdavok“|Nie je nutné pre MVP|



### **6.3 Prečo nie iné technológie** 

- **Flutter alebo React Native:** pridávajú ďalšiu vrstvu a čítanie notifikácií by som stále písala natívne v Kotline. Pre jedinú platformu nemajú výhodu. 

- **Python (Kivy, BeeWare):** slabá podpora systémových služieb Androidu, medzi nimi čítania notifikácií na pozadí. 

- **Webová aplikácia (PWA):** nemôže čítať notifikácie z iných aplikácií. 

**Kde využijem Python a Jupyter Notebook:** na prototyp parsera. Zozbierané texty notifikácií v notebooku vyskúšam s regulárnymi výrazmi, upravím ich na jednej vzorke po druhej a overené pravidlá potom prenesiem do Kotlinu (aj s pomocou AI asistenta). 

### **6.4 Ako budem vyvíjať s AI asistentom** 

- Pracujem v malých krokoch: jedna funkcia = jedna požiadavka na AI asistenta. 

- Po každom funkčnom kroku urobím commit v Gite, aby som sa mohla vrátiť späť. 

- Parser vždy testujem na reálnych vzorkách. Čísla kariet do AI nevkladám, vzorky pred odovzdaním anonymizujem. 

- Každú novú funkciu hneď vyskúšam na vlastnom telefóne. 

## **7. Plán realizácie** 

|**Etapa**|**Výstup**|**Hotovo, keď**|
|---|---|---|
|**E0 Príprava**|Nainštalované Android Studio, prázdny projekt,<br>repozitár v Gite|Prázdna aplikácia beží na mojom telefóne|
|**E1 Vzorky**|Malá aplikácia, ktorá iba zapisuje texty zachytených<br>notifikácií|Mám aspoň 20 vzoriek z Tatra banky,<br>monobanku a Sense Bank|
|**E2 Parser**|Kód na rozpoznanie sumy, meny a obchodníka<br>(najprv v Jupyter, potom v Kotline)|Testy prejdú na všetkých vzorkách|
|**E3 Dáta**|Databáza, karty, ručné pridanie, úprava a mazanie|Môžem zadať a upraviť transakciu na<br>telefóne|
|**E4 Kategórie a kurzy**|Pravidlá kategórií, stiahnutie a uloženie kurzu NBU,<br>prepočet|Každá transakcia má kategóriu a sumu v<br>základnej mene|
|**E5 Prehľad**|KPI, koláčový graf, prepínač mesiaca|Čísla v prehľade sedia s ručným<br>prepočtom|
|**E6 Ostrá prevádzka**|30 dní reálneho používania a opravy chýb|Splnené kritériá úspechu z časti 4.7|



PaySpy – popis projektu  |  Strana 7 

## **8. Riziká a otvorené body** 

|**Riziko alebo otvorený bod**|**Dopad**|**Ako ho riešim**|
|---|---|---|
|**Formát textu Google Wallet sa líši podľa**<br>**banky a nemusí obsahovať kartu**|Parser nerozpozná kartu<br>alebo obchodníka|Vzorky v etape E1. Ak karta chýba, určím ju<br>podľa meny, pri nejasnosti ju vyberiem ručne.|
|**Google zmení formát notifikácie**|Parser prestane fungovať|Parser je oddelený v jednom module a pokrytý<br>testami. Nerozpoznané notifikácie sa uložia so<br>surovým textom.|
|**Platby mimo Google Wallet sa**<br>**nezachytia**|Neúplné výdavky a zostatky|Ručné doplnenie. Po MVP zvážim notifikácie z<br>aplikácií bánk.|
|**Android ukončí službu na pozadí kvôli**<br>**úspore batérie**|Chýbajúce platby|Vypnúť optimalizáciu batérie pre PaySpy. V<br>nastaveniach ukázať stav prístupu k<br>notifikáciám.|
|**Kurz NBU sa líši od kurzu, ktorý použila**<br>**banka**|Suma v základnej mene je<br>orientačná|V aplikácii vždy ukladám aj pôvodnú sumu a<br>menu.|
|**Prevody medzi vlastnými kartami by**<br>**skreslili príjmy a výdavky**|Nesprávne KPI|Samostatný typ transakcie „Prevod“, ktorý sa<br>do príjmov a výdavkov nerátá.|
|**Strata alebo výmena telefónu**|Strata všetkých dát|Vedome prijaté v MVP. Záloha a export sú<br>prvou prioritou po MVP.|
|**Súkromie finančných údajov**|Únik citlivých údajov|Spracovanie iba v telefóne, žiadny server, do<br>internetu ide iba dopyt na kurz.|



### **Rozhodnutia, ktoré treba ešte urobiť** 

- **Jazyk aplikácie:** ukrajinčina alebo slovenčina (alebo obe). 

- **Základná mena:** návrh je EUR, s možnosťou prepnúť na UAH. Overiť, či to vyhovuje. 

- **Počiatočné zostatky:** k akému dátumu ich zadám na každej karte. 

PaySpy – popis projektu  |  Strana 8 

