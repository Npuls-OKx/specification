---
created: "2026-04-14T15:00:00+02:00"
updated: "2026-04-14T18:45:00+02:00"
human_authors:
  - "Niek Derksen (Architect OKx)"
human_reviewers: []
agent_command: "implementatie-verzoek"
agent_model: ""
related_issues: []
source_paths:
  - "meta/architecture/dr/0002-prioriteitsketen-catalogus-drielagen-fundament.md"
  - "meta/architecture/dr/0003-student-kiest-leeruitkomsten-domeinprincipes.md"
  - "meta/architecture/dr/0004-leeruitkomsten-sbu-ec-logistieke-containergrootte.md"
  - "meta/architecture/dr/0007-keuzecriteria-trechters-onderwijscatalogus.md"
  - "meta/architecture/dr/0008-scope-planning-eerst-intra-instelling.md"
  - "meta/architecture/dr/0011-keuzeniveau-leeractiviteit-leervormen-als-aanbodkenmerk.md"
  - "meta/architecture/dr/0012-leerroute-onafhankelijk-keuzegate-nominaal-maatwerk.md"
  - "meta/architecture/model/model.archimate"
notes: "Human-in-the-loop: auteurs keuren inhoud goed vóór merge. Vierde iteratie — LearningOutcome-voorbeelden met CompetentNL-taxonomieën (vaardigheden + kennisgebieden) toegevoegd."
---

# OKx OEAPI Consumer Profiel — Implementatieverzoek (v3)

## 1. Profielbeschrijving

Een OEAPI **consumer profiel** (`consumerKey: "okx"`) waarmee de **Onderwijscatalogus (OC)** — als centrale referentiecomponent — haar informatiestromen verrijkt met een **complete onderwijsspecificatie**. Niet alleen *wat* er geleerd wordt, maar ook *hoe*, *waarmee*, *door wie*, *waar* en *hoe lang* — op elk niveau van de hiërarchie.

Dit profiel maakt maximaal gebruik van het **recursieve OEAPI-datamodel** en voegt een gestructureerd specificatie-object toe waar de kern onvoldoende is. Het resultaat is bruikbaar voor:
- **Studenten** die top-down (nominaal programma) of bottom-up (zelf samenstellen) kiezen
- **Planners** die moeten bepalen of de instelling een onderwijswens kan realiseren
- **Ontwerpers** die curriculum publiceren naar de catalogus
- **Andere instellingen** die aanbod willen ontvangen en verwerken (interoperabiliteit)

**OEAPI-broncode wordt niet aangepast.** Signaleringen leiden tot OEAPI change requests.

## 2. De Onderwijscatalogus als spil

Het ArchiMate-model positioneert de **OC** als centraal distributiepunt. Alle informatiestromen in scope lopen **door** of **naar** de OC:

```
                          ┌───────────────────┐
  Curriculum ontwerptool ─┤                   ├─▶ SKS (passend aanbod)
  ("Grofmazig ontwerp")   │                   │
                          │  Onderwijs-       ├─▶ SVS (fijnmazig aanbod)
  Planningssysteem ───────┤  catalogus (OC)   │
  ("Opleidingseenheid-    │                   ├─▶ Roostersysteem (aanbod)
   specifieke planning")  │                   │
                          │                   ├─▶ LMS (aanbod + leermiddelen)
  SKS ────────────────────┤                   │
  ("Leervraag in LO,      │                   ├─▶ Planningssysteem
   domein, leervorm")     │                   │
                          │                   ├─▶ Sector Edubroker
                          │                   │   ("Alle leergelegenheden
                          │                   │    i.r.t. leeruitkomsten")
                          │                   │
                          │                   ├──▶ Curriculum ontwerptool
                          └───────────────────┘   ("Herbruikbaar fijnmazig
                                                    aanbod")
```

**Het OKx-profiel is primair het profiel waarmee de OC via OEAPI communiceert.** Elke afnemer (SKS, planner, LMS, andere instelling) ontvangt dezelfde verrijkte structuur en haalt eruit wat relevant is.

## 3. De "Student Kiest"-keten

Het ArchiMate-model nummert de kernstroom expliciet:

