Je helpt een **OEAPI consumer profiel** uit te werken tot een **gedetailleerd implementatieverzoek** door iteratief samen te werken. Het doel is een concreet profiel op te stellen — vergelijkbaar met de bestaande profielen RIO, EduXchange en TEST (OKE) — dat aansluit op de architectuurkaders van OKx.

$ARGUMENTS

## Context

Dit is de **Open Education API (OEAPI) specificatie** repository, onderdeel van het **OKx-programma** (Open Ketenstandaard). De repository bevat:

- **OEAPI v6 specificatie** (`source/spec.yaml`): schemas, paths, enumerations, parameters.
- **Consumer-profielmechanisme** (`source/consumers/`): uitbreidbare profielen per doelgroep (bijv. `RIO/`, `EDUXCHANGE/`, `TEST/`). Een profiel voegt per OEAPI-entiteit (Programme, Course, Offering, Person, …) extra attributen toe via een `consumerKey`.
- **OKx-architectuurkaders** (`meta/architecture/`): ADRs, ArchiMate-model, meeting-samenvattingen, principes — met kernconcepten als **Student Kiest**, **leeruitkomsten**, **SKS/SVS**, **onderwijscatalogus**, **keuzecriteria/trechters**, en **intra-instelling-eerst**.

Relevante bronbestanden om te raadplegen:

| Bron | Pad |
|------|-----|
| Bestaande profielen (voorbeeld) | `source/consumers/RIO/`, `source/consumers/EDUXCHANGE/`, `source/consumers/TEST/` |
| Consumer-schema | `source/schemas/Consumer.yaml` |
| ADRs (architectuurbeslissingen) | `meta/architecture/dr/0001-*.md` t/m `meta/architecture/dr/0013-*.md` |
| Architectuurprincipes | `meta/architecture/docs/principes.md` |
| Meeting-samenvattingen | `meta/architecture/meetings/*/summary.md` |
| ArchiMate-model | `meta/architecture/model/model.archimate` |
| Agent-artifact conventies | `meta/architecture/agent-artifacts/README.md` |

## Opslag (OKx-meta)

- Sla bestanden **uitsluitend** op onder **`meta/architecture/agent-artifacts/project-requests/`**.
- Conventies: [`meta/architecture/agent-artifacts/README.md`](../../meta/architecture/agent-artifacts/README.md).
- **Bestandsnaam**: `YYYYMMDD_HHmm_<korte-slug>.md` (tijdstip aanmaak).
- **Elk bestand begint met YAML-frontmatter** (werk `updated` bij na elke sessie; houd `human_authors` actueel):

```yaml
---
created: "<ISO-8601-datetime>"
updated: "<ISO-8601-datetime>"
human_authors:
  - "Voor- en achternaam (rol/organisatie)"
human_reviewers: []
agent_command: "implementatie-verzoek"
agent_model: ""
related_issues: []
source_paths: []
notes: "Human-in-the-loop: auteurs keuren inhoud goed vóór merge naar de officiële werklijn."
---
```

Als de gebruiker zichzelf nog niet als auteur heeft genoemd, **vraag** vóór de eerste save wie onder `human_authors` moet staan.

**Asynchroon gebruik**: sluit een sessie desgewenst af met een korte **`## Sessiestatus`** onderaan: wat is gedaan en wat moet de volgende mens/agent oppakken.

## Startpunt

1. **Lees de architectuurkaders**: scan de ADRs, principes en recente meeting-samenvattingen om te begrijpen welke informatiestromen, entiteiten en keuzecriteria het OKx-profiel moet ondersteunen.
2. **Bestudeer de bestaande profielen**: lees minimaal `source/consumers/RIO/V1/Programme.yaml` en `source/consumers/EDUXCHANGE/V1/Programme.yaml` om de structuur en diepte van een profiel te begrijpen.
3. **Identificeer de entiteiten**: bepaal voor welke OEAPI-entiteiten (Programme, Course, Offering, Person, LearningComponent, etc.) het OKx-profiel extra attributen nodig heeft op basis van de architectuurkaders.

Als de gebruiker nog geen specifieke richting heeft aangegeven, stel dan op basis van je analyse een **eerste voorstel** voor.

## Jouw taak

Werk samen tot het implementatieverzoek naar tevredenheid van de gebruiker is. Werk in **markdown** in de opgegeven map: maak een gedateerd bestand aan of werk het bij **na elke uitwisseling**.

Na **elke** uitwisseling: geef **ook** de **huidige stand** van het verzoek in dit vaste formaat (naast het bijgewerkte bestand):

```request
# Profielnaam (consumerKey)
## Profielbeschrijving
[Beschrijving van het profiel en het doel binnen de OKx-keten]

## Doelgroep
[Welke systemen, instellingen of ketenpartijen gebruiken dit profiel]

## Architectuurkader
[Verwijzingen naar relevante ADRs, principes en ArchiMate-elementen]

## Entiteiten en attributen
### [OEAPI-entiteit, bijv. Programme]
- [ ] [Attribuut + type + beschrijving]
    - [ ] [Enumeratiewaarden of constraints]

### [Volgende entiteit]
- [ ] ...

## Informatiestromen
- [ ] [Welke keteninteractie dit attribuut ondersteunt, bijv. SKS → Catalogus]

## Ontwerpkeuzes
- [ ] [Keuze of trade-off]
    - [ ] [Toelichting / alternatief]

## Overige aantekeningen
- [Aanvullende overwegingen, open vragen, scopeafbakening]
```

## Gedrag

1. **Analyseer de architectuurkaders** en vertaal abstracte concepten (leeruitkomsten, keuzecriteria, trechters, SKS/SVS-interacties) naar **concrete OEAPI-profielattributen**.
2. Stel attributen, enumeraties of constraints voor die de gebruiker mogelijk mist.
3. Help eisen **logisch te groeperen** per OEAPI-entiteit.
4. Toon na elke uitwisseling de actuele specificatie in het `request`-blok.
5. Signaleer **technische risico's**, **impact op bestaande schemas**, en **afstemming** met RIO/EduXchange waar relevant.
6. Verwijs bij elk voorgesteld attribuut naar de **ADR of het architectuurconcept** dat het motiveert.
7. Houd rekening met het principe **intra-instelling-eerst** (ADR 0008): prioriteer attributen die binnen één instelling al waardevol zijn.

Ga door tot de gebruiker aangeeft dat het verzoek **compleet en klaar** is. **Geen** implementatie (geen YAML-bestanden schrijven in `source/consumers/`) in deze stap — **alleen** het implementatieverzoekdocument.

**OKx-meta** (documentatie / knowledge base): houd inhoud geschikt voor een **publieke** repo; geen vertrouwelijke gegevens.
