Cvrček Architecture

Version: 0.4.1Status: Pre‑implementation (safe public version)License: Apache 2.0Author: Marek

Overview

Cvrček je architektonický koncept pre modulárnu, decentralizovanú a zero‑trust AI infraštruktúru. Predstavuje „AI Spine“ – nervový systém, ktorý riadi sieť malých autonómnych AI modulov (WRK) cez prísne definované rozhrania.

Cvrček nie je model. Je to infra vrstva, ktorá poskytuje identitu (RRID), topológiu, audit, pamäť, bezpečnostné invarianty a envelope/payload komunikáciu pre AI moduly v edge, air‑gap, priemyselných, bankových a kritických prostrediach.

Core Philosophy

AI ako nervový systém, nie monolitický mozog.

Malé modely (mravce) sú základ – rýchle, lacné, bezpečné.

CAI pracuje s meta‑vrstvou (envelope), WRK s obsahom (payload).

Zero‑trust – každý modul má RRID, stav a auditnú stopu.

Topológia je striktne riadená – žiadne WRK↔WRK spojenia.

Identita, pamäť a RAG sú centralizované v Archivári.

Roles

CAI – Control AI

Riadiaca vrstva Spine. Smeruje úlohy, koordinuje moduly, udržiava topológiu. Pracuje výhradne s envelope. Payload nikdy nevidí.

WRK – Work AI

Autonómne moduly vykonávajúce konkrétne úlohy. Malé, rýchle, deterministické, distribuované. WRK nikdy nekomunikuje priamo medzi sebou – vždy cez CAI alebo Vratník.

Vratník – Ingress Gate

Vstupná brána systému. Normalizuje, sanitizuje a voliteľne anonymizuje vstup. Režimy: sanitize_only, sanitize_and_anonymize, sanitize_no_anonymize. Envelope sa pridelí až po testoch/kontrolórovi.

Kontrolor

Modul pre stav siete, scoring, health a intervenčné rozhodnutia. Vie zastaviť alebo podržať požiadavku pred vytvorením envelope.

Guardian

Bezpečnostný modul. Monitoruje anomálie, správanie a auditné stopy.

Archivár – Central Data Store / RAG

Špecializovaný WRK modul typu central_data_store. Centrálna pamäťová a RAG služba: identita (RRID registry), kontext, vektorové indexy, auditné logy, historické envelope/payload artefakty.

Pamäťové vrstvy:

hot – RAM/tmpfs (session scratchpad)

warm – SQLite (metadata, RRID, audit)

cold – vector DB (globálna znalostná báza)

archive – POSIX filesystem (nemenné logy, age‑rotované kľúče)

Topology

Cvrček používa hierarchickú, rekurzívnu topológiu:

CAI → WRK (centrálne riadené smerovanie)

WRK môže fungovať ako lokálny CAI pre svoje pod‑WRK (WRK‑as‑CAI)

WRK nikdy nekomunikuje priamo s WRK

Archivár je cluster‑wide uzol, nie smerovací modul

Všetky požiadavky prechádzajú cez CAI alebo Vratníka

Topológia je predvídateľná, auditovateľná, bezpečná, rozšíriteľná a kompatibilná s air‑gap a offline režimom.

Communication Model (Envelope/Payload)

Envelope

Meta‑vrstva požiadavky:

RRID source/target/final

timestamp

depth_hops

visited_rrids

integrity_hash

payload_hash

audit_trace

CAI pracuje výhradne s envelope. Envelope je nemenná meta‑vrstva uložená v Archivári ako append‑only audit artefakt.

Payload

Skutočný obsah úlohy:

context

data

sealed_with (age)

rrid_payload

Spracovaný len WRK modulom. CAI payload nevidí. Archivár ukladá payload ako age‑šifrovaný artefakt.

Naming Standard (Cluster‑Wide Invariant)

Všetky envelope/payload súbory používajú jednotný formát:

TS_THREADID_REQID_TYPE.json

Kde:

TS = timestamp YYYYMMDDHH24MISS

THREADID = 16 hex znakov (64‑bit entropia)

REQID = 16 hex znakov (64‑bit entropia)

TYPE = payload alebo envelope

Štandard je definovaný v NamingStandardManifest a referencovaný vo všetkých manifestoch.

Health Model

Moduly v spine môžu byť v stavoch:

OK – modul reaguje a je dostupný

DEGRADED – zvýšená latencia, znížený výkon

CRITICAL – modul nereaguje alebo je preťažený

ISOLATED – modul je odpojený od spine

Stavy riadia smerovanie, výber WRK modulov, fallbacky a zásahy Kontrolóra/Guardiana.

Grades

🛠️ Punk Grade

Rýchly prototyp. Minimum disciplíny, maximum rýchlosti.

🏭 Industry Grade

Edge autonómia bez cloudu. Mini/pidi modely (0.5M–50M) s nízkou latenciou.

🪖 Military Grade

Splniť úlohu za každú cenu. Air‑gap, redundancia, panic‑wipe po misii.

🏦 Bank Grade

Audit, anonymizácia, stateless správanie. Každé rozhodnutie musí byť dokázateľné.

Implementation Status

Cvrček je vo fáze pre‑implementácie. Konkrétne manifesty definujú detaily identity, bezpečnosti, komunikácie, pamäťových vrstiev a auditných invariantov:

RRIDStandard

NamingStandardManifest

ArchivarModuleManifest

VratnikDefinition

CAIIngressPipelineManifest

AIEngineManifest

AgentThreadManifest

Tento dokument je bezpečná, verejná architektonická vrstva – neobsahuje privátne kľúče, deployment detaily ani interné konfigurácie.