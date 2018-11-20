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

![API](/fig/api-figur.png)

- API (application programming interface) eller *programmeringsgrensesnitt* på godt norsk,
  et grensesnitt mellom ulike systemer.
   - API-er kan være mellom forskjellige programmer på datamaskinen din eller mellom
     webtjenester. Det siste kalles også web-API-er eller webservices / webtjenester,
     men vi kommer bare til å kalle det API-er.
   - «REST» eller «RESTful» er et buzzord fra en del år tilbake som fremdeles er mye brukt
     i beskrivelser av API-er. De aller, aller fleste API-er følger disse prinsippene
     i større eller mindre grad i dag, og det er ikke noe spesielt spennende ved dem.
- Grensesnitt for å kommunisere mellom systemer ved hjelp av
  - HTTP-kall: GET for å hente data, POST for å sende inn data
  - Standardiserte dataformater som JSON og XML.
- Åpner for å kunne automatisere og trekke ut data fra ulike systemer og kombinere dem
- Som regel rimelig stabile over tid (mer stabile enn f.eks. vanlige nettsider)

<!--
  Liten digresjon: [slipsomat](https://github.com/scriptotek/alma-slipsomat)
  -- Python-script som bruker selenium for å automatisere nettleserarbeid.
  Eksempel på hva man gjør når man ikke har et API.
  Tidkrevende og sårbart.
-->

### XML og JSON

`<xml>` | `{json}`
---|---
utviklet i 1997 | utviklet i 2001
minner om html | avledet fra javascript
mer komplisert | mindre komplisert

![API](/fig/xml-json-all-time1-600x382.png)

Generelt er JSON enklere å jobbe med,
men begge formatene har sine styrker og i praksis må du forholde deg til begge deler.

# HTTP-forespørsler (HTTP-kall) med requests

Vi begynner med å importere `requests`, som er et kodebibliotek for å gjøre HTTP-forespørsler (HTTP requests):

~~~
import requests
~~~
{: .python}

HTTP-forespørsler er ekstremt lite mystisk – det er det vi gjør hver gang vi henter en nettside.
For å illustrere dette kan vi begynne med å hente forsiden til BI med `requests` slik:

~~~
requests.get('http://www.bi.no')
~~~
{: .python}

Her kjører vi funksjonen `get()` fra `requests`-biblioteket,
som igjen kaller et annet bibliotek som igjen kaller et system-API
osv. osv. Lag på lag med komplisert nettverksfunksjonalitet
som vi heldigvis slipper å forholde oss til!
Det vi får ut igjen er et responsobjekt, som vi kan gi et eller annet navn,
f.eks. "response":

~~~
response = requests.get('http://www.bi.no')
~~~
{: .python}

Et objekt er en sekk som kan inneholde forskjellige ting.
Vi kan liste ut hva objektet inneholder med `dir()`-funksjonen:

~~~
dir(response)
~~~
{: .python}

Her ser vi f.eks. at det ligger noe som heter "text", som er responsteksten vi
har fått tilbake fra serveren.
Selv om det går an å bruke `dir()` for å grave i et objekt er det i praksis ofte
enklere å lese dokumentasjonen.

Uansett har vi funnet ut at det er `response.text` vi er interessert i:

~~~
response.text
~~~
{: .python}


Det vi får ut er det samme som vi ville fått ut hvis vi hadde
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
{: .python}

Det vi gjør her er at vi bruker en funksjon som noen andre har laget,
slik at vi slipper å skrive denne funksjonen.
-->

I prinsippet kunne vi prøvd å trekke ut data fra HTML-koden, da er vi
inne i det som kalles webscraping.
Det er gjerne noe man tyr til hvis det ikke finnes et API.

<!-- Eks: Hente åpningstider fra en Facebook-side. ResearchGate, Instagram eller andre kjipe nettsteder -->

Når vi bruker et API får vi derimot ut data i et maskinvennlig format (typisk JSON eller XML).
API-er er nesten alltid dokumenterte (selv om kvaliteten på dokumentasjonen kan variere)
og de skal i utgangspunktet ikke endre seg uten forvarsel.

API-er brukes ikke bare til å hente ut data, men også endre data.
Dette krever en eller annen form for autentisering, f.eks. en hemmelig nøkkel som man legger ved hver forespørsel.

Alt dette er bra!

# Hente bibliografiske data fra Alma

Som eksempel på API skal vi hente ut bibliografiske poster over et SRU-API.
SRU er en standard som de fleste biblioteksystemer støtter.
Vi skal bruke Alma som eksempel.

<!-- Advarsel: MARC er ikke den enkleste datamodellen å jobbe med,
     og egentlig ikke det mest pedagogiske eksempelet å begynne med.
     Men vi hadde lyst til å dykke litt ned i den skitne virkeligheten.
     Men når du jobber i bibliotek er det veldig vanskelig å komme unna MARC,
     og siden dette er biblioteksløsyd så synes vi at vi måtte ha det med.
     Det finnes svært gode nettkurs for å lære programmering.
     -->

Første spørsmål når vi skal bruke et API er: hvordan bruker vi det?

Vi er interessert i å hente ut bibliografiske poster fra biblioteksystemet vårt,
og vi skal bruke Alma som eksempel, men vi skal bruke SRU, som er en standardisert søkeprotokoll
som støttes av de fleste biblioteksystemer.

API-dokumentasjonen for Almas SRU-tjeneste finnes på
[https://developers.exlibrisgroup.com/alma/integrations/SRU](https://developers.exlibrisgroup.com/alma/integrations/SRU).

La oss begynne med å prøve lenken «searchRetrieveResponse» under overskriften «Examples (in Alma's Guest sandbox environment)» helt nederst.

Denne fører hit:

https://na01.alma.exlibrisgroup.com/view/sru/TR_INTEGRATION_INST?version=1.2&operation=searchRetrieve&recordSchema=marcxml&query=alma.all_for_ui=history&maximumRecords=3&startRecord=4

Vi får ut noe data fra gjestesandkassen, men hvordan får vi det til å virke med våre data?
Vi kan gå tilbake til dokumentasjonen, der det står «The base URL for SRU requests is: https://<Alma domain>/view/sru/<institution code>».
Alma-domenet vårt kan vi finne fra [listen over Alma-instanser](https://www.bibsys.no/direktelenker-til-alma-instansene/).

Prøver med `bibsys.alma.exlibrisgroup.com` og `47BIBSYS_NETWORK`

https://bibsys.alma.exlibrisgroup.com/view/sru/47BIBSYS_NETWORK?version=1.2&operation=searchRetrieve&recordSchema=marcxml&query=alma.all_for_ui=history&maximumRecords=3&startRecord=4


Vi prøver i Python:

~~~
response = requests.get('https://bibsys.alma.exlibrisgroup.com/view/sru/47BIBSYS_NETWORK?version=1.2&operation=searchRetrieve&recordSchema=marcxml&query=alma.all_for_ui=history&maximumRecords=3&startRecord=4')
print(response.text)
~~~
{: .python}

Det funka! Men det ser rotete ut. Det som følger etter spørsmålstegnet kalles
[query string](https://en.wikipedia.org/wiki/Query_string) (spørrestreng) og
inneholder parametre som vi sender til API-et.
Vi vil skille disse ut ut parametrene som variabler så vi enklere kan jobbe med dem:

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
{: .python}

Resultatet av å kjøre denne koden er nøyaktig det samme som resultatet fra å kjøre det forrige eksempelet,
men nå har vi en kodesnutt som er lettere å bearbeide videre.

Et nytt steg som kunne være naturlig nå er å gjøre kodesnutten om til en funksjon.

Tips: Marker flere linjer tekst og trykk Tab for å indentere alle linjene samtidig.

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
{: .python}

<!--
  Dette illustrerer også et lite poeng: Man skriver typisk ikke et program fra linje 1 og nedover.
  Dette er kanskje.
-->

Nå kan vi plutselig gjøre et nytt søk med bare én linje kode:

~~~
response = sru_search_request('alma.creator="samset bjørn"')
print(response.text)
~~~
{: .python}

### Hjelp, hva gjør jeg med all denne XML-en?

Neste spørsmål er hvordan skal vi navigere i dette havet av tekst som vi får tilbake fra API-et.
Dette er biblioteksløyd, så vi skal finne frem et nytt verktøy,
nemlig et bibliotek som parser XML.
Det finnes mange forskjellige (google "python xml"), og de har ulike fordeler og ulemper.
I dag skal vi bruke et kalt `xmltodict`, som gjør XML-teksten om til en såkalt dictionary (ordliste)
eller dict, som gjør at vi kan jobbe med responsen på samme måte som vi ville gjort fra et JSON-API.

> xmltodict: Hva er forresten en dict?
> En ordbok / [dictionary](https://dbader.org/blog/python-dictionaries-maps-and-hashtables) (dict) er en sentral datatype i Python
> (i andre programmeringsspråk ofte kalt hashmap, associated array e.a.).
> En dict er et enkelt objekt som du kan putte ulike ting oppi,
> og der hvert ting har en etikett, som du kan bruke til å enkelt finne tingen igjen.
> .callout

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
{: .python}

Når vi ser på XML-en ser vi at toppnivået heter "searchRetrieveResponse".
Deretter følger "records" og "record". Denne noden er det flere av, så det er en liste.
Hver "record"-node har en "recordData"-node som igjen har en "record"-node.

![XML-strukturen til responsen](/fig/xml-response.png)

La oss prøve å navigere i XML-strukturen med Python.
Først må vi importere `xmltodict`:

~~~
import xmltodict
~~~
{: .python}

Vi ønsker å lage en liste som inneholder alle MARC-postene,
men ikke alt det som ligger rundt. Det kan vi gjøre slik:

~~~
xml = xmltodict.parse(response.text, force_list=['records', 'datafield', 'subfield'])

records = []
for rec in xml['searchRetrieveResponse']['records']['record']:
    records.append(rec['recordData']['record'])
~~~
{: .python}

I parentes: Hvis vi er nysgjerrige på hvor mange poster vi har i listen vår kan vi sjekke med `len()`-funksjonen:

~~~
len(records)
~~~
{: .python}


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
> >     xml = xmltodict.parse(response.text, force_list=['records', 'datafield', 'subfield'])
> >     records = []
> >     for rec in xml['searchRetrieveResponse']['records']['record']:
> >         records.append(rec['recordData']['record'][0])
> >     return records
> > ~~~
> > {: .python}
> {: .solution}
{: .challenge}

> ## Oppgave 2: Hente ut leader fra den første MARC-posten
> ![MARC](/fig/marc.png)
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
> > {: .python}
> >
> > (b)
> > ~~~
> > for record in records:
> >     print(record['leader'])
> > ~~~
> > {: .python}
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
> {: .python}
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
> > {: .python}
> {: .solution}
{: .challenge}



~~~
from pprint import pprint
for rec in records:
    for field in rec['controlfield']:
        if field['@tag'] == '001':
            print(field['#text'])
    for field in rec['datafield']:
        if field['@tag'] == '082':
            for subfield in field['subfield']:
                if subfield['@code'] == 'a'
                print(subfield['#text'])
~~~
{: .python}

Oppgave:
b) Slå dem opp mot `https://ub-www01.uio.no/microservices/dewey_heading.php?number=401.4`
eller http://deweysearchno.pansoft.de/webdeweysearch/rest?query=113 (funker ikke nå)

> ## Jobbe med MARC-poster 1
> Ta utgangspunkt i koden ovenfor og lag en funksjon kalt `filter_records` som
> returnerer poster


> (a) Hente ut alle Dewey-numre (082 a)
>
> > ## Løsning
> > ~~~
> > print("Hello world")
> > ~~~
> > {: .python}
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
{: .python}

Og en funksjon som laster ned en fil:

~~~
def download_file(url, filename):
    res = requests.get(url, stream=True)
    with open(filename, 'wb') as fd:
        for chunk in res.iter_content(chunk_size=1024):
            fd.write(chunk)

~~~
{: .python}

Prøv å bruk funksjonen for å laste ned omslagsbildet.


Ideer: Hente inn RSS fra Springer, sjekke hvilke bøker som finnes i katalogen,
supplere med omslagsbilde..



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
{: .python}


## Byggeklosser

Vi bygger på eksisterende byggeklosser.


> ## Oppgavetittel
> Oppgavetekst
>
> > ## Løsning
> > ~~~
> > print("Hello world")
> > ~~~
> > {: .python}
> > Mer tekst.
> {: .solution}
{: .challenge}



