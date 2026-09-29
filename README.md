# h6-Fuzzy-Feeling-Upon-Finding
Ville Suikki
Tietokannat ja tiedonhallinta - SOF001AS2A-3050 (s26), Tero Karvinen

__________________________________________________________________________________________________________________________________________________________________________

Hoikkala 2026: Fuzzing with Fuff, kalvot joohoin esityksestä kurssilta.

#Mitä fuzzing tarkoittaa?

Kyseessä on automatisoitu menetelmä, jossa ohjelmalle tai verkkopalvelu työnnetään valtavasti odottamattomia, satunnaisia syötteitä jotta mahdolliset virheet ja haavoittuvuudet paikannetaan tai mahdolliset muut täkeät piilotetut osat pakannetaan (Bruteforce)

#Mikä on ffuf ja mihin sitä käytetään?

Fuzz faster you fool tai niin kuin Juho videossaan mainitsi Fuzz faster you fucker. Työlaku on nopea komentorivillä toimiva avoimen lähdekoodin työkalu eri web-sovellusten fuzzaukseen. Pääsääntöinen käyttökohde  piilotettujen kansioiden, tiedostojen, parametrien ja virtuaalisten hostien etsimiseen. 

#Miten ffuf toimii käytännössä?

Työkalu käyttää apunaan sanakirjoja joita se käy läpi nopeasti http-pyynnöillä. Ve vertailee vastauksia esim. eri statuskoodeja http sivuille- 200, 403, 404 ja täten auttaa havaitsemaan mitkä polut on olemassa. 

#Miksi fuzzing ja ffuf ovat tärkeitä kyberturvallisuudessa / penetratiotestauksessa? 

Se on tärkeää koska sillä voidaan löytää automaattisesti sovelluksen piilotettuja tai odottamattomia syötteitä, reittejä ja käyttäytymismalleja, joita manuaalisella testaamisella olisi helppo missata.


__________________________________________________________________________________________________________________________________________________________________________

Hoikkala "joohoi" 2020: Still Fuzzing Faster (U fool). In HelSec Virtual meetup #1. (Noin tunnin mittainen)

- Käydään läpi perusteita esim: miten palvelimelle lähetetään nopeasti suuri määrä samankaltaisia HTTP-pyyntöjä ja analysoidaan saatuja vastauksia poikkeamien löytämiseksi
- ffuf kehitettiin vaihtoehdoksi muille työkaluille (kuten Gobusterille) tekijän omien tarpeiden ja toivottujen ominaisuuksien pohjalta
- ffuf-työkalulla voidaan etsiä piilotettuja kansioita ja tiedostoja verkkopalvelimelta hyödyntämällä esimerkiksi SecLists-sanastoja
- Hakuun voi lisätä myös tiedostopäätteitä (esim. .php)
- ffuf:lla voidaan kohdistaa POST-pyyntöjä kirjautumislomakkeisiin tai hyödyntää HTTP Basic Authentication -tunnistautumista käyttäjä- ja salasanalistojen avulla
- Palvelimelta voidaan etsiä erillisiä aliverkkotunnuksia ja virtuaalihosteja muokkaamalla HTTP:n Host-headeria
- Parametrien haku: Sivuilta voidaan etsiä GET-parametrejä ja testata numeerisia ID-arvoja standardisyötteen (-w -) kautta
- XSS-apuri: Etsitään suodatettuja erikoismerkkejä heijastuvista syötteistä
- Integrointi Burp Suiteen jossa ffuf-tulokset voidaan ohjata -proxy-lipun avulla proxyyn manuaalista jatkokäsittelyä varten
- Template Injection (SSTI): Etsitään mallinehaavoittuvuuksia laskutoimitusten tuloksia vertaamalla'
- Mielenkintonen osuus videost oli mielstäni Q/A osiossa jossa kysyjä kysyi mitä Payloadeja käyttäjä käyttää mimenomaan WB-fuzzaukseen, tai kannattaisi käyttää. Joissa esittäjä esitti suuria sanakirjastoja- ja painotti omia "payload -listoja" kuten Seclists.

__________________________________________________________________________________________________________________________________________________________________________

a) Tallenna itsellesi kopio säännöistä. Kirjoita omin sanoin,

