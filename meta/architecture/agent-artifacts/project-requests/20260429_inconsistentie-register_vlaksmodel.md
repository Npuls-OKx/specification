---
created: "2026-04-29T16:55:00+02:00"
source_document: "meta/architecture/agent-artifacts/project-requests/20260414_1500_okx-oeapi-consumer-profiel.md"
vlaksmodel_anchor: "§12.0"
scope: "Analyse op inconsistenties t.o.v. 6×6 (+ toetsrij) vlaks-informatiemodel; geen fixes doorvoeren."
---

# Inconsistentie-register — Vlaks-informatiemodel vs. project request

## Categorieën (C1..C9)

- **C1 Niveau-mismatch**: concept staat op verkeerde rij (kwalificatiedossier/kwalificatie/kerntaak/werkproces/les/doel/toets).
- **C2 Kolom-mismatch**: concept staat in verkeerde kolom (kader/LO/specificatie/aanbod/verbintenis/resultaat).
- **C3 Cardinaliteit/structuur-mismatch**: kerntaak ↔ werkproces ↔ leeruitkomst onjuist of impliciet 1-op-1.
- **C4 Missing-link**: LO niet consequent als sleutel door alle lagen.
- **C5 Terminologie-mismatch**: termen door elkaar voor verschillende dingen (bv. leervorm vs modesOfDelivery vs deliveryForm; leeractiviteit vs leeronderdeel vs leergelegenheid).
- **C6 Resultaatkolom onduidelijk**: resultaat/aanwezigheid hangt niet expliciet aan verbintenis/association of is dubbel gemodelleerd.
- **C7 Toetsrij scope mismatch**: formatief/summatief scope inconsistent; toetsobjecten vs toetsresultaten.
- **C8 OEAPI-mapping mismatch**: objectfamilie zonder duidelijke OEAPI-haak of intern tegenstrijdige mapping.
- **C9 Language/schema drift**: NL vs UK-English attribuutnamen en objectnamen door elkaar zonder expliciete mapping in dit document.

## Inconsistentie-matrix (INC-###)

> Notatie: regelranges verwijzen naar `meta/architecture/agent-artifacts/project-requests/20260414_1500_okx-oeapi-consumer-profiel.md`.

### INC-001 — C9 Language/schema drift in extensievelden (NL ↔ UK-English)

- **Document-locatie**: §4.3, §6, §12, §14, §16, §18 (verspreid)
- **Indicatie**: het document gebruikt door elkaar NL velden (`kwalificatieRef`, `onderwijsSpecificatie`, `hierarchieNiveau`, `deelnameVereisten`, `toetsNiveau`) én UK-English velden (`qualificationReference`, `educationSpecification`, `hierarchyLevel`, `participationRequirements`, `assessmentLevel`).
- **Waarom inconsistent met vlaksmodel**: §12.0 positioneert OEAPI-haken en OKx-extensies; zonder expliciete mapping ontstaat interpretatieverschil bij implementaties.
- **Impact**:
  - CO→OC publicatie: **hoog**
  - OC→Planning CSP: **hoog**
  - SKS-keuze: **midden**
  - Toetsketen: **midden**
- **Suggestie (nog niet uitvoeren)**:
  - Voeg één “NL→UK-English mappingtabel” toe (of kies één canonical taal voor alle schema-namen in dit document).
  - Leg expliciet vast welke velden **schema-namen** zijn (normatief) vs. **pseudocode** (illustratief).

### INC-002 — C8 Signalering-referentie “nr. 7” bestaat niet (nummering drift)

- **Document-locatie**: §9 (`Signaleringen`) vs §12.4
- **Waar**:
  - §9 bevat signaleringen **1..6**.
  - §12.4 verwijst naar “signalering 7”.
- **Wat is inconsistent**: verwijzing naar een niet-bestaande signalering maakt traceerbaarheid/afspraak onmogelijk.
- **Impact**:
  - CO→OC publicatie: laag
  - OC→Planning CSP: **midden** (RequestForOffering raakt demand-side)
  - SKS-keuze: **hoog**
  - Toetsketen: laag
