# **PaySpy** 

Špecifikácia požiadaviek (SPEC) 

|**Položka**|**Hodnota**|
|---|---|
|**Projekt**|PaySpy – osobná Android aplikácia na evidenciu výdavkov z notifikácií o platbách|
|**Verzia**|1.0 – MVP|
|**Dátum**|25. októbra 2026|
|**Súvisiaci dokument**|PaySpy_popis_projektu.docx (širší popis, plán a riziká)|



Tento dokument popisuje, čo má PaySpy robiť, tak aby mu porozumel aj cudzí človek alebo AI asistent bez ďalšieho vysvetľovania. Rozhoduje o tom, čo sa v prvej verzii postaví a čo nie. 

## **1. Problém** 

**Pri viacerých kartách v eurách a v hrivnách a ručnom zapisovaní výdavkov nie je prehľadné, kam odchádza najviac peňazí ani koľko voľných peňazí reálne zostáva.** 

Existujúce riešenia (Excel, bežné aplikácie) vyžadujú ručný zápis každej platby, a tie, ktoré platby zapisujú automaticky, sú platené. 

## **2. Používatelia a roly** 

|**Rola**|**Kto**|**Čo môže**|
|---|---|---|
|**User**|21-ročná študentka, žije na Slovensku, platí cez|Všetko v aplikácii: pridávať a upravovať|
||Google Wallet, má karty Tatra banky, monobanku a|transakcie a karty, prezerať prehľad, meniť|
||Sense Bank|nastavenia|



- **Rola Admin neexistuje.** Aplikáciu používa jedna osoba na jednom telefóne, bez prihlasovania. 

- **Externé systémy:** Google Wallet (zdroj notifikácií o platbách) a Národná banka Ukrajiny (zdroj kurzov mien). 

## **3. Funkcie** 

Každá funkcia je popísaná jednou vetou z pohľadu používateľky. 

### **1. Zachytávanie platieb** 

- **1.1** Keď zaplatím cez Google Wallet, platba sa sama uloží ako výdavok so sumou, menou, obchodníkom a časom. 

PaySpy – SPEC  |  Strana 1 

|**1.2**|Ak aplikácia text notifikácie nerozpozná, uloží záznam „Nerozpoznané“ so surovým textom, aby som ho mohla<br>opraviť ručne.|
|---|---|
|**1.3**|Tá istá platba sa neuloží dvakrát (rovnaká suma, mena a obchodník do 2 minút).|
|**1.4**|Platby sa zachytávajú aj vtedy, keď je aplikácia zatvorená.|



### **2. Transakcie** 

|**2.1**|Môžem ručne pridať výdavok, príjem alebo prevod medzi mojimi kartami so sumou, menou, kategóriou,<br>kartou, dátumom a poznámkou.|
|---|---|
|**2.2**|Môžem upraviť alebo vymazať ktorúkoľvek transakciu.|
|**2.3**|Každá platba dostane automaticky kategóriu podľa obchodníka a ja ju môžem zmeniť.|
|**2.4**|Vidím zoznam transakcií a môžem ho filtrovať podľa mesiaca, kategórie a karty.|
|**2.5**|Pri každej transakcii je suma uložená v pôvodnej mene aj v základnej mene podľa kurzu NBU z dňa platby.|



### **3. Karty a zostatky** 

|**3.1**|Môžem pridať kartu s názvom, bankou, menou a počiatočným zostatkom.|
|---|---|
|**3.2**|Pri každej karte vidím jej aktuálny zostatok (počiatočný zostatok + príjmy − výdavky ± prevody).|
|**3.3**|Vidím súčet voľných peňazí na všetkých kartách v základnej mene.|



### **4. Prehľad** 

|**4.1**|Za zvolený mesiac vidím príjmy, výdavky a bilanciu, ktorá je zelená v pluse a červená v mínuse.|
|---|---|
|**4.2**|Vidím koláčový graf výdavkov podľa kategórií a pod ním zoznam kategórií so sumou a percentom.|
|**4.3**|Po ťuknutí na kategóriu vidím transakcie, ktoré do nej patria.|
|**4.4**|Vidím, ktorá kategória je najväčšia, a aký je môj priemerný denný výdavok.|



### **5. Nastavenia** 

|**5.1**|Aplikácia mi ukáže návod, ako povoliť prístup k notifikáciám, a či je už povolený.|
|---|---|
|**5.2**|Môžem prepnúť základnú menu medzi EUR a UAH.|
|**5.3**|Môžem upraviť kategórie a pravidlá „obchodník → kategória“.|



## **4. Obmedzenia** 

|**Oblasť**|**Obmedzenie**|
|---|---|
|**Platforma**|Iba Android. Natívna aplikácia v Kotline s Jetpack Compose.|
|**Zdroj platieb**|Iba notifikácie z Google Wallet. Platby mimo Wallet sa zadávajú ručne.|
|**Banky**|Tatra banka, monobank, Sense Bank.|



