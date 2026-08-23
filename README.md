# Cvrček – AI Spine Architecture

Tento repozitár obsahuje otvorený architektonický koncept AI Spine (Cvrček).
Ide o model riadenia AI systému, v ktorom sú AI moduly oddelené od centrálnej
logickej vrstvy (CAI) a komunikujú cez definované protokoly a heartbeat
mechanizmy.

### Rozsah a cieľ (Scope)
Cvrček Spine nie je pokus o nahradenie Linuxu ani Kubernetes. Je to minimalistický 
architektonický rámec a PoC, ktorého úlohou je ukázať, že výkonné, bezpečné 
a offline-first RAG prostredie sa dá prevádzkovať na okraji (Edge/ARM64) 
bez nutnosti cloudu.

Súčasťou repozitára sú manifesty popisujúce architektúru, nie monolitickú 
implementáciu. Koncept je otvorený, nepatentovateľný a môže byť voľne používaný 
v akomkoľvek prostredí bez obmedzení.

## Licencia
Projekt je publikovaný pod licenciou **Apache 2.0**.  
To znamená, že je voľne použiteľný, modifikovateľný a distribuovateľný, bez
patentových nárokov. Plný text licencie nájdete v súbore `LICENSE`.
