---
layout:     post
title:      "Contribee - move fast and break things"
date:       2022-06-06 12:00:00
categories: HackAndTell
cover:      contribee.webp
permalink:  /lt/blog/contribee
---
2022 m. vasario 24 d. rusija įsiveržė į Ukrainą. Viso pasaulio žmonės aukojo pinigus įvairioms organizacijoms padedančioms ukrainiečiams. Tuo tarpu kontraversiškas Patreon sprendimas [suspenduoti](https://www.cnbc.com/2022/02/24/patreon-suspends-come-back-alive-page-for-ukrainian-army-donations.html) ne pelno siekiančią organizaciją "Come Back Alive" paskatino nemažai skaitmeninio tūrinio kūrėjų, bent jau Lietuvoje, [persikelti](https://twitter.com/yarlob/status/1498235520878624769) į alternatyvią platformą [Contribee](https://contribee.com). Aš, kaip ir kiti prenumeratoriai, užsiregistravau šiame puslapyje. Tačiau greitas platformos testas parodė, kad ji gali turėti saugumo spragų.

### TL; DR;

> Pranešiau apie aptiktą SQL injekciją, Reflected ir Stored XSS, nepakankamą duomenų validaciją, autorizacijos spragas ir informacijos atskleidimą per Contribee API. Pasinaudojus šiais pažeidžiamumais buvo galima prieiti prie visos duomenų bazės informacijos ir potencialiai ją keisti, atlikti veiksmus kitų vartotojų vardu, nusipirkti bet kurio kūrėjo ir lygio prenumeratą už minimalią 0.50 Eur kainą, trinti svetimus komentarus ir gauti tokią privačią informaciją kaip kūrėjo pajamos arba komentatoriaus el. pašto adresas.

### 1. Reflected XSS paieškoje

Suvedęs paieškos laukelyje [stebuklingus žodžius](https://contribee.com/search?q=%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E) pamačiau pranešimo langą:

![Reflected XSS: Alert langas su tekstu contribee.com](contribee_reflected_xss.webp "Contribee reflected XSS")

Kuo tai pavojinga ir kaip pasinaudoti tokiu pažeidžiamumu jau rašiau [praeitame straipsnyje](/lt/blog/vz-slapukai). Šį kartą tik pridėsiu, kad XSS leidžia atlikti veiksmus prisijungusio vartotojo vardu. Tam nėra būtina vogti slapukus. Pasinaudojus automobilio analogija, jei kažkas svetimas gali perimti važiuojančios mašinos vairą - tai jau pakankamai blogai, net jei jis negali pavogti jos raktų.

### 2. SQL injekcija paieškoje

Tame pačiame paieškos funkcionalume kur pastebėjau XSS buvo pasislėpusi ir SQL injekcija. Vienas langelis - dvi spragos. Šis pažeidžiamumas pirma prasprūdo pro akis, nors faktiškai tai tapo rimčiausia atrasta Contribee spraga, galėjusi leisti tiesiogiai prieiti prie visos duomenų bazės informacijos ir potencialiai ją keisti. Kaip įrodymą įvedus į paiešką [`a') AND 7143=(SELECT 7143 FROM PG_SLEEP(5))--`](<https://contribee.com/search?q=a%27)%20AND%207143=(SELECT%207143%20FROM%20PG_SLEEP(5))-->) serveris vykdė užklausą penkiomis sekundėmis ilgiau nei įprastai.

![We found 364 creators with name a') AND 7143=(SELECT 7143 FROM PG_SLEEP(5))--](contribee_sql_result.webp "SQL injekcijos rezultatas")

### 3. Prenumeratos kainos apėjimas

Contribee platformoje kiekvienas kūrėjo nustatytas prenumeratos lygis turi skirtingą kainą bei privalumus.

![Prenumeratų lygių pavyzdys: lygis 1 už 2 Eur - tekstinis turinys, lygis 2 už 5 Eur - tekstinis ir vaizdo turinys.](contribee_lygiai.webp "Prenumeratų lygių pavyzdys")

Paspaudus prenumeruoti leidžiama pasirinkti prenumeratos kainą. Jei pasirinkta kaina yra mažesnė nei nustatyta lygiui, Contribee rodo klaidos pranešimą.

![Prenumeratos kainos pasirinkimas. Klaidos pranešimas: kaina negali būti mažesnė nei pasirinkta kūrėjo.](contribee_subscription_choose.webp "Prenumeratos kainos pasirinkimas")

Tačiau šis apribojimas buvo tik naršyklės pusėje. Pamodifikavus į serverį siunčiamoje užklausoje `priceRec` reikšmę serveris sėkmingai apdorodavo ją. Reikia paminėti, kad 0.50 Eur yra minimalus Stripe, kurį Contribee naudoja pinigų surinkimui, palaikomas dydis. Kitaip, ko gero, galima būtų sumokėti ir vieną centą, arba nulį.

![HTTP užklausą į Contribee. PriceReq parametras pamodifikuotas į 0.5.](contribee_subscription_intercept.png "HTTP užklausą į Contribee")

Šis pažeidžiamumas leido sėkmingai "atrakinti" bet kurią prenumeratą už minimalią sumą.

![Mano prenumeratos. Tautvydas Marčiulaitis 0.50 Eur.](contribee_subscription_my.webp "Nauja prenumerata už 0.50 Eur")

![Sėkmingai atrakinta prenumerata kuri turėtų kainuoti 5 Eur.](contribee_subscription_success.webp "Sekmingai atrakinome prenumeratą")

Po pranešimo Contribee ištaisė šią klaidą. Tačiau atradau kitą būdą pasiekti tą patį.

Prenumeratos užsakymo nuoroda atrodo kaip [https://contribee.com/tmarciu/order/2647](https://contribee.com/adrijus-j/order/9) arba [https://contribee.com/adrijus-j/order/9](https://contribee.com/adrijus-j/order/9). Pakeitus skaičiuką nuoroda vis tiek sėkmingai veikė. Taigi kūrėjo ID nebuvo niekaip susietas su unikaliu prenumeratos numeriu. Suvedus naršyklėje dirbtinę nuorodą kaip [https://contribee.com/tmarciu/order/9](https://contribee.com/tmarciu/order/9) (kombinacija `tmarciu` ir `9`) ir apmokėjus 1 Eur prenumeratą pinigai keliavo `adrijus-j`, bet "atrakindavo" brangesnę `tmarciu` prenumeratą.

![Sėkmingai atrakinta prenumerata. Tautvydas Marciulaitis Unstoppable Trio 1 Eur per mėnesį. Aktyvi.](contribee_tmarciu_adrijus.webp "Sėkmingai atrakinome prenumeratą")

Ar kas nors iš Contribee tūrinio kūrėjų turi prenumeratą už 0.50 Eur? Tai lengva "nuscrapinti". Patikrinus visus prenumeratų numeriukus nuo 0 iki 9999 gauname:

![Prenumeratų su minimalia kaina 0 Eur sąrašas.](contribee_scrape.png "Prenumeratų su minimalia kaina 0 Eur sąrašas.")

Mano nuostabai prenumeratų su minimalia kaina 0.50 neradau, tačiau gavau sąrašą prenumeratų su minimalia kaina 0 Eur. Deja, Stripe nepraleidžia tokio mokėjimo:

![The checkout session's total amount due must add up to at least 0.50 Eur.](contribee_stripe.webp "The checkout session's total amount due must add up to at least 0.50 Eur.")

Bet tai ne bėda, nes prenumeratorius gali mokėti daugiau nei nustatyta minimali kaina.

![Sėkmingai atrakinta prenumerata. Tautvydas Marciulaitis Agnyte 0.50 Eur per mėnesį. Aktyvi.](contribee_tmarciu_agnyte.webp "Sėkmingai atrakinome prenumeratą")

Dėl PVM ir kitų mokesčių, galutinė kaina yra šiek tiek didesnė - 0.84 Eur. Ar galima pigiau? Kiekvienas gali tapti Contribee tūrinio kūrėju ir sukurti savo prenumeratą. Taigi buvo galima mokėti 0.50 Eur sau ir išlaidos būtų tik 0.34 Eur.

### 4. Stored XSS komentaruose

Vartotojo vardas buvo interpretuojamas kaip HTML, kas leido "apkrėsti" komentarus.

![Name: Jaras&lt;script&gt;console.log(13)&lt;/script&gt;](contribee_name_xss.webp "Vartotojo vardas")

![Stored XSS komentaruose. Vartotojo su vardu Jaras&lt;script&gt;console.log(13)&lt;/script&gt; komentaras ir naršyklės developer tools console tabas su išvestu skaičiumi 13 ](contribee_comment_xss.webp "Stored XSS komentaruose")

### 5. Svetimų komentarų trynimas

Contribee leidžia trinti savo paties komentarus. Užvedus pelę ant tokio komentaro atsiranda šiukšliadėžės paveiksliukas. Virš svetimų komentarų jis neatsiranda.

![Komentaras su iššokančiu langeliu kur pavaizduota šiukšliadėžė ir užrašas Delete. Šalia HTML kodas kur matosi komentaro ID.](contribee_delete_comment_ui.webp "Komentaras")

Kadangi kiekvieno komentaro numeris matosi HTML kode, galima tiesiai nusiųsti užklausą į serverį su svetimo komentaro numeriu. Ar vartotojas turi teisę ištrinti komentarą buvo tikrinama tik naršyklės pusėje.

![HTTP užklausą į Contribee: DELETE content/comment/5213/remove](contribee_comment_request.webp "Užklausa ištrinti komentarą")

### 6. Kūrėjo pajamų atskleidimas

Contribee tūrinio kūrėjai gali nustatyti ar jų pajamas per mėnesį rodys viešai arba slėps. Pvz. kur kūrėjas viešina šią informaciją:

![1033 contribees. 4959 Eur per mėnesį.](contribee_public_income.webp "Viešos pajamos per mėnesį")

HTTP užklausos lygmenyje tai atrodo taip:

```
/api/get/basic-information/2699

{"subscribers":1033,"mrr":"4,959 \u20ac","onetime-tips":"15 \u20ac","goals":[],"currency":"EUR"}
```

O taip atrodo kai kūrėjas nusprendė nerodyti šios informacijos:

![71 contribees. Per mėnesį perbrauktos akies piktograma.](contribee_private_income.webp "Neviešos pajamos per mėnesį")

Tačiau HTTP užklausos lygmenyje pajamas vis tiek rodė:

```
api/get/basic-information/2851

{"subscribers":71,"mrr":"■■■ \u20ac","onetime-tips":"0 \u20ac","goals":[],"currency":"EUR"}
```

### 7. Komentatoriaus el. pašto adreso atskleidimas

Užkraunant komentarus Contribee siunčiasi JSON duomenis su daug perteklinės informacijos. Tarp jos buvo ir vartoto el. pašto adresas kuriuo jis registravosi platformoje.

![JSON su pertekline komentatoriau informacija. Tarp jų ir el. pašto adresu.](contribee_comment_email.webp "Komentatoriaus el. pašto adresas")

### Chronologija:

2022.03.08 - Išsiųstas el. laiškas su ataskaita (Prenumeratos kainos apėjimas, Stored ir Reflected XSS, Svetimų komentarų trynimas) į Contribee support ir [NKSC](https://www.nksc.lt/).  
2022.03.09 - NKSC pranešė, kad susisiekė su Contribee telefonu.  
2022.03.13 - Išsiųstas el. laiškas apie SQL injekciją.  
2022.03.14 - Contribee padėkojo ir pranešė, kad taiso klaidas.  
2022.04.01 - Išsiųstas el. laiškas apie privataus el. adreso ir kūrėjo pajamų atskleidimą.  
2022.05.25 - Išsiųstas el. laiškas apie dar vieną prenumeratos kainos apėjimą.  
2022.06.06 - Informacijos paviešinimo data.  
