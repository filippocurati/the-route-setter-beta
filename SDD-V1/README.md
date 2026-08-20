# Pacchetto di specifiche SDD
## Applicazione per tracciatura vie climbing indoor

Questo pacchetto traduce il documento di specifiche originale (`Istruzioni.md`) in un flusso Spec-Driven Development a quattro livelli, pensato per essere fornito a un agente AI di sviluppo.

## Documenti e ordine di lettura

1. **`00-costituzione.md`** — Principi non negoziabili, validi in ogni fase. Da leggere per primo e da tenere presente per tutta la durata dello sviluppo.
2. **`01-specifica-requisiti.md`** — Cosa deve fare il sistema. Requisiti funzionali e non funzionali con ID univoci (`REQ-<DOMINIO>-<NUMERO>`) e criteri di accettazione verificabili.
3. **`02-design-tecnico.md`** — Come viene realizzato ciascun requisito: architettura, contratti API, struttura delle cartelle, decisioni tecniche. Segnala esplicitamente (sezione 10) i punti dove è stata fatta un'assunzione tecnica non esplicitamente richiesta nella specifica originale — da confermare o correggere prima di iniziare.
4. **`03-piano-implementazione.md`** — In che ordine costruire il sistema: fasi, task atomici tracciati sugli ID dei requisiti, dipendenze tra fasi, criterio di "fatto" per ciascuna fase.

## Flusso di lavoro consigliato per l'agente AI

1. Leggere per intero i quattro documenti, in ordine, prima di scrivere qualsiasi riga di codice.
2. Se una delle assunzioni tecniche della sezione 10 di `02-design-tecnico.md` non è gradita, correggerla (o farla correggere) prima di procedere: diversi task del piano vi fanno riferimento diretto.
3. Eseguire le fasi del piano nell'ordine indicato, rispettando le dipendenze dichiarate.
4. Per ciascuna fase: implementare i task elencati, eseguire i relativi test, verificare la Definition of Done della fase, **prima** di passare alla fase successiva. Non accumulare fasi non verificate.
5. In caso di dettaglio implementativo non coperto né dalla specifica dei requisiti né dal design tecnico, adottare l'interpretazione più conservativa rispetto ai principi della costituzione e segnalarla esplicitamente invece di procedere silenziosamente.
6. Se in corso d'opera si rende necessaria una modifica a un requisito, la modifica va fatta prima nel documento dei requisiti (e, se necessario, nel design tecnico), poi propagata al codice — non il contrario.

## Perché questa struttura

Il documento originale (`Istruzioni.md`) resta un'ottima base di conoscenza di dominio e di intento del progetto, ma mescola requisiti, decisioni architetturali e dettagli implementativi in un unico testo lineare. Per un progetto di questa complessità — backend .NET, frontend three.js/Rapier, fisica custom, generazione e caching di collider, tre framework di test diversi — generare il codice in un unico passaggio a partire da un documento piatto espone al rischio di deriva tra le parti generate in momenti diversi della sessione dell'agente, e non lascia punti di verifica intermedi prima che l'intero sistema sia stato scritto. La scomposizione in requisiti tracciabili, design esplicito e piano a fasi verificabili riduce questo rischio, senza introdurre formalismi (es. sintassi EARS integrale, matrici di tracciabilità) sproporzionati rispetto alla scala del progetto.
