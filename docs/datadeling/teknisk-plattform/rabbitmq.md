---
image: /prosjekter/datadeling/arbeidsomrader/integrasjonsarkitektur/dokumentasjon/img/rabbitmq-logo.png
title: RabbitMQ
---

RabbitMQ er tjeneste for å administrere meldingskøer, og forvalte din tjenestes
meldinger. Så du slipper.


## Kom i gang

For de utålmodige:

1. Bruk [Selvbetjeningsportalen for
   RabbitMQ](/docs/datadeling/teknisk-plattform/brom) for å sette opp
   meldingshåndteringen for din tjeneste. 
1. Hent ut tilkoblingsdetaljer for din tjeneste fra selvbetjeningsportalen.
   Bruk dette i din tjeneste for å både publisere og motta meldinger.

Anbefalt meldingsprotokoll: [AMQP 0.9.1](http://www.amqp.org/specification/0-9-1/amqp-org-download) på port 5671 (TLS).

Se [kode-eksempler](/docs/datadeling/kode/):

* Hvordan sende notifikasjoner (datatilbyder):  [publisering\_simpel.py](/datadeling/publisering_simpel.py)
* Hvordan motta notifikasjoner (konsument): [konsument\_simpel.py](/datadeling/konsument_simpel.py)

## Hva er RabbitMQ?

RabbitMQ er en tjeneste for å håndtere meldinger og meldingskøer. Systemet ble
valgt i IntArk fordi det følger [IntArk sine
prinsipper](/docs/datadeling/prinsippene), spesielt med god støtte for åpne
standarder i stedet for sine egne, proprietære løsninger.

Datatilbydere kan sende inn meldinger fra sin tjeneste til RabbitMQ. RabbitMQ
tar seg av å fordele disse meldingene videre til alle konsumenter som abonnerer
på meldingene. Datatilbyder trenger da ikke å måtte forholde seg til hvor mange
konsumenter det er, eller for eksempel å måtte sende ut meldinger på nytt fordi
en konsument har mistet sin melding. Alt dette håndteres av RabbitMQ.

![Melding fra datatilbyder til konsumenter](/datadeling/img/rmq-exchange.png)

Vi bruker protokollen AMQP for notifikasjoner, men RabbitMQ støtter også andre
protokoller for meldingsutveksling.

RabbitMQ er primært satt opp for å støtte integrasjonsmønsteret
[hendelsesbasert provisjonering](/docs/datadeling/god-praksis/integrasjonsmonster/hendelsesbasert),
med [notifikasjoner (tynne meldinger)](/docs/datadeling/god-praksis/notifikasjonsdesign),
men kan også brukes for andre typer meldinger og integrasjoner.


## Hvordan bruker jeg RabbitMQ?

Vi anbefaler å bruke [Selvbetjeningsportalen for
RabbitMQ](/docs/datadeling/teknisk-plattform/brom) heller enn å bruke RabbitMQ
direkte. RabbitMQ krever god teknisk innsikt, så vi har utviklet et eget
grensesnitt i IntArk for å gjøre forvaltningen av meldinger mye enklere.

RabbitMQ har et eget grensesnitt, som primært brukes av teknikere og driftere.
Se [oversikt over teknisk
plattform](/docs/datadeling/teknisk-plattform/oversikt) for URL. Hver tjeneste
får påloggingsdetaljer til RabbitMQ, som er tilgjengelig i
selvbetjeningsportalen. I RabbitMQ kan du blant annet se aktiviteten for dine
køer, sende og hente ut meldinger, og gjøre mer avanserte endringer på
oppsettet av meldingskøer og exchanges.

RabbitMQ gir deg et kraftig verktøy, med mange muligheter og innstillinger.
Dette øker også sjansen for å sette opp meldingshåndteringen ulikt for
tjenester. Selvbetjeningsportalen sikrer at oppsettet blir mer standardisert
mellom tjenestene.


## Tekniske detaljer

Litt mer tekniske detaljer om hva vi bruker i RabbitMQ. Selvbetjeningsportalen
håndterer mye av dette for deg, så du ikke skal kunne ha dyp innsikt i dette.

Kort om noen begreper: En tjeneste publiserer sine meldinger i en *exchange*,
som du kan se på som et postmottak. RabbitMQ ser på hvem som skal ha hvilke av
meldinger fra exchangen, og kopierer disse over til riktige meldingskøer.
Tjenester som abonnerer på meldinger henter disse fra en meldingskø. Når en
tjeneste henter ut en melding, må den "ack"-es for å slettes fra meldingskøen.

### AMQP

I IntArk bruker vi primært meldingsprotokollen [AMQP
0.9.1](http://www.amqp.org/specification/0-9-1/amqp-org-download). Se [RabbitMQ
sin beskrivelse av
begrepene](http://www.rabbitmq.com/tutorials/amqp-concepts.html) for mer
utdypende informasjon.


### Vhosts

En *virtuell host,* eller *vhost*, er RabbitMQ sin måte å skille prosjekt eller
tjenester fra hverandre. Alle entiteter - kø, binding og exchange - vil ligge i
*én* *vhost*.

I IntArk setter vi opp hver tjeneste i sin egen *vhost*, for å isolere dem fra
andre tjenester. Når noen blir gitt tilgang til meldinger fra en tjeneste, blir
disse kopiert mellom vhosts. Selvbetjeningsportalen tar seg av dette.


### Brukere

For å skrive inn til en exchange og lese fra meldingskøen, trenger du en
*bruker* med de rette tilgangene. Hver bruker har tilgang til sin vhost.

Du kan hente ut påloggingsdetaljer for din tjeneste fra selvbetjeningsportalen,
så du kan logge på i RabbitMQ og se hva du har tilgang til.


### Exchange

En *exchange* er et "postkontor" som en avsender sender notifikasjoner til.
Notifikasjonene distribueres (routes) videre derfra, til de forskjellige
*køene* som vil ha den relevante notifikasjonen.

I IntArk har vi en exchange per tjeneste. Meldinger distribueres videre med
metoden **Topic Exchange**.


### Kø


En kø (*queue*) er én konsuments samling av meldinger, eller hvilke typer
notifikasjonar den vil ha (se *Binding*).  *RabbitMQ* videredistribuerer
(router) de notifikasjonene køen vil ha. Konsumenten abonnerer da på denne
køen.


### Binding

Hver kø settes opp med hvilke typer notifikasjoner den vil ha - køen må være
bundet (bound) til en *exchange* med en eller flere nøkler. For *topic
exchange*, går knytningen mot *topic*.

Selvbetjeningsportalen setter opp bindings når søknader om tilganger blir
akseptert.


### Topic

Når *topic exchange* brukes for å distribuere (route) notifikasjoner fra
*exchange* til riktig kø, må avsender markere hver notifikasjon med en *message
routing key (topic)*. RabbitMQ vil se på *topic* og videresende (*route*)
notifikasjonen til de køene som har binding mot samme *topic.*

Strukturen til *message routing key (topic)*:

```
kilde.type.objekt.hendelse

```

der:


* `kilde`: Navnet til avsenderen, for eksempel "cerebrum"
* `type`: Typen melding, for eksempel "event"
* `objekt`: Vanligvis hva entitetstype notifikasjonen/hendelsen gjelder for, for eksempel "account", "person", "group" eller "employee"
* `hendelse`: Hva har skjedd, for eksempel "add", "delete" eller "modify"

Datatilbyder skal vanligvis dokumentere hvilke verdier de bruker. Dette gjør
det enklere for konsumenter å ignorere meldinger som ikke er relevante for de.



## Notifikasjoner

TODO: Flytt alt dette til "god-praksis".

I IntArk bruker vi RabbitMQ primært for det vi kaller **tynne meldinger**,
eller **notifikasjoner**. Notifikasjonene brukes primært til å informere om at
*noe* har skjedd med en bestemt entitet, men en konsument må selv spørre
kildesystemet om mer detaljer. Dette brukes i integrasjonsmønsteret
[hendelsesbasert
provisjonering](/docs/datadeling/god-praksis/integrasjonsmonster/hendelsesbasert).

Datatilbyder må dokumentere sine notifikasjoner i sin institusjons API-katalog.

For nyutviklede system, bør du vurdere å bruke standariserte format, for eksempel [CloudEvents](https://cloudevents.io/).

TODO: Legg inn dømer frå CloudEvents! Resten er mindre viktig.


For eksempel bruker Cerebrum og SAPUiO et notifikasjonsformat som er inspirert av [SCIM](http://www.simplecloud.info/) sitt [utkast til standard på et meldingsformat](https://tools.ietf.org/html/draft-hunt-idevent-scim-00), der vi bruker versjonen med minimalt med informasjon. Et eksempel på hvordan ei melding vil se ut, **omtrent**:



```

{
  // token identifier (unik id for meldingen)
  "jti": "4d3559ec67504aaba65d40b0363faad8",
  // endringer
  "eventUris": [
    "urn:ietf:params:event:SCIM:create"
  ],
  // timestamp
  "iat": 1458496404,
  // issuer
  "iss": "https://cerebrum-uio.uio.no",
  // audience (spreads/kontekst) - gjør det enklere å forkaste meldinger du ikke er interessert i
  "aud": [
   "AD\_account",
   "exchange\_acc@uio.no",
   "LDAP\_account"
  ],
  // URL til entitet (GET)
 **"sub": "https://cerebrum-ws.uio.no/v1/accounts/1234",**
  // Parametre til event - her: hvilke attributter hos objektet som er påviret - gjør det enklere å forkaste meldinger du ikke er interessert i
  "urn:ietf:params:event:SCIM:create":{
    "attributes":["id","name","userName","password","emails"],
  }
}

```

 


Andre system, som FS og TP, bruker et format som er avledet fra [JWT](https://jwt.io)-standarden. Et eksempel på en notifikasjon fra FS om at en person er oppdatert (routing key er i dette tilfelle *no.uio.fs.FS-prod.personer.update*)



```

{"sub": "personer/56154",
 "iss": "FS-prod",
 "iat": "1582344412462",
 "operation": "update",
 "jti": "35380ac0-247d-44b3-9be3-2f2dded049ee",
 "person\_id": "56154"}
```

Det kan og være lurt å ta en avgjørelse om hvordan du skal referere til ressurser i kildesystem. For eksempel er vi bundet til et spesielt domene i dette tilfellet:



```

{"sub": "https://api-waygate.uio.no/eksempeltjeneste/5"}
```

Mens følgende ikke er bundet til en spesifikk maskin:



```

{"sub": "/5",
 "iss": "eksempeltjeneste"}

```

Det samme kan du oppnå med en mer formalisert URN:


```
{"sub": "urn:eksempeltjeneste:5"}
```

I de to siste eksemplene har vi ikke inkludert informasjon om *hvor* en ressurs er lokalisert. Når du henter ressursen må du da velge å slå opp maskinnavnet. Erfaringsmessig letter dette arbeidet med å flytte tjenester mellom forskjellige miljø.



## Se også

* [Selvbetjeningsportalen for RabbitMQ](/docs/datadeling/teknisk-plattform/brom) - vi anbefaler at du bruker denne, og ikke RabbitMQ direkte.
* [RabbitMQ sine egne nettsider](https://www.rabbitmq.com/)
* [RabbitMQ sine tutorials for publisering og konsumering av meldinger](https://www.rabbitmq.com/getstarted.html)
* [RabbitMQ sin beskrivelse av begrepene i AMQP](http://www.rabbitmq.com/tutorials/amqp-concepts.html)
* [Beskrivelse av meldingsprotokollen AMQP 0.9.1](http://www.amqp.org/specification/0-9-1/amqp-org-download)
* [Teknisk oppsett av meldingskøer](/docs/datadeling/teknisk-plattform/mq-oppsett)
* [Integrasjonsmønsteret hendelsebasert provisjonering](/docs/datadeling/god-praksis/integrasjonsmonster/hendelsesbasert)