Scope. Eli kohde, joka määrittelee, mitä verkkoympäristöä testataan. Harjoituksessa kohde/scope on osoite [https://ffuf.io.fi/play](https://ffuf.io.fi/play)
xules of engagement. Eli mitä kohteelle saa tehdä. Mitä tai millaisia menetelmiä saa käyttää? Sallitut menetelmät: pelisäännöt  määrittelevät, että kohdetta saa testata ffuf-työkalulla hakemistojen, tiedostojen ja parametrien löytämiseksi. Muu toiminta, kuten palvelimen tahallinen ylikuormittaminen (DoS) tai tietokantojen rikkominen, on kielletty.
Mihin oikeutesi tehdä tietoturvatestausta tähän kohteeseen perustuu? Tietoturvatestaus ilman jörejestelmän omistajan lupaa on Suomen laissarangaistava teko. Oikeus tämän kohteen testaamiseen perustuu täysin palvelimen omistajan joohoi:n ja kurssin opettajan Tero Karvisen antamaan suostumukseen harjoitusmielessä.
Riskit ja mitigointi:

1 - Väärän kohteen skannaaminen: Kirjoitusvirhe URL-osoitteessa voi ohjata  skannauksen sivullisen palvelimelle. Mitigointi: URL-osoitteiden ja komentojen huolellinen tarkistus ennen Enteriä.
2 - Palvelunestohyökkäys (DoS): Ffuf lähettää oletuksena pyyntöjä erittäin nopeasti. Se voi vahingossa ruuhkauttaa kohteen, jolloin se kaatuu. Mitigointi: Käytetään tehokkaita, valmiiksi rajattuja sanalistoja massiivisten listojen sijaan, eikä jätetä skannereita pyörimään valvomatta.

__________________________________________________________________________________________________________________________________________________________________________

b) Asenna ffuf versio, joka tukee aivan uutta preflight-ominaisuutta.

Lähdin etenemään Fuffin asennuksella syöttämällä komennon sudo apt update && sudo apt install ffuf -y, ffuf -V jotta saan version joka oli 2.1.0-dev ja ffuf -h | grep -c preflight joka antoi numeerisen tulosteen siitä että tukeeko ko. versio tehtävää kokonaisuudessaan. Minulla ei. Etenin niin että ajoin komennon sudo apt install golang -y joka tyytää Kalin pakettienhallintaa etsimään, lataamaan ja asentamaan Go-ohjelmointikielen työkalut. Tämän jälkeen latasin fuffin komennolla go install [github.com/ffuf/ffuf/v2@latest](https://github.com/ffuf/ffuf/v2@latest) ja siirsin sudo cp ~/go/bin/ffuf /usr/local/bin/ komennolla ohjelman järjestelmä kansioon. Nyt ffuf -h | grep -c preflight sain numeeriseksi luvuksi 4 jossa tehtävänannon kokonaisuudessaan tulisi toimia.

<img width="1443" height="984" alt="image" src="https://github.com/user-attachments/assets/c87bc39a-8fd3-4036-90d7-cc6a7e1df9d8" />
__________________________________________________________________________________________________________________________________________________________________________

c1) Content discovery (Vaultline https://ffuf.io.fi/play tehtävät on numeroitu näin, käytetään tässä samoja.).

Lähdin suorittamaan tehtävää lataamalla ensin tehtävää varten tehdyn erikois-sanalistan komennolla curl -O [https://ffuf.io.fi/wordlists/content.txt](https://ffuf.io.fi/wordlists/content.txt), joka tallensi tiedoston koneelle alkuperäisellä nimellä. Tämän jälkeen käynnistin  skannauksen komennolla ffuf -w content.txt -u [https://ffuf.io.fi/FUZZ](https://ffuf.io.fi/FUZZ) -ac. Tässä komennossa -w määritti käytettävän sanalistan ja -u kohdeosoitteen, jossa FUZZ toimi merkkinä sanalistasta haettaville sanoille. Koska kohdepalvelin oli ansoitettu palauttamaan kaikkiin keskityin osoitteisiin "200 OK" -vastauksen, lisäsin komentoon -ac (auto-calibration). Se sai ohjelman analysoimaan huijaussivujen rakenteen ja suodattamaan ne automaattisesti pois. Tämän ansiosta skannaus onnistui suodattamaan roskavastaukset ja paljasti olemassa olevat piilotetut polut, kuten admin, login, .env ja api

Jostain syystä vaikka päivitin ffuffin viimeisimmän version, niin kuva silti antaa ymmärtää että versio on 2.1.0-dev vaikka koitin useammalla eri komennolla saada sen näyttämään viimeisintä versiota, tuloksetta. Onnistuin kuitenkin kaappaamaan kuvan oheiset tiedot, ja onnistuin myös -fc 200 -suodattimella paljastamaan aidot non-200 -polut, mutta työkalun versio-ongelma piilotti täysin epästandardin koodin.

<img width="1399" height="940" alt="image" src="https://github.com/user-attachments/assets/a53629bb-ab3d-43f4-b742-91ffecf9065e" />

__________________________________________________________________________________________________________________________________________________________________________

c2) The interesting non-200

Myös C2 on käytännössä suoritettu koska tein teknisesti aivan oikein ja sain rajattua tulokset tasan niihin kuuteen aitoon non-200 -polkuun. Mainittakoon että C2-kohdassa raportissa, että käytin -mc 100-599 -fc 200 -komentoa poistaaksesi 200 OK -huijaussivut, mutta ffufin sisäinen suodatusominaisuus piilottaa täysin epästandardit HTTP-koodit listauksesta.
__________________________________________________________________________________________________________________________________________________________________________

c3) Recursion

