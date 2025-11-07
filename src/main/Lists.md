# Intervjuer-notater

Noen innspill til den som skal intervjue:

Formålene med de forskjellige oppgavene, i kort, er å avdekke hvor vant kandidaten er med programmering,
gitt at Kotlin er valgt som programmeringsspråk.

Oppgaven begynner veldig enkelt, men trekker fort inn sentrale tema som høyereordens funksjoner og objektorienterte
konstruksjoner som grensesnitt (`interface`). Det er antagelig mange måter å gå seg bort på, og i en stresset
intervjusituasjon går IQ-en ned 5-15 poeng.  (Vitenskapelig bevist!  Det står i alle fall skrevet.)

Det er altså ikke realistisk at alle skal komme seg gjennom hele øvelsen, derfor bør vi ikke legge ut hele løypa og si "
kjør", men heller introdusere stegene etterhvert som kandiaten løser dem. Når tiden er ute, er den ute, og ingen trenger
ikke si noe om hvor langt man kom. Det er kjedelig om noen føler at de har kommet til kort, dessuten bør vi innhente
flere erfaringer om forventningsnivået som er realistisk på tilmålt tid.

Kommentarer til øvelsene:

## Find sublist, Find smallest element

Her vil vi sjekke at helt grunnleggende programmeringsferdigheter er til stede: Skrive en funksjon,
behandle en input, returnere et svar.

Burde forvente at kandidaten er komfortabel med utviklingsmiljøet, kjenner enkel syntaks for `fun` etc., uten for mye
håndholding.

Andre punkter:

* Om løsningen er Java-aktig imperativ kode, spør om den kan løses uten løkker
    * Hint om `filter`/`min`
* Om løsningene baserer seg på sortering, spør kandidaten om tidskompleksiteten av løsningen
    * Traversering vil ha O(n) som kompleksitet, hvordan blir det ifht. sortering?

Om kandidaten hevder å kunne Kotlin, bør man forvente bruk av `filter` og aller helst `min`.

## Find stats

Her tykner det seg litt til: Vi må returnere en verdi som implementerer et grensesnitt. Typiske løsninger er å
returnere et `object` eller lage en `data class` som holder på verdiene. (En `data class` med de samme `val`-ene vil
implisitt implementere grensesnittet.)

Man bør forvente at en Kotlin-programmerer ikke har noe problem med å implementere et grensesnitt, omtrent på samme nivå
som å skrive en `fun`.

Det er nå fire verdier som skal lages. En enkel løsning er å traversere listen fire ganger på forskjellige måter. Det er
greit for denne runden.

Hvis man vil nerde litt rundt Kotlin, så kan man diskutere hvordan man kan returnere en `Stats` som beregner svarene
sine på forespørsel, f.eks.:

* on-demand/hver gang man spør (`override val min get() = list.min()`)
* lazy/kun første gang man spør (`override val min: Int by lazy { list.min() }`)

## Find stats with provider

Det er ca. her det begynner å bli interessant.

Her snur vi ting litt på hodet, og tilbyr vår funksjon `stats` en _annen_ funksjon i stedet for en liste. Denne
funksjonen vil returnere ett og ett tall fra en serie, og avslutte med `null`. Påskuddet er at vi ikke lenger kan holde
hele listen i minnet på én gang.

Den store konsekvensen av denne endringen er at vi ikke lenger kan naivt traversere listen flere ganger. I stedet må vi
bygge opp alle svarene i løpet av ett sveip.

Noen poenger til diskusjon/hinting:

* Om kandidaten kjenner Java, bør dette minne om forskjellen på f.eks. en `Stream<T>` og en `List<T>`
* Hvordan utføres en `List::map` i Kotlin vs. en `Stream::map` i Java?
* Har Kotlin noe lignende?  (Ja, vi har `Sequence<T>`)

I tillegg skjer det noen type-endringer, da summen blir til `BigInteger` i stedet for `Long`.
Dette krever noen typekonverteringer, noe Kotlin har god støtte for om man leter seg litt frem i standardbiblioteket.

