---
author:     Anonymous
profileimage: anonymous.jpg
layout:     post
title:      "Atsakingas atskleidimas. Lietuvos pavyzdys."
date:       2019-04-29 12:00:00
categories: HackAndTell ServeriaiLT
cover:      SERVERIAILT.jpg
permalink:  /lt/blog/atsakingas-atskleidimas-lietuvos-pavyzdys
---

⚠️*Šiandien publikuoju svečio atsiųstą straipsnį:*  
Kiekvienas viešai prieinamas servisas yra pažeidžiamas. Nei viena įmonė nėra apsaugota nuo programuotojų klaidų, kad ir kiek ji investuoja į saugumą. Todėl kiekviena įmonė turi būti pasiruošusi saugumo incidentui, turėti planą.


Vienas iš efektyviausiu būdų — taip vadinamas *atsakingas atskleidimas*. Tai programa, kuri skatina programišius pranešti apie saugumo spragą, o ne bandyti pasinaudoti ja neteisėtiems veiksmams. Pagrindinė taisyklė — pranešti valdytojui ir duoti laiko ištaisyti klaidą prieš skelbiant apie ją viešai. Už tai suradusiam spragą dažnai mokamas nustatytas atlygis, o pats pranešimas vadinamas atsakingu atskleidimu (responsible disclosure).


Vienas iš atsakingų atskleidimų pasaulio pavyzdžių būtų bevielio tinklo šifravimo pažeiždiamumas KRACK - [https://www.krackattacks.com/](https://www.krackattacks.com/). Aptikus spragą, saugumo tyrinėtojai bendradarbiavo su stambiausiais programinės įrangos tiekėjais, kad kuo skubiau išleistų pataisymą. Ir tik po kurio laiko apie spragą buvo paskelbta viešai.


Iš ryškiausių Lietuvos neatsakingų atskleidimų pavyzdžių galima būtų paminėti Sveikatos sistemos spragos atvejį: [https://www.delfi.lt/news/daily/lithuania/bausme-uz-pilietiskuma-e-sveikatos-spragas-parodziusiam-vyrui-gresia-baudziamoji-byla.d?id=79204629](https://www.delfi.lt/news/daily/lithuania/bausme-uz-pilietiskuma-e-sveikatos-spragas-parodziusiam-vyrui-gresia-baudziamoji-byla.d?id=79204629)  
Laisvės TV reportažas: [https://www.youtube.com/watch?v=vFvwhYiWwdE](https://www.youtube.com/watch?v=vFvwhYiWwdE)

## Serveriai.lt saugumo spraga

2018 metų lapkritį UAB Interneto vizija (toliau IV) sistemoje buvo aptikta kritinė saugumo spraga. IV — tai viena iš stambiausių svetainių talpinimo paslaugų įmonių Lietuvoje, deklaruojanti 100 tūkstančių vartotojų (duomenys iš www.serveriai.lt).


Savo sistemoje IV naudoja Apache modulį “mod_security”, kuris yra atsakingas už pavojingų užklausų blokavimą. Tai yra, modulis yra kaip tarpininkas tarp naršyklės ir serveryje patalpintos PHP programos. Jis patikrina užklausą, ir jeigu ji atitinka tam tikrus požymius, blokuoja ją, todėl potencialiai pavojinga užklausa net nepasiekia PHP programos. Taip atrodo užblokuotų užklausų žurnalas:

```
[client 46.229.168.146] ModSecurity: Access denied with connection close (phase 1). IPmatchFromFile: "46.229.168.146" matched at REMOTE_ADDR. [file "/etc/httpd/conf/modsec/999_user_exclude.conf"]

[client 62.102.148.68] ModSecurity: Access denied with connection close (phase 1). IPmatchFromFile: "62.102.148.68" matched at REMOTE_ADDR. [file "/etc/httpd/conf/modsec/999_user_exclude.conf"] , referer: http://lesbian.vids.*******.com
```


Modulis sukurtas apsaugoti PHP programas su žinomomis spragomis nuo įsilaužimo, tačiau ir jis pats gali būti pažeidžiamas. Paaiškėjo, kad specialiai suformuota užklausa gali priversti serverį grąžinti PHP failo turinį:

![POST request](serveriailt-post-request.jpg)

Wordpress:

![Wordpress](serveriailt-burp-wordpresst.jpg)

Tokio tipo kritinė saugumo spraga programišių žargone kartais vadinama “game over”, nes leidžia visiškai užvaldyti sistemą. Atskleidus PHP turinį galima pamatyti slaptažodžius ir gauti pilną prieigą prie duomenų bazės. Kai kuriais atvejais tai suteiktų galimybę net įkelti savo PHP kodą į serverį ir toliau pasiekti vidinę infrastruktūrą (lateral movement).

## Prevencija

Ne visos PHP programos patikrinimo metu buvo pažeidžiamos. Pavyzdžiui, phpBB turi “.htaccess” konfiguraciją, kuri neleidžia web serveriui tiesiogiai atsiųsti nustatyto failo, kad ir kokio tipo jis yra. Taip jie apsaugo savo config.php failą:

[https://github.com/phpbb/phpbb/blob/master/phpBB/.htaccess](https://github.com/phpbb/phpbb/blob/master/phpBB/.htaccess)

![htaccess](serveriailt-htaccess.jpg)
  
## Atskleidimas

Apie galimą saugumo spragą buvo nedelsiant pranešta IV. Tą pačią dieną detalizuota ataskaita buvo užšifruota PGP ir išsiųsta techniniam vadovui. Spraga buvo patvirtinta ir ištaisyta po kelių dienų. Kadangi ataka turi specifinį pėdsaką — buvo patikrinti web serverio žurnalai dėl galimo pasinaudojimo (exploitation). Pėdsakų nerasta, tikėtina, kad nei IV, nei jų klientai nenukentėjo.


Išvados.
IV atvejis galėtų tapti pavyzdžiu IT verslui kurti saugią informacinę Lietuvos ateitį. Atsakingo atskleidimo programa:
1. naudinga verslui, nes sumažina galimus programinių klaidų nuostolius
2. skatina saugumo tyrinėtojų pilietiškumą
3. gerina informacinių sistemų atsparumą priešiškų šalių atakoms


## Timeline

```
2018-11-22 — pirminis pranešimas
2018-11-23 — pokalbis su IV programuotojais
2018-11-26 — gautas patvirtinimas, kad klaida ištaisyta
2019-04-29 — viešas pranešimas
```