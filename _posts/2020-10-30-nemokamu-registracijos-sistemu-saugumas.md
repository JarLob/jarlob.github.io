---
layout:     post
title:      "Nemokamų registracijos sistemų saugumas"
date:       2020-10-30 17:00:00
categories: HackAndTell Covid19
cover:      restreg.png
permalink:  /lt/blog/nemokamu-registracijos-sistemu-saugumas
---
Šiandien išėjo 15min straipsnis ["Žmonės suskubo padėti COVID-19 smūgį patyrusioms kavinėms ir barams: kuria sistemas nemokamai"](https://www.15min.lt/verslas/naujiena/bendroves/zmones-suskubo-padeti-covid-19-smugi-patyrusioms-kavinems-ir-barams-kuria-sistemas-nemokamai-663-1399998). Iniciatyva išties šauni, tik kiek keista, kad abi išvardintos ir kitos man žinomos "nemokamos" sistemos nėra atviro kodo. [Restreg.lt](https://restreg.lt) kūrėjo Facebook'e išplatintas "manifestas" skelbia, kad *"Restreg.lt sistemoje nėra reklamų , 3-iųjų šalių programų, socialinių tinklų sekimo programų"*.

[![Restreg manifestas](restreg_fb.PNG "Restreg manifestas")](restreg_fb.PNG)

Ir iš tiesų, kita man žinoma sistema [Covisit.lt](https://covisit.lt) siunčia įtartinai daug telemetrijos į Google serverius. Kavinių ir barų lankymosi informacija, konkrečių vartotojų lankymosi pomėgiai galėtų būti vertinga prekė.

[![Covisit.lt ads trafic](covisit_ads.png "Covisit.lt ads trafic")](covisit_ads.png)

Tačiau man užkliuvo ne šita vieta Facebook paskelbtam pranešime, o - *"...visiškai nemokamą ir saugią sistemą..."* ir pranešimas žemiau. You know how to trigger me...

[![Covisit.lt white hackers](restreg_whitehackers.png "Covisit.lt white hackers")](restreg_whitehackers.png)

Po keliolikos minučių puslapio tyrinėjimo mane sudomino įmonės logotipo įkėlimo funkcionalumas. Pabandęs įkelti HTML failą gavau pranešima, kad leidžiami tik PNG, JPEG ir pan. formatai. Tačiau tai buvo patikrinimas naršyklės pusėje, o tai greičiau pagerina User Experience (UX), bet nėra jokia apsauga. Tiesiogiai pamodifikavus HTTP užklausą serveris gražino klaidą. Atėjo laikas išbandyti failą poliglotą.

Poliglotais vadinami tuo pačiu metu kelių formatų failai. Pvz. tas pats specialiai paruoštas failas gali būti tvarkingai atidarytas ir su Zip programa ir su PDF skaityklė. Apačioje esantis JPEG paveiksliukas yra tuo pačiu ir validus (kind of) HTML failas:

[![XSS polyglot](xss_polyglot.jpg "XSS polyglot")](xss_polyglot.jpg)

Jei atsidarysit [šį paveiksliuką](xss_polyglot.jpg) nieko neatsitiks, nes serveris nurodys naršyklei, kad duomenų tipas yra JPEG - prisegs header'į `Content-Type: image/jpeg`. Tačiau, jei serveris to nenurodytų, naršyklė bandytų atspėti kas tai per duomenų tipas ir galėtų nuspręsti, kad tai HTML.

Pabandžiau įkelti šį failą kaip logotipą. Tačiau serveris tvarkingai rodė jį kaip paveiksliuką. Tada nurodžiau siųsti šį **jpg** failą, tačiau pamodifikavau užklausoje  plėtinį į **html**:
```
Content-Disposition: form-data; name="logo"; filename="xss_polyglot.html"
Content-Type: image/jpeg
```

Serveris priėmė, pervadino ir patalpino jį [https://restreg.lt/out/media/952342ba69383fba6bcdfb221850894e4c411f5dc5b339d2864f9937241dd99c.html](https://restreg.lt/out/media/952342ba69383fba6bcdfb221850894e4c411f5dc5b339d2864f9937241dd99c.html).

[![XSS alert restreg.lt](restreg.png "XSS alert restreg.lt")](restreg.png)

Taigi turim galimybė įkelti savo HTML kodą į serverį, tačiau vartotojas taip paprastai ten nenuklystų. Kaip galima būtų tuo pasinaudoti? Truputis socialinės inžinerijos - tokią nuorodą galima išsiųsti phishing laiške registruotos įmonės darbuotojui. Nuoroda yra į `https://restreg.lt/...` ir tikimybė, kad jis ar ji ją atsidarys ženkliai padidėja. Kitu atveju to galėtų užtekti pasiekti lankytojų duomenis, tačiau Restreg.lt reikalauja suvesti kodą suteiktą pirmos registracijos metu. Tai yra gerai, bet kadangi piktavaliai gali pilnai kontroliuoti įkeltą HTML failą, galima būtų sukurti gražų prašymą įvesti šį slaptą kodą, nes jis turi būti sugeneruotas iš naujo. Taip, tai Socialinė Inžinerija, bet Restreg.lt prašo kodo, looks legit?

Kaip tai galima būtų ištaisyti?
- Logotipo failo plėtinys turi būti tikrinamas serveryje.
- Visų failų iš `out/media` aplanko `Content-Type:` galėtų būti "nurodomas" kaip paveiksliuko.
- Vartotojų sąrašo eksporto rezultatas galėtų būti siunčiamas tik į registruotą pašto adresą (kurio negalima keisti be administratoriaus patvirtinimo).

Chronologija:  
2020.10.28 - Išsiųstas laiškas apie HTML įkėlimą kūrėjui.  
2020.10.28 - Gautas patvirtinimas, kad klaidą bus ištaisyta.  
2020.10.28-30(?) - Įmonių registracijai reikalingas administratoriaus patvirtinimas.