| Stap | Stroom | Van → Naar | OEAPI-entiteiten |
|------|--------|-----------|------------------|
| 1 | Intake resultaat (studentidentiteit, leervraag in gewenste LO's, leercontext) | Intake → SVS | `Person`, `LearningOutcome` (referenties) |
| 2 | Keuzeproces starten (administratieve aanmelding) | SVS → SKS | `Association`-referentie |
| 3 | Aanbod passend op leervraag (uitgedrukt in LO's, domein, leervorm) | SKS → **OC** | Query op `LearningOutcome`, `modesOfDelivery`, leervorm |
| 4 | Passend aanbod: **programmes, courses, learning components <> test components** | **OC** → SKS | Volledige OEAPI-hiërarchie + OKx-extensies |
| 5 | Concept-leerroute als keuze → intekening | SKS → SVS | Genest `Programme` als track |

Stap 4 noemt de OEAPI-entiteiten letterlijk. Het OKx-profiel verrijkt die entiteiten met alles wat de keten nodig heeft.

## 4. OKx-hiërarchie op het OEAPI recursieve datamodel

### 4.1 OEAPI ondersteunt recursie

```
Programme ──parentId/childIds──▶ Programme (recursief, onbeperkte diepte)
  │
  └──programmeIds──▶ Course (N:M — course kan bij meerdere programmes horen)
                       │
                       ├──courseId──▶ LearningComponent ──parentId/childIds──▶ LearningComponent (recursief)
                       │
                       └──courseId──▶ TestComponent ──parentId/childIds──▶ TestComponent (recursief)

LearningOutcome ──parentIds/childIds──▶ LearningOutcome (DAG, meerdere ouders mogelijk)
  ▲ gerefereerd via learningOutcomeIds vanuit Programme, Course, LearningComponent, TestComponent
```

### 4.2 Mapping OKx → OEAPI

| OKx concept | OEAPI entiteit | Hoe | Credential bij afronding |
|-------------|----------------|-----|--------------------------|
| **Kwalificatie / opleiding** | `Programme` (root) | `programmeType: "programme"` | **Diploma** |
| **Leerroute** (globaal, vóór inschrijving) | `Programme` (kind) | `programmeType: "track"` of `"specialisation"` | (onderdeel van diploma) |
| **Keuzedeel** | `Programme` (kind) of `Course` | `programmeType: "minor"` of als losse `Course` | **MBO-certificaat** / Keuzedeel-bewijs |
| **Opleidingsonderdeel / leertaak** | `Course` | Eigen `studyLoad`, `learningOutcomeIds`. Kan bij meerdere programmes. | **Certificaat** / **Microcredential** |
| **Leeractiviteit** (keuzeniveau student) | `LearningComponent` (niveau 1) | Collectie lesopdrachten + lesuitkomsten | **Microcredential** / badge |
| **Lesopdracht / les** | `LearningComponent` (kind, recursief) | Genest via `parentId`/`childIds` | **Badge** |
| **Toets / examen** | `TestComponent` | Onder dezelfde `Course`. Gedeelde `learningOutcomeIds` | (beoordeelt bovenliggende LO's) |
| **Leeruitkomst** (summatief) | `LearningOutcome` (root) | Gerefereerd vanuit Programme, Course, LearningComponent | — |
| **Lesuitkomst** (formatief) | `LearningOutcome` (kind) | Genest via `parentIds`/`childIds`. DAG-structuur. | — |

### 4.3 Bottom-up aggregatie: de som klopt

Een **fundamenteel ontwerpprincipe**: de onderwijsspecificatie aggregeert bottom-up. De som van alle lessen onder een course moet kloppen met de course-specificatie, en de som van alle courses onder een programme moet kloppen met het programme — en idealiter uitlijnen met het top-down kwalificatiedossier van SBB.

```
Programme "Apothekersassistent" (level: mbo-4, studyLoad: 4800 SBU)
│  ▸ learningOutcomes: [alle kerntaak-afgeleide LO's]
│  ▸ OKx: waardeDocument: diploma
│  ▸ OKx: kwalificatieRef: { dossier: "25391", kwalificatie: "25396" }
│  ▸ SOM studyLoad children = 4800 SBU ✓
│
├── Programme "Track: Regulier voltijd" (programmeType: track)
│   │  ▸ OKx: leerrouteType: regulier
│   │  ▸ SOM studyLoad courses = 4800 SBU ✓
│   │
│   ├── Course "Baliegesprekken en cliëntcommunicatie" (studyLoad: 240 SBU)
│   │   │  ▸ learningOutcomes: ["Voert professionele baliegesprekken",
│   │   │  │                     "Cliëntgericht handelen"]
│   │   │  ▸ OKx: waardeDocument: microcredential
│   │   │  ▸ OKx: onderwijsSpecificatie:
│   │   │  │    leervorm: simulatie
│   │   │  │    tijdsbesteding: { bot: 160, oot: 80, eenheid: sbu }
│   │   │  │    ruimteType: praktijkruimte_simulatie
│   │   │  │    expertiseProfiel: ["rollenspel_training", "farmaceutisch"]
│   │   │  │    leermiddelGroepen: ["simulatie_materiaal", "digitaal_werkstation"]
│   │   │  ▸ SOM componentStudyLoad children = 240 SBU ✓
│   │   │
│   │   ├── LearningComponent "Leeractiviteit: Gespreksvoering simulatie"
│   │   │   │  ▸ learningComponentType: practical
│   │   │   │  ▸ OKx: hierarchieNiveau: leeractiviteit
│   │   │   │  ▸ OKx: onderwijsSpecificatie:
│   │   │   │  │    leervorm: simulatie
│   │   │   │  │    tijdsbesteding: { bot: 80, oot: 40, eenheid: sbu }
│   │   │   │  │    ruimteType: praktijkruimte_simulatie
│   │   │   │  │    ruimteEisen: "balie, wachtruimte, kassasysteem"
│   │   │   │  │    expertiseProfiel: ["rollenspel_training"]
│   │   │   │  │    leermiddelGroepen: ["simulatie_materiaal"]
│   │   │   │  │    spreidingspatroon: "2x per week, 8 weken"
│   │   │   │  ▸ OKx: waardeDocument: microcredential
│   │   │   │  ▸ OKx: deelnameVereisten: []
│   │   │   │  ▸ learningOutcomes: ["Voert professionele baliegesprekken"]
│   │   │   │
│   │   │   ├── LearningComponent "Les: Gesprek bij emotionele cliënt"
│   │   │   │     ▸ OKx: hierarchieNiveau: lesopdracht
│   │   │   │     ▸ OKx: onderwijsSpecificatie:
│   │   │   │     │    leervorm: simulatie
│   │   │   │     │    tijdsbesteding: { bot: 20, oot: 10, eenheid: sbu }
│   │   │   │     │    ruimteType: praktijkruimte_simulatie
│   │   │   │     │    expertiseProfiel: ["rollenspel_training"]
│   │   │   │     ▸ OKx: waardeDocument: badge
│   │   │   │     ▸ learningOutcomes: [lesuitkomst: "Herkent en hanteert
│   │   │   │     │                     emoties in baliegesprek"]
│   │   │   │
│   │   │   ├── LearningComponent "Les: Medicatie-informatie verstrekken"
│   │   │   │     ▸ (zelfde structuur, andere lesuitkomsten)
│   │   │   │
│   │   │   └── LearningComponent "Les: Culturele sensitiviteit"
│   │   │         ▸ (zelfde structuur)
│   │   │
│   │   ├── LearningComponent "Leeractiviteit: Farmaceutische theorie"
│   │   │   │  ▸ learningComponentType: lecture
│   │   │   │  ▸ OKx: onderwijsSpecificatie:
│   │   │   │  │    leervorm: klassikaal
│   │   │   │  │    tijdsbesteding: { bot: 40, oot: 40, eenheid: sbu }
│   │   │   │  │    ruimteType: collegezaal
│   │   │   │  │    expertiseProfiel: ["farmaceutisch"]
│   │   │   │  │    leermiddelGroepen: ["digitaal_werkstation", "vakliteratuur"]
│   │   │   │  ▸ OKx: deelnameVereisten: []
│   │   │   │  (... geneste lesopdrachten ...)
│   │   │
│   │   └── TestComponent "Praktijkexamen baliegesprekken"
│   │         ▸ testComponentType: life_skills_test
│   │         ▸ learningOutcomes: [zelfde LO's als bovenliggende course]
│   │         ▸ OKx: toetsNiveau: summatief
│   │         ▸ OKx: onderwijsSpecificatie:
│   │         │    ruimteType: praktijkruimte_simulatie
│   │         │    expertiseProfiel: ["examinator_farmaceutisch"]
│   │         │    tijdsbesteding: { bot: 4, eenheid: sbu }
│   │
│   ├── Course "Farmaceutische kennis en medicatieveiligheid" (studyLoad: 360 SBU)
│   │   └── (... zelfde structuur, andere leervormen/LO's ...)
│   │
│   ├── Course "Beroepspraktijkvorming" (studyLoad: 1200 SBU)
│   │   │  ▸ OKx: onderwijsSpecificatie:
│   │   │  │    leervorm: werkplekleren
│   │   │  │    ruimteType: externe_werkplek
│   │   │  │    expertiseProfiel: ["praktijkbegeleider"]
│   │   │  (gedeeld via programmeIds — hoort ook bij track "Versneld")
│   │   └── (... stage-activiteiten als LearningComponents ...)
│   │
│   └── (... overige courses tot SOM = 4800 SBU ...)
│
├── Programme "Track: Versneld" (programmeType: track)
│   │  ▸ OKx: leerrouteType: versneld
│   │  ▸ SOM studyLoad = 3600 SBU (minder SBU door EVC/vrijstellingen)
│   │  ▸ Deelt courses via programmeIds (N:M)
│   └── (... subset van courses, evt. gecomprimeerd ...)
│
└── Course "Keuzedeel: Digitale vaardigheden" (studyLoad: 240 SBU)
    ▸ programmeIds: [root + beide tracks] (beschikbaar in alle routes)
    ▸ OKx: waardeDocument: mbo_certificaat
```

**De aggregatie-invariant:** `SOM(children.studyLoad) = parent.studyLoad` op elk niveau. Dit maakt het mogelijk om vanuit een willekeurig niveau omhoog te aggregeren naar het kwalificatiedossier.

**Kwalificatiedossier-alignment:** De root `Programme` verwijst via `kwalificatieRef` naar het SBB-dossier. De `learningOutcomes` op programmaniveau dekken alle kerntaken/werkprocessen. Per `Course` en `LearningComponent` is traceerbaar welke LO's (en dus welke kerntaken) worden afgedekt.

### 4.4 Voorbeeld: LearningOutcome-hiërarchie met CompetentNL-taxonomieën

[CompetentNL](https://competentnl.nl/page/view/b1741ead-e4e8-4974-8aea-1399ae22284a/data-taxonomieen-van-competentnl) is de nationale standaard voor het beschrijven van skills, ontwikkeld door SBB, UWV, TNO en CBS. De taxonomie is beschikbaar als Linked Open Data (RDF/OWL/SKOS) via een SPARQL-endpoint en API. CompetentNL onderscheidt twee hiërarchieën:

| Taxonomie | Lagen | Omvang | Basis |
|-----------|-------|--------|-------|
| **Vaardighedentaxonomie** | 3 lagen: 6 algemene → 19 generieke → 112 specifieke vaardigheidsconcepten | Hard skills (leerbaar) + soft skills (ontwikkelbaar) | ESCO, O\*Net, wetenschappelijke literatuur |
| **Kennisgebiedentaxonomie** | 4 lagen, gebaseerd op ISCED-F 2013 | Vakspecifieke feiten, principes, theorieën en praktijken | ISCED-F 2013, CBS-rubrieken |

CompetentNL koppelt skills aan **alle mbo-kwalificaties** (kwalificaties, keuzedelen, certificaten) en is bezig met uitbreiding naar hbo en non-formeel onderwijs. De relatie `cnl:requires` verbindt beroepen met skills.

#### Waarom CompetentNL als referentie voor LearningOutcome?

1. **Gedeelde taal**: Leeruitkomsten in OEAPI beschrijven *wat* een student na afronding kan. CompetentNL beschrijft *welke vaardigheden en kennisgebieden* nodig zijn op de arbeidsmarkt. De koppeling maakt leeruitkomsten matchbaar met beroepen en vacatures.
2. **Cross-instelling vergelijkbaarheid**: Als instelling A en B dezelfde CompetentNL-referenties gebruiken voor hun leeruitkomsten, is automatisch zichtbaar welke overlap en complementariteit er is.
3. **Modulair studeren**: Bij bottom-up samenstellen van een leerroute (scenario E) kan het SKS leeruitkomsten matchen op CompetentNL-skills om te bepalen welke kwalificatie-eisen al zijn afgedekt.
4. **Arbeidsmarktaansluiting**: SBB koppelt CompetentNL aan de complete mbo-kwalificatiestructuur; OEAPI LearningOutcomes met CompetentNL-referenties sluiten dus direct aan op het kwalificatiedossier.

#### OEAPI-kernvelden die CompetentNL faciliteren

Het bestaande `LearningOutcome`-schema biedt al aanknopingspunten:

| OEAPI-veld | CompetentNL-mapping |
|------------|---------------------|
| `fieldsOfStudy` (ISCED-F, 2-6 digits) | Direct bruikbaar voor CompetentNL kennisgebiedentaxonomie (laag 1-3 = ISCED-F 2013) |
| `complexityLevel` (extensible enum: bloom\_1-6, solo\_0-4) | Aanvulbaar met CompetentNL vaardigheidsniveaus (als die beschikbaar komen) |
| `otherCodes` (array IdentifierEntry) | Ideaal voor CompetentNL skill-URI's als secundaire code |
| `parentIds` / `childIds` | DAG-structuur voor leeruitkomst → lesuitkomst hiërarchie |

#### OKx-extensie op LearningOutcome voor CompetentNL

Naast de bestaande OKx-attributen (`hierarchieNiveau`, `standaardisatieStatus`, `kwalificatieRef`, `sectorReferentie`) voegen we toe:

| Attribuut | Type | Beschrijving |
|-----------|------|-------------|
| `competentNlRefs` | array of object | Referenties naar CompetentNL-concepten. Per referentie: `{ uri: string, type: enum, label: string }`. `type`: `vaardigheid_algemeen`, `vaardigheid_generiek`, `vaardigheid_specifiek`, `kennisgebied`. `uri`: de CompetentNL Linked Data URI. `label`: leesbare naam (voor display). |
| `competentNlRelatieType` | enum: `primair`, `ondersteunend` | Geeft aan of deze LO primair of ondersteunend is voor het gekoppelde CompetentNL-concept. Volgt het CompetentNL-patroon van kernrelaties vs. contextuele relaties. |

#### Uitgewerkt voorbeeld: Apothekersassistent (mbo-4)

Hieronder de LearningOutcome-boom voor het voorbeeld uit §4.3. Leeruitkomsten zijn afgeleid van SBB kwalificatiedossier 25391 "Apothekersassistent" en gekoppeld aan CompetentNL vaardigheden en kennisgebieden.

```
LearningOutcome "LO-APOTH-001" (root — summatieve leeruitkomst)
│  name: "Voert professionele baliegesprekken"
│  description: "De beginnend beroepsbeoefenaar voert zelfstandig baliegesprekken
│  │              met cliënten over medicatiegebruik, bijwerkingen en
│  │              gezondheidsadvies, rekening houdend met de cliënt-context."
│  fieldsOfStudy: "0916"  (ISCED-F: Pharmacy)
│  complexityLevel: bloom_3  (Apply)
│  ▸ OKx: hierarchieNiveau: leeruitkomst
│  ▸ OKx: standaardisatieStatus: afgestemd
│  ▸ OKx: kwalificatieRef:
│  │    dossier: "25391"
│  │    kerntaak: "B1-K1"
│  │    werkproces: "B1-K1-W1"
│  ▸ OKx: competentNlRefs:
│  │    - uri: "cnl:skill/specifiek/mondelinge-communicatie"
│  │      type: vaardigheid_specifiek
│  │      label: "Mondelinge communicatie"
│  │      relatieType: primair
│  │    - uri: "cnl:skill/specifiek/klantgericht-handelen"
│  │      type: vaardigheid_specifiek
│  │      label: "Klantgericht handelen"
│  │      relatieType: primair
│  │    - uri: "cnl:knowledge/0916"
│  │      type: kennisgebied
│  │      label: "Farmacie"
│  │      relatieType: primair
│  │    - uri: "cnl:skill/generiek/communiceren"
│  │      type: vaardigheid_generiek
│  │      label: "Communiceren"
│  │      relatieType: primair
│  │    - uri: "cnl:skill/specifiek/empathie-tonen"
│  │      type: vaardigheid_specifiek
│  │      label: "Empathie tonen"
│  │      relatieType: ondersteunend
│  ▸ otherCodes:
│  │    - codeType: "competentnl-skill"
│  │      code: "cnl:skill/specifiek/mondelinge-communicatie"
│  │    - codeType: "competentnl-skill"
│  │      code: "cnl:skill/specifiek/klantgericht-handelen"
│  │    - codeType: "sbb-werkproces"
│  │      code: "B1-K1-W1"
│  │
│  ├── LearningOutcome "LO-APOTH-001a" (kind — formatieve lesuitkomst)
│  │     name: "Herkent en hanteert emoties in baliegesprek"
│  │     description: "De student herkent emotionele reacties bij cliënten
│  │     │              (angst, boosheid, verdriet) en past de gespreksvoering
│  │     │              aan met actief luisteren en empathische bevestiging."
│  │     fieldsOfStudy: "0916"
│  │     complexityLevel: bloom_4  (Analyse)
│  │     ▸ OKx: hierarchieNiveau: lesuitkomst
│  │     ▸ OKx: standaardisatieStatus: concept
│  │     ▸ OKx: kwalificatieRef:
│  │     │    dossier: "25391"
│  │     │    kerntaak: "B1-K1"
│  │     │    werkproces: "B1-K1-W1"
│  │     ▸ OKx: competentNlRefs:
│  │     │    - uri: "cnl:skill/specifiek/empathie-tonen"
│  │     │      type: vaardigheid_specifiek
│  │     │      label: "Empathie tonen"
│  │     │      relatieType: primair
│  │     │    - uri: "cnl:skill/specifiek/conflicthantering"
│  │     │      type: vaardigheid_specifiek
│  │     │      label: "Conflicthantering"
│  │     │      relatieType: ondersteunend
│  │     │    - uri: "cnl:skill/generiek/sociaal-communicatief"
│  │     │      type: vaardigheid_generiek
│  │     │      label: "Sociaal-communicatief handelen"
│  │     │      relatieType: primair
│  │
│  ├── LearningOutcome "LO-APOTH-001b" (kind — formatieve lesuitkomst)
│  │     name: "Verstrekt correcte medicatie-informatie aan cliënt"
│  │     description: "De student geeft gestructureerde en begrijpelijke
│  │     │              mondelinge uitleg over dosering, bijwerkingen,
│  │     │              interacties en bewaarcondities van gangbare medicijnen."
│  │     fieldsOfStudy: "0916"
│  │     complexityLevel: bloom_3  (Apply)
│  │     ▸ OKx: hierarchieNiveau: lesuitkomst
│  │     ▸ OKx: standaardisatieStatus: concept
│  │     ▸ OKx: kwalificatieRef:
│  │     │    dossier: "25391"
│  │     │    kerntaak: "B1-K1"
│  │     │    werkproces: "B1-K1-W2"
│  │     ▸ OKx: competentNlRefs:
│  │     │    - uri: "cnl:knowledge/0916"
│  │     │      type: kennisgebied
│  │     │      label: "Farmacie"
│  │     │      relatieType: primair
│  │     │    - uri: "cnl:skill/specifiek/mondelinge-communicatie"
│  │     │      type: vaardigheid_specifiek
│  │     │      label: "Mondelinge communicatie"
│  │     │      relatieType: primair
│  │     │    - uri: "cnl:skill/specifiek/informatieverstrekking"
│  │     │      type: vaardigheid_specifiek
│  │     │      label: "Informatieverstrekking"
│  │     │      relatieType: primair
│  │     │    - uri: "cnl:knowledge/091601"
│  │     │      type: kennisgebied
│  │     │      label: "Farmacologie"
│  │     │      relatieType: primair
│  │
│  └── LearningOutcome "LO-APOTH-001c" (kind — formatieve lesuitkomst)
│        name: "Past communicatie aan bij culturele achtergrond cliënt"
│        description: "De student herkent culturele invloeden op
│        │              gezondheidsbeleving en past taalgebruik, non-verbale
│        │              communicatie en adviesstijl hierop aan."
│        fieldsOfStudy: "0916"
│        complexityLevel: bloom_5  (Evaluate)
│        ▸ OKx: hierarchieNiveau: lesuitkomst
│        ▸ OKx: standaardisatieStatus: concept
│        ▸ OKx: competentNlRefs:
│        │    - uri: "cnl:skill/specifiek/interculturele-communicatie"
│        │      type: vaardigheid_specifiek
│        │      label: "Interculturele communicatie"
│        │      relatieType: primair
│        │    - uri: "cnl:skill/generiek/communiceren"
│        │      type: vaardigheid_generiek
│        │      label: "Communiceren"
│        │      relatieType: primair
│        │    - uri: "cnl:skill/specifiek/diversiteitsbewustzijn"
│        │      type: vaardigheid_specifiek
│        │      label: "Diversiteitsbewustzijn"
│        │      relatieType: ondersteunend

LearningOutcome "LO-APOTH-002" (root — summatieve leeruitkomst)
│  name: "Bereidt farmaceutische producten"
│  description: "De beginnend beroepsbeoefenaar bereidt zelfstandig magistrale
│  │              en generieke farmaceutische producten volgens GMP-richtlijnen,
│  │              voert kwaliteitscontroles uit en documenteert het bereidingsproces."
│  fieldsOfStudy: "0916"
│  complexityLevel: bloom_3  (Apply)
│  ▸ OKx: hierarchieNiveau: leeruitkomst
│  ▸ OKx: standaardisatieStatus: afgestemd
│  ▸ OKx: kwalificatieRef:
│  │    dossier: "25391"
│  │    kerntaak: "B1-K2"
│  │    werkproces: "B1-K2-W1"
│  ▸ OKx: competentNlRefs:
│  │    - uri: "cnl:skill/specifiek/prepareren"
│  │      type: vaardigheid_specifiek
│  │      label: "Prepareren en bereiden"
│  │      relatieType: primair
│  │    - uri: "cnl:skill/specifiek/kwaliteitscontrole"
│  │      type: vaardigheid_specifiek
│  │      label: "Kwaliteitscontrole uitvoeren"
│  │      relatieType: primair
│  │    - uri: "cnl:knowledge/0916"
│  │      type: kennisgebied
│  │      label: "Farmacie"
│  │      relatieType: primair
│  │    - uri: "cnl:skill/specifiek/nauwkeurig-werken"
│  │      type: vaardigheid_specifiek
│  │      label: "Nauwkeurig werken"
│  │      relatieType: primair
│  │    - uri: "cnl:skill/generiek/procedures-volgen"
│  │      type: vaardigheid_generiek
│  │      label: "Procedures en protocollen volgen"
│  │      relatieType: primair
│  │    - uri: "cnl:skill/specifiek/documenteren"
│  │      type: vaardigheid_specifiek
│  │      label: "Documenteren en registreren"
│  │      relatieType: ondersteunend
│  │
│  ├── LearningOutcome "LO-APOTH-002a" (kind — formatieve lesuitkomst)
│  │     name: "Weegt en meet grondstoffen conform voorschrift"
│  │     complexityLevel: bloom_3
│  │     ▸ OKx: hierarchieNiveau: lesuitkomst
│  │     ▸ OKx: competentNlRefs:
│  │     │    - uri: "cnl:skill/specifiek/nauwkeurig-werken"
│  │     │      type: vaardigheid_specifiek
│  │     │      label: "Nauwkeurig werken"
│  │     │      relatieType: primair
│  │     │    - uri: "cnl:skill/specifiek/meten-en-wegen"
│  │     │      type: vaardigheid_specifiek
│  │     │      label: "Meten en wegen"
│  │     │      relatieType: primair
│  │
│  ├── LearningOutcome "LO-APOTH-002b" (kind — formatieve lesuitkomst)
│  │     name: "Voert eindcontrole uit op bereid product"
│  │     complexityLevel: bloom_5  (Evaluate)
│  │     ▸ OKx: hierarchieNiveau: lesuitkomst
│  │     ▸ OKx: competentNlRefs:
│  │     │    - uri: "cnl:skill/specifiek/kwaliteitscontrole"
│  │     │      type: vaardigheid_specifiek
│  │     │      label: "Kwaliteitscontrole uitvoeren"
│  │     │      relatieType: primair
│  │     │    - uri: "cnl:skill/specifiek/kritisch-denken"
│  │     │      type: vaardigheid_specifiek
│  │     │      label: "Kritisch denken"
│  │     │      relatieType: ondersteunend
│  │
│  └── LearningOutcome "LO-APOTH-002c" (kind — formatieve lesuitkomst)
│        name: "Documenteert bereidingsproces in apotheekinformatiesysteem"
│        complexityLevel: bloom_3
│        ▸ OKx: hierarchieNiveau: lesuitkomst
│        ▸ OKx: competentNlRefs:
│        │    - uri: "cnl:skill/specifiek/documenteren"
│        │      type: vaardigheid_specifiek
│        │      label: "Documenteren en registreren"
│        │      relatieType: primair
│        │    - uri: "cnl:skill/specifiek/digitale-vaardigheden"
│        │      type: vaardigheid_specifiek
│        │      label: "Digitale vaardigheden"
│        │      relatieType: ondersteunend

LearningOutcome "LO-APOTH-003" (root — summatieve leeruitkomst)
│  name: "Handelt medicatieveilig"
│  description: "De beginnend beroepsbeoefenaar signaleert, voorkomt en
│  │              rapporteert medicatiefouten en -risico's conform de geldende
│  │              veiligheidsprotocollen en wet- en regelgeving."
│  fieldsOfStudy: "0913"  (ISCED-F: Nursing and caring)
│  complexityLevel: bloom_5  (Evaluate)
│  ▸ OKx: hierarchieNiveau: leeruitkomst
│  ▸ OKx: standaardisatieStatus: afgestemd
│  ▸ OKx: kwalificatieRef:
│  │    dossier: "25391"
│  │    kerntaak: "B1-K3"
│  │    werkproces: "B1-K3-W1"
│  ▸ OKx: competentNlRefs:
│       - uri: "cnl:skill/specifiek/veiligheidsprotocollen-toepassen"
│         type: vaardigheid_specifiek
│         label: "Veiligheidsprotocollen toepassen"
│         relatieType: primair
│       - uri: "cnl:skill/specifiek/risicosignalering"
│         type: vaardigheid_specifiek
│         label: "Risico's signaleren"
│         relatieType: primair
│       - uri: "cnl:skill/generiek/kwaliteitsbewust-handelen"
│         type: vaardigheid_generiek
│         label: "Kwaliteitsbewust handelen"
│         relatieType: primair
│       - uri: "cnl:knowledge/0916"
│         type: kennisgebied
│         label: "Farmacie"
│         relatieType: primair
│       - uri: "cnl:knowledge/0413"
│         type: kennisgebied
│         label: "Management en administratie"
│         relatieType: ondersteunend
```

#### DAG-structuur: meerdere ouders, hergebruik

Het OEAPI `LearningOutcome`-model ondersteunt meerdere ouders (`parentIds` is een array). Dit maakt hergebruik van lesuitkomsten over courses heen mogelijk:

```
LO-APOTH-001  "Voert professionele baliegesprekken"
  ├── LO-APOTH-001a  "Herkent en hanteert emoties"
  ├── LO-APOTH-001b  "Verstrekt correcte medicatie-informatie"
  └── LO-APOTH-001c  "Past communicatie aan bij culturele achtergrond"

LO-APOTH-003  "Handelt medicatieveilig"
  ├── LO-APOTH-001b  "Verstrekt correcte medicatie-informatie"  ← GEDEELD
  │     parentIds: [LO-APOTH-001, LO-APOTH-003]
  └── ...
```

Lesuitkomst `LO-APOTH-001b` hoort bij twee summatieve leeruitkomsten: "Baliegesprekken" en "Medicatieveiligheid". Bij het correct verstrekken van medicatie-informatie draag je aan beide bij. Dit is essentieel voor:
- **Modulair studeren**: een student die course "Baliegesprekken" afrond, heeft ook deels aan "Medicatieveiligheid" voldaan.
- **Cross-instelling erkenning**: instelling B ziet dat de student deze lesuitkomst al heeft behaald en hoeft dat deel niet opnieuw te toetsen.

#### CompetentNL-referenties als matchingsleutel

```
Student kiest in SKS: "Ik wil werken aan klantgericht handelen in de farmacie"

SKS query naar OC:
  → filter LearningOutcomes waar competentNlRefs bevat:
      uri LIKE "cnl:skill/specifiek/klantgericht-handelen"
      AND fieldsOfStudy = "0916"

OC retourneert:
  → LO-APOTH-001 "Voert professionele baliegesprekken"
    → gekoppeld aan Course "Baliegesprekken en cliëntcommunicatie" (240 SBU)
    → gekoppeld aan LearningComponent "Gespreksvoering simulatie"
  → Student ziet: leervorm = simulatie, 80 BOT, praktijkruimte, 8 weken

Planner berekent:
  → CompetentNL expertiseProfiel "rollenspel_training" + "farmaceutisch"
    → match met beschikbare docenten
```

---

## 5. Leerroute-scenario's

De Npuls-leerroutes (1-9) zijn allemaal expresseerbaar in het OEAPI-datamodel met OKx-extensies. Hieronder drie perspectieven per relevant scenario.

### Scenario A — Regulier (leerroute 1): Jochem wil apothekersassistent worden

> *Jochem schrijft zich in voor een voltijd mbo-4 opleiding van 3 jaar. Na 3 jaar behaalt hij zijn diploma.*

**Onderwijsontwerper (top-down)**
- Ontwerpt `Programme "Apothekersassistent"` met `curriculumType: nominaal`.
- Vertaalt kwalificatiedossier SBB 25391 naar `LearningOutcome`-hiërarchie.
- Maakt per kerntaak een of meer `Courses` met vaste `LearningComponents` (leeractiviteiten).
- Specificeert per component: leervorm (simulatie/klassikaal/werkplek), BOT/OOT, ruimtetype, expertiseprofiel, leermiddelen.
- Publiceert naar OC → alles staat klaar.

**Planner**
- Ontvangt volledige specificatie uit OC. Per `LearningComponentOffering`:
  - *"Gespreksvoering simulatie"*: 80 BOT, praktijkruimte met balie, docent met rollenspel-expertise, 2x/week, 8 weken.
  - *"Farmaceutische theorie"*: 40 BOT, collegezaal, farmaceutisch docent.
- Berekent: 120 studenten × deze specificaties = X lokalen, Y docenten, Z leermiddelen.
- Voedt `beschikbarePlaatsen` en `cohortGrootte` terug naar OC.

**Student (Jochem)**
- Ziet in SKS één programma met één track. Kiest niet, volgt nominale route.
- Elke afgeronde les → badge. Elke afgeronde leeractiviteit → microcredential. Course → certificaat. Alles → diploma.

---

### Scenario B — Versneld (leerroute 3): Linda heeft horeca-ervaring

> *Linda volgt Leidinggevende Bediening versneld. Ze heeft al horeca-ervaring en kan sneller door het programma.*

**Onderwijsontwerper**
- Dezelfde `Programme` root als regulier, maar voegt een `Programme "Track: Versneld"` toe (`leerrouteType: versneld`).
- Track "Versneld" deelt courses met Track "Regulier" via `programmeIds` (N:M), maar met minder totaal SBU (vrijstellingen o.b.v. EVC).
- Sommige `Courses` staan bij beide tracks; sommige alleen bij regulier.

**Planner**
- Ziet twee tracks onder hetzelfde programma. Kan berekenen:
  - Track Regulier: 30 studenten, Track Versneld: 5 studenten.
  - Gedeelde courses: 35 studenten in dezelfde `CourseOffering`.
  - Niet-gedeelde courses: aparte offerings met minder capaciteitsvraag.

**Student (Linda)**
- SKS toont track "Versneld" met minder courses. LO's die ze al beheerst (EVC) zijn afgevinkt.
- Kan alsnog meteen examen doen voor courses die ze overslaat → `TestComponent` is bereikbaar onafhankelijk van het volgen van de bijbehorende `LearningComponents`.

---

### Scenario C — Personaliseren binnen instelling (leerroute 4): Kyra combineert Pabo en ALO

> *Kyra wil het klaslokaal en de gymzaal combineren en gaat voor de dubbele bachelor Pabo-ALO.*

**Onderwijsontwerper**
- `Programme "Pabo"` en `Programme "ALO"` bestaan als losse root-programmes.
- Sommige `Courses` (bijv. "Pedagogiek", "Didactiek") horen bij **beide** programmes via `programmeIds`.
- Keuzedelen/minors zijn `Programme`-kinderen met `programmeType: "minor"`.

**Planner**
- Ziet dat `Course "Pedagogiek"` bij twee programmes hoort.
- Kan één `CourseOffering` plannen voor studenten uit beide opleidingen.
- Onderwijsspecificatie is identiek → zelfde ruimte, docent, leermiddelen.

**Student (Kyra)**
- SKS toont overlap: *"Deze 5 courses tellen voor beide diplomas."*
- Bottom-up: haar combinatie van gekozen courses aggregeert naar de LO's van **beide** kwalificaties.
- Na 4,5 jaar: twee diploma's, omdat de `learningOutcomes` van beide programmes zijn afgedekt.

---

### Scenario D — Personaliseren buiten instelling, binnen sector (leerroute 5): Macca doet Data Science bij universiteit B

> *Macca studeert voedingswetenschappen aan universiteit A. Ze vult haar programma aan met Data Science vakken van universiteit B.*

**Cross-instelling interoperabiliteit — waarom de standaard nodig is**

- Universiteit B publiceert `Course "Data Science Fundamentals"` in haar OC, met OKx-profiel:
  - `learningOutcomes`, `onderwijsSpecificatie` (leervorm, BOT/OOT, ruimtetype, etc.)
  - `studyLoad: 5 ECTS`
  - `waardeDocument: microcredential`
- Universiteit A ontvangt dit via **Sector Edubroker** of directe OEAPI-koppeling.
- **Omdat beide instellingen hetzelfde profiel gebruiken**, kan universiteit A:
  - De `learningOutcomes` matchen met haar eigen kwalificatie-eisen.
  - De `studyLoad` optellen in Macca's totaal.
  - De `onderwijsSpecificatie` tonen aan Macca (leervorm, locatie, etc.).

**Planner (universiteit B)**
- Plant de offering op basis van eigen onderwijsspecificatie.
- `beschikbarePlaatsen` wordt gedeeld via OC → Edubroker.

**Student (Macca)**
- SKS bij universiteit A toont aanbod van universiteit B als matchend op haar leervraag.
- `Course` van B wordt onderdeel van haar persoonlijke programme bij A (via `programmeIds`).
- Na afronding: microcredential van B + opname in diploma van A.

---

### Scenario E — Vrije keuze / modulair studeren (leerroute 7-9): Sinead en Chen

> *Sinead volgt losse modules voor bijscholing (vrije keuze). Chen bundelt modules uit mbo-opleidingen rond energietransitie (bundelen). Michelle stapelt modules richting diploma (stapelen).*

**Onderwijsontwerper**
- Publiceert `Courses` als **zelfstandige eenheden** met `keuzeMogelijk: true`.
- Elke course heeft eigen `learningOutcomes`, `onderwijsSpecificatie` en `waardeDocument: microcredential`.
- Courses kunnen los gevolgd worden — geen verplichte `Programme`-parent.

**Bottom-up aggregatie (de student bouwt zelf)**

```
Sinead kiest 3 losse courses:
  Course "Cloud Security" (5 ECTS, microcredential)
  Course "Threat Analysis" (5 ECTS, microcredential)
  Course "Incident Response" (5 ECTS, microcredential)
→ Geen diploma, wel 3 microcredentials + 15 ECTS in wallet

Chen bundelt 6 courses uit 2 opleidingen:
  Courses uit "Technicus Smart Energy" (mbo)
  Courses uit "Technicus Engineering" (mbo)
  Courses uit "Elektrotechniek" (hbo)
→ Thematische bundel, microcredentials, cross-instelling

Michelle stapelt modules:
  Begint met 4 courses → 4 microcredentials
  Instelling ziet: LO's dekken 60% van kwalificatie niveau-4
  → Aanbod: "volg nog 6 courses + examen → diploma"
  → Programme wordt retroactief samengesteld uit behaalde courses
```

**De aggregatie werkt bottom-up:** de `learningOutcomes` van de gekozen courses worden opgeteld en gematcht tegen het kwalificatiedossier. Als de som alle LO's dekt → diplomawaardig.

**Planner**
- Ziet per `CourseOffering`: aanmeldingen van zowel reguliere studenten als modulaire studenten.
- Onderwijsspecificatie is identiek ongeacht hoe de student er komt — dezelfde leervorm, ruimte, docent.
- Kan `minimaalAantalDeelnemers` hanteren voor go/no-go.

---

## 6. Het onderwijsSpecificatie-object (fase 1 — kern)

Het informatiemodel Onderwijsontwerp in ArchiMate toont dat op elk niveau niet alleen *wat* maar ook *hoe*, *waarmee*, *door wie*, *waar* en *hoe lang* wordt vastgelegd. Dit vertaalt zich naar een gestructureerd consumer-extensie-object.

### 6.1 Structuur `onderwijsSpecificatie`

Toepasbaar op `LearningComponent`, `Course` en `TestComponent`. Op hogere niveaus (Course, Programme) beschrijft het het overkoepelende kader; op lagere niveaus (LearningComponent) de concrete specificatie.

```yaml
onderwijsSpecificatie:
  leervorm:
    type: string enum         # simulatie, klassikaal, werkplekleren,
                              # projectonderwijs, zelfstudie_begeleid,
                              # stage, onderzoek, co_teaching, blended
    strategie: string         # optioneel: overkoepelende didactische
                              # strategie (bijv. "4CID", "probleemgestuurd")
  tijdsbesteding:
    bot: number               # begeleid onderwijs tijd (SBU of uren)
    oot: number               # onbegeleid onderwijs tijd
    eenheid: string enum      # sbu, ects, uur
    spreidingspatroon: string # "2x per week, 8 weken" / "doorlopend"
  ruimteType: string enum     # praktijkruimte_simulatie, collegezaal,
                              # werkplaats, lab, online, externe_werkplek,
                              # examenzaal, hybride
  ruimteEisen: string         # vrije specificatie: "balie, wachtruimte"
  expertiseProfielen:
    - profiel: string         # competentie-aanduiding
                              # bijv. "rollenspel_training", "farmaceutisch"
  leermiddelGroepen:
    - groep: string           # "digitaal_werkstation", "vakliteratuur",
                              # "simulatie_materiaal", "gereedschap"
      specificatie: string    # "Chromebook + MS Word licentie"
```

### 6.2 Aanvullende OKx-extensies per entiteit

**Programme** (`consumerKey: "okx"`)

| Attribuut | Type | Beschrijving |
|-----------|------|-------------|
| `curriculumType` | enum: `nominaal`, `flexibel`, `hybride` | Structuurtype. Bepaalt of tracks vast zijn of student vrij combineert. |
| `keuzegateType` | enum: `nominaal`, `maatwerk`, `continu` | Keuzemoment. `continu` = reversibele overgang (ADR 0012). |
| `leerrouteType` | enum: `regulier`, `versneld`, `temporiserend`, `personalisatie_intra`, `personalisatie_sector`, `personalisatie_cross_sector`, `vrije_keuze`, `bundelen`, `stapelen` | Npuls leerroute-classificatie (1-9). |
| `waardeDocument` | object: `{ type: enum, register: string }` | Credential bij afronding. `type`: `diploma`, `certificaat`, `mbo_certificaat`, `deelkwalificatie`, `microcredential`. `register`: bijv. "DUO", "edubadges.nl". |
| `kwalificatieRef` | object: `{ dossier: string, kwalificatie: string, kerntaak: string }` | Referentie naar SBB kwalificatiedossier. `dossier` = dossiernummer, `kwalificatie` = kwalificatienummer, `kerntaak` = optioneel specifieke kerntaak. |
| `leeruitkomstDekking` | enum: `volledig`, `gedeeltelijk`, `ontbreekt` | Mate waarin LO's gekoppeld zijn aan courses/components. |
| `onderwijsSpecificatie` | object (zie §6.1) | Overkoepelend specificatiekader op programmaniveau. |

**Course** (`consumerKey: "okx"`)

| Attribuut | Type | Beschrijving |
|-----------|------|-------------|
| `onderwijsSpecificatie` | object (zie §6.1) | Specificatie op cursusniveau: leervorm, tijdsbesteding, ruimte, expertise, leermiddelen. |
| `waardeDocument` | object: `{ type, register }` | Credential: `microcredential`, `certificaat`, `mbo_certificaat`, `badge`. |
| `keuzeMogelijk` | boolean | Kan onderdeel zijn van een maatwerk-leerroute. |
| `deelnameVereisten` | array of `{ courseId: UUID, type: "afgerond" \| "gelijktijdig" }` | Volgordelijkheid. Welke courses eerst afgerond moeten zijn. |
| `kwalificatieRef` | object | Optioneel: mapping naar specifieke kerntaak/werkproces in het dossier. |

**LearningComponent** (`consumerKey: "okx"`)

| Attribuut | Type | Beschrijving |
|-----------|------|-------------|
| `hierarchieNiveau` | enum: `leeractiviteit`, `lesopdracht` | Positie in OKx-hiërarchie. |
| `onderwijsSpecificatie` | object (zie §6.1) | Concrete specificatie: leervorm, BOT/OOT, ruimtetype + eisen, expertiseprofiel, leermiddelen, spreidingspatroon. |
| `waardeDocument` | object: `{ type, register }` | `microcredential`, `badge`, of `null`. |
| `componentStudyLoad` | object: `{ bot: number, oot: number, eenheid: enum }` | SBU/ECTS op componentniveau. Splitsing BOT/OOT. |
| `deelnameVereisten` | array of `{ learningComponentId: UUID, type: enum }` | Prerequisites tussen componenten. |

**TestComponent** (`consumerKey: "okx"`)

| Attribuut | Type | Beschrijving |
|-----------|------|-------------|
| `toetsNiveau` | enum: `formatief`, `summatief` | Summatief = geldend voor diploma, gekoppeld aan opleidingsonderdeel/kwalificatie. |
| `onderwijsSpecificatie` | object (subset: ruimteType, expertiseProfiel, tijdsbesteding) | Wat nodig is om de toets af te nemen. |
| `kwalificatieRef` | object | Mapping naar kerntaak/werkproces die geëxamineerd wordt. |

**LearningOutcome** (`consumerKey: "okx"`)

| Attribuut | Type | Beschrijving |
|-----------|------|-------------|
| `hierarchieNiveau` | enum: `leeruitkomst`, `lesuitkomst` | Positie in LO-hiërarchie. |
| `standaardisatieStatus` | enum: `concept`, `afgestemd`, `vastgesteld`, `verouderd` | Status sectorale standaardisatie. |
| `kwalificatieRef` | object: `{ dossier, kerntaak, werkproces }` | Traceerbaarheid naar SBB-kwalificatiedossier. |
| `sectorReferentie` | string | Referentie naar sectoraal register. |
| `competentNlRefs` | array of `{ uri: string, type: enum, label: string }` | Referenties naar [CompetentNL](https://competentnl.nl) vaardigheden en kennisgebieden. `type`: `vaardigheid_algemeen`, `vaardigheid_generiek`, `vaardigheid_specifiek`, `kennisgebied`. `uri`: Linked Data URI. Zie §4.4 voor voorbeelden. |
| `competentNlRelatieType` | enum: `primair`, `ondersteunend` | Geeft aan of deze LO primair of ondersteunend is voor het gekoppelde CompetentNL-concept. |

---

## 7. Cross-instelling interoperabiliteit

### Waarom de standaard nodig is

Als instelling A een `Course` publiceert met OKx-profiel, moet instelling B deze kunnen:

1. **Ontvangen** — via Sector Edubroker of directe OEAPI-koppeling
2. **Begrijpen** — dankzij gestandaardiseerde `onderwijsSpecificatie`, `learningOutcomes` en `kwalificatieRef`
3. **Matchen** — `learningOutcomes` van course B matchen met kwalificatie-eisen van programme A
4. **Inplannen** — `onderwijsSpecificatie` vertelt welke resources nodig zijn
5. **Erkennen** — `waardeDocument` en `learningOutcomes` maken erkenning/vrijstelling mogelijk

### Wat gestandaardiseerd moet zijn per entiteit

| Aspect | Waarom standaard | Voorbeeld |
|--------|-----------------|-----------|
| `learningOutcomes` met `kwalificatieRef` | Anders kan B niet matchen met eigen kwalificatie | LO "Voert professionele baliegesprekken" → SBB B1-K1-W1 |
| `onderwijsSpecificatie.leervorm` | Instelling B moet weten of het werkplekleren of klassikaal is | Relevant voor planning en studentverwachting |
| `onderwijsSpecificatie.tijdsbesteding` (BOT/OOT) | Instelling B moet weten hoeveel contacttijd nodig is | Relevant voor inpassing in eigen rooster |
| `studyLoad` in gedeelde eenheid | Optelbaarheid cross-instelling | ECTS (hbo) of SBU (mbo) |
| `waardeDocument` | Erkenning van wat de student elders heeft behaald | Microcredential van B telt mee bij A |
| `deelnameVereisten` | Instelling B moet weten of student kwalificeert | "Eerst course X afgerond" |

### Wat instelling-specifiek mag blijven

| Aspect | Waarom niet standaard |
|--------|----------------------|
| Fysiek lokaal (OEAPI `Room`) | Instelling-eigen faciliteiten |
| Specifieke docent (OEAPI `Person`) | Instelling-eigen personeel |
| Roostering / tijdslots | Instelling-eigen planning |
| Prijs / bekostigingsmodel | Instelling-eigen beleid |

---

## 8. Fasering (herzien)

### Fase 1 — Curriculum ontwerp → OC (onderwijsspecificatie publiceren)

**Doel:** Instelling publiceert compleet curriculum als OEAPI-structuur met onderwijsspecificatie. Bruikbaar voor zowel klassiek nominaal onderwijs als flexibel modulair aanbod.

**Scope:** `Programme`, `Course`, `LearningComponent`, `TestComponent`, `LearningOutcome` met alle OKx-extensies uit §6. Bottom-up aggregatie-invariant (SOM klopt per niveau). Kwalificatiedossier-referenties.

### Fase 2 — OC → SKS (keuze, trechters en leerroutes)

**Doel:** OC levert gefilterd aanbod aan SKS. Student kan kiezen op leervorm, LO's, beschikbaarheid, budget, regio. Alle 9 leerroute-typen worden ondersteund.

**Aanvullende extensies:** `instroomEisen` (gestructureerd), `uitstroomProfiel`, `leerrouteType`, `beschikbaarheidsType`, `budgetIndicatie`, `instroomMomenten`, `beschikbarePlaatsen`, `regioAanbod`.

### Fase 3 — OC ↔ Planningssysteem (realisatie)

**Doel:** Planningssysteem gebruikt onderwijsspecificatie om haalbaarheid te berekenen. Terugkoppeling naar OC.

**Aanvullende extensies:** `planningHorizon`, `minimaalAantalDeelnemers`, `parallelGroepen`, `cohortGrootte`, `doorlooptijdWeken`. De `onderwijsSpecificatie` (leervorm, BOT/OOT, ruimtetype, expertiseprofiel, leermiddelen) uit fase 1 is hier direct bruikbaar — geen aparte planning-extensies nodig voor de kernvraag "kan de instelling dit realiseren?".

---

## 9. Signaleringen (buiten extensiemogelijkheden OEAPI)

| # | Probleem | Impact | Workaround | Aanbeveling |
|---|---------|--------|-----------|-------------|
| 1 | `studyLoad` ontbreekt op `LearningComponent`/`TestComponent` | BOT/OOT per component alleen via extensie; niet interoperabel | `componentStudyLoad` als OKx-extensie | OEAPI change request: `studyLoad` op alle entiteiten |
| 2 | `modesOfDelivery` te grof voor OKx-leervormen | Simulatie, werkplekleren, projectonderwijs niet expresseerbaar | `onderwijsSpecificatie.leervorm` als extensie | Uitbreiden `x-ooapi-extensible-enum` met OKx-waarden |
| 3 | Geen prerequisite-mechanisme | Volgordelijkheid niet uitdrukbaar in kern | `deelnameVereisten` als extensie | OEAPI change request: `prerequisiteIds` op `Course`/`LearningComponent` |
| 4 | Geen credential/waardedocument-veld | Niet duidelijk welk bewijs bij afronding hoort | `waardeDocument` als extensie | Evalueer OEAPI-uitbreiding |
| 5 | Keuze/plaatsingsobject ontbreekt | SKS ↔ SVS interactie buiten OEAPI | Separaat koppelvlak | OKx-koppelvlakspecificatie voor SKS ↔ SVS |
| 6 | Fijnmazige roostering (recurrence) ontbreekt | Geen "elke dinsdag 10-12" | Basiskenmerken via extensie | Aansluiting iCal/RFC 5545 onderzoeken |

---

## 10. Ontwerpkeuzes

| # | Keuze | Toelichting | Alternatief |
|---|-------|-------------|-------------|
| 1 | **`onderwijsSpecificatie` als gestructureerd object** | Eén consistent object op elk niveau. Planner, student en LMS halen er elk uit wat ze nodig hebben. | Losse attributen per concern (ruimte apart, docent apart, leermiddel apart) — verliest samenhang. |
| 2 | **Bottom-up aggregatie als invariant** | `SOM(children) = parent` op elk niveau. Maakt cross-instelling erkenning en modulair studeren mogelijk. | Geen aggregatie-eis — verliest controleerbaarheid. |
| 3 | **`waardeDocument` per niveau** | Maakt de credentialing-cascade (badge → microcredential → certificaat → diploma) expliciet en gestandaardiseerd. | Alleen op programmaniveau — mist de bottom-up motivatie. |
| 4 | **`kwalificatieRef` op elk niveau** | Traceert van lesopdracht tot kwalificatiedossier. Essentieel voor summatieve toetsing en cross-instelling erkenning. | Alleen op programmaniveau — verliest granulaire alignment. |
| 5 | **Alle 9 Npuls-leerroutes via `leerrouteType`** | Expliciet classificeren maakt het mogelijk om in SKS op leerroute-type te filteren. | Afleiden uit structuur — niet eenduidig. |
| 6 | **BOT/OOT-splitsing in `tijdsbesteding`** | Cruciaal voor planning: BOT = docent + ruimte nodig, OOT = zelfstandig. | Alleen totaal SBU — planner kan niet berekenen wat nodig is. |
| 7 | **Cross-instelling door gedeeld profiel** | Eén `consumerKey: "okx"` met gestandaardiseerde semantiek. Instelling B kan aanbod van A begrijpen, matchen en inplannen. | Bilaterale afspraken — niet schaalbaar. |

---

## 11. Overige aantekeningen

- **Architectuurkader incompleet:** ADRs staan op "Voorstel". Het profiel evolueert mee.
- **Afstemming RIO/EduXchange:** Vermijd overlap. RIO heeft `sector`, `studyChoiceCheck`; EduXchange heeft `alliances`. OKx vult aan.
- **64 OKx business processes** in ArchiMate vormen de functionele validatie.
- **Twee catalogi:** OC (fijnmazig) en Onderwijsprogrammacatalogus (grofmazig). OEAPI: expand = fijnmazig, geen expand = grofmazig.
- **Voorbeelden:** Bij YAML-implementatie: `source/consumers/OKx/V1/` met examples conform RIO-patroon.

## Sessiestatus

**Gedaan (v3):**
- Onderwijsspecificatie als gestructureerd object met leervorm, BOT/OOT, ruimtetype, expertiseprofiel, leermiddelen, spreidingspatroon
- Bottom-up aggregatie met SOM-invariant en kwalificatiedossier-alignment
- 5 uitgewerkte scenario's over Npuls-leerroutes (regulier, versneld, personalisatie intra/inter-instelling, modulair)
- 3 perspectieven per scenario (ontwerper, planner, student)
- Cross-instelling interoperabiliteit: wat moet standaard zijn, wat mag instelling-specifiek blijven
- Credentialing-cascade (badge → microcredential → certificaat → diploma) met `waardeDocument`

**Gedaan (v4):**
- LearningOutcome-voorbeelduitwerkingen met CompetentNL-taxonomieën (§4.4)
- CompetentNL vaardighedentaxonomie (6 → 19 → 112) en kennisgebiedentaxonomie (ISCED-F) als referentiekader
- Twee nieuwe OKx-extensieattributen: `competentNlRefs` en `competentNlRelatieType`
- Drie uitgewerkte root-leeruitkomsten (B1-K1, B1-K2, B1-K3) met geneste lesuitkomsten
- DAG-structuur voorbeeld: gedeelde lesuitkomst met meerdere ouders
- Matchingscenario: student zoekt op CompetentNL-skills → OC retourneert passend aanbod

**Volgende stappen:**
- [ ] Review kernteam: kloppen scenario's met praktijk pilotinstellingen?
- [ ] Validatie CompetentNL-URI's: zijn de gebruikte URI's reëel in de publieksversie van CompetentNL?
- [ ] Validatie: kan voorbeeld (§4.3 + §4.4) door bestaande OEAPI-implementaties worden geserveerd?
- [ ] Detaillering enum-waarden (leervorm, ruimtetype, expertiseprofiel) met instellingen
- [ ] Uitwerking modulair studeren: hoe werkt retroactieve programme-samenstelling?
- [ ] OEAPI change requests voor signaleringen
- [ ] Featureplan via `/maak-plan` voor YAML-profielbestanden (opgeleverd: `feature-plans/20260414_1800_okx-oeapi-consumer-profiel.md`)
