# **PaySpy** 

Akceptačné kritériá (ACCEPTANCE_CRITERIA) 

|**Položka**|**Hodnota**|
|---|---|
|**Projekt**|PaySpy – osobná Android aplikácia na evidenciu výdavkov z notifikácií o platbách|
|**Verzia**|1.0 – MVP, návrh na schválenie|
|**Dátum**|25. októbra 2026|
|**Súvisiace dokumenty**|SPEC.docx (špecifikácia), PaySpy_popis_projektu.docx (popis a plán)|



## **1. Účel a rozsah** 

Dokument určuje, kedy sa kľúčová funkcia PaySpy dá považovať za hotovú. Každé kritérium je zapísané tak, aby ho vedela overiť tretia osoba bez ďalšieho vysvetľovania: má presné vstupy, postup a číselne alebo logicky jednoznačný výsledok. Vety ako „aplikácia je rýchla“ sa tu nepoužívajú. 

Pokrýva štyri funkcie: **F1** zachytenie platby, **F3** prepočet mien a kurzy, **F6** KPI prehľad a bilancia, **F9** ochrana pred duplicitami. Funkcia je akceptovaná, iba ak sú splnené **všetky** jej kritériá. 

## **2. Pojmy a spoločné pravidlá** 

|**Pojem**|**Význam**|
|---|---|
|**Záznam**|Jeden riadok v tabuľke Transakcia (výdavok, príjem, prevod alebo „Nerozpoznané“).|
|**Čas prijatia**|Čas, kedy Android doručil notifikáciu aplikácii PaySpy (postTime notifikácie).|
|**Čas zápisu**|Čas uloženia záznamu do databázy. Ukladá sa v poli createdAt pri každom zázname.|
|**Základná mena**|EUR, ak kritérium neuvádza inak.|
|**Kurz**|Počet UAH za 1 jednotku cudzej meny podľa Národnej banky Ukrajiny (NBU). Pre EUR ide o<br>hodnotu rate z odpovede NBU API.|
|**Mock kurz**|Pevný kurz zadaný v teste namiesto reálneho API. Vďaka nemu je výsledok vždy rovnaký a<br>dá sa ručne prepočítať.|
|**Zaokrúhľovanie**|Sumy v základnej mene sa zaokrúhľujú na 2 desatinné miesta, pri presne 5 smerom nahor<br>(half-up).|
|**Počítanie súm**|Všetky sumy sa ukladajú a sčítavajú v centoch ako celé čísla, nie ako desatinné čísla.|
|**Testovací telefón**|Fyzický Android telefón s Google Wallet, povoleným prístupom k notifikáciám a vypnutou<br>optimalizáciou batérie pre PaySpy.|



PaySpy – Akceptačné kritériá  |  Strana 1 

## **3. Prehľad pokrytých funkcií** 

|**Funkcia**|**Názov**|**Súvisí so SPEC**|**Počet kritérií**|
|---|---|---|---|
|**F1**|Zachytenie platby z notifikácie|1.1, 1.2, 1.4|7|
|**F3**|Jedna mena a kurzy NBU|2.5, 5.2|9|
|**F6**|KPI prehľad a bilancia|4.1, 4.4|10|
|**F9**|Ochrana pred duplicitami|1.3|10|



## **4. F1 – Zachytenie platby z notifikácie** 

Použitý text notifikácií v príkladoch je ilustračný. Reálne vzorky z Tatra banky, monobanku a Sense Bank sa zozbierajú v etape E1 a uložia sa ako testovacie dáta. 

