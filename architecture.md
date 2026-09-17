# Cvrček Architecture Specification
**Version:** 0.6.5  
**Status:** Pre-implementation Refactored (Safe public version)  
**License:** Apache-2.0  
**Author:** ganagas-rgb  
**Updated at:** 2026-09-17  
---
## Overview
Cvrček je architektonický koncept pre modulárnu, decentralizovanú a zero-trust AI infraštruktúru (AI Spine). Predstavuje nervový systém riadiaci sieť autonómnych AI modulov (WRK) prostredníctvom prísne definovaných rozhraní a transportných línií. Poskytuje identitu (RRID), topológiu, audit, pamäť, bezpečnostné invarianty a oddelené envelope/payload spracovanie pre edge, air-gapped, bankové a kritické enterprise prostredia.
---
## Core Philosophy
* **Nervový systém, nie monolit:** Malé špecializované modely (SLM) tvoria rýchly a bezpečný základ.
* **Strict Envelope/Payload Separation:** CAI spracováva výhradne meta-vrstvu (envelope), WRK pracuje s obsahom (payload).
* **Zero-Trust Topológia:** Každá požiadavka má RRID, stav a nemennú auditnú stopu. Žiadna priama komunikácia WRK <-> WRK.
* **Zero-Dependency Native Parsing:** Plochý dátový kontrakt (flat key=value / INI-style) garantuje spracovateľnosť natívnymi POSIX nástrojmi (Perl 5 / awk / ksh93) bez externalít ako `jq`.
* **Model nie je agent:** Identita (RRID) a pamäť patria agentovi, nie modelu ani inference engine.
---
## Identity & Failover
### Heartbeat Rules
* **Normal Interval:** 5–30s
* **Forced Interval:** 5–60min
* **Invariant:** RRID, name, birth_number a depth sú počas heartbeat-u nemenné.
### Depth & Depth Hops

| Parameter | Popis a Pravidlá |
| :--- | :--- |
| **Depth** | Topologická pozícia uzla v Spine DAG (vlastnosť uzla). Deterministický atribút priraďovaný pri registrácii pod CAI. Vlastní ho výhradne CAI. Pri CAI failoveri sa prepočíta (nový CAI = depth 0). |
| **Depth Hops** | Monotónny per-request counter v rámci jednej požiadky na detekciu rekurzie/smyčiek. Pri každom presmerovaní sa inkrementuje o 1. Resetuje sa len pri novej požiadavke, CAI failover ho NEresetuje. |

**Loop Detection Rule:**
`IF depth_hops > max_depth OR rrid_final IN visited_rrids` $\rightarrow$ `mark_thread("loop_detected")` $\rightarrow$ `append_log("recursive_loop_aborted")`. *(Strop `max_depth` určuje deployment manifest per grade).*
### CAI Failover
* **Triggery:** CAI heartbeat loss, Control plane timeout.
* **Správanie:** Standby CAI preberá rolu aktívneho CAI okamžite. K8s etcd quorum ($2\times\text{ active WRK} + 1\times\text{ MNG}$) garantuje kontinuitu stavu. Nový CAI pokračuje v routovaní bez straty kontextu požiadavky.
* **Post-Failover:** Pôvodný CAI sa po obnovení re-registruje ako WRK; depth sa prepočíta. RRID zostáva nemenné.
### RRID Stack Invariants
* **Visited RRIDs Required:** `true`
* **Štruktúra:** LIFO stack v envelope viazaný na jednu požiadavku. Pri každom hop-e sa pridá aktuálny RRID. Stack sa počas trvania požiadavky nikdy nečistí.
---
## Roles
* **CAI (Control AI):** Riadiaca vrstva Spine. Smeruje úlohy, koordinuje moduly a udržiava topológiu výhradne na základe envelope.
* **WRK (Work AI):** Autonómne moduly vykonávajúce konkrétne úlohy. Bežia ako izolované OCI/K8s kontajnery (`llama-server`) s CPU/RAM limitmi.
* **Vratník (Ingress Gate & Proxy):** Fyzické a logické vstupné rozhranie. Network boundary tvoria HAProxy listener na bastione (`127.0.0.1` cez SSH tunnel). Logical boundary normalizuje, sanitizuje a anonymizuje vstup + vykonáva health checky (`/health`).
* **Kontrolór:** Vyhodnocuje stav siete, zdravie hostiteľov (`SWAP_FREE_PCT`) a latenciu. Zadrží požiadavku pri preťažení.
* **Guardian:** Bezpečnostný modul na monitoring anomálií a auditných stôp.
* **Archivár (Central Data Store / RAG Backend):** Centrálny RAG a pamäťový modul (zero-daemon envelope-based graph store).
  * **Hot Storage:** `tmpfs` append-only buffer, `.git` história na disku.
  * **Warm Storage:** `age` + `pqc` šifrované bundly s Tail Indexom.
  * **Cold Storage:** Read-only `mmap`, immutable, `YEAR_MONTH/DAY` partitioning.
  * **Archive Storage:** POSIX filesystem, off-site, export cez JSON stream.
  * **Audit Invariants:** Vyžaduje sa `envelope_hash` aj `payload_hash` (SHA-256).
