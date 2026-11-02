---
name: hent-ledige-stillinger
description: Finn ledige stillinger som matcher en persons CV/bakgrunn og send resultatet som en oversiktlig liste på e-post. Bruk denne skillen når brukeren ber om å finne, søke opp eller hente ledige jobber/stillinger basert på CV-en sin, nevner å jobbsøke og vil ha en liste tilsendt, eller ber om å "hente ledige stillinger", "finne relevante jobber til meg", "søke opp stillinger som matcher bakgrunnen min" e.l. — selv om de ikke bruker akkurat de ordene. Trigger også når brukeren laster opp en CV/bakgrunnsdokument sammen med en jobbsøk-relatert forespørsel.
---

# Hent ledige stillinger

Denne skillen automatiserer arbeidsflyten med å finne relevante stillingsannonser
basert på en persons CV/bakgrunn, og sende dem videre som en strukturert liste på
e-post. Dette er en research- og kommunikasjonsoppgave, ikke en kodeoppgave — bruk
vanlige verktøy (Read, WebSearch, Gmail) fremfor å skrive skript.

## Fremgangsmåte

### 1. Les CV og bakgrunnsdokumenter

Bruk Read-verktøyet på opplastede filer (PDF/docx). For binære `.docx`-filer som
Read ikke støtter direkte, pakk ut tekst med en liten Python-oneliner via Bash
(zipfile + XML-parsing av `word/document.xml`) — dette er raskere og mer pålitelig
enn å be brukeren konvertere filen selv.

Trekk ut:
- Utdanning
- Arbeidserfaring (roller, bransjer, ansvarsområder)
- Kjernekompetanse / ferdigheter
- Nåværende bosted/arbeidssted (styrer standard søkeområde)

Hvis brukeren ikke har lastet opp noe CV/bakgrunnsdokument ennå, spør om dette
først — uten det kan ikke relevans vurderes.

### 2. Avklar søkeparametere

Hvis ikke allerede oppgitt av brukeren, spør (gjerne med AskUserQuestion) om:
- **Geografisk område** (f.eks. en by, et fylke, "hele Norge")
- **Rolletyper/kategorier** som er aktuelle, basert på CV-en (foreslå 3–5 alternativer
  utledet fra kompetansen/erfaringen, merk det mest opplagte som anbefalt)
- **Hvilke jobbsider** som skal søkes (standard: Finn.no og NAV Arbeidsplassen/
  arbeidsplassen.nav.no — de to største jobbportalene i Norge)

Ikke gjett deg frem til brede antagelser på områder som endrer resultatet mye
(sted og rolletype) — dette er verdt et raskt spørsmål.

### 3. Søk etter stillinger

Bruk WebSearch med målrettede søk som kombinerer rolle + sted + kilde, f.eks.:
- `site:finn.no <rolle> <sted>`
- `finn.no "<stillingstittel>" <kommune/fylke> ledig stilling`
- `arbeidsplassen.nav.no <rolle> <fylke>`

Kjør flere søk parallelt (ett per rolletype/kategori) for effektivitet.

**Nettverksbegrensning:** WebFetch direkte mot `finn.no` eller
`arbeidsplassen.nav.no` kan være blokkert av sandkasse-miljøets nettverkspolicy
(egress proxy). Hvis WebFetch feiler med `EGRESS_BLOCKED`:
- Ikke gi opp — fortsett med WebSearch alene, som ofte går utenom sperren.
- Fortell brukeren kort at direkte tilgang er blokkert, og at de kan endre dette i
  miljøinnstillingene (nettverkstilgang → tillatte domener) hvis de ønsker en mer
  fullstendig/oppdatert liste via direkte sidehenting.
- Hvis brukeren sier "bare fortsett", gå videre med WebSearch-resultatene uten å
  spørre igjen.

### 4. Vurder relevans og kvalitet

For hvert treff:
- Sammenlign mot CV-ens utdanning, erfaring og kompetanse — inkluder kun reelt
  relevante stillinger.
- **Bruk aldri diktede eller gjettede URL-er.** Ta kun med lenker som faktisk kom
  frem i søkeresultatene. Er du usikker på om en lenke er reell/gyldig, heller
  utelat den enn å gjette.
- Flagg annonser som virker gamle eller mulig utløpt (f.eks. publisert for lenge
  siden, eller søkeresultatet antyder at den ikke lenger er aktiv) i stedet for å
  stille dem opp som like ferske som resten.

### 5. Sett sammen listen

For hver stilling, oppgi:
- Stillingstittel
- Bedrift/arbeidsgiver
- Sted
- Kort (én linje) begrunnelse for hvorfor den er relevant for denne personen
- Direkte lenke til annonsen

Grupper gjerne etter rolletype/kategori for lesbarhet. Avslutt gjerne med en kort
anbefaling om hvilke 1–2 stillinger som peker seg mest ut, og lenker til de fulle
søkesidene (f.eks. Finn.no-søk for hele området) slik at brukeren selv kan
utforske videre utover det som ble funnet.

### 6. Send som e-post

Bruk `mcp__Gmail__send_message` til brukerens e-postadresse. Skriv e-posten på
samme språk som brukerens forespørsel (typisk norsk). Inkluder et kort forbehold
øverst i e-posten dersom søket var basert på WebSearch alene (uten direkte
sidehenting), slik at mottakeren vet at søknadsfrister bør dobbeltsjekkes direkte
på annonsen.

Bekreft til slutt i chatten at e-posten er sendt, og oppsummer kort hva som ble
inkludert (antall stillinger, kategorier, eventuelle forbehold).