PaySpy – SPEC  |  Strana 2 

|**Oblasť**|**Obmedzenie**|
|---|---|
|**Meny**|Základná mena EUR (prepínateľná na UAH). Prepočet kurzom NBU z dňa platby, kurz sa uloží<br>s transakciou a spätne sa nemení.|
|**Údaje**|Iba lokálne v telefóne. Žiadny účet, server ani cloud. Do internetu ide iba dopyt na kurz.|
|**Používatelia**|Jedna osoba, jeden telefón, bez prihlasovania.|
|**Povolenia**|Prístup k notifikáciám, internet. Odporúčané: vypnúť optimalizáciu batérie pre PaySpy.|
|**Technológie**|NotificationListenerService, Room, DataStore, WorkManager, Retrofit, Vico. Jeden modul,<br>architektúra MVVM, bez zbytočných knižníc.|
|**Distribúcia**|Súbor APK inštalovaný do telefónu. Nie Google Play.|
|**Vývoj**|Jedna vývojárka so základmi Pythonu a AI asistentom. Preto malé kroky a jednoduchá<br>štruktúra.|
|**Súkromie**|Reálne čísla kariet sa nevkladajú do AI asistenta. Vzorky notifikácií sa pred odovzdaním<br>anonymizujú.|



## **5. Mimo rozsahu** 

Vo verzii 1 úmyselne nie: 

|**Čo**|**Prečo**|
|---|---|
|**Biometrický zámok alebo PIN**|Telefón má vlastný zámok, dáta sú iba lokálne a používam ich iba ja.|
|**Čítanie SMS z banky**|Nahradené notifikáciami. SMS vyžadujú citlivé oprávnenie a šablónu pre<br>každú banku.|
|**Notifikácie z aplikácií bánk**|Každá má iný formát a hrozia duplicity s Google Wallet. MVP má jeden<br>parser.|
|**Samoučenie kategórií**|Stačia pravidlá a ručná zmena kategórie.|
|**Mesačné limity a upozornenia**|Najprv treba overiť, že údaje sú správne.|
|**Porovnanie období a ročné prehľady**|Na začiatku nie je dosť dát.|
|**Záloha, export a import, prenos medzi**<br>**telefónmi**|Mám jeden telefón. Riziko straty dát prijímam vedome, ide o prvú prioritu<br>po MVP.|
|**iOS**|iOS neumožňuje čítať notifikácie z iných aplikácií.|
|**Zverejnenie v Google Play**|Osobné použitie, prísna kontrola Google Play pri prístupe k notifikáciám.|
|**Účty, cloud, viac používateľov**|Súkromie a jednoduchosť.|
|**Priame napojenie na banky (API, Open**<br>**Banking)**|Zložitá autorizácia, pre každú banku iné riešenie.|
|**Skenovanie bločkov, rozpočty, sporiace**<br>**ciele**|Mimo jadra problému.|



PaySpy – SPEC  |  Strana 3 

## **6. Otvorené otázky** 

Pracovný predpoklad platí, kým sa otázka nerozhodne. 

|**Otázka**|**Pracovný predpoklad**|
|---|---|
|V akom jazyku bude rozhranie: ukrajinsky, slovensky, alebo<br>oboma?|Zatiaľ nerozhodnuté, treba rozhodnúť pred začiatkom<br>etapy s obrazovkami.|
|Čo je „mesiac“: kalendárny mesiac, alebo obdobie od výplaty<br>či štipendia (napr. od 15. dňa)?|Kalendárny mesiac.|
|Ako určiť kartu platby, ak notifikácia Google Wallet nenesie<br>názov ani číslice karty?|Podľa meny platby; pri nejasnosti vyberiem kartu ručne.<br>Overí sa na reálnych vzorkách.|
|Čo sa stane, ak kurz NBU pre daný deň chýba (víkend, sviatok,<br>bez internetu)?|Použije sa posledný známy kurz.|
|Prepočíta sa kurz, keď neskôr upravím sumu alebo dátum<br>transakcie?|Nie, kurz zostane pôvodný.|
|Posiela Google Wallet notifikáciu aj pri vrátení peňazí (refund)<br>alebo zamietnutej platbe, a ako ich rozpoznať?|Neznáme, overí sa na vzorkách. Dovtedy sa takéto<br>notifikácie ukladajú ako „Nerozpoznané“.|
|K akému dátumu zadám počiatočné zostatky kariet?|K dňu prvého spustenia aplikácie.|
|Má sa pri prevode medzi kartami v rôznych menách zadávať<br>aj suma prijatá na druhej karte?|Áno, pri prevode medzi rôznymi menami sa zadávajú obe<br>sumy.|



PaySpy – SPEC  |  Strana 4 

