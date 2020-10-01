---
layout:     post
title:      "Tele2 savitarnos autorizacijos apėjimas"
date:       2020-05-21 12:00:00
categories: HackAndTell MobileInternet
cover:      tele2.png
permalink:  /lt/blog/tele2-savitarnos-autorizacijos-apejimas
---
Visi mobiliojo ryšio operatoriai turi savitarnos svetaines. Naujas vartotojas registraciją gali patvirtinti įvedęs kodą atsiųsta SMS žinute ir susikurti slaptažodį kitiems prisijungimams. Taip pat besijungiant yra galimybė naudoti mobilųjį parašą.

Dar kai kurie operatoriai vartotojui suvedus nešifruoto prisijungimo `http://` savitarnos adresą į telefono naršyklę automatiškai nustato jo telefono numerį ir prijungia prie savitarnos be slaptažodžio. Tačiau noras palengvinti prisijungimo procesą gali iškrėsti piktą pokštą. Per didelis pasitikėjimas HTTP Header Enrichment technologija kurią aprašiau [praeitame straipsnyje](/en/blog/mobile-data-leak) Tele2 Lietuva atveju leido pasiekti bet kokio vartotojo savitarną.

Tam tereikėjo susiinstaliuoti bet kurį iš laisvai prieinamų web proxy serverių ([Burp](https://portswigger.net/burp), [Zap](https://www.zaproxy.org/), [Fiddler](https://www.telerik.com/fiddler) ar pan.), nustatyti jį įterpti į kiekvieną siunčiamą užklausą suklastotą `x-tele2-subid` identifikatorių ir sukonfigūruoti naršyklę darbui su šiuo proxy serveriu.  
Įvedus naršyklėje [http://mano.tele2.lt](http://mano.tele2.lt) buvo patenkama į pasirinkto vartotojo savitarną. Aukščiau paminėtas identifikatorius yra tik vidinis IP adresas kaip pvz. `10.2.113.113`. Kaip matom kitų vartotojų numerį lengva surasti perrinkimo būdu. Bet kadangi [Tele2 siuntė šį numerį visiems iš eilės](/en/blog/mobile-data-leak), reikiamą numerį buvo galima sužinoti įviliojus mobiliojo interneto vartotoją į specialų puslapį.

Tam, kad nepažeisčiau kitų vartotojų privatumo kaip *Proof of Concept* prisijungiau per kabelinį internetą nukopijavęs savo paties `x-tele2-subid`. Pasinaudojus šita, kaip [teigia Tele2 (16 min. 43 sec.)](https://www.ziniuradijas.lt/laidos/skaitmeniniai-horizontai/kaip-m-transportas-lt-programeles-leido-atskleisti-naudotoju-banko-korteliu-informacija), "teorine spraga", "kuriai reikėjo eilės kitų veiksmų", o taip pat "turėti eiles žinių apie Tele2 sistemų struktūrą" ir "labai daug informacijos iš vidinių sistemų", buvo galima:
1. Atsisiųsti skambučių išklotinę. Tele2 nereikalauja papildomo patvirtinimo kaip pvz. kodo atsiųsto SMS žinute tokiai operacijai.
2. Siųsti SMS iš suklastoto vartotojo numerio.
3. Atlikti kitus veiksmus svetimoje savitarnoje.

Nagrinėdamas savitarnos svetainę tam, kad įvertinčiau pažeidžiamumo įtaką aptikau automatinio prisijungimo nustatymą [senoje Tele2 svetainėje](https://old-mano.tele2.lt/pagrindinis) (Bendrovė neseniai pristatė naują savitarnos svetainę). Į ją galima patekti per [nuorodą](https://old-mano.tele2.lt/pagrindinis) arba naujoje svetainėje paspaudus `Grįžti į senąją savitarną`. Tada - `Mano paskyra -> Kiti prisijungimo būdai`:  

![Kiti prisijungimo būdai](tele2-1.png)

Įdomu tai, kad paspaudus išjungti yra parodomas sekantis pranėšimas:  
![Tele2 savitarnos perspėjimas](tele2-2.png)

Dauguma vartotojų net nesusimąsto, kai dalinasi internetu, kad tai galėtų plačiai atverti duris į jų savitarną visiems besijungiantiems. Taip pat neprisimenu kad kada nors bučiau keitęs šį nustatymą. Drįsčiau teigti, kad jis įjungtas visiems pagal nutylėjimą ir dauguma vartotoju nėra susipažinę su susijusiais pavojais.

Tai, kartu su galimybe suklastoti vartotojo identifikatorių, leido prisijungti prie bet kurio vartotojo savitarnos netgi jei jis niekada nesidalino internetu. Tam nereikėjo būti Tele2 vartotojų ar apskritai būti Lietuvoje. Bet kokiu atveju šitas nustatymas nebeveikia - po mano pranėšimo Tele2 išjungė automatinį prisijungimą be slaptažodžio prie savitarnos, tačiau ne prie [http://narsyk.tele2.lt](http://narsyk.tele2.lt).

Apskritai paėmus, nors [Tele2 neigia spragos rimtumą (16 min. 43 sec.)](https://www.ziniuradijas.lt/laidos/skaitmeniniai-horizontai/kaip-m-transportas-lt-programeles-leido-atskleisti-naudotoju-banko-korteliu-informacija) *(Užtektų tiesiog pripažinti, kad saugumo klaida buvo greitai ištaisyta, o vidinis patikrinimas nenustatė, kad kas nors tuo pasinaudojo. Juk įmonė privalėjo ne tik ištaisyti spragą, bet ir patikrinti įrašus ieškodama galimų panaudojimo pėdsakų.)* mano patirtis su Tele2 Lietuva išlieka teigiama. Jie parodė, kad vartotojų saugumas jiems yra svarbus ir gana operatyviai ištaisė klaidas.  Deja, bandymai *post factum* sumenkinti saugumo problemą neprisideda prie atsakingo atskleidimo praktikos ir neskatina saugumo tyrinėtojų pirma pranešti įmonei apie atrastą spragą prieš atskleidžiant tai viešai.

P.S. Labas.lt taip pat leidžia prisijungti prie [savaitarnos](http://mano.labas.lt) per mobilųjį ryšį be slaptažodžio jeigu suvedamas nešifruotas adresas `http://`. Skirtingai nuo Tele2 galimybės apsimesti kitu vartotoju nebuvo. Tačiau nebuvo galimybės išjungti šio funkcionalumo ir jis buvo įjungtas visiems vartotojams iš eilės be perspėjimo apie galimas grėsmes. Po pranešimo Bitė įdiegė galimybę tai valdyti savitarnos nustatymuose ir išjungė automatinį atpažinimą esamiems vartotojams. Beje panašu, kad įsivėlė klaida ir įjungti šio funkcionalumo nepavyksta :)  
![Labas savitarnos perspėjimas](labas-savitarna.png)

## Chronologija
2020-02-29 - Išsiųstas pranešimas apie spragą Tele2.lt  
2020-03-03 - Gautas patvirtinimas, kad spraga ištaisyta 2020-03-02  
2020-04-01 - Išsiųstas pranešimas Bite.lt  
2020-05-21 - Labas.lt įdiegti pakeitimai  