|**ID**|**Východisková situácia**|**Postup**|**Splnené, ak**|
|---|---|---|---|
|**F1-01**|Aplikácia je zatvorená<br>(zmazaná zo zoznamu<br>posledných aplikácií) a<br>obrazovka je uzamknutá.<br>Karta „Tatra“ existuje.|1. Zaplať cez Google Wallet reálnu<br>platbu.<br>2. Odomkni telefón a otvor PaySpy<br>› Transakcie.|Existuje presne 1 nový záznam.<br>Typ = výdavok, zdroj = notifikácia.<br>Suma, mena a obchodník sú rovnaké<br>ako v notifikácii Google Wallet.<br>Záznam má kategóriu (aj<br>„Nezaradené“).<br>Čas zápisu − čas prijatia ≤ 5 s.|
|**F1-02**|V súbore testovacích dát je<br>pre každú banku (Tatra banka,<br>monobank, Sense Bank) aspoň<br>20 reálnych vzoriek notifikácií,<br>s ručne zapísanou očakávanou<br>sumou, menou a<br>obchodníkom.|Spusti jednotkový test parsera nad<br>všetkými vzorkami.|Test prejde pre 100 % vzoriek (60 z 60<br>pri minimálnom počte).<br>Pre každú vzorku sa suma, mena a<br>obchodník zhodujú s očakávanou<br>hodnotou na znak presne (obchodník)<br>a na cent presne (suma).|
|**F1-03**|Prístup k notifikáciám je<br>povolený. Aplikácia nebola po<br>zapnutí telefónu nikdy<br>otvorená.|1. Reštartuj telefón.<br>2. Neotváraj PaySpy.<br>3. Zaplať cez Google Wallet alebo<br>pošli simulovanú notifikáciu.<br>4. Otvor PaySpy › Transakcie.|Platba sa uložila ako záznam so<br>správnou sumou, menou a<br>obchodníkom.|
|**F1-04**|Notifikácia z inej aplikácie než<br>Google Wallet (napr. správa z<br>Messengera so slovami<br>„zaplatil som 15 EUR“).|Pošli telefónu takú notifikáciu a<br>počkaj 30 s.<br>Otvor PaySpy › Transakcie.|Počet záznamov sa nezmenil.<br>V zozname nie je žiadny nový záznam,<br>ani „Nerozpoznané“.|
|**F1-05**|Notifikácia z Google Wallet,<br>ktorej text parser nevie<br>spracovať. Príklad vstupu:<br>„Platba bola spracovaná“ (bez<br>sumy a meny).|Doruč takú notifikáciu (simulovane<br>v debug verzii).<br>Otvor PaySpy › Transakcie.|Vznikol 1 záznam s typom<br>„Nerozpoznané“.<br>Pole surový text sa rovná textu<br>notifikácie znak po znaku, vrátane<br>diakritiky a medzier.<br>Záznam je v zozname viditeľne<br>označený ako „Nerozpoznané“.<br>Aplikácia nespadla a ďalšia platná<br>notifikácia sa spracuje správne.|



PaySpy – Akceptačné kritériá  |  Strana 2 

|**ID**|**Východisková situácia**|**Postup**|**Splnené, ak**|
|---|---|---|---|
|**F1-06**|Jednotkový test parsera.|Zavolaj parser postupne s týmito<br>vstupmi: (a) prázdny text, (b) text<br>bez čísla, (c) text s číslom bez meny,<br>(d) text so sumou v neznámej mene<br>„XYZ“.|Pre každý z 4 vstupov parser vráti<br>výsledok „nerozpoznané“.<br>Pre žiadny vstup parser nevyhodí<br>výnimku.<br>Pre žiadny vstup parser nevráti sumu.|
|**F1-07**|Existuje záznam<br>„Nerozpoznané“ z F1-05.|1. Otvor záznam.<br>2. Doplň sumu 12,40, menu EUR,<br>obchodník „Tesco“, kartu.|Typ záznamu sa zmenil na výdavok a<br>má zadané hodnoty.<br>Pole surový text zostalo nezmenené.|
|||3. Ulož.|Záznam už nie je označený ako<br>„Nerozpoznané“.|



## **5. F3 – Prepočet mien a kurzy NBU** 

Pri kritériách F3-01 až F3-08 sa používa mock kurz. Ak nie je uvedené inak, základná mena je EUR a kurz EUR/UAH je 50,0000 pre 20. 9. 2026 (deň D) a 51,0000 pre 21. 9. 2026 (deň D+1). Vzorec: suma v EUR = suma v UAH ÷ kurz. 

