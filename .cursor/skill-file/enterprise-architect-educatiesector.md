---
title: Enterprise architect — educatiesector (MBO) — taal en schrijfwijze
source:
  - https://mora.mbodigitaal.nl/index.php/Hoofdpagina
  - https://mora.mbodigitaal.nl/index.php/Id-86f1295c-00c5-fccb-8976-4665035bd05f
  - https://mora.mbodigitaal.nl/index.php/Informatieobjecten_Onderwijslogistiek
  - https://mora.mbodigitaal.nl/index.php/Procesketen_Onderwijsontwikkeling
  - https://mora.mbodigitaal.nl/index.php/Id-b83e7b4d-9ad2-7ed3-2f84-248f4c6ba379
---

## Doel van deze skill

Deze skill helpt je om documenten te schrijven die **herkenbaar zijn voor onderwijsprofessionals** (docent, roosteraar, planner, onderwijsontwikkelaar, begeleider) en tegelijk **architectuur-proof** blijven. We gebruiken MORA als terminologie-anker: procesnamen, “resultaat”, en “vastgelegd in”.

## 1) Basisprincipe: schrijf rolgericht en resultaatgericht

- **Schrijf in werkwoorden** die onderwijsprofessionals gebruiken: *plannen en roosteren*, *intake en plaatsen*, *ontwikkelen en voorbereiden onderwijs*, *verzorgen onderwijs*.
- **Maak het beleefbaar**: “De student…”, “De planner…”, “De roosteraar…”.
- **Eindig een stap met een resultaat**: “Resultaat: …” of “Vastgelegd in: …”.

Voorbeeld (stijl):
- Slecht: “De lifecycle van offerings gaat van draft naar active.”
- Goed: “De planner maakt planbaar aanbod zichtbaar voor studenten. Resultaat: aanbod met capaciteit en periode. Vastgelegd in: jaarplanning en (planbaar) aanbod.”

## 2) Voorkeurswoorden (MORA-woordgebruik)

Gebruik waar mogelijk deze termen in proza (niet als technische veldnamen):

- **Plannen en roosteren**: jaarplanning, jaarkalender, rooster, roosterwijzigingen, publiceren rooster, clusteren leeractiviteiten, matchen mensen en middelen.
- **Instroom / keuze**: leervraag, persoonlijke leerroute, intake en plaatsen, inschrijven, (aanmelden voor) geroosterde activiteit, wachtlijst/loting (bij instroombeperking).
- **Onderwijsontwikkeling**: onderwijsplan, onderwijsprogramma, opleidingsonderdeel, OER, leermiddelenlijst, (summatieve/formatieve) resultaatstructuur, examenplan.
- **Mensen en middelen**: inzetplanning mensen en middelen, faciliteiten, middelen, lesmateriaal.

Schrijf bij voorkeur “**jaarplanning**” en “**rooster**” (onderwijslogistiek), niet “planninghorizon” of “schedule”.

## 3) Anti-jargon: woorden die snel schuren (en wat je dan schrijft)

Gebruik in onderwijsgerichte tekst liever niet:

- **lifecycle** → schrijf: “van ontwerp naar (planbaar) aanbod naar (geroosterd) aanbod”
- **consumer/producer** → schrijf: “afnemer/leverancier” of nog concreter: “student”, “planningssysteem”, “roostersysteem”, “onderwijscatalogus”
- **payload / endpoint** → schrijf: “bericht”, “koppeling”, “uitwisseling”, “informatiespecificatie”
- **canonical fieldnames** → schrijf: “technische veldnamen (intern)” of verplaats naar een technische bijlage
- **CSP / solver** (als dat niet noodzakelijk is) → schrijf: “haalbaarheidsberekening” of “planningsberekening”
- **objects / entities** → schrijf: “informatieobjecten” (MORA) of “gegevens” / “registratie”
- **offerings** → schrijf: “aanbod” (planbaar aanbod / geroosterd aanbod) en specificeer indien nodig: “met capaciteit” of “met tijd en plaats”

Als je een Engelse term *moet* gebruiken (bijv. omdat hij in een standaard staat), doe dan:
- **Eerst Nederlands, dan Engels tussen haakjes**, en herhaal daarna alleen het Nederlands.
- Geen computerscience termen in het algemeen. 

## 4) Vertaalregels (technisch ↔ onderwijs)

Gebruik deze patronen om een technische boodschap onderwijs-leesbaar te maken:

- **Technisch detail → onderwijszin + ‘vastgelegd in’**  
  “We publiceren X” → “We maken X zichtbaar voor student/docent. Vastgelegd in: …”

- **Data-structuur → werkproces**  
  “We hebben een statusveld” → “De roosteraar publiceert het rooster periodiek en na wijzigingen.”

- **Integratie → ketenafspraak**  
  “Systemen moeten synchroniseren” → “Er is een ketenafspraak nodig: wanneer is aanbod ‘definitief’ en hoe worden wijzigingen doorgegeven?”

## 5) Schrijfpatronen die goed werken in scenario’s

Gebruik voor scenario’s deze vaste blokken:

- **Doel**: één zin in onderwijs-taal.
- **Rollen**: student/docent/planner/roosteraar/onderwijsontwerper/onderwijsontwikkelaar (alleen die relevant zijn).
- **Hoofdflow**: genummerd, maximaal 8–12 stappen, elke stap: *wie doet wat*.
- **Beslispunten**: “als … dan …” (bijv. instroombeperking, te weinig deelnemers, roosterconflict).
- **Vastgelegd in**: 3–6 informatieobjecten (MORA-termen).

## 6) MORA-anker: voorbeeldzinnen (kopieerbaar)

- “**Resultaat**: optimale inzet van mensen en middelen ten behoeve van onderwijs geven en onderwijs krijgen.”
- “**Vastgelegd in**: jaarplanning, rooster, werkverdeling team.”
- “De jaarplanning is het resultaat van het **matchen** van onderwijsaanbod, studenten, BPV, medewerkers en faciliteiten.”
- “De student meldt zich actief aan voor een geroosterde leeractiviteit wanneer er voorwaarden of een maximum aantal plaatsen is.”

## 7) Mini-checklist vóór je publiceert

- Spreekt de tekst over **wat mensen doen** (student/docent/planner/roosteraar) in plaats van wat systemen doen?
- Is elk belangrijk processtuk te herleiden naar een MORA-term (proces of informatieobject)?
- Staat er ergens “lifecycle/consumer/payload/endpoint” in proza? Vervang of verplaats naar bijlage.
- Is bij scenario’s duidelijk: **wat wordt vastgelegd** (persoonlijke leerroute / jaarplanning / rooster / inschrijving)?