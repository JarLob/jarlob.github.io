---
layout:       post
title:        "Kaip aš nulaužiau Vilnius.lt"
date:         2019-04-12 12:00:00
categories:   HackAndTell Vilnius.lt
cover:        roger.png
permalink:    /lt/blog/Kaip-as-nulauziau-vilnius-lt
---
Tiesą sakant, visai nesiruošiau [toliau užsiimti](/lt/blog/kodel-hakeriai-nemoka-baudu-2) vilnius.lt. Bet taip išėjo, kad ėjau arbatos ir kolega pakvietė užeiti į jo kambarį. "Žiūrėk! Jie turi dar tokį daugiabuciai.vilnius.lt" - pasakė jis - "Nenori jo nulaužti?.."

Jis parodė man ne pradinį daugiabuciai.vilnius.lt puslapį, o kažkokį pdf failą Uploads aplanke: [https://daugiabuciai.vilnius.lt/dms/uploads/administrator_meta/ADMI.pdf](https://daugiabuciai.vilnius.lt/dms/uploads/administrator_meta/ADMI.pdf).

Išbandę [https://daugiabuciai.vilnius.lt/dms/uploads/administrator_meta/](https://daugiabuciai.vilnius.lt/dms/uploads/administrator_meta/) ir [https://daugiabuciai.vilnius.lt/dms/uploads/](https://daugiabuciai.vilnius.lt/dms/uploads/) gavom 403 Forbidden. Bet [https://daugiabuciai.vilnius.lt/dms/](https://daugiabuciai.vilnius.lt/dms/) pasirodė įdomus:
![DMS](daugiabuciai.png)

Aptikom viešai prieinamus duomenis su namus administruojančių įmonių vadovų ir bendrijų pirmininkų vardais, telefonais ir elektroninio pašto adresais. Gal tai ir vieša informacija, bet verta pastebėti, kad pagrindiniam puslapyje viso to nėra.

Viršutiniame kampe dešinėje matėsi prisijungimo puslapio nuoroda, bet neautorizuotam vartotojui leido tik peržiūrėti duomenis ir naudotis paieška. Pabandę įvesti įvairius paieškos žodžius greitai gavom pranešimą:
![SQL klaida](daugiabuciai_error.png)
Kambarį sudrebino juokas. Radom SQL injekciją, o aš net nespėjau nueiti arbatos. Pasakiau: "Atsiųsk man nuorodą".

Deja, skirtingai nei [praeitą kartą](/lt/blog/kodel-hakeriai-nemoka-baudu-2), tai buvo akloji injekcija - t.y. negalėjau matyti užklausos rezultato, bet galėjau spręsti ar ji pavyko pagal tai po kiek laiko gaudavau klaidą. Tokio tipo injekcijos labai lėtos, nes iš vienos užklausos, kuri trunka kelias sekundes, gali gauti tik vieną bitą informacijos - taip arba ne. Bet yra programų kurios automatizuoja šį procesą, todėl nors lėtai, bet po kelių dienų gavau duomenų bazės lentelių struktūrą ir parsisiunčiau puslapio administratoriaus slaptažodžio Hash reikšmę. Norėčiau pastebėti, kad vilnius.lt serverio administratoriai galėtų vis dėlto kartą per parą pasidomėti ar keistai nepadaugėjo duomenų bazės klaidų.

![Miegantis sargas](sleeping_guard.jpg)

Administratoriaus slaptažodžio apsaugai ir vėl buvo naudojamas MD5 algoritmas. Šį kartą net be druskos. Beje iš HTML kodo buvo akivaizdu, kad [daugiabuciai.vilnius.lt/dms](https://daugiabuciai.vilnius.lt/dms) naudoja [Xataface](http://xataface.com/). Šita meilė MD5 man yra nesuprantama, nes Xataface palaiko ir [geresnes alternatyvas](http://xataface.com/wiki/encryption). Kolega, kurį minėjau prieš tai, yra didelis slaptažodžių laužymo entuziastas, todėl paprašiau jo nulaužti šį Xataface administratoriaus slaptažodį. O pats tuo tarpu nusprendžiau paskaityti programos dokumentaciją.

Xataface leidžia lengvai sukurti interaktyvų puslapį su lentelėmis, kurias galima redaguoti. Tam praktiškai nereikia nieko programuoti, tik nurodyti duomenų bazę ir lenteles konfigūracijos failuose. Skaitydamas [pradinukų tutorialą](http://xataface.com/wiki/How_to_build_a_PHP_MySQL_Application_with_4_lines_of_code) supratau, kad viename iš tų failų turėtų būti duomenų bazės slaptažodis. Nors dokumentacijoje buvo paryškintai pažymėta, kad šį failą reikia apsaugoti, o pateiktos instrukcijos veikia tik Apache serveriui (daugiabuciai.vilnius.lt naudoja Nginx), be abejo jis buvo [atviras visiems](https://daugiabuciai.vilnius.lt/dms/conf.ini). Tiesa sakant tai jau [sistematinė Vilniaus savivaldybės darbuotojų klaida](/lt/blog/kodel-hakeriai-nemoka-baudu-2) pasikartojanti su visais vilnius.lt puslapiais.

Po kelių valandų(!) kolega pranėšė, kad nulaužė slaptažodį. Prisijungęs kaip administratorius gavau galimybę redaguoti įrašus. Iškart norėjau patikrinti kas bus, jei vietoj pdf failo kurį mačiau pradžioj įkelsiu PHP failą. Nuojauta neapgavo - serveris įvykdė mano PHP kodą iš Uploads aplanko. Jeigu SQL injekcijos pažeidžiamumo kaltę dar galima suversti Xataface kūrėjams, tai PHP kodo vykdymas iš Uploads yra konfigūracijos klaida, nes kaip tai teisingai padaryti yra [aprašyta dokumentacijoje](http://xataface.com/documentation/how-to/how-to-handle-file-uploads).

Deja, tai buvo lemtinga klaida, nes įvykdę savo PHP kodą serverio pusėje hakeriai galėjo gauti pilną prieigą, ne tik prie [https://daugiabuciai.vilnius.lt](https://daugiabuciai.vilnius.lt/), bet ir prie visų kitų puslapių esamų tame pačiame serveryje, jų duomenų bazių ir skverbtis toliau į vidinį tinklą.

![Direktorijos](vilnius_ls.png)

Savivaldybės darbuotojai turėjo užtikrinti, kad vieno subdomeno pažeidžiamumas nereikštų viso serverio kompromitacijos. Taip pat reikėtų skirti resursų ne tik funkcionalumo, bet saugumo testavimui. Tai, kaip lengvai ir greitai buvo atrastos skylės, net baugina.

Chronologija:  
2019.04.07 - Išsiųstas laiškas apie SQL injekciją Xataface kūrėjui.  
2019.04.07 - Išleidžiama [nauja Xataface versija](https://github.com/shannah/xataface/releases/tag/2.2.3) su pataisymais.  
2019.04.07 - Išsiųstas laiškas į Vilniaus Savivaldybę.  
2019.04.11 - Gautas patvirtinimas, kad klaidos ištaisytos.