Lähdin sselvittämään, mitä aiemmin löydettyjen kansioiden (kuten backup, api ja enumerate) sisälle on piilotettu. Syötin komentoriville komennon /usr/local/bin/ffuf -w content.txt -u [https://ffuf.io.fi/FUZZ](https://ffuf.io.fi/FUZZ) -recursion -recursion-depth 1 -ac. Tässä komennossa -recursion käskee ohjelmaa aloittamaan sanalistan läpikäymisen automaattisesti alusta jokaisen löydetyn uuden kansion kohdalla. Lisäsin mukaan vivun -recursion-depth 1, joka rajoittaa kansion sisään sukeltamisen vain yhden tason syvyyteen jottei skannaus karkaa loputtomiin alikansioihin.

Tulosteesta huomasin, että ohjelma lisäsi löytämänsä hakemistot työjonoon (esim. Adding a new job to the queue) ja alkoi skannata niitä vuorotellen. Tämän avulla sain esiin syvemmälle piilotettuja tiedostoja, joita pelkkä etusivun skannaus ei olisi ikinä löytänyt. Tuloksista paljastui muun muassa /backup/-kansion sisältä tietokannan kopio db.sql.bak, sekä muiden kansioiden sisältä uusia polkuja kuten v2, 2024, 2025 ja 2026.

<img width="1442" height="983" alt="image" src="https://github.com/user-attachments/assets/e69fad4e-182f-4f68-bda7-51706f507abe" />

__________________________________________________________________________________________________________________________________________________________________________

c4) Virtual hosts

Lähdin suorittamaan C4-tehtävää, jossa tafoite oli löytää saman palvelimen taustalta piilotettuja virtuaalipalvelimia. Koska suora URL-osoitteen fuffaaminen olisi rikkonut SSL-varmennekättelyn, kohdistin skannauksen pääosoitteeseen -u [https://ffuf.io.fi](https://ffuf.io.fi) ja hyödynsin HTTP-otsaketta -H "Host: FUZZ.ffuf.io.fi".   Aluksi latasin tätä tehtävää varten suunnatun erillisen sanalistan komennolla curl -O [https://ffuf.io.fi/wordlists/vhosts.txt](https://ffuf.io.fi/wordlists/vhosts.txt). Ennen  ajoa tutkin vastauksia ja totesin, että virheelliset tai olemattomat aliverkkotunnukset palauttavat sivun, jonka pituus on poikkeuksetta 377 sanaa. Ajoin skannauksen komennolla /usr/local/bin/ffuf -w vhosts.txt -u [https://ffuf.io.fi](https://ffuf.io.fi) -H "Host: FUZZ.ffuf.io.fi" -fw 377, jossa -fw 377 -vivulla suodatin pois kyseiset roskavastaukset. Suodatuksen johdosta skannaus ohitti oletussivut ja nosti esiin tasan kolme aidosti olemassa olevaa virtuaalipalvelinta: admin, staging ja dev.   

<img width="1398" height="939" alt="image" src="https://github.com/user-attachments/assets/3a3958ca-a0a4-4479-ad15-7f7ece956cd1" />

__________________________________________________________________________________________________________________________________________________________________________
c9) The login you cannot replay (Has preflight! Has CSRF token!)

Tähän tehtävään minulla valitettavasti tyssäsi enkä muiden opintojen nojalla kerinnyt tekemään tehtävää loppuun saakka. 

__________________________________________________________________________________________________________________________________________________________________________

Lähteet:
Hoikkala 2026: Fuzzing with Fuff
https://terokarvinen.com/tunkeutumistestaus/hoikkala-2026-fuzzing-with-ffuf.pdf
https://github.com/resources/articles/what-is-fuzz-testing 
https://github.com/ffuf/ffuf/blob/master/README.md
https://ffuf.io.fi/play
https://www.youtube.com/watch?v=mbmsT3AhwWU
https://www.youtube.com/watchv=9Hik0xy9qd0&t=1546s
__________________________________________________________________________________________________________________________________________________________________________
