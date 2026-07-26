# Cvrček Architecture  
**Version:** 0.3.1  
**Status:** Pre‑implementation (safe public version)  
**License:** Apache 2.0  
**Author:** Marek  

---

## Overview  
Cvrček je architektonický koncept pre modulárnu, decentralizovanú a zero‑trust AI infraštruktúru.  
Je navrhnutý ako „AI Spine“ – nervový systém, ktorý riadi sieť malých autonómnych AI modulov (WRK).  
Cvrček nie je model, ale infra vrstva, ktorá umožňuje bezpečné, auditovateľné a predvídateľné správanie AI modulov.

Cvrček je navrhnutý pre:

- edge AI  
- air‑gap prostredia  
- priemyselné systémy  
- bankový sektor  
- kritickú infraštruktúru  
- decentralizované AI siete  

---

## Core Philosophy  
- **AI ako nervový systém, nie ako monolitický mozog.**  
- **Malé modely (mravce) sú základ – rýchle, lacné, bezpečné.**  
- **CAI je miecha a krvný obeh – riadi, smeruje, koordinuje.**  
- **WRK sú reflexy – vykonávajú úlohy autonómne.**  
- **Zero‑trust – každý modul má identitu, stav a je auditovateľný.**  
- **Topológia je striktne riadená – žiadny chaos, žiadne WRK↔WRK spojenia.**

---

## Roles  
### CAI – Control AI  
Riadiaca vrstva.  
Zodpovedá za smerovanie úloh, koordináciu modulov a udržiavanie topológie.  
CAI neinterpretuje obsah úloh – pracuje len s meta‑informáciami.

### WRK – Work AI  
Autonómne moduly vykonávajúce konkrétne úlohy.  
WRK sú malé, rýchle, deterministické a distribuované.  
WRK nikdy nekomunikujú priamo medzi sebou.

### Kontrolor  
Modul zodpovedný za stav siete, registry, scoring a health.  
Zabezpečuje prehľad o stave WRK modulov.

### Vratník  
Vstupná brána systému.  
Sanitizuje a validuje požiadavky predtým, než vstúpia do spine.

### Guardian  
Bezpečnostný modul.  
Monitoruje anomálie, správanie a auditné stopy.

### Archivar  
Dlhodobá pamäť systému.  
Poskytuje historické dáta, auditné logy a RAG.

---

## Topology  
Cvrček používa hierarchickú, rekurzívnu topológiu:

- **CAI → WRK**  
- **WRK môže fungovať ako CAI pre svoje vlastné WRK (WRK‑as‑CAI)**  
- **WRK nikdy nekomunikuje priamo s WRK**  
- **Všetky požiadavky prechádzajú cez CAI alebo Vratníka**  

Topológia je navrhnutá tak, aby bola:

- predvídateľná  
- auditovateľná  
- bezpečná  
- rozšíriteľná  

---

## Communication Model  
Cvrček používa model **envelope/payload**:

### Envelope  
Meta‑informácie o požiadavke:

- identita modulu  
- časová pečiatka  
- stav  
- typ úlohy  
- auditné údaje  

CAI pracuje výhradne s envelope.

### Payload  
Skutočný obsah úlohy.  
Je spracovaný len WRK modulom.  
CAI payload nevidí.

---

## Health Model  
Moduly v spine môžu byť v stavoch:

- **OK** – modul reaguje a je dostupný  
- **DEGRADED** – zvýšená latencia, znížený výkon  
- **CRITICAL** – modul nereaguje alebo je preťažený  
- **ISOLATED** – modul je odpojený od spine  

Tieto stavy slúžia na riadenie topológie a smerovanie úloh.

---

## Grades  
Cvrček podporuje viac úrovní nasadenia:

### Basic Grade  
Domáce laboratórium, tablet, mobil.  
Minimálna bezpečnosť, fokus na funkčnosť.

### Punk Grade  
Komunitné, DIY nasadenia.  
Flexibilné, experimentálne.

### Industry Grade  
Banky, enterprise, priemysel.  
Audit, governance, stabilita.

### Military Grade  
Air‑gap, kritická infraštruktúra.  
Maximálna kontrola, minimálna dôvera.

---

## Implementation Status  
Cvrček je vo fáze pre‑implementácie.  
Detaily protokolov, identity, bezpečnosti a komunikácie budú doplnené až po prvých testoch.

Tento dokument obsahuje iba bezpečné, neprivlastniteľné časti architektúry.

---