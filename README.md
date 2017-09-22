# Kyber, lyhyt oppimäärä

Tästä materiaalipankista saat perustietoa verkkosovelluksen tietoturvasta ja sen testaamisesta. Materiaali on valikoitu ja tiivistetty palvelemaan ensisijaisesti ohjelmistokehittäjiä, jotka haluavat tehdä turvallisia järjestelmiä.

# Perusasiat

# Referenssi materiaali

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
 


# Lisenssi

!(88x31.png)

Creative Commons, Attribution-NonCommercial CC BY-NC
Kts. [LICENSE].

