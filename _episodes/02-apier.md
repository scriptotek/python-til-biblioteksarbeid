---
title: "Biblioteks-APIer"
teaching: 45
exercises: 45
questions:
- Hvordan kan jeg hente ut data fra et API?
objectives:
- Kunne kommunisere med et API ved hjelp av `requests`-pakken
- Kunne orientere seg i og trekke ut data fra XML- og JSON-responsobjekter.
keypoints:
- API-er (programmeringsgrensesnitt) er en annen inngang til et system enn den du gjerne bruker til vanlig.
- API-er kommuniserer over HTTP og du kan bruke et hvilket som helst programmeringsspråk for å snakke med dem.
- API-er utveksler data i programmeringsvennlige formater, typisk JSON og/eller XML.
- API-er er vanligvis stabile (endrer seg ikke uten forvarsel), i motsetning til vanlige nettsider som endrer seg uten forvarsel.
- API-er er vanligvis dokumenterte, men det kan være mye ny terminologi å forholde seg til.

---

# Hva er et API?

- API (application programming interface) eller også *programmeringsgrensesnitt* på godt norsk –
  et grensesnitt mot et system (f.eks biblioteksystemet ditt).
    - I praksis skal vi bare se på web-API-er, altså API-er vi snakker med over internett.
      Det finnes også API-er mellom programmer og annet på maskinen din.
    - Hvis dere leser API-dokumentasjon kommer dere fort over betegnelsen «REST» eller «RESTful».
      De aller, aller fleste API-er følger disse prinsippene i større eller mindre grad i dag,
      og det er ikke noe spesielt spennende ved dem egentlig.
    - Java-utviklere liker å snakke om "web service" (webtjeneste).
      Tenk på det som et synonym for API.
- API-er er som regel mye mer stabilt over tid enn selve systemene bak.
- API-er er som regel dokumenterte.
- Åpner for å kunne automatisere og trekke ut data fra ulike systemer og kombinere dem

