---
layout: page
title: osa 3
permalink: /osa3/
---

## Osan 3 oppimistavoitteet

- Web-sovellusten toiminnan perusteet
  - RESTful-periaatteiden alkeet
    - HTTP-pyyntötyypit
    - statuskoodit
    - resurssiperustaiset URL:it
    - HTTP-pyyntöjen- safety ja idempotence-ominaisuudet
- node.js/express
  - npm
    - sovelluksen luominen
    - riippuvuuksien hallinta
    - npm-skriptien alkeet
    - versiointi
    - nodemon
  - HTTP-pyyntöjen käsittely
    - reittien määrittely
    - pyyntöihin vastaaminen eri statuskoodein
  - middlewaret
  - node.js-sovellusten debuggaaminen
  - sovelluksen vieminen tuotantoon
  - sovelluskehitys- ja tuotantoversion tietokantojen erottaminen
- mongo
  - dokumenttitietokantojen perusteet
  - mlab:in (tai vastaavan palvelun) mongon peruskäyttö
  - mongoose
    - skeema ja model
    - olioiden hakeminen, tallettaminen, editointi ja poisto



## Node.js

Siirrämme tässä osassa fokuksen backendiin, eli palvelimella olevaan toiminnallisuuteen.

Backendin toiminnallisuuteen käytämme [Node.js](https://nodejs.org/en/):ää, joka on melkein missä vaan, erityisesti palvelimilla ja omalla koneellasikin toimiva, Googlen [chrome V8](https://developers.google.com/v8/) -javascriptmoottoriin perustuva javascriptin suoritusympäristö.

Kurssimateriaalia tehtäessä on ollut käytössä Node.js:n versio _v8.6.0_. Huolehdi että omasi on vähintää yhtä tuore (ks. komentoriviltä _node -v_).

Kuten [osassa 1](osa1#Javascriptiä) todettiin, selaimet eivät vielä osaa uusimpia javascriptin ominaisuuksia ja siksi selainpuolen koodi täytyy kääntää eli _transpiloida_ esim [babel](https://babeljs.io/):illa. Backendissa tilanne on kuitenkin toinen, uusin node hallitsee riittävissä määrin myös javascriptin uusia versioita (muutamia vielä standardoimattomia ominaisuuksia lukuunottamatta), joten suoritamme nodella suoraan kirjoittamaamme koodia ilman transpilointivaihetta.

Tavoitteenamme on tehdä [osan 2](/osa2) muistiinpano-sovellukseen sopiva backend. Aloitetaan kuitenkin ensin perusteiden läpikäyminen toteuttamalla perinteinen "hello world"-sovellus.

Osassa 2 oli jo puhe [npm](osa2#npm):stä, eli javascript-projektien hallintaan liittyvästä, alunperin node-ekosysteeminstä kotoisin olevasta työkalusta. Mennään sopivaan hakemistoon ja luodaan projektimme runko komennolla _npm init_. Vastaillaan kysymyksiin sopivasti ja tuloksena on hakemiston juureen sijoitettu projektin tietoja kuvaava tiedosto _package.json_

```json
{
  "name": "muistiinpanot-backend",
  "version": "0.0.1",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Matti Luukkainen",
  "license": "MIT"
}
```

Tiedosto määrittelee mm. että ohjelmamme käynnistyspiste on tiedosto _index.js_.

Tehdään kenttään _scripts_ pieni lisäys:

```json
{
  // ...
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ...
}
```

Luodaan sitten sovelluksen ensimmäinen versio, eli projektin juureen sijoitettava tiedosto _index.js_ ja sille seuraava sisältö:

```js
console.log('hello word')
```

Voimme suorittaa ohjelman joko "suoraan" nodella, komentorivillä

```bash
node index.js
```

tai [npm scriptinä](https://docs.npmjs.com/misc/scripts)

```bash
npm start
```

npm-skripti _start_ toimii koska määrittelimme sen tiedostoon _package.json_

```json
{
  // ...
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ...
}
```

Vaikka esim. projektin suorittaminen onnistuukin suoraan käyttämällä komentoa _node index.js_, on npm-projekteille suoritettavat operaatiot yleensä tapana määritellä nimenomaan npm-skripteinä.

Oletusarvoinen _package.json_ määrittelee valmiiksi myös toisen yleisesti käytetyn npm-scriptin eli _npm test_. Koska projektissamme ei ole vielä testikirjastoa, ei _npm test_ kuitenkaan tee vielä muuta kun suorittaa komennon

```bash
echo "Error: no test specified" && exit 1
```

### Yksinkertainen web-palvelin

Muutetaan sovellus web-palvelimeksi:

```js
const http = require('http')

const app = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' })
  res.end('Hello World')
})

const port = 3001
app.listen(port)
console.log(`Server running on port ${port}`)
```

Konsoliin tulostuu

```bash
Server running on port 3001
```

Voimme avata selaimella osoitteessa <http://localhost:3001> olevan vaatimattoman sovelluksemme:

![]({{ "/assets/3/1.png" | absolute_url }})

Palvelin toimii itseasiassa täsmälleen samalla tavalla riippumatta urlin loppuosasta, eli myös sivun <http://localhost:3001/foo/bar> sisältö on sama.

Tarkastallaan koodia hiukan. Ensimmäinen rivi

```js
const http = require('http')
```

ottaa käyttöön noden sisäänrakennetun [web-palvelimen](https://nodejs.org/docs/latest-v8.x/api/http.html) määrittelevän moduulin. Kyse on käytännössä samasta asiasta, mihin olemme selainpuolen koodissa tottuneet hieman syntaksiltaan erilaisessa muodossa:

```js
import http from 'http'
```

Selaimen puolella käytetään (nykyään) ES6:n moduuleita, eli moduulit määritellään [exportilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) ja otetaan käyttöön [importilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import).

Node.js kuitenkin käyttää ns. [CommonJS](https://en.wikipedia.org/wiki/CommonJS)-moduuleja. Syy tälle on siinä, että node-ekosysteemillä oli tarve moduuleihin jo kauan ennen kuin Javascript tuki kielen tasolla moduuleja. Node ei toistaiseksi tue ES-moduuleja, mutta tuki on todennäköisesti jossain vaiheessa [tulossa](https://nodejs.org/api/esm.html).

CommonJS-moduulit toimivat kohtuullisessa määrin samaan tapaan kuin ES6-moduulit, ainakin tämän kurssin tarpeiden puitteissa.

Koodi jatkuu seuraavasti:

```js
const app = http.createServer((request, response) => {
  response.writeHead(200, { 'Content-Type': 'text/plain' })
  response.end('Hello World')
})
```

koodi luo [http](https://nodejs.org/docs/latest-v8.x/api/http.html)-palvelimen metodilla _createServer_ web-palvelimen, jolle se rekisteröi _tapahtumankäsittelijän_, joka suoritetaan _jokaisen_ osoitteen <http:/localhost:3000/> alle tulevan HTTP-pyynnön yhteydessä.

Pyyntöön vastataan statuskoodilla 200, asettamalla _Content-Type_-headerille arvo _text/plain_ ja asettamalla palautettavan sivun sisällöksi merkkijono _Hello World_.

Viimeiset rivit sitovat muuttujaan _app_ sijoitetun http-palvelimen kuuntelemaan porttiin 3001 tulevia HTTP-pyyntöjä:

```js
const PORT = 3001
app.listen(PORT)
console.log(`Server running on port ${PORT}`)
```

Koska tällä kurssilla palvelimen rooli on pääasiassa tarjota frontille JSON-muotoista "raakadataa", muutetaan heti palvelinta siten, että se palauttaa kovakoodatun listallisen JSON-muotoisia muistiinpanoja:

```js
let notes = [
  {
    id: 1,
    content: 'HTML on helppoa',
    date: '2017-12-10T17:30:31.098Z',
    important: true
  },
  {
    id: 2,
    content: 'Selain pystyy suorittamaan vain javascriptiä',
    date: '2017-12-10T18:39:34.091Z',
    important: false
  },
  {
    id: 3,
    content: 'HTTP-protokollan tärkeimmät metodit ovat GET ja POST',
    date: '2017-12-10T19:20:14.298Z',
    important: true
  }
]

const app = http.createServer((request, response) => {
  response.writeHead(200, { 'Content-Type': 'application/json' })
  response.end(JSON.stringify(notes))
})
```
Käynnistetään palvelin uudelleen ja ladataan sivu selaimessa, ja päivitetty sisältö näkyy selaimessa.

Headerin _Content-Type_ arvolla _application/json_ kerrotaan että kyse on JSON-muotoisesta datasta. Taulukko muutetaan jsoniksi metodilla <code>JSON.stringify(notes)</code>.


Kun avaamme selaimen, on tulostusasu sama kuin [osassa 2](osa2/#datan-haku-palvelimelta) käytetyn [json-serverin](https://github.com/typicode/json-server) tarjoamalla muistiinpanojen listalla:

![]({{ "/assets/3/2.png" | absolute_url }})

Voimme jo melkein ruveta käyttämään uutta backendiämme osan 2 muistiinpano-frontendin kanssa. Mutta vain _melkein_, sillä kun käynnistämme frontendin, tulee konsoliin virheilmoitus

![]({{ "/assets/3/3.png" | absolute_url }})

Syy virheelle selviää pian, parantelemme kuitenkin ensin koodia muilta osin.

## Express

Palvelimen koodin tekeminen suoraan noden sisäänrakennetun web-palvelimen [http](https://nodejs.org/docs/latest-v8.x/api/http.html):n päälle on mahdollista, mutta työlästä, erityisesti jos sovellus kasvaa hieman isommaksi.

Nodella tapahtuvaa web-sovellusten ohjelmointia helpottamaan onkin kehitelty useita _http_:tä miellyttävämmän ohjelmoitirajapinnan tarjoamia kirjastoja. Näistä ylivoimaisesti suosituin on [express](http://expressjs.com).

Otetaan express käyttöön määrittelemällä se projektimme riippuvuudeksi komennolla

```bash
npm install express --save
```

Riippuvuus tulee nyt määritellyksi tiedostoon _package.json_:

```json
{
  // ...
  "dependencies": {
    "express": "^4.16.2"
  }
}
```

Riippuvuuden koodi asentuu kaikkien projektin riippuvuuksien tapaan projektin juuressa olevaan hakemistoon _node_modules_. Hakemistosta löytyy expressin lisäksi suuri määrä muutakin tavaraa

<img src="/assets/3/4.png" height="200">

Kyseessä ovat expressin riippuvuudet ja niiden riippuvuudet ym... eli projektimme [transitiiviset riippuvuudet](https://lexi-lambda.github.io/blog/2016/08/24/understanding-the-npm-dependency-model/).

Projektiin asentui expressin versio 4.16.2. Mitä tarkoittaa _package.json:issa_ versiomerkinnän edessä oleva väkänen, eli miksi muoto on

```json
"express": "^4.16.2"
```

npm:n yhteydessä käytetään ns. [semanttista versiointia](https://docs.npmjs.com/getting-started/semantic-versioning).

Merkintä _^4.16.2_ tarkoittaa, että jos/kun projektin riippuvuudet päivitetään, asennetaan expressistä versio, joka on vähintään _4.16.2_, mutta asennetuksi voi tulla versio, jonka _patch_ eli viimeinen numero tai _minor_ eli keskimmäinen numero voi olla suurempi. Pääversio eli _major_ täytyy kuitenkin olla edelleen sama.


Voimme päivittää projektin riippuvuudet komennolla

```bash
npm update
```

Vastaavasti jos aloitamme projektin koodaamisen toisella koneella, saamme haettua ajantasaiset, _package.json_:in määrittelyn kanssa yhteensopivat riippuvuudet komennolla

```bash
npm install
```


Jos riippuvuuden _major_-versionumero ei muutu, uudempien versioiden pitäisi olla [taaksepäin yhteensopivia](https://en.wikipedia.org/wiki/Backward_compatibility), eli jos ohjelmamme käyttäisi tulevaisuudessa esim. expressin versiota 4.99.175, tässä osassa tehtävän koodin pitäisi edelleen toimia ilman muutoksia. Sen sijaan tulevaisuudessa joskus julkaistava express 5.0.0. voi sisältää sellaisia muutoksia, että koodimme ei enää toimisi.

### Web ja express

Palataan taas sovelluksen ääreen ja muutetaan se muotoon:

```js
const express = require('express')
const app = express()

let notes = [
  ...
]

app.get('/', (req, res) => {
  res.send('<h1>Hello World!</h1>')
})

app.get('/notes', (req, res) => {
  res.json(notes)
})

const PORT = 3001
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

Sovellus ei muutu paljoa. Heti alussa otetaan käyttöön _express_, joka on tällä kertaa _funktio_, jota kutsumalla luodaan express-sovellusta vastaava olio:

```js
const express = require('express')
const app = express()
```

Seuraavaksi määritellään sovellukselle kaksi _routea_. Näistä ensimmäinen tapahtumankäsittelijän, joka hoitaa sovelluksen juureen eli polkuun _/_ tulevia HTTP GET -pyyntöjä:

```js
app.get('/', (request, response) => {
  response.send('<h1>Hello World!</h1>')
})
```

Tapahtumankäsittelijäfunktiolla on kaksi parametria. Näistä ensimmäinen eli [request](http://expressjs.com/en/4x/api.html#req) sisältää kaikki HTTP-pyynnön tiedot ja toisen parametrin [response](http://expressjs.com/en/4x/api.html#res):n avulla määritellään, miten pyyntöön vastataan.

Koodissa pyyntöön vastataan käyttäen _response_-olion metodia [send](http://expressjs.com/en/4x/api.html#res.send), jonka kutsumisen seurauksena palvelin vastaa HTTP-pyyntöön lähettämällä vastaukseksi _send_:in parametrina olevan merkkijonon _<h1>Hello World!</h1>_. Koska parametri on merkkijono, tulee vastauksessa _content-type_-headerin arvoksi _text/html_, statuskoodiksi tulee oletusarvoisesti 200.

Routeista toinen määrittelee tapahtumankäsittelijän, joka hoitaa sovelluksen polkuun _notes_ tulevia HTTP GET -pyyntöjä:

```js
app.get('/notes', (request, response) => {
  response.json(notes)
})
```

Pyyntöön vastataan _response_-olion metodilla [json](http://expressjs.com/en/4x/api.html#res.json), joka lähettää HTTP-pyynnön vastaukseksi parametrina olevaa javascript-olioa (eli taulukko _notes_) vastaavan JSON-muotoisen merkkijonon. Headerin _Content-type_ arvoksi tulee _application/json_.

Pieni huomio JSON-muodossa palautettavasta datasta.

Aiemmassa, pelkkää nodea käyttämässämme versiossa, jouduimme muuttamaan palautettavan datan json-muotoon metodilla _JSON.stringify_:

```js
response.end(JSON.stringify(notes))
```

Expressiä käyttässä tämä ei ole tarpeen, sillä muunnos tapahtuu automaattisesti.

Kannattaa huomata, että [JSON](https://en.wikipedia.org/wiki/JSON) on merkkijono, eikä javascript-olio kuten muuttuja _notes_.

Seuraava interaktiivisessa [node-repl](https://nodejs.org/docs/latest-v8.x/api/repl.html):issä suoritettu kokeilu havainnollistaa asiaa:

<img src="/assets/3/5.png" height="200">

Saat käynnistettyä interaktiivisen node-repl:in kirjoittamalla komentoriville _node_. Esim. joidenkin komentojen toimivuuttaa on koodatessa kätevä tarkastaa konsolissa, suosittelen!

## nodemon

Jos muutamme sovelluksen koodia joudumme uudelleenkäynnistämään sovelluksen (eli ensin sammuttamaan konsolista _ctrl_ ja _c_ ja sitten käynnistämään uudelleen), jotta muutokset tulisivat voimaan. Verrattuna Reactin mukavaan workflowhun missä selain päivittyi automaattisesti koodin muuttuessa tuntuu uudelleenkäynnistely kömpelöltä.

Ongelmaan ratkaisu on [nodemon](https://github.com/remy/nodemon):

> nodemon will watch the files in the directory in which nodemon was started, and if any files change, nodemon will automatically restart your node application.

Asennetaan nodemon määrittelemällä se _kehitysaikaiseksi riippuvuudeksi_ (development dependency) komennolla:

```bash
npm install --save-dev nodemon
```

Tiedoston _package.json_ sisältö muuttuu seuraavasti:

```json
{
  //...
  "dependencies": {
    "express": "^4.16.2"
  },
  "devDependencies": {
    "nodemon": "^1.13.3"
  }
}
```

Jos nodemon-riippuvuus kuitenkin meni normaaliin "dependencies"-ryhmään, päivitä package.json manuaalisesti vastaamaan yllä näkyvää (versiot kuitenkin säilyttäen).

Kehitysaikaisilla riippuvuuksilla tarkoitetaan työkaluja, joita tarvitaan ainoastaan sovellusta kehitettäessä, esim. testaukseen tai sovelluksen automaattiseen uudelleenkäynnistykseen kuten _nodemon_.

Kun sovellusta suoritetaan tuotantomoodissa, eli samoin kun sitä tullaan suorittamaan tuotantopalvelimella (esim. Herokussa, mihin tulemme kohta siirtämään sovelluksemme), ei kehitysaikaisia riippuvuuksia tarvita.

Voimme nyt käynnistää ohjelman _nodemon_:illa seuraavasti:

```bash
node_modules/.bin/nodemon index.js
```

Sovelluksen koodin muutokset aiheuttavat nyt automaattisen palvelimen uudelleenkäynnistyksen. Kannattaa huomata, että vaikka palvelin uudelleenkäynnistyy automaattisesti, selain täytyy kuitenkin refreshata, sillä toisin kuin Reactin yhteydessä, meillä ei nyt ole eikä tässä skenaariossa (missä palautamme JSON-muotoista dataa) edes voisikaan olla selainta päivittävää [hot reload](https://gaearon.github.io/react-hot-loader/getstarted/) -toiminnallisuutta.

Komento on ikävä, joten määritellään sitä varten _npm-skripti_ tiedostoon _package.json_:

```json
{
  // ..
  "scripts": {
    "start": "node index.js",
    "watch": "node_modules/.bin/nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  // ..
}
```

Voimme nyt käynnistää palvelimen sovelluskehitysmoodissa komennolla

```bash
npm run watch
```

Toisin kuin skriptejä _start_ tai _test_ suoritettaessa, joudumme sanomaan myös _run_.

## Lisää routeja

Laajennetaan sovellusta siten, että se toteuttaa samanlaisen RESTful-periaatteeseen nojaavan HTTP-rajapinnan kun [json-server](https://github.com/typicode/json-server#routes).

### REST

Representational State Transfer eli REST on Roy Fieldingin vuonna 2000 ilmestyneessä [väitöskirjassa](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) määritelty skaalautuvien web-sovellusten rakentamiseksi tarkoitettu arkkitehtuurityyli.

Emme nyt rupea määrittelemään REST:iä Fieldingiläisittäin tai rupea väittämään mitä REST on tai mitä se ei ole vaan otamme hieman [kapeamman näkökulman](https://en.wikipedia.org/wiki/Representational_state_transfer#Applied_to_Web_services) miten REST tai RESTful API:t yleensä tulkitaan Web-sovelluksissa. Alkuperäinen REST-periaate ei edes sinänsä rajoitu Web-sovelluksiin.

Mainitsimme jo [edellisestä osassa](osa3/#REST-API:n-käyttö), että yksttäisiä asioita, meidän tapauksessamme muistiinpanoja kutsutaan RESTful-ajattelussa _resursseiksi_. Jokaisella resurssilla on yksilöivä osoite.

Erittäin yleinen konventio on muodostaa resurssien yksilöivät urlit liittäen resurssityypin nimi ja resurssin yksilöivä tunniste.

Oletetaan että palvelumme juuriosoite on _www.example.com/api_

Jos nimitämme muistiinpanoja _note_-resursseiksi, yksilöidään yksittäinen muistiinpano, jonka tunniste on 10 urlilla _www.example.com/api/notes/10_.

Kaikkia muistiinpanoja edustavan kokoelmaresurssin url taas on _www.example.com/api/notes_

Resursseille voi suorittaa erilaisia operaatiota. Suoritettavan operaation määrittelee HTTP-operaation tyyppi, jota kutsutaan usein myös _verbiksi_:

| URL | verbi           |  toiminnallisuus |
|------- | --- | --- |
| notes/10 &nbsp;&nbsp;  | GET | hakee yksittäisen resurssin |
| notes    | GET         | hakee kokoelman kaikki resurssit |
| notes    | POST        | luo uuden resurssin pyynnön mukana olevasta datasta |
| notes/10 | DELETE &nbsp;&nbsp;    | poistaa yksilöidyn resurssin |
| notes/10 | PUT         | korvaa yksilöidyn resurssin pyynnön mukana olevalla datalla |
| notes/10 | PATCH       | korvaa yksilöidyn resurssin osan pyynnön mukana olevalla datalla |
|          |  |  |

Näin määrittyy suurin piirtein asia, mitä REST kutsuu nimellä [uniform interface](https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints), eli jossain määrin yhtenäinen tapa määritellä rajapintoja, jotka mahdollistavat (tietyin tarkennuksin) järjestelmien yhteiskäytön.


Tämänkaltaista tapaa tulkita REST:iä on nimitetty kolmiportaisella asteikolla [kypsyystason 2](https://martinfowler.com/articles/richardsonMaturityModel.html) REST:iksi. REST:in kehittäjän Roy Fieldingin mukaan tällöin kyseessä ei vielä ole ollenkaan asia, jota tulisi kutsua [REST-apiksi](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven). Maailman "REST"-apeista valtaosa ei täytäkään puhdasverisen Fieldingiläisen REST-API:n määritelmää.

Jotkut yhteyksissä (ks. esim [Richardsom, Ruby: RESTful Web Services](http://shop.oreilly.com/product/9780596529260.do)) edellä esitellyn kaltaista suoraviivaisehkoa resurssien [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)-tyylisen manipuloinnin mahdollistavaa API:a nimitetään REST:in sijaan [resurssipohjaiseksi](https://en.wikipedia.org/wiki/Resource-oriented_architecture) arkkitehtuurityyliksi. Emme nyt kuitenkaan takerru liian tarkasti määritelmällisiin asioihin vaan jatkamme sovelluksen parissa.

### Yksittäisen resurssin haku

Laajennetaan nyt sovellusta siten, että se tarjoaa muistiinpanojen operointiin REST-rajapinnan. Tehdään ensin [route](http://expressjs.com/en/guide/routing.html) yksittäisen resurssin katsomista varten.

Yksittäisen muistiinpanon identifioi url, joka on muotoa _notes/10_, missä lopussa oleva numero vastaa resurssin muistiinpanon id:tä.

Voimme määritellä expressin routejen poluille [parametreja](http://expressjs.com/en/guide/routing.html) käyttämällä kaksoispistesyntaksia:

```js
app.get('/notes/:id', (request, response) => {
  const id = request.params.id
  const note = notes.find(note => note.id === id )
  response.json(note)
})
```

Nyt <code>app.get('/notes/:id', ...)</code> käsittelee kaikki HTTP GET -pyynnöt, jotka ovat muotoa _note/JOTAIN_, missä _JOTAIN_ on mielivaltainen merkkijono.


Polun parametrin _id_ arvoon päästään käsiksi olion _request_ kautta:

```js
const id = request.params.id
```

Jo tutuksi tulleella taulukon _find_-metodilla haetaan taulukosta parametria vastaava muistiinpano ja palautetaan se pyynnön tekijälle.

Kun sovellusta testataan menemällä selaimella osoitteeseen <http://localhost:3001/notes/1>, havaitaan että se ei toimi. Tämä on tietenkin softadevaajan arkipäivää, ja on ruvettava debuggaamaan.

Vanha hyvä keino on alkaa lisäillä koodiin _console.log_-komentoja:

```js
app.get('/notes/:id', (request, response) => {
  const id = request.params.id
  console.log(id)
  const note = notes.find(note => note.id === id)
  console.log(note)
  response.json(note)
})
```

Konsoliin tulostuu

<pre>
1
undefined
</pre>

eli halutun muistiinpanon id välittyy sovellukseen aivan oikein, mutta _find_ komento ei löydä mitään.

Päätetään tulostella konsoliin myös _find_-komennon sisällä olevasta vertailijafunktiosta, joka onnistuu helposti kun tiiviissä muodossa oleva funktio <code>note => note.id === id</code> kirjoitetaan eksplisiittisen returnin sisältävässä muodossa:

```js
app.get('/notes/:id', (request, response) => {
  const id = request.params.id
  const note = notes.find(note => {
    console.log(note.id, typeof note.id, id, typeof id, note.id === id)
    return note.id === id
  })
  console.log(note)
  response.json(note)
})
```

Jokaisesta vertailufunktion kutsusta tulostetaan nyt monta asiaa. Konsolin tulostus on seuraava:

<pre>
1 'number' '1' 'string' false
2 'number' '1' 'string' false
3 'number' '1' 'string' false
</pre>

ongelman syy selviää: muuttujassa _id_ on tallennettuna merkkijono '1' kun taas muistiinpanojen id:t ovat numeroita. Javascriptissä === vertailu katsoo kaikki eri tyyppiset arvot oletusarvoisesti erisuuriksi, joten 1 ei ole '1'.

Korjataan ongelma, muuttamalla parametrina oleva merkkijonomuotoinen id [numeroksi](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number):

```js
app.get('/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  const note = notes.find(note => note.id === id)
  response.json(note)
})
```

ja nyt yksittäisen resurssin hakeminen toimii

<img src="/assets/3/6.png" height="200">

toiminnallisuuteen jää kuitenkin pieni ongelma.

Jos haemme muistiinpanoa sellaisella indeksillä, mitä vastaavaa muistiinpanoa ei ole olemassa, vastaa palvelin seuraavasti

![]({{ "/assets/3/7.png" | absolute_url }})

HTTP-statuskoodi on onnistumisesta kertova 200. Vastaukseen ei liity dataa, sillä headerin _content-length_ arvo on 0, ja samaa todistaa selain: mitään ei näy.

Syynä tälle käyttäytymiselle on se, että muuttujan _note_ arvoksi tulee _undefined_ jos muistiinpanoa ei löydy. Tilanne tulisi käsitellä palvelimella järkevämmin, eli statuskoodin 200 sijaan tulee vastata statuskoodilla [404 not found](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.5).

Tehdään koodiin muutos

```js
app.get('/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  const note = notes.find(note => note.id === id)

  if ( note ) {
    response.json(note)
  } else {
    response.status(404).end()
  }
})
```

Koska vastaukseen ei nyt liity mitään dataa käytetään statuskoodin asettavan metodin [status](http://expressjs.com/en/4x/api.html#res.status) lisäksi metodia [end](http://expressjs.com/en/4x/api.html#res.end) ilmoittamaan siitä, että pyyntöön tulee vastata ilman dataa.

Koodin haarautumisessa hyväksikäytetään sitä, että mikä tahansa Javascript-olio on [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy), eli katsotaan todeksi vertailuoperaatiossa. undefined taas on [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) eli epätosi.

Nyt sovellus toimii, eli palauttaa oikean virhekoodin. Sovellus ei kuitenkaan palauta mitään käyttäjälle näytettävää kuten web-sovellukset yleensä tekevät jos mennään osoitteeseen jota ei ole olemassa. Emme kuitenkaan tarvitse nyt mitään näytettävää, sillä REST API:t ovat ohjelmalliseen käyttöön tarkoitettuja rajapintoja ja pyyntöön liitetty virheestä kertova statuskoodi on riittävä.

### Resurssin poisto

Toteutetaan seuraavaksi resurssin poistava route. Poisto tapahtuu tekemällä HTTP DELETE -pyyntö resurssin urliin:


```js
app.delete('/notes/:id', (request, response) => {
  const id = Number(request.params.id)
  notes = notes.filter(note => note.id !== id)

  response.status(204).end()
})
```

Jos poisto onnistuu, eli poistettava muistiinpano on olemassa, vastataan statuskoodilla [204 no content](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.2.5) sillä mukaan ei lähetetä mitään dataa.

Ei ole täyttä yksimielisyyttä siitä mikä statuskoodi DELETE-pyynnöstä pitäisi palauttaa jos poistettavaa resurssia ei ole olemassa. Vaihtoehtoja ovat lähinnä 204 ja 404. Yksinkertaisuuden vuoksi sovellus palauttaa nyt molemmissa tilanteissa statuskoodin 204.

### Postman

Herää kysymys miten voimme testata poisto-operaatiota? HTTP GET -pyyntöjä on helppo testata selaimessa. Voisimme toki kirjoittaa javascript-koodin, joka testaa deletointia, mutta jokaiseen mahdolliseen tilanteeseen testikoodinkaan tekeminen ei ole aina paras ratkaisu.

On olemassa useita backendin testaamista helpottavia työkaluja, eräs näistä on edellisessä osassa nopeasti mainittu komentorivityökalu [curl](https://curl.haxx.se).

Käytetään nyt kuitenkin [postman](https://www.getpostman.com/)-nimistä sovellusta.

**HUOM:** jos käytät Visual Studio Codea, voit postmanin sijaan käyttää VS Coden
[REST client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) -pluginia.

Asennetaan postman ja kokeillaan

![]({{ "/assets/3/8.png" | absolute_url }})

Postmanin käyttö on tässä tilanteessa suhteellisen yksinkertaista, riittää määritellä url ja valita oikea pyyntötyyppi.

Palvelin näyttää vastaavan oikein. Tekemällä HTTP GET osoitteeseen _http://localhost:3001/notes_ selviää että poisto-operaatio oli onnistunut, muistiinpanoa, jonka id on 2 ei ole enää listalla.

Koska muistiinpanot on talletettu palvelimen muistiin, uudelleenkäynnistys palauttaa tilanteen ennalleen.

### Datan vastaanottaminen

Toteutetaan seuraavana uusien muistiinpanojen lisäys, joka siis tapahtuu tekemällä HTTP POST -pyyntö osoitteeseen _http://localhost:3001/notes_ ja liittämällä pyynnön mukaan eli [bodyyn](https://www.w3.org/Protocols/rfc2616/rfc2616-sec7.html#sec7) luotavan muistiinpanon tiedot JSON-muodossa.

Jotta pääsisimme pyynnön mukana lähetettyyn dataan helposti käsiksi tarvitsemme [body-parser](https://github.com/expressjs/body-parser)-kirjaston apua. Määritellään kirjasto projektin riippuvuudeksi

```bash
npm install body-parser --save
```

Otetaan sitten body-parser käyttöön ja luodaan alustava määrittely HTTP POST -pyynnön käsittelyyn

```js
const express = require('express')
const app = express()
const bodyParser = require('body-parser')

app.use(bodyParser.json())

//...

app.post('/notes', (request, response) => {
  const note = request.body
  console.log(note)

  response.json(note)
})
```

Tapahtumankäsittelijäfunktio pääsee dataan käsiksi viitaamalla _request.body_.

Ilman body-parser-kirjaston määrittelyä pyynnön kentän _body_ arvo olisi ollut määrittelemätön. body-parserin toimintaperiaatteena on, että se ottaa pyynnön mukana olevan JSON-muotoisen datan, muuttaa sen Javascript-olioksi ja sijoittaa _request_-olion kenttään _body_ ennen kuin routen käsittelijää kutsutaan.

Toistaiseksi sovellus ei vielä tee vastaanotetulle datalle, mitään muuta kuin tulostaa sen konsoliin ja palauttaa sen pyynnön vastauksessa.

Ennen toimintalogiikan viimeistelyä varmistetaan ensin postmanilla että, lähetetty tieto menee varmasti perille. Pyyntötyypin ja urlin lisäksi on määriteltävä myös pyynnön mukana menevä data eli _body_:


![]({{ "/assets/3/9.png" | absolute_url }})

Näyttää kuitenkin siltä, että mitään ei mene perille, palvelin vastaanottaa ainoastaan tyhjän olion. Missä on vika? Olemme unohtaneet määritellä headerille _Content-Type_ oikean arvon:

<img src="/assets/3/10.png" height="200">

Nyt kaikki toimii! Ilman oikeaa headerin arvoa palvelin ei osaa parsia dataa oikeaan muotoon. Se ei edes yritä arvailla missä muodossa data on, sillä potentiaalisia datan siirtomuotoja eli _Content-Typejä_ on olemassa [suuri määrä](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types).

> **sivuhuomio**
>
> Välillä debugatessa tulee vastaan tilanteita, joissa backendissä on tarve selvittää mitä headereja HTTP-pyynnöille on asetettu. Eräs menetelmä tähän on _request_-olion melko kehnosti nimetty metodi [get](http://expressjs.com/en/4x/api.html#req.get), jonka avulla voi selvittää yksittäisen headerin arvon. _request_-oliolla on myös kenttä _headers_, jonka arvona ovat kaikki pyyntöön liittyvät headerit.

Palataan taas sovelluksen pariin. Kun tiedämme, että sovellus vastaanottaa tiedon oikein, voimme viimeistellä sovelluslogiikan:

```js
app.post('/notes', (request, response) => {
  const maxId = notes.length > 0 ? notes.map(n => n.id).sort().reverse()[0] : 0
  const note = request.body
  note.id = maxId + 1

  notes = notes.concat(note)

  response.json(note)
})
```

Uudelle muistiinpanolle tarvitaan uniikki id. Ensin selvitetään olemassaolevista id:istä suurin muuttujaan _maxId_. Uuden muistiinpanon id:ksi asetetaan sitten _maxId+1_. Tämä tapa ei ole itseasiassa kovin hyvä, mutta emme nyt välitä siitä sillä tulemme pian korvaamaan tavan miten muistiinpanot talletetaan.

Tämän hetkisessä versiossa on vielä se ongelma, että voimme HTTP POST -pyynnöllä lisätä mitä tahansa kenttiä sisältäviä olioita. Parannellaan sovellusta siten, että kenttä _content_ vaaditaan. Kentille _important_ ja _date_ asetetaan oletusarvot. Kaikki muut kentät hylätään:

```js
const generateId = () => {
  const maxId = notes.length > 0 ? notes.map(n => n.id).sort().reverse()[0] : 1
  return maxId + 1
}

app.post('/notes', (request, response) => {
  const body = request.body

  if (body.content === undefined) {
    response.status(400).json({error: 'content missing'})
  }

  const note = {
    content: body.content,
    important: body.important|| false,
    date: body.date || new Date(),
    id: generateId()
  }

  notes = notes.concat(note)

  response.json(note)
})
```

Tunnisteena toimivan id-kentän arvon generointilogiikka on eriytetty funktioon _generateId_.

Jos kenttä _content_ puuttuu, vastataan statuskoodilla [400 bad request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.1). Muussa tapauksessa luodaan muistiinpano syötteen perusteella. Jos kenttä _important_ tai _date_ puuttuvat, generoidaan niille oletusarvo.

Kenttien _important_ ja _date_ oletusarvot generoidaan nyt hieman erikoisella tavalla. date:n arvoksi asetetaan

```js
body.date || new Date()
```

jos sovelluksen vastaanottamassa muuttujaan _body_ talletetussa datassa on kenttä _data_, tulee lausekkeelle sen arvo. Jos kenttää ei ole olemassa, tulee lausekkeen arvoksi oikeanpuoleinen osa eli _new Date()_.

## Tehtäviä

Tee nyt tehtävät [40-45](../tehtavat#expressin-alkeet)

## Huomioita HTTP pyyntötyyppien käytöstä

[HTTP-standardi](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) puhuu pyyntötyyppien yhteydessä kahdesta ominaisuudesta, **safe** ja **idemponent**.

HTTP-pyynnöistä GET:in tulisi olla _safe_:

> In particular, the convention has been established that the GET and HEAD methods SHOULD NOT have the significance of taking an action other than retrieval. These methods ought to be considered "safe".

Safety siis tarkoittaa, että pyynnön suorittaminen ei saa aiheutta palvelimelle _sivuvaikutuksia_ eli esim. muuttaa palvelimen tietokannan tilaa, pyynnön tulee ainoastaan palauttaa palvelimella olevaa dataa.

Mikään ei automaattisesti takaa, että GET-pyynnöt olisivat luonteeltaan _safe_, kyseessä onkin HTTP-standardin suositus palvelimien toteuttajille. RESTful-periaatetta noudattaessa GET-pyyntöjä käytetäänkin aina siten, että ne ovat safe.

HTTP-standardi määrittelee myös pyyntötyypin [HEAD](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.4), jonka tulee olla safe. Käytännössä HEAD:in tulee toimia kuten GET, mutta se ei palauta vastauksenaan muuta kuin statuskoodin ja headerit, viestin bodyä HEAD ei palauta ollenkaan.

HTTP-pyynnöistä muiden paitsi POST:in tulisi olla _idemponent_:

> Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request. The methods GET, HEAD, PUT and DELETE share this property

Eli jos pyynnöllä on sivuvaikutuksia, lopputulos on sama suoritetaanko pyyntö yhden tai useamman kerran.

Esim. jos tehdään HTTP PUT pyyntö osoitteeseen _/notes/10_ ja pyynnön mukana on <code>{ content: "ei sivuvaikutuksia", important: true }</code>, on lopputulos sama riippumatta siitä kuinka monta kertaa pyyntö suoritetaan.

Kuten metodin GET _safety_ myös _idempotence_ on HTTP-standardin suositus palvelimien toteuttajille. RESTful-periaatetta noudattaessa GET, HEAD, PUT ja DELETE-pyyntöjä käytetäänkin aina siten, että ne ovat idemponent.

HTTP pyyntötyypeistä POST on ainoa joka ei ole safe eikä idemponent. Jos tehdään 5 kertaa HTTP POST -pyyntö osoitteeseen _/notes_ siten että pyynnön mukana on <code>{ content: "monta samaa", important: true }</code>, tulee palvelimelle 5 saman sisältöistä muistiinpanoa.

## Middlewaret

Äsken käyttöönottamamme [body-parser](https://github.com/expressjs/body-parser) on terminologiassa niin sanottu [middleware](http://expressjs.com/en/guide/using-middleware.html).

Middlewaret ovat funktioita, joiden avulla voidaan käsitellä _request_- ja _response_-olioita.

Esim. body-parser ottaa pyynnön mukana tulevan raakadatan _request_-oliosta, parsii sen Javascript-olioksi ja sijoittaa olion _request_:in kenttään _body_

Middlewareja voi olla käytössä useita jolloin ne suoritetaan peräkkäin siinä järjestyksessä kun ne on määritelty.

Toteutetaan itse yksinkertainen middleware, joka tulostaa konsoliin palvelimelle tulevien pyyntöjen perustietoja.

Middleware on funktio, joka saa kolme parametria:

```js
const logger = (request, response, next) => {
  console.log('Method:',request.method)
  console.log('Path:  ', request.path)
  console.log('Body:  ', request.body)
  console.log('---')
  next()
}
```

Middleware kutsuu lopussa parametrina olevaa funktiota _next_, jolla se siirtää kontrollin seuraavalle middlewarelle.

Middleware otetaan käyttöön seuraavasti:

```js
app.use(logger)
```

Middlewaret suoritetaan siinä järjestyksessä, jossa ne on otettu käyttään metodilla sovellusolion _use_. Middlewaret tulee myös määritellä ennen routeja jos ne halutaan suorittaa ennen niitä. On myös eräitä tapauksia, joissa middleware tulee määritellä vasta routejen jälkeen, käytännössä tällöin on kyse middlewareista, joita suoritetaan vain, jos mikään route ei käsittele HTTP-pyyntöä.

Lisätään routejen jälkeen seuraava middleware, jonka ansiosta saadaan routejen käsittelemättömistä virhetilanteista JSON-muotoinen virheilmoitus:

```js
const error = (request, response) => {
  response.status(404).send({error: 'unknown endpoint'})
}

app.use(error)
```

## Tehtäviä

Tee nyt tehtävät [46 ja 47](../tehtavat#lisää-middlewareja)

## Yhteys frontendiin

Palataan yritykseemme käyttää nyt tehtyä backendiä [osassa 2](/osa2) tehdyllä React-frontendillä. Aiempi yritys lopahti seuraavaan virheilmoitukseen

![]({{ "/assets/3/3.png" | absolute_url }})

Frontendin tekemä GET-pyyntö osoitteeseen <http://localhost:3001/notes> ei jostain syystä toimi. Mistä on kyse? Backend toimii kuitenkin selaimesta ja postmanista käytettäessä ilman ongelmaa.

### Same origin policy ja CORS

Kyse on asiasta nimeltään CORS eli Cross-origin resource sharing. [Wikipedian](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) sanoin

> Cross-origin resource sharing (CORS) is a mechanism that allows restricted resources (e.g. fonts) on a web page to be requested from another domain outside the domain from which the first resource was served. A web page may freely embed cross-origin images, stylesheets, scripts, iframes, and videos. Certain "cross-domain" requests, notably Ajax requests, are forbidden by default by the same-origin security policy.

Lyhyesti sanottuna meidän kontekstissa kyse on seuraavasta: sivulla oleva javascript-koodi saa oletusarvoisesti kommunikoida vain samassa [originissa](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) olevan palvelimen kanssa. Koska palvelin on localhostin portissa 3001 ja frontend localhostin portissa 3000, niiden origin ei ole sama.


Korostetaan vielä, että [same origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) ja CORS eivät ole mitenkään React- tai node-spesifisiä asioita, vaan yleismaailmallisia periaatteita Web-sovellusten toiminnasta.

Voimme sallia muista _origineista_ tulevat pyynnöt käyttämällä noden [cors](https://github.com/expressjs/cors)-middlewarea.

Asennetaan _cors_ komennolla

```bash
npm install cors --save
```

Otetaan middleware käyttöön ja sallitaan kaikki origineista tulevat pyynnöt:

```js
const cors = require('cors')

app.use(cors())
```

Nyt frontend toimii! Tosin muistiinpanojen tärkeäksi muuttavaa toiminnallisuutta backendissa ei vielä ole.

CORS:ista voi lukea tarkemmin esim. [Mozillan sivuilta](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

## Sovellus internettiin

Kun koko "stäkki" on saatu vihdoin kuntoon, siirretään sovellus internettiin. Viime aikoina on tullut uusia mielenkiintoisa sovellusten hostausmahdollisuuksia, esim. [Zeit](https://zeit.co). Käytetään seuraavassa vanhaa kunnon [Herokua](https://www.heroku.com).

Lisätään projektin juureen tiedosto _Procfile_, joka kertoo herokulle, miten sovellus käynnistetään

```bash
web: node index.js
```

Muutetaan tiedoston _index.js_ lopussa olevaa sovelluksen käyttämän portin määrittelyä seuraavasti:

```js
const PORT = process.env.PORT || 3001
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`)
})
```

Nyt käyttöön tulee [ympäristömuuttujassa](https://en.wikipedia.org/wiki/Environment_variable) _PORT_ määritelty portti tai 3001 jos ympäristömuuttuja _PORT_ ei ole määritelty. Heroku konfiguroi sovelluksen portin ympäristömuuttujan avulla.

Tehdään projektihakemistosta git-repositorio, lisätään _.gitignore_ jolla seuraava sisältö

```bash
node_modules
```

Luodaan heroku-sovellus komennolla _heroku create_ ja deployataan sovellus komennoilla _git add -A_, _git commit -m \"Initiate app.\"_ ja _git push heroku master_.

Jos kaikki meni hyvin, sovellus toimii. Jos ei, vikaa voi selvittää herokun lokeja lukemalla, eli komennolla _heroku logs_.

Esim. tätä materiaalia tehdessä törmättiin ongelmaan joka aiheutti seuraavan tulostuksen lokeihin

![]({{ "/assets/3/11.png" | absolute_url }})

Syynä ongelmalle oli se, että middlewarea _cors_ asennettaessa oli unohtunut antaa optio __--save__, joka tallentaa tiedon riippuvuudesta tiedostoon _package.json_. Koska näin kävi, ei Heroku ollut asentanut corsia sovelluksen käyttöön.

Myös frontend toimii herokussa olevan backendin avulla. Voit varmistaa asian muuttamalla frontendiin määritellyn backendin osoitteen viittaamaan _localhost:3001_:n sijaan herokussa olevaan backendiin.

Seuraavaksi herää kysymys miten saamme myös frontendin internettiin? Vaihtoehtoja on useita.

### Frontendin tuotantoversio

Olemme toistaiseksi suorittaneet React-koodia _sovelluskehitysmoodissa_, missä sovellus on konfiguroitu antamaan havainnollisia virheilmoituksia, päivittämään koodiin tehdyt muutokset automaattisesti selaimeen ym.

Kun sovellus viedään tuotantoon, täytyy siitä tehdä [production build](https://reactjs.org/docs/optimizing-performance.html#use-the-production-build)
eli tuotantoa varten optimoitu versio.

create-react-app:in avulla tehdyistä sovelluksista saadaan muodostettua tuotantoversio komennolla [npm run build](https://github.com/facebookincubator/create-react-app#npm-run-build-or-yarn-build).

Komennon seurauksena syntyy hakemiston _build_ (joka sisältää jo sovelluksen ainoan html-tiedoston _index.html_) sisään hakemisto _static_, minkä alle generoituu sovelluksen javascript-koodin [minifioitu](https://en.wikipedia.org/wiki/Minification_(programming)) versio. Vaikka sovelluksen koodi on kirjoitettu useaan tiedostoon, generoituu kaikki javascript yhteen tiedostoon, samaan tiedostoon tulee itseasiassa myös kaikkien sovelluksen koodin tarvitsemien riippuvuuksien koodi.

Minifioitu koodi ei ole miellyttävää luettavaa. Koodin alku näyttää seuraavalta:

```js
!function(e){function t(r){if(n[r])return n[r].exports;var o=n[r]={i:r,l:!1,exports:{}};return e[r].call(o.exports,o,o.exports,t),o.l=!0,o.exports}var n={};t.m=e,t.c=n,t.d=function(e,n,r){t.o(e,n)||Object.defineProperty(e,n,{configurable:!1,enumerable:!0,get:r})},t.n=function(e){var n=e&&e.__esModule?function(){return e.default}:function(){return e};return t.d(n,"a",n),n},t.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},t.p="/",t(t.s=12)}([function(e,t,n){"use strict";function r(e){return"[object Array]"===E.call(e)}function o(e){return"[object ArrayBuffer]"===E.call(e)}function a(e){return"undefined"!==typeof FormData&&e instanceof FormData}function i(e){return"undefined"!==typeof ArrayBuffer&&ArrayBuffer.isView?ArrayBuffer.isView(e):e&&e.buffer&&e.buffer instanceof ArrayBuffer}function u(e){return"string"===typeof e}function l(e){return"number"===typeof e}function s(e){return"undefined"===typeof e}function c(e){return null!==e&&"object"===typeof
```

### Staattisten tiedostojen tarjoaminen backendistä

Eräs mahdollisuus frontendin tuotantoon viemiseen on kopioida tuotantokoodi, eli hakemisto _build_ backendin repositorion juureen ja määritellä backend näyttämään pääsivunaan frontendin _pääsivu_, eli tiedosto _build/index.html_.

Jotta saamme expressin näyttämään _staattista sisältöä_ eli sivun _index.html_ ja sen lataaman javascriptin ym. tarvitsemme expressiin sisäänrakennettua middlewarea [static](http://expressjs.com/en/starter/static-files.html).

Kun lisäämme muiden middlewarejen määrittelyn yhteyteen seuraavan

```js
app.use(express.static('build'))
```

tarkastaa express GET-tyyppisten HTTP-pyyntöjen yhteydessä ensin löytyykö pyynnön polkua vastaavan nimistä tiedostoa hakemistosta _build_. Jos löytyy, palauttaa express tiedoston.

Nyt HTTP GET -pyyntö osoitteeseen _www.palvelimenosoite.com/index.html_ tai _www.palvelimenosoite.com_ näyttää Reactilla tehdyn frontendin. GET-pyynnön esim. osoitteeseen _www.palvelimenosoite.com/notes_ hoitaa backendin koodi.

Koska tässä tapauksessa sekä frontend että backend toimivat samassa osoitteessa, voidaan React-sovelluksessa tapahtuva backendin _baseUrl_ määritellä [suhtellisena](https://www.w3.org/TR/WD-html40-970917/htmlweb.html#h-5.1.2) URL:ina, eli ilman palvelinta yksilöivää osaa:

```js
import axios from 'axios'
const baseUrl = '/notes'

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

// ...
```

Muutoksen jälkeen on luotava uusi production build ja kopioitava se backendin repositorioin juureen.

Kun sovellus pushataan uudelleen herokuun, [se](https://radiant-plateau-25399.herokuapp.com) toimii moitteettomasti lukuunottamatta vielä backendiin toteuttamatonta muistiinpanon tärkeyden muuttamista.

Sovelluksemme tallettama tieto ei ole ikuisesti pysyvää, sillä sovellus tallettaa muistiinpanot muuttujaan. Jos sovellus kaatuu tai se uudelleenkäynnistetään, kaikki tiedot katoavat.

Tarvitsemme sovelluksellemme tietokannan. Ennen tietokannan käyttöönottoa katsotaan kuitenkin vielä muutamaa asiaa.

### Backendin urlit

Backendin tarjoama muistiinpanojen käsittelyn rajapinta on nyt suoraan sovelluksen URL:in <https://radiant-plateau-25399.herokuapp.com> alla. Eli <https://radiant-plateau-25399.herokuapp.com/notes> on kaikkien mustiinpanojen lista ym. Koska backendin roolina on tarjota frontendille koneluettava rajapinta, eli API, olisi ehkä parempi erottaa API:n tarjoama osoitteisto selkeämmin, esim. aloittamalla kaikki sanalla _api_.

Tehdään muutos ensin muuttamalla käsin kaikki backendin routet:

```js
//...
app.get('/api/notes', (request, response) => {
  response.json(notes)
})
//...
```

Frontendin koodiin riittää seuraava muutos

```js
import axios from 'axios'
const baseUrl = '/api/notes'

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

// ...
```

> Sivuhuomautus: API:en versiointi
>
>Joskus API:n urleissa ilmaistaan myös API:n versio. Eri versioita saatetaan tarvita, jos aikojen kuluessa API:in tehdään laajennuksia, jotka ilman versiointia hajoittaisivat olemassaolevia osia ohjelmista. Versioinnin avulla voidaan tuoda vanhojen rinnalle uusia, hieman eri tavalla voimivia versioita API:sta.
>
>API:n version ilmaiseminen URL:issa ei kuitenkaan ole välttämättä, ainakaan kaikkien mielstä järkevää vaikka tapaa paljon käytetäänkin. Oikeasta tavasta API:n versiointiin [kiistellään ympäri internettiä](https://stackoverflow.com/questions/389169/best-practices-for-api-versioning).

### Proxy

Frontendiin tehtyjen muutosten seurauksena on nyt se, että kun suoritamme frontendiä sovelluskehitysmoodissa, eli käynnistämällä sen komennolla _npm start_, yhteys backendiin ei toimi.

Syynä tälle on se, että backendin osoite muutettiin suhteellisesti määritellyksi:

```js
const baseUrl = '/api/notes'
```

Koska frontend toimii osoitteessa _localhost:3000_, menevät backendiin tehtävät pyynnöt väärään osoitteeseen _localhost:3000/api/notes_. Backend toimii kuitenkin osoitteessa _localhost:3001_

create-react-app:illa luoduissa projekteissa ongelma on helppo ratkaista. Riittää, että tiedostoon _package.json_ lisätään seuraava määritelmä:

```bash
{
  "scripts": {
    // ...
    "proxy": "http://localhost:3001
  }
}
```

Nyt Reactin sovelluskehitysympäristö toimii [proxynä](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#proxying-api-requests-in-development) ja jos React-koodi tekee HTTP-pyynnön palvelimen _http://localhost:3000_ johonkin osoitteeseen joka ei ole React-sovelluksen vastuulla (eli kyse ei ole esim. sovelluksen javascript-koodista tai CSS:stä), lähetetään pyyntö edelleen osoitteessa _http://localhost:3001_ olevalle palvelimelle.

Nyt myös frontend on kunnossa, se toimii sekä sovelluskehitysmoodissa että tuotannossa yhdessä palvelimen kanssa.

Eräs negatiivinen puoli käyttämässämme lähestymistavassa on se, että sovelluksen uuden version tuotantoon vieminen edellyttää ikävän manuaalisen askeleen: frontendin koodin tuotantoversion kopioimisen backendin repositorioon. Tämä taas hankaloittaa automatisoidun [deployment pipelinen](https://martinfowler.com/bliki/DeploymentPipeline.html) toteuttamista. Deployment pipelinellä tarkoitetaan automatisoitua ja hallittua tapaa viedä koodi sovelluskehittäjän koneelta erilaisten testien ja laadunhallinnallisten vaiheiden kautta tuotantoympäristöön.

Tähänkin on useita erilaisia ratkaisuja (esim. sekä frontendin että backendin [sijoittaminen samaan reporitorioon](https://github.com/mars/heroku-cra-node)), emme kuitenkaan nyt mene niihin.

Myös frontendin koodin deployaaminen omana sovelluksenaan voi joissain tilanteissa olla järkevää. create-react-app:in avulla luotujen sovellusten osalta se on [suoraviivaista](https://github.com/mars/create-react-app-buildpack).


## Tehtäviä

Tee nyt tehtävät [48-50](../tehtavat#yhteys-frontendiin-ja-vienti-tuotantoon)

## Node-sovellusten debuggaaminen

Node-sovellusten debuggaaminen on jossain määrin hankalampaa kuin selaimessa toimivan javascriptin.

Vanha hyvä keino on tietysti konsoliin tulostelu. Se kannattaa aina.

### Visual Studio Code

Visual Studio Coden debuggeri voi olla hyödyksi joissain tapauksissa. Seuraavassa screenshot, joka tosin on otettu koodin tämän osan lopun versiosta:

![]({{ "/assets/3/17.png" | absolute_url }})

Koodiin on asetettu muutama breakpoint ja konsolissa on evaluoitu muuttujan _request.params_ arvo. Vasemmalla olevassa ikkunassa on nähtävillä myös muuta ohjelman tilaan liittyvää.

Ylhäällä olevista nuolista yms. voidaan kontrolloida debuggauksen etenemistä.

Itse en juurikaan käytä Visual Studio Code debuggeria.

### Chromen dev tools

Debuggaus onnisuu myös Chromen developer-konsolilla, käynnistämällä sovellus komennolla:

```bash
node --inspect index.js
```

Debuggeriin pääsee käsiksi kirjoittamalla chromen osoiteriville

```bash
chrome://inspect
```

Avautuvasta näkymästä valitaan debugattava sovellus:

![]({{ "/assets/3/18.png" | absolute_url }})

Debuggausnäkymä toimii kuten React-koodia debugattaessa, _source_-välilehdelle voidaan esim. asettaa breakpointeja, eli kohtia joihin suoritus pysähtyy:

![]({{ "/assets/3/19.png" | absolute_url }})


Kaikki sovelluksen console.log-tulostukset tulevat debuggerin _Console_-välilehdelle. Voit myös tutkia siellä muuttujien arvoja ja suorittaa mielivaltaista javascript-koodia:

![]({{ "/assets/3/20.png" | absolute_url }})

### Epäile kaikkea

Full Stack -sovellusten debuggaaminen vaikuttaa alussa erittäin hankalalta. Kun kohta kuvaan tulee myös tietokanta ja frontend on yhdistetty backendiin, on potentiaalisia virheenlähteitä todella paljon.

Kun sovellus "ei toimi", onkin selvitettävä missä vika on. On erittäin yleistä, että vika on sellaisessa paikassa, mitä ei osaa ollenkaan epäillä, ja menee minuutti-, tunti- tai jopa päiväkausia ennen kuin oikea ongelmien lähde löytyy.

Avainasemassa onkin systemaattisuus. Koska virhe voi olla melkein missä vaan, kaikkea pitää epäillä, ja tulee pyrkiä poissulkemaan ne osat tarkastelusta, missä virhe ei ainakaan ole. Konsoliin kirjoitus, Postman, debuggeri ja kokemus auttavat.

Virheiden ilmaantuessa huonoin strategia on jatkaa koodin kirjoittamista. Se on tae siitä, että koodissa on pian kymmenen ongelmaa lisää ja niiden syyn selvittäminen on entistäkin vaikeampaa. Toyota Production Systemin periaate [Stop and fix](http://gettingtolean.com/toyota-principle-5-build-culture-stopping-fix/#.Wjv9axP1WCQ) toimii tässäkin yhteydessä paremmin kuin hyvin.

## Mongo

Jotta saisimme talletettua muistiinpanot pysyvästi, tarvitsemme tietokannan. Useimmilla laitoksen kursseilla on käytetty relaatiotietokantoja. Tällä kurssilla käytämme [MongoDB](https://www.mongodb.com/):tä, joka on ns. [dokumenttitietokanta](https://en.wikipedia.org/wiki/Document-oriented_database).

Dokumenttitietokannat poikkeavat jossain määrin relaatiotietokannoista niin datan organisointitapansa kuin kyselykielensäkin suhteen. Dokumenttitietokantojen ajatellaan kuuluvan sateenvarjotermin [NoSQL](https://en.wikipedia.org/wiki/NoSQL) alle. Lisää dokumenttitietokannoista ja NoSQL:stä Tietokantojen perusteiden [viikon 7 materiaalista](https://materiaalit.github.io/tikape-s17/part7/).

**Lue nyt Tietokantojen perusteiden dokumenttitietokantoja kuvaava osuus.** Jatkossa oletetaan, että hallitset käsitteet _dokumentti_ ja _kokoelma_ (collection).


MongoDB:n voi luonnollisesti asentaa omalle koneelle. Internetistä löytyy kuitenin myös palveluna toimivia Mongoja (esim [mlab](https://mlab.com/) ja [MongoDbCloud](https://www.mongodb.com/cloud/atlas)), ja koska Herokussa oleville sovelluksille on suhteellisen suoraviivaista konfiguroida Mongo-tietokanta, seuraavissa esimerkeissä käytetään Herokun kautta käyttöönotettavaa Mongoa.

Käyttöönotto tapahtuu sovelluksen repositoriossa komennolla

```bash
heroku addons:create mongolab:sandbox
```

** HUOM** vaikka Heroku tarjoaa ilmaisen Mongon, joudut ehkä antamaan Herokulle luottokorttinumerosi, että saat Mongon asennettua. Voit kiertää tämän tarpeen stackoverflow:sta löytyvää [ohjetta](https://stackoverflow.com/questions/36321385/deploy-nodejs-mongodb-on-heroku-but-need-verify-credit-card-when-install-add-o) seuraten.

Kuten komennon tuloste kertoo, kysessä on [mlab](https://mlab.com/):n tarjoama Mongo:

![]({{ "/assets/3/12.png" | absolute_url }})

Pääset Herokusta sovelluksesi sivun kautta mlabin mongo-hallintanäkymään.

Tietokannan osoite, eli _MONGODB_URI_ selviää komentoriviltä komennolla _heroku config_.

Mongon käyttäminen javascript-koodista suoraan [MongoDB Node.js driver](https://mongodb.github.io/node-mongodb-native/) -kirjaston avulla on varsin työlästä. Käytämmekin [mongoose](http://mongoosejs.com/index.html)-kirjastoa.

Mongoosesta voisi käyttää luonnehdintaa _object document mapper_ (ODM), ja sen avulla javascript-olioiden tallettaminen mongon dokumenteiksi on suoraviivaista.

Asennetaan mongoose:

```bash
npm install mongoose --save
```

Ei lisätä mongoa käsittelevää koodia heti backendin koodin sekaan, vaan tehdään erillinen kokeilusovellus tiedostoon _mongo.js_:

```js
const mongoose = require('mongoose')

const url = 'mongodb://...'

// voit tarkistaa mongoosen version tiedostosta package.json
// jos käytössäsi on mongoosen versio 4.x seuraava rivi tulee antaa muodossa
// mongoose.connect(url, { useMongoClient: true })

mongoose.connect(url)
mongoose.Promise = global.Promise;

const Note = mongoose.model('Note', {
  content: String,
  date: Date,
  important: Boolean
})

const note = new Note({
  content: 'HTML on helppoa',
  date: new Date(),
  important: true
})

note
.save()
.then(response => {
  console.log('note saved!')
  mongoose.connection.close()
})
```

Kun koodi suoritetaan komennolla _node mongo.js_ lisää mongoose tietokantaaan uuden dokumentin.

Mlab:in hallintanäkymä (minne siis pääsee sovelluksen heroku-sivun kautta) näyttää lisäämämme datan:

![]({{ "/assets/3/13.png" | absolute_url }})

Kuten näkymä kertoo, on muistiinpanoa vastaava _dokumentti_ lisätty kokoelmaan (collection) nimeltään _notes_.

Koodi sisältää muutamia mielenkiintoisia asioita. Aluksi avataan yhteys ja määritellään, että mongoose käyttää _promiseja_, eikä oldschool-takaisinkutsufunktioita:

```js
const mongoose = require('mongoose')

const url = 'mongodb://...'

mongoose.connect(url)
mongoose.Promise = global.Promise
```

Valitettavasti mongoosen dokumentaatiossa käytetään joka paikassa takaisinkutsufunktioita, joten sieltä ei kannata suoraan copypasteta koodia, sillä promisejen ja vanhanaikaisten callbackien sotkeminen samaan koodiin ei ole kovin järkevää.

### Skeema

Yhteyden avaamisen jälkeen määritellään mustiinpanoa vastaava [model](http://mongoosejs.com/docs/models.html):

```js
const Note = mongoose.model('Note', {
  content: String,
  date: Date,
  important: Boolean
})
```

Modelin parametrina määritellään _muistiinpanon_ [skeema](http://mongoosejs.com/docs/guide.html), joka kertoo mongooselle, miten muistiinpano-oliot tulee tallettaa tietokantaan.

Ensimmäisenä parametrina oleva _Note_ määrittelee, että mongoose tallettaa muistiinpanoa vastaavat oliot kokoelmaan nimeltään _notes_.

Mongoosen dokumentaatiossa skeema ja sitä vastaava model määritellään kumpikin erikseen:

```js
const noteSchema = new mongoose.Schema({
  content: String,
  date: Date,
  important: Boolean
})

const Note = mongoose.model('Note', noteSchema);
```

Koska meillä ei ole skeema-oliolle muuta käyttöä kuin modelin parametrina, käytämme hyväksemme sitä, että skeema voidaan määritellä modeleille suoraan antamalla toisena parametrina skeeman määrittelevä olio.

Dokumenttikannat, kuten Mongo ovat _skeemattomia_, eli tietokanta itsessään ei välitä mitään sinne talletettavan tiedon muodosta. Samaan kokoelmaankin on mahdollista tallettaa olioita joilla on täysin eri kentät.

Mongoosea käytettäessä periaatteena on kuitenkin se, että tietokantaan talletettavalle tiedolle määritellään _sovelluksen koodin tasolla skeema_, joka määrittelee minkä muotoisia olioita kannan eri kokoelmiin talletetaan.

### Olioiden luominen ja tallettaminen

Seuraavaksi sovellus luo muistiinpanoa vastaavan [model](http://mongoosejs.com/docs/models.html):in avulla muistiinpano-olion:

```js
const note = new Note({
  content: 'Selain pystyy suorittamaan vain javascriptiä',
  date: new Date(),
  important: false
})
```

Modelit ovat ns. _konstruktorifunktioita_, jotka luovat parametrien perusteella javascript-olioita. Koska oliot on luotu modelien konstruktirifunktiolla, nillä on kaikki modelien ominaisuudet, eli joukko metodeja, joiden avulla olioita voidaan mm. tallettaa tietokantaan.

Tallettaminen tapahtuu metodilla _save_. Metodi palauttaa _promisen_, jolle voidaan rekisteröidä _then_-metodin avulla tapahtumankäsittelijä:

```js
note
  .save()
  .then(result => {
    console.log('note saved!')
    mongoose.connection.close()
  })
```

Kun olio on tallennettu kantaan, kutsutaan _then_:in parametrina olevaa tapahtumankäsittelijää, joka sulkee tietokantayhteyden komennolla <code>mongoose.connection.close()</code>. Ilman yhteyden sulkemista ohjelman suoritus ei pääty.

Tallennusoperaation tulos on takaisinkutsun parametrissa _result_. Yhtä olioa tallentaessamme tulos ei ole kovin mielenkiintoinen, olion sisällön voi esim. tulostaa konsoliin jos haluaa tutkia sitä tarkemmin sovelluslogiikassa tai esim. debugatessa.

Talletetaan kantaan myös pari muuta muistiinpanoa muokkaamalla dataa koodista ja suorittamalla ohjelma uudelleen.

### Olioiden hakeminen tietokannasta

Kommentoidaan koodista uusia muistiinpanoja generoiva osa, ja korvataan se seuraavalla:

```js
Note
  .find({})
  .then(result => {
    result.forEach(note => {
      console.log(note)
    })
    mongoose.connection.close()
  })
```

Kun koodi suoritetaan, kantaan talletetut muistiinpanot tulostuvat.

Oliot haetaan kannasta _Note_-modelin metodilla [find](http://mongoosejs.com/docs/api.html#model_Model.find). Metodin parametdina on hakuehto. Koska hakuehtona on tyhjä olio <code>{}</code>, saimme kannasta kaikki _notes_-kokoelmaan talletetut oliot.

Hakuehdot noudattavat mongon [syntaksia](https://docs.mongodb.com/manual/reference/operator/).

Voisimme hakea esim. ainoastaan tärkeät muistiinpanot seuraavasti:

```js
Note
  .find({ important: true })
  .then(result => {
    // ...
  })
```

### Tehtäviä

Tee nyt tehtävät [51 ja 52](../tehtavat#mongoosen-alkeet)

## Tietokantaa käyttävä backend

Nyt meillä on periaatteessa hallussamme riittävä tietämys ottaa mongo käyttöön sovelluksessamme.

Aloitetaan nopean kaavan mukaan, copypastetaan tiedostoon _index.js_ mongoosen määrittelyt, eli

```js
const mongoose = require('mongoose')

const url = 'mongodb://...'

mongoose.connect(url)
mongoose.Promise = global.Promise

const Note = mongoose.model('Note', {
  content: String,
  date: Date,
  important: Boolean
})
```

ja muutetaan kaikien muistiinpanojen hakemisesta vastaava käsittelijä seuraavaan muotoon

```js
app.get('/api/notes', (request, response) => {
  Note
    .find({})
    .then(notes => {
      response.json(notes)
    })
})
```

Voimme todeta selaimella, että backend toimii kaikkien dokumenttien näyttämisen osalta:

<img src="/assets/3/14.png" height="200">

Toiminnallisuus on muuten kunnossa, mutta frontend olettaa, että olioiden yksikäsitteinen tunniste on kentässä _id_. Emme myöskään halua näyttää frontendille mongon versiointiin käyttämää kenttää <em>\_\_v</em>. Tehdään pieni apufunktio, jonka avulla yksittäinen muistiinpano saadaan muutettua mongon sisäisestä esitysmuodosta haluamaamme muotoon:

```js
const formatNote = (note) => {
  return {
    content: note.content,
    date: note.date,
    important: note.important,
    id: note._id
  }
}
```

ja palautetaan HTTP-pyynnön vastauksena funktion avulla muotoiltuja oliota:

```js
app.get('/api/notes', (request, response) => {
  Note
    .find({})
    .then(notes => {
      response.json(notes.map(formatNote))
    })
})
```

Nyt siis muuttujassa _notes_ on taulukollinen mongon palauttamia olioita. Kun suoritamme operaation <code>notes.map(formatNote)</code> seurauksena on uusi taulukko, missä on jokaista alkuperäisen taulukon alkiota vastaava funktion _formatNote_ avulla muodostettu alkio.

Jos kannasta haettavilla olioilla olisi suuri määrä kenttiä, apufunktio _formatNote_ kannattaisi muotoilla hieman geneerisemmässä muodossa, esim:

```js
const formatNote = (note) => {
  const formattedNote = { ...note._doc, id: note._id }
  delete formattedNote._id
  delete formattedNote.__v

  return formattedNote
}
```

Ensimmäinen rivi luo uuden olion, mihin kopioituu kaikki vanhan olion kentät. Uuteen olioon lisätään myös kenttä _id_:

```js
const formattedNote = { ...note._doc, id: note._id }
```

Ennen olion palauttamista turhat kentät poistetaan.

Jos ohjelma käyttäisi muunkin tyyppisiä olioita kuin _muistiinpanoja_ sopisi sama funktio niidenkin muotoiluun. Jatkon kannalta on kuitenkin parempi, että pidämme alkuperäisen version funktiosta.

On myös mahdollista estää mongoosea palauttamasta tiettyjen kenttien arvoa, tai pyytää sitä palauttamaan vaan tietyt kentät. Saamme estettyä parametrin <em>\_\_v</em>:n lisäämällä _find_-metodiin toiseksi parametriksi _{\_\_v: 0}_ seuraavasti:

```js
app.get('/api/notes', (request, response) => {
  Note
    .find({}, {__v: 0})
    .then(notes => {
      response.json(notes.map(formatNote))
    })
})
```

Kyselyjen palauttamien kenttien määrittely tapahtuu Mongon [syntaksin mukaan](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/).

### Tietokantamäärittelyjen eriyttäminen omaksi moduuliksi

Ennen kuin täydennämme backendin muutkin osat käyttämään tietokantaa, eriytetään mongoose-spesifinen koodi omaan moduuliin.

Tehdään moduulia varten hakemisto _models_ ja sinne tiedosto _note.js_:

```js
const mongoose = require('mongoose')

const url = 'mongodb://...'

mongoose.connect(url)
mongoose.Promise = global.Promise

const Note = mongoose.model('Note', {
  content: String,
  date: Date,
  important: Boolean
})

module.exports = Note
```

Noden [moduulien](https://nodejs.org/docs/latest-v8.x/api/modules.html) määrittely poikkeaa hiukan osassa 2 määrittelemistämme frontendin käyttämistä [ES6-moduuleista](osa3/#refaktorointia---moduulit)

Moduulin ulos näkyvä osa määritellään asettamalla arvo muuttujalle _module.exports_. Asetamme arvoksi määritellyn modelin _Note_. Muut moduulin sisällä määritellyt asiat, esim. muuttujat _mongoose_ ja _url_ eivät näy moduulin käyttäjälle.

Moduulin käyttöönotto tapahtuu lisäämällä tiedostoon _index.js_ seuraava rivi

```js
const Note = require('./models/note')
```

Näin muuttuja _Note_ saa arvokseen saman olion, jonka moduuli määrittelee.

### Muut operaatiot

Muutetaan nyt kaikki operaatiot tietokantaa käyttävään muotoon.

Uuden muistiinpanon luominen tapahtuu seuraavasti:

```js
app.post('/api/notes', (request, response) => {
  const body = request.body

  if (body.content === undefined) {
    response.status(400).json({error: 'content missing'})
  }

  const note = new Note({
    content: body.content,
    important: body.important || false,
    date: new Date()
  })

  note
    .save()
    .then(savedNote => {
      response.json(formatNote(savedNote))
    })
})
```

Muistiinpano-oliot siis luodaan _Note_-konstruktorifunktiolla. Pyyntöön vastataan _save_-operaation konstruktorifunktion sisällä. Näin varmistutaan, että operaatio vastaus tapahtuu vain jos operaatio on onnistunut. Palaamme virheiden käsittelyyn myöhemmin.

Takaisinkutsufunktion parametrina _savedNote_ on talletettu muistiinpano. HTTP-pyyntöön palautetaan kuitenkin siitä funktiolla _formatNote_ formatoitu muoto:

```js
response.json(formatNote(savedNote))
```

Yksittäisen muistiinpanon tarkastelu muuttuu muotoon

```js
app.get('/api/notes/:id', (request, response) => {
  Note
    .findById(request.params.id)
    .then(note => {
      response.json(formatNote(note))
    })
})
```

### Frontendin ja backendin yhteistominnallisuuden varmistaminen

Kun backendia laajennetaan, kannattaa sitä testailla aluksi **ehdottomasti selaimella ja postmanilla**. Vasta kun kaikki on todettu toimivaksi, kannattaa siirtyä testailemaan että muutosten jälkeinen backend toimii yhdessä myös frontendin kanssa. Kaikkien kokeilujen tekeminen ainoastaan frontendin kautta on todennäköisesti varsin tehotonta.

Todennäköisesti voi olla kannattavaa edetä frontin ja backin integroinnissa toiminnallisuus kerrallaan, eli ensin voidaan toteuttaa esim. kaikkien muistiinpanojen näyttäminen backendiin ja testata että toiminnallisuus toimii selaimella. Tämän jälkeen varmistetaan, että frontend toimii yhteen muutetun backendin kanssa. Kun kaikki on todettu olevan kunnossa, siirrytään seuraavan ominaisuuden toteuttamiseen.

Kun kuvioissa on mukana tietokanta, on myös tietokannan tilan tarkastelu mlabin hallintanäkymästä varsin hyödyllistä, usein myös suoraan tietokantaa käyttävät node-apuohjelmat, kuten tiedostoon _index.js_ kirjoittamamme koodi auttavat sovellukehityksen edetessä.

### Tehtäviä

Tee nyt tehtävät [53 ja 54](tehtavat#backend-ja-tietokanta)

### Virheiden käsittely

Jos yritämme mennä selaimella sellaisen yksittäisen muistiinpanon sivulle, jota ei ole olemassa, eli esim. urliin <http://localhost:3001/api/notes/5a3b80015b6ec6f1bdf68d> missä _5a3b80015b6ec6f1bdf68d_ ei ole minkään tietokannassa olevan muistiinpanon tunniste, jää selain "jumiin" sillä palvelin ei vastaa pyyntöön koskaan.

Palvelimen konsolissa näkyykin virheilmoitus:

![]({{ "/assets/3/15.png" | absolute_url }})

Kysely on epäonnistunut ja kyselyä vastaava promise mennyt tilaan _rejected_. Koska emme käsittele promisen epäonnistumista, ei pyyntöön vastata koskaan. Osassa 2 tutustuimme jo [promisejen virhetilanteiden käsittelyyn](osa2/#promise-ja-virheet).

Lisätään tilanteeseen yksinkertainen virheidenkäsittelijä:

```js
app.get('/api/notes/:id', (request, response) => {
  Note
    .findById(request.params.id)
    .then(note => {
      response.json(formatNote(note))
    })
    .catch(error => {
      console.log(error)
      response.status(404).end()
    })
})
```

Kaikissa virheeseen päättyvissä tilanteissa HTTP-pyyntöön vastataan statuskoodilla 404 not found. Konsoliin tulostetaan tarkempi tieto virhestä.

Tapauksessamme on itseasiassa olemassa kaksi erityyppistä virhetilannetta. Toinen vastaa sitä, että yritetään hakea muistiinpanoa virheellisen muotoisella _id_:llä, eli sellasiella mikä ei vastaa mongon id:iden muotoa.

Jos teemme näin tulostuu konsoliin:

<pre>
Method: GET
Path:   /api/notes/5a3b7c3c31d61cb9f8a0343
Body:   {}
---
{ CastError: Cast to ObjectId failed for value "5a3b7c3c31d61cb9f8a0343" at path "_id"
    at CastError (/Users/mluukkai/opetus/_fullstack/osa3-muisiinpanot/node_modules/mongoose/lib/error/cast.js:27:11)
    at ObjectId.cast (/Users/mluukkai/opetus/_fullstack/osa3-muisiinpanot/node_modules/mongoose/lib/schema/objectid.js:158:13)
    ...
</pre>

Toinen virhetilanne taas vastaa tilannetta, missä haettavan muistiinpanon id on periaatteessa oikeassa formaatissa, mutta tietokannasta ei löydy indeksillä mitään:

<pre>
Method: GET
Path:   /api/notes/5a3b7c3c31d61cbd9f8a0343
Body:   {}
---
TypeError: Cannot read property '_doc' of null
    at formatNote (/Users/mluukkai/opetus/_fullstack/osa3-muisiinpanot/index.js:46:33)
    at Note.findById.then.note (/Users/mluukkai/opetus/_fullstack/osa3-muisiinpanot/index.js:65:21)
</pre>

Nämä tilanteen on syytä erottaa toisistaan, ja itseasiassa jälkimmäinen poikkeus on oman koodimme <code>/Users/mluukkai/opetus/\_fullstack/osa3-muisiinpanot/index.js:46</code> aiheuttama.

Muutetaan koodia seuraavasti:

```js
app.get('/api/notes/:id', (request, response) => {
  Note
    .findById(request.params.id)
    .then(note => {
      if (note) {
        response.json(formatNote(note))
      } else {
        response.status(404).end()
      }
    })
    .catch(error => {
      console.log(error)
      response.status(400).send({ error: 'malformatted id' })
    })
})
```

Jos kannasta ei löydy haettua olioa, muuttujan _note_ arvo on _undefined_ ja koodi ajautuu _else_-haaraan. Siellä vastataan kyselyyn _404 not found_.

Jos id ei ole hyväksyttävässä muodossa, ajaudutaan _catch_:in avulla määriteltyyn virheidenkäsittelijään. Sopiva statauskoodi on [400 bad request](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.1) koska kyse on juuri siitä:

> The request could not be understood by the server due to malformed syntax. The client SHOULD NOT repeat the request without modifications.

Vastaukseen on lisätty myös hieman dataa kertomaan virheen syystä.

Promisejen yhteydessä kannattaa melkeinpä aina lisätä koodiin myös virhetilainteiden käsittely, muuten seurauksena on usein hämmentäviä vikoja.

Ei ole koskaan huono idea tulostaa poikkeuksen aiheuttanutta olioa konsoliin virheenkäsittelijässä:

```js
.catch(error => {
  console.log(error)
  response.status(400).send({ error: 'malformatted id' })
})
```

Virheenkäsittelijään joutumisen syy voi olla joku ihan muu kuin mitä on tullut alunperin ajatelleeksi. Jos virheen tulostaa konsoliin, voi säästyä pitkiltä ja turhauttavilta väärää asiaa debuggaavilta sessioita.

### loput operaatiot

Toteutetaan vielä jäljellä olevat operaatiot, eli yksittäisen muistiinpanon poisto ja muokkaus.

Poisto onnistuu helpoiten metodilla [findByIdAndRemove](http://mongoosejs.com/docs/api.html#model_Model.findByIdAndRemove):

```js
app.delete('/api/notes/:id', (request, response) => {
  Note
    .findByIdAndRemove(request.params.id)
    .then(result => {
      response.status(204).end()
    })
    .catch(error => {
      response.status(400).send({ error: 'malformatted id' })
    })
})
```

Vastauksena on statauskoodi _204 no content_ molemmissa "onnistuneissa" tapauksissa, eli jos olio poistettiin tai olioa ei ollut mutta _id_ oli periaatteessa oikea. Takaisinkutsun parametrin _result_ perusteella olisi mahdollisuus haarautua ja palauttaa tilanteissa eri statuskoodi jos sille on tarvetta.

Muistiinpanon tärkeyden muuttamisen mahdollistava olemassaolevan muistiinpanon päivitys onnistuu helposti metodilla [findOneAndUpdate](http://mongoosejs.com/docs/api.html#model_Model.findOneAndUpdate):

```js
app.put('/api/notes/:id', (request, response) => {
  const body = request.body

  const note = {
    content: body.content,
    important: body.important
  }

  Note
    .findByIdAndUpdate(request.params.id, note, { new: true } )
    .then(updatedNote => {
      response.json(formatNote(updatedNote))
    })
    .catch(error => {
      console.log(error)
      response.status(400).send({ error: 'malformatted id' })
    })
})
```

Operaatio mahdollistaa myös muistiinpanon sisällön editoinnin. Päivämäärän muuttaminen ei ole mahdollista.

Huomaa, että metodin _findOneAndUpdate_ parametrina tulee antaa normaali javascript-olio, eikä uuden olion luomisessa käytettävä _Note_-konstruktorifunktiolla luotu olio.

Pieni, mutta tärkeä detalji liittyen operaatioon _findOneAndUpdate_. Oletusarvoisesti tapahtumankäsittelijä saa parametrikseen _updatedNote_ päivitetyn olion [ennen muutosta](http://mongoosejs.com/docs/api.html#model_Model.findOneAndUpdate) olleen tilan. Lisäsimme operaatioon parametrin <code>{ new: true }</code> jotta saamme muuttuneen olion palautetuksi kutsujalle.

Backend vaikuttaa nyt toimivan postmanista tehtyjen kokeilujen perusteella, muistiinpanojen tärkeyden muuttaminen frontendissa kuitenkin sotkee muistiinpanojen järjestyksen. Syynä on se, että _id_-kentät eivät ole enää numeroja vaan stringejä ja joudummekin muuttamaan järjestämisessä käytettävän metodissa _render_ olevan funktion esim. seuraavaan muotoon:

```js
const byId = (note1, note2) =>
  note1.id < note2.id ? -1 : 1
```

Koska javascriptissa merkkijonojen leksikaalista aakkosjärjestystä on mahdollista vertailla < -operaattorilla, teemme vertailun ja palautamme vertailun tulokseen perustuen joko -1 tai 1.

### Tehtäviä

Tee nyt tehtävät [55-57](tehtavat#lisää-operaatioita)

## refaktorointia - promisejen ketjutus

Useat routejen tapahtumankäsittelijöistä muuttivat palautettavan datan oikeaan formaattiin kutsumalla metodia _formatNote_:

```js
const formatNote = (note) => {
  return {
    id: note._id,
    content: note.content,
    date: note.date,
    important: note.important
  }
}
```

esim uuden muistiinpanon luomisessa metodia kutsutaan _then_:in parametrina palauttama olio parametrina:

```js
app.post('/api/notes', (request, response) => {
  // ...

  note
    .save()
    .then(savedNote => {
      response.json(formatNote(savedNote))
    })

})
```

Voisimme tehdä saman myös hieman tyylikkäämmin [promiseja ketjuttamalla](https://javascript.info/promise-chaining):

```js
app.post('/api/notes', (request, response) => {
  // ...

  note
    .save()
    .then(savedNote => {
      return formatNote(savedNote)
    })
    .then(savedAndFormattedNote => {
      response.json(savedAndFormattedNote)
    })

})
```

Eli ensimmäisen _then_:in takaisinkutsussa otamme mongoosen palauttaman olion _savedNote_ ja formatoimme sen. Operaation tulos palautetaan returnilla. Kuten osassa 2 [todettiin](osa2/#palvelimen-kanssa-tapahtuvan-kommunikoinnin-eristäminen-omaan-moduuliin), promisen then-metodi palauttaa myös promisen. Eli kun palautamme _formatNote(savedNote)_:n takaisinkutsufunktiosta, syntyy promise, jonka arvona on formatoitu muistiinpano. Pääsemme käsiksi arvoon rekisteröimällä _then_-kutsulla uuden tapahtumankäsittelijän.

Itseasiassa selviämme vieläkin tiiviimmällä koodilla:

```js
app.post('/api/notes', (request, response) => {
  // ...

  note
    .save()
    .then(formatNote)
    .then(savedAndFormattedNote => {
      response.json(savedAndFormattedNote)
    })

})
```

koska _formatNote_ on viite funktioon, on oleellisesti ottaen kyse samasta kuin kirjoittaisimme:

```js
app.post('/api/notes', (request, response) => {
  // ...

  note
    .save()
    .then(savedNote => {
      const formattedNote = { ...savedNote._doc, id: savedNote._id }
      delete formattedNote._id
      delete formattedNote.__v

      return formattedNote
    })
    .then(savedAndFormattedNote => {
      response.json(savedAndFormattedNote)
    })

})
```

## Sovelluksen vieminen tuotantoon

Sovelluksen pitäisi toimia tuotannossa, eli herokussa sellaisenaan. Frontendin muutosten takia on tehtävä siitä uusi tuotantoversio ja kopioitava se backendiin.

Sovellusta voi käyttää sekä frontendin kautta <https://radiant-plateau-25399.herokuapp.com/>, mutta myös API:n <https://radiant-plateau-25399.herokuapp.com/api/notes> suora käyttö selaimella ja postmanilla onnistuu.

Sovelluksessamme on tällä hetkellä eräs ikävä piirre. Tietokannan osoite on kovakoodattu backendiin ja samaa tietokantaa käytetään sekä tuotannossa, että sovellusta kehitettäessä.

Tarvitsemme oman kannan sovelluskehitystä varten. Eräs vaihtoehto on luoda käyttäjätunnus [mlab](https://www.mlab.com):iin ja luoda sinne uusi tietokanta.

Huomaa, että kun luot mlab:issa tietokannan, tarkoitetaan käyttäjätunnuksella ja salasanalla tietokannalle määriteltyä tietoja, ei niitä millä kirjaudut mlabiin.

![]({{ "/assets/3/16.png" | absolute_url }})

Tietokannan osoitetta ei kannata kirjoittaa koodiin. Eräs hyvä tapa tietokannan osoitteen määrittelemiseen on [ympäristömuuttujien](https://en.wikipedia.org/wiki/Environment_variable) käyttö. Itseasiassa Herokussa sovelluksemme tietokannan osoite on talletettuna ympäristömuuttujaan _MONGODB_URI_, tämän kertoo myös komentoriviltä annettava komento _heroku config_

Ympäristömuuttujiin pääsee Node-sovelluksesta käsiksi seuraavasti:

```js
const mongoose = require('mongoose')

const url = process.env.MONGODB_URI

// ...

module.exports = Note
```

tämän muutoksen jäkeen sovellus ei toimi paikallisesti, koska ympäristömuuttujalla _MONGODB_URI_ ei ole mitään arvoa. Tapoja määritellä ympäristömuuttujalle arvo on monia, käytetään nyt [dotenv](https://www.npmjs.com/package/dotenv)-kirjastoa.

Asennetaan kirjasto komennolla

```bash
npm install dotenv --save
```

Sovelluksen juurihakemistoon tehdään sitten tiedosto nimeltään _.env_, minne tarvittavien ympäristömuuttujien arvot asetetaan

```bash
MONGODB_URI=mongodb://....
```

Tiedosto .env **tulee heti gitignorata** sillä emme halua julkaista .env -tiedoston sisältöä verkkoon.

dotenvissä määritellyt ympäristömuuttujat otetaan koodissa käyttöön komennolla

```js
require('dotenv').config()
```

ja niihin viitataan Nodessa kuten "normaaleihin" ympäristömuuttujiin syntaksilla _process.env.MONGODB_URI_

Otetaan dotenv käyttöön seuraavasti:

```js
const mongoose = require('mongoose')

if ( process.env.NODE_ENV !== 'production' ) {
  require('dotenv').config()
}

const url = process.env.MONGODB_URI

// ...

module.exports = Note
```

Nyt dotenvissä olevat ympäristömuuttujat otetaan käyttöön ainoastaan silloin kun sovellus ei ole _production_- eli tuotantomoodssa (kuten esim. Herokussa).

Uudelleenkäynnistyksen jälkeen sovellus toimii taas paikallisesti.

Node-sovellusten konfigurointiin on olemassa ympäristömuuttujien ja dotenvin lisäksi lukuisia vaihtoehtoja, mm. [node-conf](https://github.com/lorenwest/node-config). Ympäristömuuttujien käyttö riittää meille nyt, joten emme rupea overengineeraamaan. Palaamme aiheeseen kenties myöhemmin.

### Tehtäviä

Tee nyt osan viimeiset tehtävät [58-](/tehtavat#loppuhuipennus)
