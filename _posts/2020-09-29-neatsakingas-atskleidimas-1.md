---
layout:     post
title:      "(Ne)atsakingas atskleidimas - 1-oji dalis"
date:       2020-09-29 12:00:00
categories: HackAndTell ResponsibleDisclosure
cover:      
permalink:  /lt/blog/neatsakingas-atskleidimas-1
---
<div style="text-align: right">"Žmonės, kurie niekada gyvenime neatskleidinėjo saugumo spragų, turi „įdomiausių“ nuomonių apie tai kaip tai teisinga daryti." <i>Thomas H. Ptacek, Matasano</i></div>
<br>
<div style="text-align: right">"Aš sutinku kalbėtis apie „atsakingą atskleidimą“ kai mes pradėsim kalbėtis apie neatsakingą (nesantį) saugumo kokybės užtikrinimą iš gamintojo pusės." <i>Thomas Dullien, Google Project Zero</i></div>

> Sąžiningas atskleidimas: Šis straipsnis stipriai įtakotas [Rashomon of disclosure](http://addxorrol.blogspot.com/2019/08/rashomon-of-disclosure.html).

„Atsakingas atskleidimas“ vėl karšta tema dėka to, kad Krašto Apsaugos Ministerija (KAM) paruošė įstatymo pakeitimus siekdama reglamentuoti kibernetinių saugumo spragų atskleidimo tvarką. Tai ko gero sveikintina iniciatyva, nes, nors kiekvieną kartą privačiai atskleisdamas saugumo spragą programinės įrangos valdytojui, aš tikiu jo sveiku protu ir supratimu, kad jam daroma paslauga, tačiau kiti atskleidimo pavyzdžiai Lietuvoje nėra tokie sėkmingi ir saugumo tyrinėtojai sulaukia nuo grasinančių klausimų „Ar tikrai tyrimą atlikote ir neviešus elektroninius duomenis rinkote nepažeidžiant galiojančių įstatymų?“ iki realiai iškeltų baudžiamųjų bylų. Beje aukščiau paminėti „nevieši duomenys“ yra ne kas kitas, bet bandymas „pritempti“ nesusijusį teisės aktą prie kibernetinio saugumo spragų paviešinimo siekiant išvengti visuomenės teisės žinoti apie išleistus potencialiai nesaugius produktus. Todėl aiškesnis reglamentavimas galėtų išeiti visiems į naudą.

Verta paminėti, kad tarptautinėje informacinio saugumo bendruomenėje vis dar periodiškai kyla karštos diskusijos dėl to kaip turėtų būti atskiedžiamos saugumo spragos, kas yra atsakinga, o kas ne. Tai nėra jokiu būdu pilnai apibrėžta sąvoka ir kiekviena suinteresuota pusė (valstybės saugumo tarnybos, kibernetinio saugumo tyrinėtojai, programinės įrangos gamintojai, vartotojai, IT ūkio administratoriai, programinės įrangos skirtos aptikti kibernetines atakas gamintojai) savaip įsivaizduoja tobulą atskleidimo procesą.

Pradėti reikėtų ko gero nuo to kokie yra galimi atskleidimo scenarijai. Aptikęs saugumo spragą tyrinėtojas turi iš esmės keturis pasirinkimus:
#### 1. Nepranešti niekam.
Tam gali būti įvairių priežasčių. Pvz. iOS gamintojas Apple bandydamas užtikrinti savo gaminių saugumą tuo pačiu apsunkina saugumo tyrinėtojų darbą, nes norint stebėti vidinius programų procesus pirmą reikia „nulaužti“ telefoną ir gauti reikiamas teises paleisti specializuotus įrankius. Tikėtina, kad pasilikęs šią spragą sau asmuo įgautų pranašumą tiriant ir atrandant kitas spragas.

#### 2. Ribotas atskleidimas.
Informacija pranešama tik tam tikram ratui suinteresuotų šalių. Tai galėtų būti Nacionalinis Kibernetinio Saugumo Centras (NKSC) dar žinomas kaip CERT, programinės įrangos valdytojas, KAM ir, kaip dabar madinga sakyti, Kritinės Infrastruktūros dalyviai.

Šito pasirinkimo bėda ta, kad kuo daugiau žmonių žino paslaptį, tuo sunkiau ją išlaikyti. Informacinio saugumo eros pradžioje egzistavo tokia susirašinėjimo elektroninių paštų grupė [Zardoz]( https://en.wikipedia.org/wiki/Zardoz) kurioje „baltakepuriai“ hakeriai diskutavo ir dalinosi aptiktomis spragomis tam, kad jos galėtų būti ištaisytos be „visuomenės žinios“. Rezultate kiekvienas tų metų save gerbiantis hakeris ar žvalgybos skyrius siekė prisijungti prie Zardoz grupės. Dažniausiai sėkmingai.

Istorija rodo vėl ir vėl, kad mažose grupėse kurios dalinasi nevieša informacija, visada būna bent vienas sukompromituotas grupės narys ir piktavaliai skaito visą komunikaciją. Galima daryti prielaidą, kad tas pats galiotų ir centralizuotiems elektroninio pašto adresams skirtiems siųsti pranešimus apie saugumo spragas. Kadangi juose esanti informacija tokia vertinga, bet kokios gerai finansuojamos, pvz. valstybės, hakerių komandos užduotis būtų gauti prieigą prie jos. Zardoz atvejis nėra unikalus. Ankstyvais 2000-aisiais internete cirkuliavo įvairūs vidiniai nacionalinių Saugumo Incidentų Reagavimo Skyrių (CERT) susirašinėjimai. Tikėtina, kad žvalgybos tarnybų galimybės ir šiais laikas siekia panašų lygį.

#### 3. Pilnas atskleidimas.
Informacija paskelbiama viešai ir tampa prieinama tuo pačiu metu ir vartotojams ir gamintojams ir, neišvengiamai, piktavaliams.

Neskubėkit smerkti, kad toks pasirinkimas stumia vartotojus į nesaugią padėtį. Deja, bet skirtingos visuomenės grupės skirtingai rizikuoja. Vieni sakys, kad ši spraga jiems nerupi, nes jie „neturi ką slėpti“. Kitiems tai sukels tam tikrų nepatogumų ar finansinių nuostolių. O dar kitiems, kaip pvz. disidentams Baltarusijoje, tai galbūt mirties ir gyvybės klausimas. Tokiems žmonėms labai svarbu vengti arba atsisakyti nesaugių technologijų. Tai yra visų pirmą etinė problema, kurią kažin ar galima apibrėžti teisiškai, nes galima užduoti provokuojantį klausimą: „Ar verta sukelti tam tikrų nepatogumų 100m žmonių jei tai gali išgelbėti gyvybę kitiems penkiems?“.

#### 4. Atsakingas atskleidimas.
Pats pavadinimas tarsi užkrauna visą atsakomybės naštą ant atskleidusio asmens pečių, todėl pastaruoju metu vis dažniau naudojamas kitas terminas – „koordinuotas atskleidimas“. Tuo pabrėžiama ir programinės įrangos gamintojo atsakomybė dėl išleisto plačiam naudojimui nesaugaus produkto.

Informacija apie spragą yra privačiai perduodama programinės įrangos gamintojui ir paviešinama tik tada, kai išleidžiamas pataisymas arba baigiasi apibrėžtas laikas. Šiuo metu standartas *de facto* yra 90 kalendorinių dienų. Bet pasitaiko ir kitų terminų. Iš esmės tai yra tyrinėtojo teisė nustatyti kiek laiko jis sutinka tylėti apie **esamą saugumo spragą vartotojų aktyviai naudojamame produkte**. Pasitaiko atvejų kai laukiama ir metus. Koordinuoto atskleidimo atveju įrangos gamintojas turi laiko paruošti ir ištestuoti reikiamus pakeitimus ir vartotojai gali atnaujinti savo programinę įrangą.

Vėl gi, neskubėkite džiaugtis.

Dažnai naiviai galvojama, kad iki atskleidžiant spragą plačiajai publikai, šis kibernetinis pažeidžiamumas yra žinomas tik gamintojui ir apie jį pranešusiam asmeniui. Neretai tai nėra tiesa. Yra žinoma daug atvejų kai ta pati spraga yra atrandama skirtingų žmonių. Užtenka paskaityti kad ir pvz. Microsoft pranešimus apie ištaisytas klaidas. Prie kiekvienos spragos dažnai nurodoma kam įmonė dėkoja už pranešimą apie ją. Kartais ten nurodomi net penki nesusiję tyrėjai ar jų komandos. Tie kurie ieško – randa. Didelė tikimybė, kad ta pati spraga jau yra žinoma kitoms šalims. Kuo ilgiau vartotojai laikomi nežinomybėje, tuo ilgiau rizikuojama jų saugumu.

90 dienų laiko tarpas yra išbandytas laiko. Dažniausiai net didelės korporacijos turinčios sudėtingus vidinius procesus sugeba išleisti pataisymus per nurodytą terminą. Bet ne visada. Tada informacija yra atskleidžiama viešai net jei nėra pataisymo ir visuomenė vėl sukyla diskutuodama ar tai atsakinga. Tiesa yra tai, kad šis laiko terminas privalo būti nepatogus gamintojui. Tobulam pasaulyje, tik gavęs pranešimą apie pažeidžiamumą, gamintojas iškart puola jį taisyti ir išleidžia pataisymą kuo greičiau. Deja šiuolaikiniame pasaulyje įmonės užsidirba išleisdamos produktus su naujomis galimybėmis. Vadovai ir programuotojai kyla karjeros laiptais laiku pristatydami naujoves. Kai saugumo spragos tampa žinomos jie jau būna pakilę keturiais laipteliais arba perėję į kitą departamentą ar įmonę į aukštesnę poziciją. Niekas dar nėra praradęs darbo ar pažemintas pareigose dėl to, kad išleido nesaugų produktą. Atskleidimo terminas yra nustatomas tam, kad gamintojas nustatytų teisingą prioritetą saugumo pataisymams.

P.S. Gavosi šiek tiek ilgai, todėl apie Lietuvos realijas parašysiu antroje dalyje.