---
## Engine Microkernel
* **Binary Size Target:** $<100\text{ kB}$ (C99 / POSIX)
* **Interpreter:** `wasm3-embedded` (zero-dependency)
* **Účel:** Slepý hardvérový akcelerátor. Formáty, kvantizácia, tokenizácia a RAG routing sú modulárne WASM knižnice mimo jadra.
* **Hot Swap:** Ľubovoľný WASM modul je vymeniteľný za chodu bez reštartu jadra.
---
## Worker Pipeline & Security
**Stages:**
1. **Blind-index gate:** Bezstavový C99/WASM3 filter, spúšťa `age` dekrypciu pri zhode.
2. **Age-decrypt:** Post-quantum age dekrypcia (ML-KEM-768 + X25519).
3. **Sanitizer:** Agresívne PII stripping + normalizácia.
4. **LLM-screen:** Lokálny screening model, JSON skóre (toxicity/pii/hallucination).
5. **Local-RAG:** Offline KB lookup.
**Execution Policy & Security:**
* **Mode:** `STRICT_DETERMINISTIC`
* **Retry Policy:** 3 pokusy (`idempotency_guarantee: RAM_ONLY_PIPE_ZERO_SIDE_EFFECTS`)
* **Failure Exhausted:** Presun do `dead_letter`, admin alert
* **Network & IPC:** `air_gapped: true`, `allow_external_network: false`, IPC cez `POSIX_UNIX_PIPES`.
---
## Topology & Quorum
* **Data Flow:** UNIX/AIX Agent (Tcl/Perl) $\rightarrow$ SSH Tunnel $\rightarrow$ Bastion Ingress (HAProxy:8080) $\rightarrow$ K8s Service $\rightarrow$ WRK Pod (`llama-server`).
* **Isolation:** Legacy uzly sa nepripájajú priamo do K8s klastra, komunikujú výhradne cez bastion proxy.
* **Control Plane Quorum:** RKE2 K8s klaster drží etcd quorum: $2\times\text{ Active WRK}$ (ESXi, voting members) $+ 1\times\text{ Standby Quorum Node}$ (MNG, tie-breaker).
---
## Data Contract & Naming Standard
**Standard:** Flat Data Contract (Bank Grade Compliance) pre natívny Perl/awk/ksh93 parsing bez `jq`.
* **Delimiter:** Nové riadky (LF) oddeľujú kľúče; prvý znak `=` oddeľuje kľúč a hodnotu.
* **Escapes:** Percent-encoded hodnoty, bez netlačiteľných znakov a odriadkovaní.
* **LIFO Stack Order:** Appendovanie na koniec (napr. `visited_rrids=rrid001:rrid002:rrid003`).
**Naming Invariant:** `TS_THREADID_REQID_TYPE.json`
* **TS:** Timestamp `YYYYMMDDHH24MISS`
* **THREADID / REQID:** 16 hex znakov (64-bit entropia)
* **TYPE:** `envelope` \| `payload`
---
## Swap Modes & Deployment Grades
### Swap Modes
* **Hot Swap:** Plynulý failover bez prerušenia požiadavky (vyžaduje K8s HA cluster a redundantný CAI/WRK).
* **Model Swap:** Single-node prostredie; prebiehajúca odpoveď pri výmene padne a požiadavku treba zopakovať.
### Deployment Grades
* **Punk:** Prototyping na lokálnom ARM64/Termux prostredí (Day-1, zero governance).
* **Industry:** Edge autonómia s malými modelmi 0.5B–3.8B (Day-2 stable operation).
* **Military:** Air-gapped operácie, etcd quorum, auto-wipe po misii.
* **Bank:** Striktný audit, stateless správanie, POSIX zero-dependency parsing, air-gapped RKE2 lifecycle management.
---
## Manifest References & Rejected Alternatives
**Zastrešené Manifesty:**
* **Identity & Control:** `CvrcekEnvelope v1.4.1`, `ControlManifest v3.0.2`, `AgentThreadManifest v1.0.2`
* **Naming & Formats:** `NamingStandardManifest v0.0.1`, `cvrcek-core-storage-formats v0.0.1`
* **RAG & Storage:** `cvrcek-rag-db v1.0.3`, `cvrcek-rag-tmp-scratchpad`, `cvrcek-rag-tmp-session`, `cvrcek-rag-log-thread v0.1.2`, `cvrcek-rag-main-global`
* **Engine & Pipeline:** `AIEngineManifest v0.1.4`, `AIHybridDefinition v0.7.1`, `wrk-pqc-envelope-gate-v3`
**Rejected Alternatives:**
* Kvantová terminológia a emulátory (napr. `wasm3_tensor_emulator`, `qubits_emulated`)
* Python závislosti pre POSIX agentov
* Hlboko zanorený JSON pre malé modely
* Centralizovaná distribúcia kľúčov v produkcii
* PKI pre vector obfuscation