Dette er en mulig snublestein, men det kan fungere som en liten test av hvor komfortabel man er med å finne frem i
tredjepartsbiblioteker. Hvis kandidaten blir unødig opphengt i disse detaljene, er det nok best å hjelpe vedkommende
over kneika med disse, da det ikke er det mest interessante med oppgaven. Pluss for å kjenne detaljene, større
pluss for å finne ut av det selv.

### Hjelpefunksjonen for testing

Som en liten oppvarming til dette må vi ha en hjelpefunksjon som kan konvertere en liste til en funksjon på formen
`() -> Int?`.

Dette kan løses på mange måter, men krever kjennskap til høyereordens funskjoner: Vår
hjelpefunksjon må returnere en _ny_ funksjon. Denne skal sendes til `stats` for å representere listen av tall.

Hvis tiden løper ut, eller kandidaten sliter og ikke er plagsomt god på høyereordens funksjoner, kan det være greit å
bare gi en definisjon av funksjonen og hoppe til neste del, da den også har en imperativ løsning.

### Strategier for å bygge opp svaret:

Det er flere måter å bygge opp svaret, de to viktigste er:

#### Imperativ stil

Deklarer fire temporære variabler som oppdateres for hvert tall:

1. Minste tall til nå
2. Største tall til nå
3. Sum så langt
4. Antall så langt

Da kan man til slutt regne ut snittet og returnere verdiene. Dette fungerer bra i imperativ stil.

Diskusjonspunkter:

* Om man har en sterk kandidat kan man spørre hvordan det kan gjøres funksjonelt.  ("Hvis du ikke fikk lov å bruke en
  løkke ...")
* `fold` er en naturlig funksjon (ekv. med `reduce` i Java), hint gjerne om denne

#### Funksjonell (objektorientert?) stil

Definer f.eks.:

* en ikke-muterbar `data class` som holder på tilstandene
* (denne kan gjerne implementere Stats i seg selv)
* en medlems-funksjon på denne (eller en extension-funksjon) som tar ett tall og returnerer en oppdatert instans:
  `stats.add(1)` gir en oppdatert `stats'`

Diskusjonspunkter:

* Dette er en mer elegant løsning, men medfører mer minnebruk og gc-aktivitet
* Om man har en kjempesterk kandidat, kan man diskutere en halerekursiv løsning!

### Strategier for å iterere

På ett eller annet vis må vi kalle funksjonen til den begynner å returnere `null`. For hvert kall må vi bygge opp en
tilstand, og deretter returnere den.

Også her er imperativ stil en mulighet. En `while`-løkke kaller funksjonen frem til den svarer `null`. Dette fungerer
fint, uansett hvordan man velger å bygge opp svaret.

Men funksjonell stil er det vi prøver å hinte til her!  For de som kjenner Kotlin godt, er det et hint i typesignaturen
`() -> Int?`. Det finnesdet nemlig en `generateSequence`-funksjon i standardbiblioteket, som tar nettopp funksjoner på
denne formen og returnerer en `Sequence<T>`.

Sekvenser i Kotlin tilsvarer streams i Java, men mange tenker ikke over å bruke dem, siden kolleksjoner i Kotlin har
sine `filter`/`map`-metoder osv. Det kan være interessant å diskutere denne forskjellen med kandidaten; å kalle
`filter` på en Kotlin-`List` produserer en ny liste der og da, og prosesserer alle elementene. Men `filter` på en
`Sequence` gir en ny sekvens, der ingenting evalueres før man kaller en _terminerende_ funksjon på sekvensen.

Uansett: Med en slik sekvens på plass kan man enkelt `fold`-e seg gjennom strømmen. (Som er en terminerende funksjon.)

## Provide a hook

Dette er en generalisering av `stats`, kalt `process`. Det handler i det store og hele om å abstrahere
tilstandsoppbyningen og skille traverseringen ut fra denne.

Med de byggeklossene vi har på plass allerede, og en kandiat som har kommet seg helt hit, burde dette ikke være noe
problem. I så fall På en annen side, å komme seg helt hit kan ha vært mentalt utmattende, og IQ-en er gått ned enda noen
poeng. 