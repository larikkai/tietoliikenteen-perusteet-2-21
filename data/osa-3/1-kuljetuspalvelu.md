---
path: '/osa-3/1-kuljetuspalvelu'
title: 'Kuljetuspalvelun toiminta'
hidden: false
---

<text-box variant='learningObjectives' name='Oppimistavoitteet'>

- Osaat kuvata kuljetuskerroksen palvelujen toiminnan yleisellä tasolla.
- Tiedät mitä tarkoittavat segmentti ja lomitus. Osaat kertoa, mitä ne ovat ja miten lomitus toimii.

</text-box>



## Kuljetuspalvelu

Kuljetuspalvelun tehtävänä on huolehtia sen kuljetettavaksi annettu viesti lähettäjältä vastaanottajalle. Tämän kurssin edellisessä osassa ja aiemmalla kurssilla on jo käyty läpi sovelluskerroksen toiveita kuljetuskerrokselle. Palaute ne mieleesi nyt. Käy tarvittaessa aiempia materiaaleja läpi.


Kuljetuskerroksen keskeinen tehtävä on täyttää sovelluskerroksen toiveet ja vaatimukset ja näin piilottaa se seikka, että verkkokerros ei näitä pysty tarjoamaan.

Luotettavan kuljetuspalvelun keskeinen tehtävä on siis piilottaa verkkokerroksen epäluotettavuus sovelluskerrokselta:
* Sovelluskerros edellyttää kuljetuspalvelulta virheettömyyttä ja luotettavuutta, mutta verkkokerros voi muuttaa segmentin bittejä tai kadottaa koko segmentin
* sovelluskerros edellyttää järjestyksen säilymistä, mutta verkkokerros voi toimittaa segmentit epäjärjestyksessä
* sovelluskerros edellyttää kaksoiskappaleiden karsimista, mutta verkkokerros voi monistaa segmenttejä eli toimittaa useita kopioita samasta segmentistä
* sovelluskerros edellyttää mielivaltaisen pitkiä sanomia, mutta verkkokerros voi rajoittaa segmentin kokoa
* verkkokerros voi lisäksi viivyttää segmenttejä satunnaisen pitkän ajan

Ennen kuin käydään yksityiskohtaisemmin läpi kuljetuskerroksen TCP:n ja UDP:n toimintaa, niin tarkastellaan asioita ensin hiukan teoreettisemmin. Edellytämme protokollilta ja niiden toteutuksilta, että ne toimivat virheettömästi ja tekevät sen mitä lupaavat. Tämän arviointi suunnitteluvaiheessa perustuu protokollan mallintamiseen ja tämän mallin toiminnallisuuden ja oikeellisuuden arviointiin. Vaikka emme tällä kurssilla opettele verifioimaan ja validoimaan protokollia ja niiden oikeellisuutta, niin yleiskuva mallinnuksesta on kuitenkin tarpeen. Mallinnuksessa kuten yleensä tietojenkäsittelyssä abstrahoimme asioita ja tarkastelemme vain sillä hetkellä kiinnostavia piirteitä.


## Mallinnuksesta

Kun mallinnamme tietoliikennejärjestelmän toimintaa, käytämme usein termiä kanava (engl. channel) kuvaamaan varsinaista tietoa siirtävää elementtiä. Kanava-termillä mallia ei sidota tiettyyn järjestelmään, vaan se sopii kaikkiin sellaisiin ympäristöihin, joissa kanavalle määritellyt ominaisuudet ovat samoja. Kanava on siis vain joukko ominaisuuksia, jotka kuvaavat, miten tieto siirtyy lähettäjältä vastaanottajalle. Kanava voi esimerkiksi kadottaa viestejä, jolloin kaikki lähetetyt viestit eivät pääse perille. Kanava voi myös luvata toimittaa kaikki viestit virheettömästi perille. Mallin suunnittelija määrittelee kanavan ominaisuudet siten, että mallilla saadaan tarkasteltua haluttua toiminnallisuutta.

<img src="../img/sovelluskerros-kuljetuspalvelu.svg" alt="Sovelluskerroksen prosessit lähettävät dataa toisilleen kuljetuskerroksen 'putken' läpi."/>
KUVA: Sovelluskerroksen näkökulmasta kuljetuskerroksen tarjoama kuljetuspalvelu on yhtenäinen luotettava kanava, johon lähettäjä laittaa viestin ja josta se sitten kulkee vastaanottajalle. Kanavassa kaikki viestit etenevät järjestyksessä ja pääsevät myös perille.

