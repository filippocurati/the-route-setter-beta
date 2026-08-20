# 10 — Open Decisions and Source Gaps

Questo file è importante: contiene esclusivamente punti che la fonte `Istruzioni(4).md` non determina completamente.

Non sono da interpretare automaticamente come requisiti.

## OPEN-001 — Nomi e URI delle API

La fonte impone REST/OpenAPI ma non definisce URI, metodi e DTO specifici.

**Decisione richiesta:** definire il contratto OpenAPI durante la progettazione backend.

## OPEN-002 — Formato del file Convex Hull

La fonte richiede vertici e, se disponibili, indici delle facce, ma non definisce un formato file preciso.

**Decisione:** scegliere un formato semplice, stabile e facilmente consumabile dal frontend, documentandolo.

## OPEN-003 — Formato esatto del manifest

Il manifest è richiesto ma non è definito nel dettaglio.

**Decisione:** definire DTO e schema OpenAPI.

## OPEN-004 — Libreria .NET Convex Hull

La fonte propone MIConvexHull o equivalente.

**Decisione:** scegliere la libreria compatibile con i requisiti di progetto e documentare la scelta, preferendo se non ci sono complicazione l'utilizzo di MIConvexHull.

## OPEN-005 — Sintassi esatta Rapier

La fonte richiede `@dimforge/rapier3d-compat` e KinematicCharacterController, ma non fissa una versione.

**Decisione:** fissare una versione npm durante l'implementazione e adattare la sintassi alla versione scelta senza modificare il comportamento richiesto.

## OPEN-006 — Dettaglio algoritmo di snap

Il comportamento richiesto è definito, ma non sono prescritti tutti i dettagli dell'algoritmo geometrico per trovare il punto più vicino.

**Decisione:** scegliere un algoritmo robusto e testarlo con superfici piane e non piane.

## OPEN-007 — Regola precisa per superfici complesse

La fonte definisce la normale nel punto di contatto ma non descrive tutti i casi degeneri di geometria, spigoli e vertici.

**Decisione:** definire una politica deterministica durante l'implementazione e coprirla con test.

## OPEN-008 — Movimento prima dello snap

È definito il comportamento dopo lo snap, ma non è completamente specificato il comportamento iniziale della presa prima dell'aggancio.

**Decisione:** definire nel design dell'interazione prima dell'implementazione della fase relativa.

## OPEN-009 — Shortcut

La scelta è esplicitamente delegata all'implementazione.

Devono essere intuitivi e documentati.

## OPEN-010 — Gestione caching GLB dopo il primo utilizzo

La fonte prescrive caricamento lazy, ma non definisce se il GLB debba essere mantenuto in cache dopo l'utilizzo.

**Decisione:** scegliere in funzione della memoria e delle prestazioni, senza violare la distinzione modello/istanza.

## OPEN-011 — Rendering/illuminazione

La fonte non definisce camera iniziale, luci, materiali o tone mapping in dettaglio.

**Decisione:** adottare una configurazione semplice e leggibile senza introdurre requisiti funzionali aggiuntivi.

## OPEN-012 — Risoluzione JPG

È richiesto JPG ad alta risoluzione, ma non è definita una risoluzione numerica.

**Decisione:** fissare una risoluzione target durante il design della funzione.

## OPEN-013 — Metodo di misura FPS

Il target è circa 30 FPS, ma non è definito hardware di riferimento né metodologia di misura.

**Decisione:** definire un benchmark ripetibile.

## OPEN-014 — Session ID

Il documento finale richiede RequestId ed ErrorId nel logging, ma non prescrive più un SessionId.

**Decisione:** non considerarlo requisito obbligatorio; può essere aggiunto come miglioramento se utile e documentato.

## OPEN-015 — HTTP authentication

La fonte non specifica autenticazione/autorizzazione.

**Decisione:** non introdurre un sistema di autenticazione come requisito funzionale senza richiesta esplicita.

## OPEN-016 — Persistenza dei log

È richiesta conservazione/rotazione su file giornalieri, ma non un database di log.

**Decisione:** usare una soluzione file-based coerente con i requisiti, salvo diversa esigenza.