|**ID**|**Východisková situácia**|**Postup**|**Splnené, ak**|
|---|---|---|---|
|**F3-01**|Základná mena EUR. Mock<br>kurz pre deň D = 50,0000.|Pridaj výdavok 1 000,00 UAH s<br>dátumom D.|Suma v základnej mene = 20,00 EUR.<br>Pôvodná suma = 1 000,00 UAH,<br>pôvodná mena = UAH.<br>Uložený kurz = 50,0000 a uložený<br>dátum kurzu = D.|
|**F3-02**|Základná mena EUR. Kurz pre<br>deň D = 50,0000, druhý prípad<br>s kurzom 48,5000.|Pridaj výdavky s dátumom D: (a)<br>333,33 UAH pri kurze 50,0000, (b)<br>100,00 UAH pri kurze 48,5000, (c)<br>0,01 UAH pri kurze 50,0000.|(a) = 6,67 EUR (6,6666 zaokrúhlené<br>nahor).<br>(b) = 2,06 EUR (2,0619 zaokrúhlené<br>nadol).<br>(c) = 0,00 EUR (0,0002 zaokrúhlené<br>nadol).|
|**F3-03**|Základná mena EUR.|Pridaj výdavok 12,40 EUR.|Suma v základnej mene = 12,40 EUR.<br>Uložený kurz = 1,0000.<br>Nevykonala sa žiadna požiadavka na<br>NBU kvôli tejto transakcii.|
|**F3-04**|Základná mena EUR. Kurz pre<br>deň D = 50,0000, pre deň D+1<br>= 51,0000.|Pridaj dva výdavky po 1 000,00<br>UAH: jeden s dátumom D, druhý s<br>dátumom D+1.|Prvý = 20,00 EUR, kurz 50,0000.<br>Druhý = 19,61 EUR (19,6078<br>zaokrúhlené nahor), kurz 51,0000.<br>Dnešný dátum sa pri výpočte nepoužil.|
|**F3-05**|Stav po F3-04. K dnešnému<br>dňu pribudne mock kurz<br>60,0000.|1. Spusti stiahnutie kurzov.<br>2. Otvor oba záznamy.<br>3. U prvého zmeň sumu na 2 000,00<br>UAH.|Po kroku 2: prvý stále 20,00 EUR s<br>kurzom 50,0000 a druhý 19,61 EUR s<br>kurzom 51,0000.<br>Po kroku 3: prvý = 40,00 EUR, kurz<br>zostal 50,0000.|
|**F3-06**|Základná mena EUR. Kurz<br>existuje pre D+1 (51,0000),<br>pre D+2 (víkend) nie.|Pridaj výdavok 1 000,00 UAH s<br>dátumom D+2.|Suma v základnej mene = 19,61 EUR.<br>Uložený kurz = 51,0000.<br>Uložený dátum kurzu = D+1 (posledný<br>známy kurz).|



PaySpy – Akceptačné kritériá  |  Strana 3 

|**ID**|**Východisková situácia**|**Postup**|**Splnené, ak**|
|---|---|---|---|
|**F3-07**|Prvé spustenie bez internetu,<br>tabuľka Kurz je prázdna.|Pridaj výdavok 1 000,00 UAH.<br>Potom zapni internet a spusti<br>stiahnutie kurzov.|Pred stiahnutím: záznam sa uložil s<br>pôvodnou sumou 1 000,00 UAH a je<br>označený „bez kurzu“, aplikácia<br>nespadla.<br>Po stiahnutí: označenie zmizlo a suma<br>v základnej mene je vypočítaná<br>kurzom NBU platným v deň platby.|
|**F3-08**|Lietadlový režim. Záznamy: 1<br>000,00 UAH a 12,40 EUR s<br>dátumom D (kurz 50,0000).<br>Základná mena EUR.|V Nastaveniach prepni základnú<br>menu na UAH.|Záznam v UAH ukazuje 1 000,00 UAH.<br>Záznam v EUR ukazuje 620,00 UAH<br>(12,40 × 50,0000).<br>Pôvodné sumy a kurzy v databáze sa<br>nezmenili.<br>Prepnutie funguje bez internetu.|
|**F3-09**|Telefón s internetom, reálne<br>NBU API.|1. Pridaj výdavok v UAH s dátumom<br>3 rôznych dní (napr. pracovný deň,<br>sobota, nedeľa).<br>2. Pre každý deň otvor v prehliadači<br>adresu NBU API pre EUR<br>(bank.gov.ua, NBUStatService,<br>exchange s parametrami<br>valcode=EUR a date=RRRRMMDD)<br>a prečítaj hodnotu rate.|Uložený kurz sa zhoduje s hodnotou<br>rate z NBU API pre ten istý deň, na 4<br>desatinné miesta, alebo je pri<br>chýbajúcom kurze dňa uvedený dátum<br>použitého staršieho kurzu.|



