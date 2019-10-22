---
author:     Anonymous
profileimage: anonymous.jpg
layout:     post
title:      "XSS serveriai.lt puslapyje"
date:       2019-10-21 12:00:00
categories: HackAndTell ServeriaiLT
cover:      serveriai-xss.png
permalink:  /lt/blog/xss-serveriai-lt-puslapyje
---

⚠️*Svečio atsiųstas straipsnis:*  
Kas yra XSS? XSS yra sutrumpintas Cross Site Scripting spragos pavadinimas. Tai saugumo spraga, kuri leidžia priversti web puslapį įvykdyti bet kokį JavaScript kodą.

Pavyzdžiui, XSS leidžia gauti prieigą prie vartotojo web sessijos ir vykdyti bet kokius veiksmus vartotojo vardu. Įsivaizduokite, jūs prisijungėte prie filmų nuomos puslapio. Kitas vartotojas parašo jums žinutę "Labas!". O po kelių dienų jus pastebėjote, kad iš jūsų sąskaitos buvo apmokėta filmo nuoma. Tai įvyko dėl to, kad žinutėje buvo paslėptas JavaScript kodas, kuris leido kitam vartotojui pažiūrėti filmą jūsų sąskaita.


## Serveriai.lt WHOIS paslauga

Serveriai.lt sukūrė patogų viešą įrankį, kuris leidžia gauti viešą informaciją apie domeną:

![WHOIS įrankis](xss-serveriai-whois.jpg)

[Plačiau apie WHOIS](https://en.wikipedia.org/wiki/WHOIS)

Kartu su WHOIS informacija, įrankis leidžia matyti patalpinto serverio IP adresą, jo DNS vardą, o taip pat - atsakymo HTTP headerius, jeigu serverio adresu yra paleistas Web serveris.

Tačiau HTTP headeriai buvo atvaizduojami nesaugiai: bet kokia atsakyme pateikta reikšmė buvo tiesiogiai atvaizduojama puslapyje. Todėl sukūrus testinį adresą ir įdėjus į jo atsakymą specialų HTTP headerį su paveiksliuko kodu buvo gautas štai toks rezultatas:

![HTTP headers](xss-serveriai-http-headers.jpg)

Pakeitus paveiksliuko kodą į JavaScript, vartotojo naršyklėje būtų įvykdytas XSS kodas:

![Alert popup](xss-serveriai-alert-popup.jpg)

Tereikėtų sulaukti, kol koks nors smalsus serveriai.lt darbuotojas su `*.serveriai.lt` Cookie patirkintų jūsų domeną.


## Atskleidimas

Elektroniniu paštu susisiekiau su įmone ir pranešiau apie spragą. Jau kitą dieną gavau patvirtinimą, kad klaida ištaisyta.


## Timeline

```
2019-04-10 - aptikta spraga
2019-04-11 - pranešimas apie spragą
2019-04-12 - gautas patvirtinimas, kad klaida ištaisyta
2019-10-21 - viešas atskleidimas
```