Kuljetuskerroksen kuljetuspalvelun näkökulmasta tilanne on paljon monimutkaisempi. Koska verkkokerros ei tarjoa luotettavaa siirtoa, on kuljetuspalvelun itse toteutettava tämä luotettava kuljetus epäluotettavan verkkokerroksen päälle. Tähän tarvitaan kuljetusprotokollaa, jonka pitää huolehtia kaikkien viestien perillemenosta oikeassa järjestyksessä ja oikealla sisällöllä, vaikka kanava sen alapuolella voi kadottaa, muuttaa, monistaa ja sekoittaa viestejä.

<img src="../img/kuljetuspalvelu-verkkokerros.svg" alt="Kuljetusprotokollan eri päissä (lähettäjä ja vastaanottaja) toimivat protokollan palaset saavat/välittävät dataa sovelluskerrokselle ja lähettävät/vastaanottavat segmenttejä verkkokerroksen 'putkesta'. Verkkokerroksen 'putki' yhdistää nämä protokollan palat toisiinsa."/>
KUVA: Kuljetuspalvelun näkökulmasta verkkokerroksen tarjoama palvelu on epäluotettava kanava, joka voi muuttaa viestin sisältöä, kadottaa viestin, monistaa viestin ja vaihtaa viestien järjestyksen.

Kuljetuskerros siis piilottaa verkkokerroksen puutteita ja parantaa sovellukselle näkyvän palvelun laatua. Aivan kaikkea kuljetuskerros ei kuitenkaan pysty piilottamaan. Esimerkiksi viestien viivästymiset tai verkkoyhteyden katkeaminen ovat tilanteita, jolloin kuljetuskerros ei pysty välittämään viestejä ja jotka siten välttämättä näkyvät myös sovelluskerrokselle.

Yksinkertaisissa mallinnuksissa ei tyypillisesti oteta kantaa viestien kulkuaikoihin vaan ainoastaan sisällön oikeellisuuteen ja järjestykseen. Yhteyden katkeaminenkin voidaan mallissa ajatella vain todella pitkäksi viivästymiseksi. Kun viivästyminen joskus päättyy, niin viestit kuitenkin pitää välittää sisällön ja järjestyksen mukaan oikein. Malliin on toki mahdollista ottaa mukaan myös aika, mutta tällä kurssilla otamme käyttöön vain ajastimet, emme varsinaisia aika-arvioita viestien kulkemisiin.

Seuraavassa aliluvussa käsitellään kuljetuspalvelun toimintaa ensin mallien avulla.


## Segmenttien lomitus

Yksi kuljetuspalvelun keskeisistä tehtävistä on saapuvien viestien toimittaminen oikealle sovelluskerroksen vastaanottajalle. Samassa koneessa voi olla samanaikaisesti useita prosesseja, jotka odottavat saapuvia viestejä.

Vastaanottovaiheessa linkkikerros poimii saapuvista viesteistä ne, jotka linkkikerrokse osoitteen perustella on tarkoitettu tälle koneelle. Se antaa nämä viestit verkkokerrokselle, joka tarkistaa verkkokerroksen tiedoista viestin varsinaisen vastaanottajan. Jos vastaanottaja ei ollut tämä laite, niin viesti reititetään kohti lopullista vastaanottajaa ja annetaan linkkikerrokselle toimitettavaksi eteenpäin. Kun verkkokerroksen mielestä vastaanottaja on tällä koneella, se antaa viestin edelleen kuljetuskerrokselle.

Kun kuljetuskerros saa viestin verkkokerrokselta, on kuljetuskerroksen selvitettävä, mille sovelluskerroksen vastaanottajalle viesti on oikeasti menossa. Jokainen verkkokerrokselta saapuva viesti on siis selvitettävä erikseen. Ne voivat olla menossa eri vastaanottajille.

Vastaavasti sovelluskerrokselta kuljetuskerrokselle tulevat viestit saapuvat eri kohteista ja lähetysvaiheessa ne voivat lomittua keskenään.

<img src="../img/lomitus.svg" alt="Kuvassa on kolme laitetta A, B ja C. Niissä jokaisessa on koko protokollapino kuvattuna. Laitteet on yhdistetty samaan kanavaan. "/>
KUVA: Kuvassa A:n ja C:n sovelluskerroksen prosessit kommunikoivat keskenään porttinumeroiden 2476 (A:lla) ja 3614 (C:llä) kautta. Vastaavasti B:n ja C:n sovelluskerroksen prosessit kommunikoivat keskenään porttinumeroiden 1258 (B:llä) ja  5625 (C:llä) kautta. C:llä on siis kaksi sovellusprosessia (tai yksi, joka kommunikoi useamman pistokkeen/porttinumeron kautta eri laitteissa olevien sovellusten kanssa), jotka käyttävät eri portteja ja kommunikoivat vastapuoliensa kanssa niiden välityksellä.


<quiz id="888b75a6-9244-53ec-8f8b-e68f203150ba"></quiz>