## **6. F6 – KPI prehľad a bilancia** 

Vzorec: **Bilancia = Príjmy − Výdavky** , počíta sa za zvolený mesiac v základnej mene (EUR). 

**Testovacia sada T6** (použitá, ak kritérium neuvádza inak): 

|**Dátum**|**Typ**|**Suma**|**Kategória / poznámka**|
|---|---|---|---|
|**1. 9. 2026**|Príjem|300,00 €|Štipendium|
|**15. 9. 2026**|Príjem|150,50 €|Brigáda|
|**3. 9. 2026**|Výdavok|12,40 €|Potraviny|
|**5. 9. 2026**|Výdavok|8,60 €|Potraviny|
|**10. 9. 2026**|Výdavok|45,00 €|Doprava|
|**12. 9. 2026**|Výdavok|120,00 €|Bývanie a účty|
|**8. 9. 2026**|Prevod|200,00 €|z karty Tatra na kartu monobank|
|**28. 8. 2026**|Príjem|1 000,00 €|mimo septembra|
|**2. 10. 2026**|Výdavok|50,00 €|mimo septembra|



PaySpy – Akceptačné kritériá  |  Strana 4 

|**ID**<br>**F6-01**|**Východisková situácia**<br>Testovacia sada T6. Zvolený<br>mesiac: september 2026.|**Postup**<br>Otvor Prehľad.|**Splnené, ak**<br>Príjmy = 450,50 € (300,00 + 150,50).<br>Výdavky = 186,00 € (12,40 + 8,60 +<br>45,00 + 120,00).<br>Bilancia = 264,50 €, zobrazená so<br>znamienkom + a zelenou farbou.<br>Prevod 200,00 € nie je započítaný v<br>príjmoch ani vo výdavkoch.<br>Záznamy z augusta a októbra nie sú<br>započítané.|
|---|---|---|---|
|**F6-02**|T6 + výdavok 500,00 € s<br>dátumom 20. 9. 2026.|Otvor Prehľad pre september 2026.|Výdavky = 686,00 €.<br>Bilancia = −235,50 €, zobrazená so<br>znamienkom − a červenou farbou.|
|**F6-03**|Mesiac s jedným príjmom<br>100,00 € a jedným výdavkom<br>100,00 €.|Otvor Prehľad pre tento mesiac.|Príjmy = 100,00 €, výdavky = 100,00 €,<br>bilancia = 0,00 €.<br>Bilancia nie je zelená ani červená<br>(neutrálna farba).|
|**F6-04**|Mesiac bez akýchkoľvek<br>záznamov (napr. november<br>2026 v T6).|Otvor Prehľad pre tento mesiac.|Príjmy = 0,00 €, výdavky = 0,00 €,<br>bilancia = 0,00 €.<br>Nikde sa nezobrazí „NaN“, prázdne<br>políčko ani chyba.<br>Aplikácia nespadla.|
|**F6-05**|T6 + výdavok 1 000,00 UAH s<br>dátumom 14. 9. 2026 a<br>uloženým kurzom 50,0000 (v<br>základnej mene 20,00 €).<br>Dnešný mock kurz sa potom<br>zmení na 60,0000.|Otvor Prehľad pre september 2026<br>pred zmenou kurzu aj po nej.|Výdavky = 206,00 € (186,00 + 20,00)<br>pred zmenou kurzu aj po nej.<br>Bilancia = 244,50 €.<br>Výpočet použil uloženú sumu v<br>základnej mene, nie dnešný kurz.|
|**F6-06**|Jednotkový test výpočtu KPI.|Zadaj (a) tri výdavky po 0,10 €, (b) 1<br>000 výdavkov po 0,01 €.|(a) Výdavky = 0,30 € presne.<br>(b) Výdavky = 10,00 € presne.<br>Výpočet sa robí v centoch ako celé<br>čísla.|
|**F6-07**|T6, zobrazený Prehľad pre<br>september 2026.|1. Zmeň výdavok 45,00 € na 55,00 €<br>a vráť sa na Prehľad.<br>2. Vymaž tento výdavok a vráť sa na<br>Prehľad.|Po kroku 1: výdavky = 196,00 €,<br>bilancia = 254,50 €.<br>Po kroku 2: výdavky = 141,00 €,<br>bilancia = 309,50 €.<br>Hodnoty sa zmenili bez reštartu<br>aplikácie.|
|**F6-08**|T6.|1. Prepni mesiac na august 2026.<br>2. Prepni späť na september 2026.|August: príjmy = 1 000,00 €, výdavky =<br>0,00 €, bilancia = +1 000,00 €.<br>September: príjmy = 450,50 €, výdavky<br>= 186,00 €, bilancia = 264,50 €.|



