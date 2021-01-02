---
author:     Anonymous
profileimage: anonymous.jpg
layout:     post
title:      "Asmens duomenų šmikiai"
date:       2019-06-21 12:00:00
categories: HackAndTell 1A
cover:      1A.jpg
permalink:  /lt/blog/1a-asmens-duomenu-smikiai
---

⚠️*Svečio atsiųstas straipsnis:*  
Kas yra BDAR? BDAR tai  “Bendras duomenų apsaugos reglamentas”. Tai ES reglamentas, įsigaliojęs 2018 metais. Pažeidus BDAR nuostatas, gali būti skiriamos administracinės baudos, kurios kiekvienu konkrečiu atveju turi būti veiksmingos, proporcingos ir atgrasomos. Bauda gali siekti iki 2–4 proc. ankstesnių finansinių metų bendros metinės pasaulinės apyvartos, arba iki 10000000–20000000 EUR.

Vienas iš garsiausiai nuskambėjusių atvejų Lietuvoje, kai įmonė buvo patraukta atsakomybėn už BDAR pažeidimus – tai elektroninių pinigų įstaiga MisterTango, kuriai Valstybinė duomenų apsaugos inspekcija (VDAI) skyrė 61,5 tūkstančio eurų baudą. Bendrovė tapo viena pirmųjų bendrojo duomenų apsaugos reglamento (BDAR) aukų Lietuvoje. MisterTango netinkamai tvarkė duomenis, jų rinko per daug, jie nutekėjo, o apie incidentus nebuvo pranešta: [Straipsnis 15MIN apie Mister Tango](https://www.15min.lt/verslas/naujiena/bendroves/bdar-auka-mistertango-nubausta-61-5-tukstanciu-euru-bauda-663-1145494)

Šiandien susipažinsime su dar viena netvarkinga įmone - UAB "1A". 2018 metais juos įsigijo Kesko Senukai kompanija. Šiuo metu elektroninės parduotuvės 1A kartu su Senukais siūlo plačiausią prekių asortimentą Lietuvoje.

## 1A saugumo spraga

Užsisakius prekes 1A parduotuvėje pirkėjas gauna užsakymo sekimo nuorodą:

[![1A užsakymo nuoroda](1A_nuoroda.jpg "1A užsakymo nuoroda")](1A_nuoroda.jpg)

[https://www.1a.lt/order/progress/order_number/123456/token/XXXXXXXXXXXXXXXX](https://www.1a.lt/order/progress/order_number/123456/token/XXXXXXXXXXXXXXXX)

Kur XXXXXXXXX - tai unikalus 32 simbolių kodas, kuris leidžia peržiūrėti užsakymą tik tam, kas žino pilną nuorodą. Iš nuorodos aišku, kad užsakymo numeris yra lengvai nuspėjamas - tai tiesiog eilės numeris. Tačiau duomenys apsaugoti, nes atspėti 32 atsitiktinių simbolių neįmanoma. Ar įmanoma?

Gal palikti tuščią? Ne, gaunam klaidą. Gal pakišti reikšmę iš kito užsakymo - ne, netinka, matyt reikšmė tikrai unikali kiekvienam užsakymui. Specialius simbolius? Irgi ne…

O ką jeigu įrašysime tiesiog “0”? Bingo!

[https://www.1a.lt/order/progress/order_number/123456/token/0](https://www.1a.lt/order/progress/order_number/123456/token/0)

Ir jums atsiveria stebuklingas pasaulis, marketologo aukso puodas - visi įmonės užsakymai nuo pat elektroninės parduotuvės atidarymo!

Pavyzdžiui:

[https://www.1a.lt/order/progress/order_number/435483/token/0](https://www.1a.lt/order/progress/order_number/435483/token/0)

[![1A užsakymas](1A_uzsakymas.jpg "1A užsakymas")](1A_uzsakymas.jpg)

Kažkas užsisakė svarstykles Esperanza EKS003 už 6,98 eurus su pristatymu už 1,99 eur į Vilniaus OMNIVA paštomatą Naugarduko gatvėje. Ir kas gi čia užsakovas?

[https://www.omniva.lv/privats/sutijuma_atrasanas_vieta?barcode=XXXXXXXXXXXXX](https://www.omniva.lv/privats/sutijuma_atrasanas_vieta?barcode=XXXXXXXXXXXXX)

Mes gerbiame Agnės privatumą ir neviešinsime jos pavardės:

[![OMNIVA gavėjas](1A_omniva.jpg "OMNIVA gavėjas")](1A_omniva.jpg)

Agnė atsiėmė siuntą 09.04.2019 12:50. Labai operatyviai! "Dėkojame, jog pasirinkote www.1a.lt! Gero naudojimo!"

Atsisiųsti visų užsakymų duombazę nėra sudėtinga:

[![Visi 1A užsakymai](1A_visi_uzsakymai.jpg "Visi 1A užsakymai")](1A_visi_uzsakymai.jpg)



## Atskleidimas

Elektroniniu paštu susisiekiau su įmone ir pranešiau, kad bet kam yra pasiekiami visų užsakymo duomenys. Gavau pranešimą, kad laiškas pristatytas ir užregistruotas. Tačiau atsakymo taip ir negavau. Po savaitės vėl nusiunčiau pranešimą elektroniniu paštu, kad nesiėmus veiksmų, informacija bus paviešinta. Vėl jokios reakcijos.

Nuo 2019-06-20 elektroninė parduotuvė 1A buvo pakeista į Senukų platformą, todėl yra saugu ir etiška pranešti apie šį atvejį viešai. Apsipirkdami 1A arba Senukuose reikalaukite, kad jūsų asmens duomenys būtų tinkamai apsaugoti!


## Timeline

```
2019-04-09 - aptikta spraga
2019-04-10 - išsiųstas pranešimas 1A
2019-04-18 - išsiųstas pakartotinis priminimas 1A
2019-06-20 - spraga nebepasiekiama viešai, platforma pakeista į Senukų
2019-06-21 - viešas atskleidimas ir pranešimas Asmens duomenų inspekcijai
```