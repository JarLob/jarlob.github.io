---
layout:     post
title:      "Kodėl hakeriai nemoka baudų?"
date:       2019-02-20 12:00:00
categories: HackAndTell TvarkauMiesta
cover:      TM.png
permalink:  /lt/blog/kodel-hakeriai-nemoka-baudu
---
*Straipsnis pirma publikuotas 2019-02-20 [Linkedin](https://www.linkedin.com/pulse/kod%C4%97l-hakeriai-nemoka-baud%C5%B3-jaroslav-loba%C4%8Devski/).*  
Manau [https://tvarkaumiesta.lt](https://tvarkaumiesta.lt) yra puikus puslapis suteikiantis galimybę teikti pasiūlymus miesto tvarkymo klausimais arba įskųsti kokį pažeidėją ;) Teko ir pačiam pasinaudoti juo. Mano dėmesį patraukė keturios ikonos po paties sukurtu pranešimu kurių iš pradžių net nepastebėjau:

![Pranešimas](tm_icons.png)

Antra ikona buvo neaktyvi. Tai pastūmėjo manę patikrinti ar tai kas uždrausta iš naršyklės taip pat neveikia jei kreipčiausi į serverį tiesiogiai. Susikūriau naują anoniminį pranešimą testavimui ir supratau, kad tokio tipo pranešimai išviso neturi redagavimo mygtukų. Teko susikurti antrą pranešimą jau savo vardu, paspausti trinti ir pagauti naršyklės siunčiamą užklausą serveriui:

![Request](tm_request.png)

Serveriui buvo siunčiamas problem_id numeris, kuris matomas kiekvienam pranešimui adresų juostoje. Antras parametras problem_oid - ilga skaičių ir raidžių seka. Įdomu ar galėčiau gauti jį anoniminiams pranešimams? Jį radau kiekvieno pranešimo HTML kode. Pabandžiau išsiųsti užklausą su prieš tai paties sukurto anoniminio pranešimo problem_id ir problem_oid. Pavyko - pranešimas dingo! Pabandžiau neprisijungęs (be cookies) ištrinti savo ne anoniminį pranešimą - ir vėl sėkmė!

Supratau, kad net neautorizuotas vartotojas (anonimas) gali nusiųsti užklausą į [https://tvarkaumiesta.lt/user_problems_list](https://tvarkaumiesta.lt/user_problems_list) su viešai prieinamais problem_id ir problem_oid ir ištrinti svetimą pranešimą. Bandžiau tai daryti tik su pranešimais kurių statusas yra "Registruota", bet gali būti, kad tai veikė ir pranešimams kurie jau buvo pradėti nagrinėti. Panašu, kad panešimas nėra iš tikrųjų ištrinamas iš duomenų bazės - jis tik dingsta iš puslapio, bet savivaldybės darbuotojui turėtų niekuo nesiskirti nuo to lyg pats pranešėjas būtų jį ištrynęs.

Šitą saugumo spragą galima būtų panaudoti:

1. Tiesiog vandalizmui.
2. Trinti pranešimams apie ne vietoje pastatytus automobilius, kol ši informacija nepasiekė savivaldybės darbuotojų. (Automobilio valstybinis numeris viešai matosi pranešime).

Savivaldybės programuotojai sureagavo labai profesionaliai ir pašalino saugumo spragą per vieną valandą(!) nuo mano laiško išsiuntimo.