- **Suggestie (nog niet uitvoeren)**: nummering harmoniseren (óf §9 uitbreiden met #7 RequestForOffering, óf verwijzingen aanpassen).

### INC-003 — C9 `kwalificatieRef.dossier=25391` conflicteert met kwalificatiedossierbron (Crebo 23450)

- **Document-locatie**: §4.3 (voorbeeld), §4.4 (LO-boom), §5 scenario A, §16.1, §18.1
- **Waar**: o.a. **L116–L123**, **L257–L275**, **L553–L558**, **L1318–L1333**, **L1710–L1714**
- **Wat is inconsistent**: het document fixeert “kwalificatiedossier 25391” als sleutel in voorbeelden/queries, terwijl de repo-bron `meta/architecture/docs/kwalificatiedossier/Apothekersassistent-2.pdf` een **Crebo-dossier 23450** bevat (en kwalificatie 27141).
- **Waarom**: het vlaksmodel gebruikt “Kwalificatiekader” als normatieve basis; de concrete identifier moet consistent zijn door alle lagen.
- **Impact**:
  - CO→OC publicatie: **hoog** (refs op programme/LO)
  - OC→Planning CSP: **midden**
  - SKS-keuze: **hoog** (queryparameter)
  - Toetsketen: **midden**
- **Suggestie (nog niet uitvoeren)**:
  - Maak onderscheid tussen **SBB dossier-id** vs **Crebo dossier** en leg vast welke “dossier”-sleutel `qualificationReference` precies bedoelt.
  - Veranker één canonical identifier-set in §12.0 (en hergebruik in alle voorbeelden).

### INC-004 — C5 Leervorm-terminologie loopt door elkaar (`modesOfDelivery` vs `leervorm` vs `deliveryForm`)

- **Document-locatie**: §6.1, §9, §13, §14, §16, §18
- **Waar**:
  - §6.1 toont `onderwijsSpecificatie.leervorm`.
  - §13 gebruikt `educationSpecification.deliveryForm`.
  - §18 query note gebruikt `modesOfDelivery=...`.
- **Wat is inconsistent**: drie verschillende representaties voor hetzelfde selectiecriterium “hoe wordt onderwezen”.
- **Waarom**: in het vlaksmodel is “Onderwijsspecificatie” het sjabloon; veldnamen moeten daar eenduidig zijn en consistent naar aanbod-query doorwerken.
- **Impact**:
  - CO→OC publicatie: **midden**
  - OC→Planning CSP: **hoog**
  - SKS-keuze: **hoog**
  - Toetsketen: laag
- **Suggestie (nog niet uitvoeren)**:
  - Kies één canonical veldnaam in OKx-extensie (bijv. `educationSpecification.deliveryForm`) en definieer hoe die zich verhoudt tot OEAPI `modesOfDelivery`.

### INC-005 — C6 Resultaatkolom: `RESULT_RECORD` conceptueel, maar zonder koppelvlak/plaats in OEAPI

- **Document-locatie**: §12.0.3 ERD + toelichting
- **Waar**: **L904–L1031**
- **Wat is inconsistent**: het vlaksmodel introduceert `RESULT_RECORD` om de resultaatkolom zichtbaar te maken, maar elders wordt resultaat beschreven als “primair in `Association.state`” en/of in velden op offerings. Het document specificeert niet waar “bewijsvoering op LO/lesuitkomst” daadwerkelijk wordt uitgewisseld.
- **Impact**:
  - CO→OC publicatie: laag
  - OC→Planning CSP: laag
  - SKS-keuze: **midden**
  - Toetsketen: **hoog**
- **Suggestie (nog niet uitvoeren)**:
  - Maak expliciet: (a) minimum: alleen `Association.state`, (b) uitbreidingspad: apart resultaat-koppelvlak buiten OEAPI, of (c) OKx-extensie op association/offering (met risico op OEAPI-change).

### INC-006 — C3 Kerntaak/werkproces/LO cardinaliteit wordt soms als “1 LO-set per course” gepresenteerd

- **Document-locatie**: §4.3 voorbeeldstructuur en scenario’s
- **Waar**: **L127–L137**, **L152–L163**, **L553–L558**
- **Wat is inconsistent**: het vlaksmodel stelt expliciet:
  - kerntaak = meerdere werkprocessen,
  - werkproces = meerdere leeruitkomsten,
  - en (bottom-up) lesuitkomsten kunnen aggregeren.
  
  In het voorbeeld lijkt een `Course` soms gelijkgesteld aan “opleidingsonderdeel” met een klein aantal LO’s, zonder expliciet te maken dat één werkproces meerdere LO’s kan opleveren (en dat LO’s over meerdere courses/components verspreid kunnen zijn).
- **Impact**:
  - CO→OC publicatie: **midden**
  - OC→Planning CSP: laag
  - SKS-keuze: **midden**
  - Toetsketen: **hoog** (summatieve dekking)
- **Suggestie (nog niet uitvoeren)**:
  - Voeg een expliciete regel toe: `WorkProcess (1..*) LearningOutcome` en `LearningOutcome (0..*) Course/LC/TC`, met voorbeelden van “LO verdeeld over meerdere components”.

### INC-007 — C8 `formalDocument` (OEAPI) vs `credentialDocument` (OKx) inconsistent gebruik

- **Document-locatie**: §14.2 tabel
- **Waar**: **L1229–L1231**
- **Wat is inconsistent**: de tabel noemt `formalDocument` (OEAPI-kern) én `credentialDocument` (OKx) maar in andere secties wordt `waardeDocument` gebruikt (NL). Zonder normatieve keuze is onduidelijk welke velden implementaties moeten vullen.
- **Impact**:
  - CO→OC publicatie: **midden**
  - OC→Planning CSP: laag
  - SKS-keuze: **midden**
  - Toetsketen: midden
- **Suggestie (nog niet uitvoeren)**: één canonical objectnaam + veldset kiezen, en andere als alias/mapping documenteren.

### INC-008 — C2 Kolom-mismatch: “Leergelegenheid” (werkproces-rij) vs OEAPI `LearningComponentOffering`

- **Document-locatie**: §12.0.2 rijen-tabel
- **Waar**: **L895–L902**
- **Wat is inconsistent**: het vlaksmodel gebruikt de term `Leergelegenheid` als aanbod op werkproces-rij. In OEAPI bestaat het aanbod als `LearningComponentOffering` (en de specificatie als `LearningComponent`). Zonder mappingregel is “leergelegenheid” ambigu (les? leeractiviteit? beide?).
- **Impact**:
  - CO→OC publicatie: laag
  - OC→Planning CSP: **midden**
  - SKS-keuze: **midden**
  - Toetsketen: laag
- **Suggestie (nog niet uitvoeren)**: definieer “leergelegenheid = offering van learning component op hierarchyLevel=learning_activity” en “lesgelegenheid = offering van child learning component (lesson assignment)”.

### INC-009 — C8 Queryparameter `qualificationDossier` in §18.1 heeft geen OEAPI-haak

- **Document-locatie**: §18.1
- **Waar**: **L1710–L1714**
- **Wat is inconsistent**: de query note gebruikt `&qualificationDossier=25391`, maar elders is de haak beschreven als `qualificationReference`/OKx `kwalificatieRef` op entiteiten. Als queryparameter moet dit consistent zijn met OEAPI-filtering of expliciet “OKx-query-extensie”.
- **Impact**:
  - CO→OC publicatie: laag
  - OC→Planning CSP: laag
  - SKS-keuze: **hoog**
  - Toetsketen: laag
- **Suggestie (nog niet uitvoeren)**: maak “trechterquery-parameters” normatief: welke zijn OEAPI-standaardfilters, welke zijn OKx-extensiefilters.

### INC-010 — C7 Toetsrij: `toetsNiveau`/`assessmentLevel` scope nog niet hard gekoppeld aan werkproces/LO-set

- **Document-locatie**: §4.3 voorbeeld + §6.2 TestComponent + §12.0.2 toetsrij
- **Waar**: **L182–L189**, **L750–L757**, **L902**
- **Wat is inconsistent**: het vlaksmodel wil toetsing kunnen scopes op “lesuitkomst-set / leeruitkomst-set / werkproces-set”. In het document wordt dit deels benoemd, maar niet gekoppeld aan een expliciet objectveld (en niet consequent in NL/EN).
- **Impact**:
  - CO→OC publicatie: midden
  - OC→Planning CSP: laag
  - SKS-keuze: laag
  - Toetsketen: **hoog**
- **Suggestie (nog niet uitvoeren)**: voeg één normatieve “assessmentScope” representatie toe (bijv. `assessmentTargets: { learningOutcomeIds: [...], workProcessCodes: [...] }` als concept).

## Samenvatting (kwalitatief)

- **Hoogste impact**: INC-001 (taaldrift), INC-003 (kwalificatiekader-id drift), INC-004 (leervorm drift), INC-005 (resultaatmodel), INC-009 (trechterquery).
- **Quick wins**: nummering/refs (INC-002), terminologie-mappingregels (INC-008), credential-veldkeuze (INC-007).