PaySpy – Akceptačné kritériá  |  Strana 5 

|**ID**|**Východisková situácia**|**Postup**|**Splnené, ak**|
|---|---|---|---|
|**F6-09**|(a) T6, september 2026 (30<br>dní). (b) Test s nastaveným<br>dnešným dátumom 25. 10.<br>2026 a októbrovými<br>výdavkami 50,00 € (2. 10.) a<br>125,00 € (20. 10.).|Pozri hodnotu „Priemerný denný<br>výdavok“ v (a) pre september a v<br>(b) pre október.|(a) 186,00 ÷ 30 = 6,20 €.<br>(b) 175,00 ÷ 25 = 7,00 € (dni od 1. do<br>dnešného dňa vrátane).|
|**F6-10**|T6, september 2026.|Pozri hodnotu „Najväčšia<br>kategória“.|Zobrazí sa „Bývanie a účty“.<br>Podiel = 65 % (120,00 ÷ 186,00 = 64,5<br>%, zaokrúhlené na celé percento).|



## **7. F9 – Ochrana pred duplicitami** 

Pravidlo: notifikácia je duplicitná, ak v databáze už existuje záznam s rovnakou sumou, rovnakou menou a rovnakým obchodníkom (bez ohľadu na veľkosť písmen a medzery na okrajoch) a rozdiel časov prijatia je najviac **120 sekúnd** . Duplicitná notifikácia sa neuloží. Testy používajú nastaviteľné hodiny alebo debug simuláciu s presnými časmi. Základ: platba „Tesco“, 12,40 EUR, karta Tatra, čas t. 