![API](https://scriptotek.github.io/python-til-biblioteksarbeid/fig/api-figur.png)

<!--
  Liten digresjon: [slipsomat](https://github.com/scriptotek/alma-slipsomat)
  -- Python-script som bruker selenium for å automatisere nettleserarbeid.
  Eksempel på hva man gjør når man ikke har et API.
  Tidkrevende og sårbart.
-->

### XML og JSON

Nesten alle API-er returnerer data som XML og/eller JSON.

`<xml>` | `{json}`
---|---
utviklet i 1997 | utviklet i 2001
minner om html | avledet fra javascript
mer komplisert | mindre komplisert

![API](https://scriptotek.github.io/python-til-biblioteksarbeid/fig/xml-json-all-time1-600x382.png)

Generelt er JSON enklere å jobbe med, men begge formatene har sine styrker,
og i praksis må en forholde seg til begge deler.


# HTTP-forespørsler (HTTP-kall) med requests

Vi begynner med å importere `requests`, som er en Python-pakke (eller et Python-bibliotek)
for å gjøre HTTP-forespørsler (HTTP requests):

~~~
import requests
~~~
{: .language-python}

HTTP-forespørsler er ekstremt lite mystisk – det er det vi gjør hver gang vi henter en nettside.
For å illustrere dette kan vi begynne med å hente forsiden til BI med `requests` slik:

~~~
requests.get('http://www.bi.no')
~~~
{: .language-python}

Her kjører vi funksjonen `get()` fra `requests`-pakken
(requests-pakken kaller igjen et annet bibliotek som igjen kaller et system-API
osv. osv. Lag på lag med komplisert nettverksfunksjonalitet
som vi heldigvis slipper å forholde oss til – puh!)
Det vi får ut igjen fra `get()`-funksjonen er et responsobjekt.
For å kunne bruke det videre «putter vi det i en variabel» som vi kaller "response":

~~~
response = requests.get('http://www.bi.no')
~~~
{: .language-python}

Men hva befinner seg inni dette objektet?
Et objekt er en sekk som kan inneholde forskjellige ting.
Vi kan liste ut hva objektet inneholder med `dir()`-funksjonen:

~~~
dir(response)
~~~
{: .language-python}

Her ser vi f.eks. at det ligger noe som heter "text".
La oss bruke `print()`-funksjonen for å se hva som ligger der:

~~~
print(response.text)
~~~
{: .language-python}

Ser dere hva dette er? HTML! Det samme som vi ville fått ut hvis vi hadde
[vist kildekoden til nettsiden](https://kb.iu.edu/d/agao) i nettleseren vår.

<!--
Om vi ønsker å trekke ut informasjon fra et HTML-dokument er `BeautifulSoup`
et utmerket verktøy.
Akkurat som requests et BeautifulSoup som noen andre har laget,
og som inneholder nyttige funksjoner som vi kan bruke fremfor å måtte skrive dem selv.

~~~
from bs4 import BeautifulSoup

parsed_html = BeautifulSoup(response.text, 'lxml')
parsed_html.body.findAll('h2')
~~~
{: .language-python}

Det vi gjør her er at vi bruker en funksjon som noen andre har laget,
slik at vi slipper å skrive denne funksjonen.
-->

I prinsippet kunne vi prøvd å trekke ut data fra HTML-koden, da er vi
inne i det som kalles webscraping.
Det er gjerne noe en tyr til for å få ut data fra systemer som ikke tilbyr et API
(eller ikke tilbyr dataene du trenger gjennom API-et).

<!-- Eks: Hente åpningstider fra en Facebook-side. ResearchGate, Instagram eller andre kjipe nettsteder -->

Når vi bruker et API får vi derimot ut data i et maskinvennlig format (typisk JSON eller XML).
API-er er nesten alltid dokumenterte (selv om kvaliteten på dokumentasjonen kan variere)
og de skal i utgangspunktet ikke endre seg uten forvarsel.

API-er brukes ikke bare for å hente ut data, men også for å endre data.
Dette krever en eller annen form for autentisering, f.eks. en hemmelig nøkkel som man legger ved hver forespørsel.


# Hente data fra Cristin-API-et

Hvis du googler "api cristin" kommer du til <http://api.cristin.no/>

<!--Dette er det nye Cristin-API-et, men det er ikke helt ferdig.
Foreløpig får du ikke ut vitenskapelige publikasjoner – som jo er det morsomste!
Så vi bruker det gamle API-et, som er dokumentert her:
<https://www.cristin.no/ressurser/dokumentasjon/web-service/>
-->

## Noen eksempler

Vi kan f.eks. se på bøker utgitt av en bestemt person med ID 22846. Hvem er det? Prøv og se:
<https://api.cristin.no/ws/hentVarbeiderPerson?lopenr=22846&fra=1999&til9999&hovedkategori=BOK&format=json>

La oss så bruke `requests` for å hente inn det samme resultatet med Python:

~~~
response = requests.get('https://api.cristin.no/ws/hentVarbeiderPerson', params={
  'lopenr': '22846',
  'fra': '1999',
  'til': '9999',
  'hovedkategori': 'BOK',
  'format': 'json',
})
~~~
{: .language-python}

Dette API-et støtter både XML og JSON, men her ber vi om å få JSON fordi det er enklest å jobbe med, og vi lærer noe nytt.
I stad brukte vi `xmltodict`-pakken for å parse XML.
Nå skal vi bruke `json`-pakken for å parse JSON.

~~~
import json
data = json.loads(response.text)
~~~
{: .language-python}

Dette er igjen et object av type `dict`.
For å få et inntrykk av JSON-strukturen kan det enkleste være å bruke Chrome.
Når vi har funnet ut hva vi er ute etter, kan vi hente det ut:

~~~
for resultat in data['forskningsresultat']:
    print(resultat['fellesdata']['ar'], resultat['fellesdata']['tittel'])
~~~
{: .language-python}



* Den nyeste boka hens:
  <https://api.cristin.no/ws/hentVarbeiderPerson?lopenr=22846&fra=1999&til9999&hovedkategori=BOK&utplukk=nyeste&maksantall=1&format=json>

* Artiklene hens:
  <https://api.cristin.no/ws/hentVarbeiderPerson?lopenr=22846&fra=1999&til9999&hovedkategori=TIDSSKRIFTPUBL&format=json>

* Nyeste 10 artikler:
  <https://api.cristin.no/ws/hentVarbeiderPerson?lopenr=22846&fra=1999&til9999&hovedkategori=TIDSSKRIFTPUBL&utplukk=nyeste&maksantall=1&format=json>

* De 10 nyeste bøkene fra personer med UiO-tilknytning (UiO har institusjonsnr 185. For andre institusjoner, se f.eks. [denne PDF-en](https://www.cristin.no/statistikk-og-rapporter/nvi-rapportering/tidligere-ar/nvi-resultater2015/uh-nvi2015.pdf)):
  <https://api.cristin.no/ws/hentVarbeidSted?instnr=185&utplukk=nyeste&maksantall=10&&hovedkategori=BOK&format=json>


Her er et eksempel på hvordan data fra Cristin-APIet presenteres på UiOs nettsider for å vise de nyeste artiklene fra et bestemt forskningssenter:
<https://www.med.uio.no/klinmed/forskning/sentre/seraf/publikasjoner/cristin/>

<!--
instnr=
(NTNU=194, UiB=184, UiO=185, UiTø=186, BI = 584).
Alle institusjoners nr:
-->


# Hente bibliografiske data fra API-et til et biblioteksystem

Som eksempel på API skal vi hente ut bibliografiske poster fra et API

> ## [SRU](http://www.loc.gov/standards/sru/)
> SRU er en standard søkeprotokoll fra Library of Congress som så godt som
> alle biblioteksystemer støtter (Det er riktignok ikke alle bibliotek som
> har *åpne* SRU-APIer, men mange har det). Et eksempel på en nettside
> som snakker med SRU-APIer fra mange forskjellige bibliotek er
> [Karlsruher Virtueller Katalog](http://kvk.bibliothek.kit.edu/).
>
> Hva betyr det at det er en standard søkeprotokoll? At parametrene er standardiserte,
> så vi kan bruke de samme parameternavnene uansett hvilket biblioteksystem
> vi snakker med. Selve søkefeltene (søkeindeksene) er imidlertid ikke standardiserte,
> så de kan variere fra system til  system.
{: .callout}

<!-- Advarsel: MARC er ikke den enkleste datamodellen å jobbe med,
     og egentlig ikke det mest pedagogiske eksempelet å begynne med.
     Men vi hadde lyst til å dykke litt ned i den skitne virkeligheten.
     Men når du jobber i bibliotek er det veldig vanskelig å komme unna MARC,
     og siden dette er biblioteksløsyd så synes vi at vi måtte ha det med.
     Det finnes svært gode nettkurs for å lære programmering.
     -->

Vi skal bruke Alma som eksempel. Da begynner vi typisk med å google "alma sru",
og kommer vi til API-dokumentasjonen for Almas SRU-tjeneste på
[https://developers.exlibrisgroup.com/alma/integrations/SRU](https://developers.exlibrisgroup.com/alma/integrations/SRU).

I API-dokumentasjonen kan vi se om det er noen eksempler vi kan prøve rett ut av boksen.
Under lenken «searchRetrieveResponse» under overskriften «Examples (in Alma's Guest sandbox environment)»
helt nederst finner vi ett. Trykker vi på lenken gjør nettleseren vår en HTTP-forespørsel og vi får tilbake noe XML-data:

<https://na01.alma.exlibrisgroup.com/view/sru/TR_INTEGRATION_INST?version=1.2&operation=searchRetrieve&recordSchema=marcxml&query=alma.all_for_ui=history&maximumRecords=3&startRecord=4>

Dataene kommer fra gjestesandkassen, men hvordan får vi det til å virke med *våre* data?
Hvis vi går tilbake til dokumentasjonen ser vi at det står
«The base URL for SRU requests is: https://&lt;Alma domain&gt;/view/sru/&lt;institution code&gt;».
Alma-domenet kan vi finne fra [listen over Alma-instanser](https://www.bibsys.no/direktelenker-til-alma-instansene/).

Vi kan f.eks. prøve med domenet `bibsys.alma.exlibrisgroup.com` og institusjonskoden `47BIBSYS_NETWORK`:

<https://bibsys.alma.exlibrisgroup.com/view/sru/47BIBSYS_NETWORK?version=1.2&operation=searchRetrieve&recordSchema=marcxml&query=alma.all_for_ui=history&maximumRecords=3&startRecord=4>

(Prøv evt. gjerne også med verdier fra din egen favorittinstitusjon!)

Nå kan vi prøve å hente den samme URL-en i Python med `requests`-biblioteket som vi importerte i stad:

~~~
response = requests.get('https://bibsys.alma.exlibrisgroup.com/view/sru/47BIBSYS_NETWORK?version=1.2&operation=searchRetrieve&recordSchema=marcxml&query=alma.all_for_ui=history&maximumRecords=3&startRecord=4')
print(response.text)
~~~
{: .language-python}

Dette funker, men det er vanskelig å lese hvor det ene parameteret slutter og det neste starter.
Parametrene er det som følger etter spørsmålstegnet, og samlet kalles det
[query string](https://en.wikipedia.org/wiki/Query_string) (spørrestreng).
For å gjøre koden ryddigere ønsker vi skille disse ut som variabler:

~~~
start = 4
limit = 3
query = 'alma.all_for_ui=history'

response = requests.get('https://bibsys.alma.exlibrisgroup.com/view/sru/47BIBSYS_NETWORK', params={
    'version': '1.2',
    'operation': 'searchRetrieve',
    'startRecord': start,
    'maximumRecords': limit,
    'query': query,
})
print(response.text)
~~~
{: .language-python}

*Resultatet* fra å kjøre denne koden er nøyaktig det samme som resultatet fra å kjøre det forrige eksempelet,
*men* nå har vi en kodesnutt som er lettere å bearbeide videre – og det er bra!
For å gjøre det enda lettere å bruke denne kodesnutten vil vi gjøre den om til en funksjon.

Jupyter-tips: Marker flere linjer tekst og trykk Tab for å indentere alle linjene samtidig.

~~~
def sru_search_request(query, start=1, limit=50):
    response = requests.get('https://bibsys.alma.exlibrisgroup.com/view/sru/47BIBSYS_NETWORK', params={
        'version': '1.2',
        'operation': 'searchRetrieve',
        'startRecord': start,
        'maximumRecords': limit,
        'query': query,
    })
    return response
~~~
{: .language-python}

<!--
  Dette illustrerer også et lite poeng: Man skriver typisk ikke et program fra linje 1 og nedover.
  Dette er kanskje.
-->

Nå kan vi gjøre et nytt søk med bare én linje kode!

~~~
response = sru_search_request('alma.creator="hurum, jørn"')
print(response.text)
~~~
{: .language-python}


### Hjelp, hva gjør jeg med all denne XML-teksten?

Neste spørsmål er hvordan skal vi navigere i dette havet av XML-tekst som vi får tilbake fra API-et.

Dette er biblioteksløyd, så vi skal prøve et nytt verktøy, nemlig en Python-pakke som
parser XML (parse betyr å tolke teksten og gjøre den om til et objekt vi kan jobbe med).

Det finnes mange forskjellige slike pakker (google "python xml"), og de har ulike fordeler og ulemper.
I dag skal vi bruke et kalt `xmltodict`, som gjør XML-teksten om til en såkalt dictionary (ordliste)
eller dict, som gjør at vi kan jobbe med responsen på samme måte som vi ville gjort fra et JSON-API.

> ## Apropos xmltodict, hva er egentlig en `dict`?
> En ordbok / [dictionary](https://dbader.org/blog/python-dictionaries-maps-and-hashtables) (dict) er en sentral datatype i Python
> (i andre programmeringsspråk ofte kalt hashmap, associated array e.a.).
> En dict er et enkelt objekt som du kan putte ulike ting oppi,
> og der hvert ting har en etikett, som du kan bruke til å enkelt finne tingen igjen.
{: .callout}

<!--
TODO: Oppgave med dict?
-->

For å finne ut strukturen til XML-filen kan det enkleste noen ganger være å forhåndsvise den
i nettleseren din, f.eks. Chrome.
Digresjon: Det finnes selvfølgelig en pakke du kan bruke for å åpne nettleseren din også:

~~~
import webbrowser
webbrowser.open(response.url)
~~~
{: .language-python}

Når vi ser på XML-en ser vi at toppnivået heter "searchRetrieveResponse".
Deretter følger "records" og "record". Denne noden er det flere av, så det er en liste.
Hver "record"-node har en "recordData"-node som igjen har en "record"-node.

![XML-strukturen til responsen](https://scriptotek.github.io/python-til-biblioteksarbeid/fig/xml-response.png)

La oss prøve å navigere i XML-strukturen med Python.
Først må vi importere `xmltodict`:

~~~
import xmltodict
~~~
{: .language-python}

Vi ønsker å lage en liste som inneholder alle MARC-postene,
men ikke alt det som ligger rundt. Det kan vi gjøre slik:

~~~
xml = xmltodict.parse(response.text, force_list=['record', 'datafield', 'subfield'])

records = []
for rec in xml['searchRetrieveResponse']['records']['record']:
    records.append(rec['recordData']['record'][0])
~~~
{: .language-python}

I parentes: Hvis vi er nysgjerrige på hvor mange poster vi har i listen vår kan vi sjekke med `len()`-funksjonen:

~~~
len(records)
~~~
{: .language-python}


> ## Oppgave 1: Bygge om `sru_search_request`-funksjonen til å returnere en liste med MARC-poster
> Litt tidligere laget vi funksjonen `sru_search_request`, som utfører et søk og returerner
> et responsobjekt. Men vi vil heller at den heller skal returnere en liste med
> MARC-poster. Kan du utvide funksjonen slik at den gjør det? Bruk koden over.
> For å unngå forvirring er det lurt å kopiere den gamle funksjonen inn i en ny celle og
> gi den et nytt navn: `sru_search_request2`.
>
> > ## Løsning
> > ~~~
> > def sru_search_request2(query, start=1, limit=50):
> >     response = requests.get('https://bibsys.alma.exlibrisgroup.com/view/sru/47BIBSYS_NETWORK', params={
> >         'version': '1.2',
> >         'operation': 'searchRetrieve',
> >         'startRecord': start,
> >         'maximumRecords': limit,
> >         'query': query,
> >     })
> >     xml = xmltodict.parse(response.text, force_list=['record', 'datafield', 'subfield'])
> >     records = []
> >     for rec in xml['searchRetrieveResponse']['records']['record']:
> >         records.append(rec['recordData']['record'][0])
> >     return records
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

> ## Oppgave 2: Hente ut leader fra den første MARC-posten
> ![MARC](https://scriptotek.github.io/python-til-biblioteksarbeid/fig/marc.png)
>
> Vi har `records`, som er en liste. Du kan hente ut det første elementet fra listen
> (altså den første MARC-posten) med `records[0]`.
> Se på strukturen til XML-filen, enten i nettleseren din eller figuren over.
>
> - (a) Klarer du å hente ut verdien til feltet
>       [leader](https://www.loc.gov/marc/bibliographic/bdleader.html) for den *første* posten?
> - (b) Kan du lage en `for`-løkke og bruke `print`-funksjonen for å skrive ut leader for *alle* postene?
>
> > ## Løsning
> > (a)
> > ~~~
> > records[0]['leader']
> > ~~~
> > {: .language-python}
> >
> > (b)
> > ~~~
> > for record in records:
> >     print(record['leader'])
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}

> ## Oppgave 3: Fyll inn det som mangler (______)
>
> Kopier kodesnutten under til din notebook. Prøv å finn ut hva du må
> erstatte "______" med for å få koden til å virke (det er ikke det samme begge stedene).
> Hva gjør koden?
>
> ~~~
> for rec in records:
>     for field in ______['datafield']:
>         if field['@tag'] == '245':
>             for subfield in ______['subfield']:
>                 if ______['@code'] == 'a':
>                     print(subfield['#text'])
> ~~~
> {: .language-python}
>
> > ## Løsning
> > ~~~
> > for rec in records:
> >     for field in rec['datafield']:
> >         if field['@tag'] == '245':
> >             for subfield in field['subfield']:
> >                 if subfield['@code'] == 'a':
> >                     print(subfield['#text'])
> > ~~~
> > {: .language-python}
> {: .solution}
{: .challenge}


Skrive ut tittel og Dewey-nummer:

~~~
for rec in records:

    # Det kan være lurt å tømme variablene først
    tittel = ''
    dewey_numre = []

    for field in rec['datafield']:

        if field['@tag'] == '245':
            for subfield in field['subfield']:
                if subfield['@code'] == 'a':
                    tittel = subfield['#text']

        if field['@tag'] == '082':
            for subfield in field['subfield']:
                if subfield['@code'] == 'a':
                    dewey_numre.append(subfield['#text'])

    print(tittel, dewey_numre)
~~~
{: .language-python}

Hvis vi bare vil ha *unike* numre kan vi gjøre om listen til et sett: `set(dewey_numre)`

Tenk om det fantes et API der vi kunne sende inn et Dewey-nummer og få ut klassebetegnelsen
fra norsk Webdewey. La oss prøve det.

~~~
for rec in records:

    # Det kan være lurt å tømme variablene først
    tittel = ''
    dewey_numre = []

    for field in rec['datafield']:

        if field['@tag'] == '245':
            for subfield in field['subfield']:
                if subfield['@code'] == 'a':
                    tittel = subfield['#text']

        if field['@tag'] == '082':
            for subfield in field['subfield']:
                if subfield['@code'] == 'a':
                    dewey_numre.append(subfield['#text'])

    print('Tittel:' + tittel)
    for dewey_nummer in set(dewey_numre):
        dewey_response = requests.get('https://ub-www01.uio.no/microservices/dewey_heading.php', params={
            'number': dewey_nummer,
        })
        print('    ' + dewey_nummer + ' ' + dewey_response.text)
~~~
{: .language-python}


## I kombinasjon med Dewey-API-et

<!-- http://deweysearchno.pansoft.de/webdeweysearch/rest?query=113 funker ikke nå -->

<!-- ## Jobbe med MARC-poster 1
> Ta utgangspunkt i koden ovenfor og lag en funksjon kalt `filter_records` som
> returnerer poster


> (a) Hente ut alle Dewey-numre (082 a)
>
> > ## Løsning
> > ~~~
> > print("Hello world")
> > ~~~
> > {: .language-python}
> > Mer tekst.
> {: .solution}
{: .challenge}


Du har en kodesnutt som henter ut `856 $a` for 856-felt med `856 $c = 'Omslagsbilde`:

~~~
from pprint import pprint
for rec in records:
    # Variabler vi vil hente fra hver post:
    mms_id = ''
    tittel = ''
    har_omslagsbilde = False
    omslagsbilde_url = ''
    for field in rec['controlfield']:
        if field['@tag'] == '001':
            mms_id = field['#text']
    for field in rec['datafield']:
        if field['@tag'] == '245':
            for subfield in field['subfield']:
                if subfield['@code'] == 'a':
                    tittel = subfield['#text']
        if field['@tag'] == '856':
            for subfield in field['subfield']:
                if subfield['@code'] == 'a':
                    omslagsbilde_url = subfield['#text']
                if subfield['@code'] == '3' and subfield['#text'] == 'Omslagsbilde':
                    pprint(field)
                    har_omslagsbilde = True
    if har_omslagsbilde == True:
        print(tittel + " har omslagsbilde")
    else:
        print(tittel + " har ikke omslagsbilde")
~~~
{: .language-python}

Og en funksjon som laster ned en fil:

~~~
def download_file(url, filename):
    res = requests.get(url, stream=True)
    with open(filename, 'wb') as fd:
        for chunk in res.iter_content(chunk_size=1024):
            fd.write(chunk)

~~~
{: .language-python}

Prøv å bruk funksjonen for å laste ned omslagsbildet.


Ideer: Hente inn RSS fra Springer, sjekke hvilke bøker som finnes i katalogen,
supplere med omslagsbilde..
-->

<!--
## Andre API-er

- [Wikipedia](https://en.wikipedia.org/api/rest_v1/) (ett av flere)
- [OCLC](https://www.oclc.org/developer/home.en.html)
- [ExLibris](http://developers.exlibrisgroup.com)

Se også:

 - [Bibsys sin dokumentasjon](https://www.bibsys.no/wp-content/uploads/2016/11/BIBSYS_Alma_SRU.pdf)

Network zone gir tilgj. for alle bibliotek, men ikke samlingsinfo:

https://bibsys.alma.exlibrisgroup.com/view/sru/47BIBSYS_NETWORK?version=1.2&operation=searchRetrieve&recordSchema=marcxml&query=alma.isbn=%209788243011182

Eksempler på andre API-er dere nå er klare til å utforske
* http://api.cristin.no/
* https://authority.bibsys.no/authority/
* https://dev.elsevier.com/api_docs.html



Eksempel på generering av MARCXML med `xmlwitch`:
https://github.com/uio-library/hansteen-brev/blob/master/scripts/json2marc.py

-----------------------------------

- Søk i Cristin og Scopus etter forfatter.

> ## Tips
> it may include some code
{: .callout}


## Overskrift

### Underoverskrift

Kodeeksempel:

~~~
print("Hello world")
~~~
{: .language-python}


## Byggeklosser

Vi bygger på eksisterende byggeklosser.


> ## Oppgavetittel
> Oppgavetekst
>
> > ## Løsning
> > ~~~
> > print("Hello world")
> > ~~~
> > {: .language-python}
> > Mer tekst.
> {: .solution}
{: .challenge}

-->

