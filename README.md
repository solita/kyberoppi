# Kyber, lyhyt oppimäärä

Tästä materiaalipankista saat perustietoa verkkosovelluksen tietoturvasta ja sen testaamisesta. Materiaali on valikoitu ja tiivistetty palvelemaan ensisijaisesti ohjelmistokehittäjiä, jotka haluavat tehdä turvallisia järjestelmiä. 

# Perusasiat web-sovellusten tietoturvasta

Perustiedot on koottu erilliseen dokumenttiin: [Perusasiat](perusasiat.md).

# Referenssimateriaali

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

## HTTP headerit

|Header|Mikä se on?| Selaintuki | Suositeltava arvo, miksi? | Muuta? |
|------|-----------|:-----------:|:----------------------:|------:|
X-Content-Type-Options: nosniff|Estää selainta arvaamasta uudelleen MIME-typeä.|Chrome, IE8|kokoelma-hederi. nosniff estää tietoturvaongelmia.|http://stackoverflow.com/questions/18337630/what-is-x-content-type-options-nosniff|
|Content-Type |MIME-type..| | | |
|Keep-Alive| | | | |
|X-Frame-Options:SAMEORIGIN| Estää sovelluksen avaamisen frameen mielivaltaisesta domainista.| |SAMEORIGIN, ellei ole erityistä syytä sallia iframeja kaikkialta, estetään ne. Tietoturvan kannalta estää clickjacking tyyppisiä hyökkäyksiä.|http://en.wikipedia.org/wiki/Clickjacking|
|Cookie + secure| Salaa cookien. Toimii vain jos on HTTPS, eli ei välttämättä devausympäristössä.| |https://www.owasp.org/index.php/SecureFlag|
|cookie + HTTP Only| Cookien käsittely javascriptilla estetty. Session cookieen ei ole mitenkään tarpeellista päästä käsiksi Javascriptilla clientin päässä. Tietoturva-asia. Triviaali hoitaa kuntoon.| | | https://www.owasp.org/index.php/HttpOnly|
|Strict-Transport-Security| Ohjeistaa selainta käyttämään aina HTTPS:ää. Ignoroidaan HTTP:tä käytettäessä.|Muut kuin IE|Mikäli sivustoa käytetään aina HTTPS:n yli, kannattaa headeri ottaa käyttöön|http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security|
|Cache-Control|IE:n kanssa monenlaisia ongelmia luvassa. Voi toimia myös eri tavalla HTTPS-protokollassa.| | | |
|X-XSS-Protection|Ehdottaa selaimelle, että sisällössä voi olla potentiaalisesti XSS sisältöä.|IE, Chrome, Safari| 1 (jos sivulla näytetään käyttäjän sisältöä), selain voi tämän perusteella käynnistää XSS filtterinsä jos käyttäjä on sen jostain syystä disabloinut.|OWASP-ZAP antaa low-tason varoituksen tämän puuttumisesta.|
|Same-Site|CSRF-esto|Chrome, Opera| lax (strict jossain tapauksissa)|https://www.owasp.org/index.php/SameSite |
 

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

Lisätietoa:
* Javascriptin enkoodaukset: https://mathiasbynens.be/notes/javascript-escapes
* HTML enkoodaukset: https://mathiasbynens.be/notes/ambiguous-ampersands
* Erikoismerkit verkko-osoitteissa: [Punycode](https://en.wikipedia.org/wiki/Punycode)


## Javascriptin ajaminen

Javascriptin ujuttamiseen ajoon on useita eri tapoja jos sovelluksessa on huolimattomuuden takia jonkinlainen XSS-aukko tai muu ongelma. Tässä on listattu joitakin:

* ```<script>``` tagin käyttö.
* ```onError=alert(1)``` erilaiset event-käsittelijät, joista onError on yksi.  
* ```<svg/onload=alert(1)>``` SVG-kuvaformaation hyväksikäyttö + event handler.
* XML-dokumentin sisällä (CDATA)
* SVG-kuvan käyttö alustana (CDATA + script)

Lisätietoa:
* https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot
* https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20injection

## Erikoiset URL:t

Selaimet tunnistavat useita URL-osoitteita, jotka eivät ole normaaleja verkko-osoitteita ja niitä voidaan hyötykäyttää. Riippuu selaimen ja tietokoneen asetuksista mitä kaikkea voidaan tehdä, mutta tässä on joitakin esimerkkejä.

* data:text/html - esimerkiksi ```data:text/html,<script>alert(1)</script>```
* ```javascript:alert(1)```
* ```http://;URL=javascript:alert(1)```
* Base64-enkoodauksen hyväksikäyttö: ```data:text/html;base64,PHN2Zy9vbmxvYWQ9YWxlcnQoMik+```
* ```mailto:```  -sähköpostin lähetys
* ```callto:```  -puhelinsoitto (esimerkiksi maksulliseen numeroon)
* ```tel:``` - puhelinsoitto (esimerkiksi maksulliseen numeroon)

Data-tyyppistä "verkko-osoitetta" voidaan käyttää myös dynaamisesti kuvien tai äänen muodostamiseen verkkosivulla jos HTML-koodissa osoitetaan datasisältö sen avulla.

Lisätietoa:
* https://github.com/ouspg/urlhandlers


# Lisenssi

![lisenssi](88x31.png)

Creative Commons, Attribution-NonCommercial CC BY-NC
Kts. [LISENSSI](LICENSE).