|**ID**|**Východisková situácia**|**Postup**|**Splnené, ak**|
|---|---|---|---|
|**F9-01**|Prázdna databáza, karta Tatra<br>so zostatkom 100,00 €.|Doruč dve rovnaké notifikácie<br>(Tesco, 12,40 EUR) v časoch t a t +<br>10 s.|V Transakciách je presne 1 záznam.|
|**F9-02**|Prázdna databáza.|(a) Doruč notifikácie v časoch t a t +<br>120 s.<br>(b) Vymaž databázu a doruč<br>notifikácie v časoch t a t + 121 s.|(a) 1 záznam (hranica 120 s je<br>zahrnutá).<br>(b) 2 záznamy.|
|**F9-03**|Prázdna databáza.|Doruč notifikácie v časoch t a t + 10<br>s: Tesco 12,40 EUR a Tesco 12,41<br>EUR.|2 záznamy.|
|**F9-04**|Prázdna databáza.|Doruč notifikácie v časoch t a t + 10<br>s: Tesco 12,40 EUR a Tesco 12,40<br>UAH.|2 záznamy.|
|**F9-05**|Prázdna databáza.|Doruč notifikácie v časoch t a t + 10<br>s: Tesco 12,40 EUR a Billa 12,40<br>EUR.|2 záznamy.|
|**F9-06**|Prázdna databáza.|Doruč notifikácie v časoch t a t + 10<br>s: „TESCO “ (s medzerou na konci) a<br>„Tesco“, obe 12,40 EUR.|1 záznam.|
|**F9-07**|Prázdna databáza.|Doruč tri rovnaké notifikácie v<br>časoch t, t + 100 s a t + 200 s.|2 záznamy: jeden s časom t a jeden s<br>časom t + 200 s.<br>Notifikácia z času t + 100 s sa neuložila.|
|**F9-08**|Prázdna databáza.|1. Doruč notifikáciu v čase t.<br>2. Ukonči proces aplikácie (adb<br>shell am force-stop).<br>3. Doruč rovnakú notifikáciu v čase<br>t + 30 s.|1 záznam.<br>Duplicita sa rozpoznala aj po ukončení<br>procesu (kontrola sa nespolieha na<br>pamäť aplikácie).|



PaySpy – Akceptačné kritériá  |  Strana 6 

|**ID**|**Východisková situácia**|**Postup**|**Splnené, ak**|
|---|---|---|---|
|**F9-09**|Po F9-01 existuje záznam|Do 2 minút ručne pridaj výdavok|Ručný záznam sa uložil (celkovo 2|
||Tesco 12,40 EUR z času t.|Tesco 12,40 EUR.|záznamy).<br>Aplikácia ho nezamietla ako duplicitu.|
|**F9-10**|Stav po F9-01, začiatočný<br>zostatok karty Tatra 100,00 €.|Pozri zostatok karty Tatra.|Zostatok = 87,60 € (100,00 − 12,40),<br>nie 75,20 €.|
||||Duplicitná notifikácia nezmenila ani<br>príjmy, ani výdavky v Prehľade.|



## **8. Podmienky testovateľnosti a predpoklady** 

### **Čo musí aplikácia umožňovať, aby sa kritériá dali overiť** 

- **Pole createdAt** (čas zápisu) pri každom zázname. Bez neho sa meranie „do 5 s“ (F1-01) nedá urobiť. Treba ho pridať do dátového modelu. 

- **Pole surový text** a **dátum použitého kurzu** pri každom zázname. 

- **Debug verzia** s povoleným balíkom com.android.shell, aby sa dali simulovať notifikácie príkazom adb. Táto výnimka nesmie byť v ostrej verzii. 

- **Nastaviteľné hodiny a nastaviteľný zdroj kurzov** v testoch (jednotkové testy s mock kurzom a pevným časom). 

- **Súbor testovacích dát** so vzorkami notifikácií od všetkých troch bánk (anonymizované, bez čísla karty). 

### **Hodnoty a rozhodnutia navrhnuté v tomto dokumente** 

Tieto hodnoty som zvolil ja, treba ich potvrdiť alebo zmeniť. 

|**Predpoklad**|**Navrhnutá hodnota**|
|---|---|
|**Čas od notifikácie po záznam (F1)**|Najviac 5 sekúnd|
|**Hranica duplicity (F9)**|120 sekúnd vrátane|
|**Ručne pridaný záznam sa neblokuje ako duplicita (F9-09)**|Áno|
|**Zaokrúhľovanie súm v základnej mene**|Na 2 desatinné miesta, half-up|
|**Úprava sumy transakcie nemení uložený kurz (F3-05)**|Áno|
|**Chýbajúci kurz v daný deň (F3-06)**|Použije sa posledný známy kurz, uloží sa jeho dátum|
|**Prvé spustenie bez kurzu (F3-07)**|Záznam sa uloží s označením „bez kurzu“ a dopočíta<br>sa po stiahnutí|
|**Neutrálna farba pri nulovej bilancii (F6-03)**|Áno|
|**Presný tvar adresy NBU API**|Overiť pri implementácii v dokumentácii NBU|



PaySpy – Akceptačné kritériá  |  Strana 7 

