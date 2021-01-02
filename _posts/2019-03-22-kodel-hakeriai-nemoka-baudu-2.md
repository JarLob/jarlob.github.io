---
layout:     post
title:      "Kodėl hakeriai nemoka baudų? 2-oji dalis."
date:       2019-03-22 12:00:00
categories: HackAndTell TvarkauMiesta
cover:      TM2.png
permalink:  /lt/blog/kodel-hakeriai-nemoka-baudu-2
---
Po praeito [straipsnio](/lt/blog/kodel-hakeriai-nemoka-baudu) kai kas atkreipė mano dėmesį į tai, kad [tvarkaumiesta.lt](https://www.tvarkaumiesta.lt/) (toliau TM) kodas yra atviras ir netgi galima pasižiūrėti [kaip buvo ištaisyta saugumo skylė](https://bitbucket.org/emiestas/tvarkau-miesta_moved/commits/f03a936134a7e27982df54350a8960bdb04ea05a). Tai paskatino mane paanalizuoti TM giliau.

> tl;dr: Radau saugumo pažeidimų kurie leido pilną prieigą prie puslapio duomenų bazės su vartotojų asmens kodas ir kontaktais, prisijungimą prie savivaldybės vidinės dokumentų valdymo sistemos, prieigą prie privačių šifravimo raktų naudojamų darbui su [e.valdžios vartais](https://www.epaslaugos.lt/portal/login) ir dar daugiau.

Visų pirma noriu pasakyti, kad programinio kodo atvirumas nedaro puslapio nei mažiau, nei daugiau saugiu. Tai tik padidina skaidrumą kuris yra naudingas ir blogiečiams ir gėriečiams.

Pasižiūrėjus į pataisymą pasidarė aišku kam tas antras parametras `oid`. TM turi savo registruotų pranešimų duomenų bazę, bet tuo pačiu metu dirba ir su vidine savivaldybės dokumentų valdymo sistema Avilys. `Oid` yra dokumento numeris Avilio sistemoje. Programuotojai pataisė skylę pridėję pamirštą patikrinimą ar pranešimą norintis ištrinti vartotojas yra jo kūrėjas:

[![Pranešimas](tmfix.png "Pranešimas")](tmfix.png)

Patikrinimas buvo standartinis, toks pats kaip visur kitur programoje. Tik viena problema - buvo tikrinamas TM pranešimo `id`, bet ne Avilio `oid`. Vartotojas galėjo sukurti savo pranešimą ir nusiųsti modifikuotą užklausą ištrinti savo `id`, bet svetimą `oid`. Tai paliktų svetimą pranešimą TM, bet ištrintų jį savivaldybėje. Ši loginė patikrinimo klaida egzistavo visoms operacijoms, ne tik trinimui. Mano nuomone `oid` išvis neturėtų būti rodomas vartotojui.

Tiesa sakant, išeities kodas šiai saugumo skylei atrasti net nebuvo būtinas. Viena iš taisyklių tinkamų bet kokiam testavimui yra nedaryti prielaidų “ko gero jie apie tai pagalvojo” ir “kitaip greičiausiai neveiks”. Padariau prielaidą, kad `id` turi atitikti `oid` ir praleidau klaidą. Galimybė pasižiūrėti į kodą tik išryškino klaidas.

Bet tam, kad leistų ištrinti pranešimą reikia būti registruotu vartotoju ir pirmą kartą reikia jungtis per valdžios vartų portalą. Ar yra būdas tai apeiti? Mano dėmesys nukrypo į login.php. Pastebėjau, kad nusiuntus specialią prisijungimo su Facebook ar Google užklausą TM paslaugiai įleidžia vidun be jokios registracijos:

```
POST /login.php HTTP/1.1
...
fb_response_id=abc&type=login
```

Taip pat pastebėjau būdą kaip perimti pranešimų autorystę iš bet kurio vartotojo:

```php
else if($_POST['type']=='join'){
			updateUserInfo($_SESSION['USER_ID'],$_POST['fb_response_id'],'fb');
			updateProblemsChangeUser($_POST['user_to_join'],$_SESSION['USER_ID']);
			deleteUser($_POST['user_to_join']);
}
```

Tereikia išsiųsti sekančią užklausą ir visi jo pranešimai bus priskirti jums, o jo vartotojas ištrintas:

```
POST /login.php HTTP/1.1
...
fb_response_id=abc&type=join&user_to_join=123
```

Šitas būdas nors veiksmingas, bet brutalus - svetimo vartotojo id aš nežinau, o spelioti arba periminėti absoliučiai visus nenoriu.

[Gitignore failas](https://bitbucket.org/emiestas/tvarkau-miesta_moved/src/6c100468d919a59e9c4576906a7356e3c0694087/.gitignore?at=master&amp;fileviewer=file-view-default) pasufleravo kokios TM direktorijos vis dėlto nėra atviro kodo ir kur ieškoti serverio log failo. Beje jį peržiūrėti galėjo bet kas, tereikėjo įvesti naršyklėje [https://tvarkaumiesta.lt/logs/nginx.log](https://tvarkaumiesta.lt/logs/nginx.log), bet nieko naudingo sau ten neradau. Užtat

```php
$xmlsec = new xmlsec(dirname(__FILE__) . '/viisp/keys.xml', dirname(__FILE__) . '/temp/');
```

atvedė prie [privataus kriptografinio rakto](https://tvarkaumiesta.lt/viisp/keys.xml), naudojamo užklausų siunčiamų į e.valdžios portalą TM vardu pasirašymui. Jį, beje, irgi galėjo parsisiųsti bet kas.

Atviras kodas tai ir atvira pakeitimų istorija. Vartotojų nuotraukoms ir kitiems dokumentams saugoti TM naudoja [Minio](https://www.minio.io/). 2017-ųjų spalį programuotojas [pašalino iš kodo prisijungimo prie Minio kodus](https://bitbucket.org/emiestas/tvarkau-miesta_moved/diff/class/myFunctions.php?diff2=c0cffd37c077&amp;at=map):

[![MinioKeys](miniokeys.png "MinioKeys")](miniokeys.png)

bet patingėjo juos pakeisti, nes... niekas gi nesužinos. Bet blogiausia pasirodė tai, kad jų net nereikėjo žinoti - bet kas galėjo atsidaryti naršyklėje pvz. [https://minio.vilnius.lt/minio/answers.vilnius/](https://minio.vilnius.lt/minio/answers.vilnius/) ir parsisiųsti laiškus su potencialiai asmenine informacija.

Jeigu programuotojai ir padarė ką teisingai, tai buvo sprendimas 2018-ųjų vasarį perrašyti darbą su duomenų baze ir naudoti parametrizuotas užklausas. Žiūrėdamas į tai, kaip saugomi vartotojų duomenys ir prisijungimo prie Avilio slaptažodžiai, jaučiausi kaip katinas stebintis akvariumą su žuvytėmis. Visas bendravimas su duomenų baze vyko per pagalbinę [MysqliDb](https://bitbucket.org/emiestas/tvarkau-miesta_moved/src/ea84d88c7b29acfbbbc2777a02205e0d8fcfa277/class/Database.php?at=11332&amp;fileviewer=file-view-default) klasę. Nors kode nebuvo užuominų, kad tai trečios šalies atviras kodas, greita paieška rado [šį GitHub projektą](https://github.com/ThingEngineer/PHP-MySQLi-Database-Class). Buvo akivaizdu, kad tai tas pats kodas, tik TM versija yra šiek tiek senesnė. Palyginęs failus ir peržiūrėjęs istoriją naujausioj versijoj neradau jokių saugumo klaidų pataisymų, kuriuos galima būtų panaudoti. Vienintelis šansas buvo surasti ką nors naujo pačiam. Po kiek laiko kai ką aptikau:

```php
public function where($whereProp, $whereValue = 'DBNULL', $operator = '=', $cond = 'AND'){
        // forkaround for an old operation api
        if (is_array($whereValue) && ($key = key($whereValue)) != "0") {
            $operator = $key;
            $whereValue = $whereValue[$key];
        }
        if (count($this->_where) == 0) {
            $cond = '';
        }
        $this->_where[] = array($cond, $whereProp, $operator, $whereValue);
        return $this;
}
```

Įtartinas patikrinimas naudojo masyvo indeksą kaip operatorių formuoti SQL sakinį jei jis nebuvo nulis. Šita funkcija paprastai naudojama taip:

```php
$db->where('ID', $_POST['id']);

$name = $db->getValue('USERS', 'name');
```

`$_POST['id']` yra vartotojo duomenys kurie perduodami į `$whereValue` ir jei nėra papildomų patikrinimų kaip `is_numeric($_POST['id'])` vartotojas gali įterpti savo SQL sakinį. Pvz: `id[= ? or 1=1 --]=0` Viskas kas yra tarp laužtinių skliaustų yra vykdoma duomenų bazės kaip užklausos dalis. Šiek tiek užtruko išsiaiškinti kaip perduoti masyvą PHP ir surasti patogiausią vietą kur tai padaryti, bet galų gale turėjau:

```
POST /login.php HTTP/1.1
...
fb_response_id%5B%3D%20%3F%20or%201%3D2%20union%20select%20PSW,0,0,0,0,0,0,0,0,0,0,0,0,0,0%20from%20PRS_DVS_CFG%20where%20CITY_ID%20%3d%201%20--%5D=0&type=add
```

Turėjau puikų "orakulą" - užklausos rezultatas buvo patogiai rodomas naršyklės lange:

```php
else if($_POST['type']=='add'){
			$user=getUserByFB($_POST['fb_response_id']);
			if($user){
				echo $user['ID'];
			}
```

Gavau Avilio prisijungimo duomenis visiems miestams, bet pasirodė, kad tik vienintelė Vilniaus savivaldybė laiko savo vidinę dokumentų valdymo sistemą [prieinamą visam pasauliui](https://ivs.vilnius.lt/) (kad ir apsaugotą slaptažodžiu):

[![Avilys](avilys.png "Avilys")](avilys.png)

Atskiro paminėjimo vertas vartotojų slaptažodžių saugojimas duomenų bazėje. TM naudoja MD5 su druska. Pagrindinė problema yra tai, kad MD5 "nulaužimo" greitis perrinkimo būdu yra mažiausiai 25 tūkstančių kartų didesnis, nei moderni saugi alternatyva. Šiuo atveju kuo greičiau, tuo blogiau, nes galimų slaptažodžių perrinkimas turi būti kuo lėtesnis. Šiuolaikinis kompiuteris su viena grafine plokšte nulaužtų sudėtingą aštuonių įvairiausių simbolių slaptažodį per kelias dienas. O yra kompiuterių ir su [aštuoniom grafinėm plokštėm](https://gist.github.com/epixoip/a83d38f412b4737e99bbef804a270c40).

Liko pasiekti paskutinį užsibrėžtą tikslą - gauti slaptus serverio failus kaip `configuration/config.php` arba `viisp/xmlclasstst.php`. Tam galima būtų panaudoti Path Traversal pažeidžiamumą, bet situaciją apsunkino tai, kad failų kopijavimas buvo simetriškas, panašiai kaip:

```php
copy($uploadPath . '/' . _POST['foto'], $minioBucketName . '/' . _POST['foto']);
```

Taigi įterpus vietoj nuotraukos `../configuration/config.php` kopijavimas nepavyktų, nes Minio pusėje nebuvo tokio `configuration` aplanko. Šiuo atveju galima buvo pasinaudoti atskleistu Minio prisijungimo slaptažodžiu ir susikurti ten aplanką reikalingu pavadinimu rankiniu būdu. Dažnai sakoma, kad sistema yra tiek saugi, kiek saugi silpniausia jos grandis. Bet kartais vienas pažeidžiamumas atveria duris kitam.

Chronologija:  
2019.03.08 - Išsiųstas laiškas su rastų problemų aprašymu ir pataisymų rekomendacijomis.  
2019.03.12-21 - Papildomi komentarai dėl pataisymų.  
2019.03.21 - Savivaldybės darbuotojai patikina, kad visi suplanuoti pakeitimai atlikti.  
