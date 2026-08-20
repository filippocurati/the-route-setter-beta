# Piano di implementazione
## Applicazione per tracciatura vie climbing indoor

## Come usare questo piano (istruzioni per l'agente AI)

1. Leggere per intero, prima di iniziare, `00-costituzione.md`, `01-specifica-requisiti.md` e `02-design-tecnico.md`. Questo piano non ripete i dettagli già presenti in quei documenti: ogni task li referenzia tramite ID (`REQ-...`) o numero di sezione del design.
2. Eseguire le fasi nell'ordine indicato, salvo dove esplicitamente segnalata la possibilità di parallelizzare (sezione "Dipendenze tra fasi").
3. Non passare alla fase successiva finché la **Definition of Done** della fase corrente non è soddisfatta.
4. Ad ogni task completato, verificare che nessun principio di `00-costituzione.md` sia stato violato, anche se il task non lo richiama esplicitamente.
5. In caso di ambiguità non coperta né dai requisiti né dal design tecnico, adottare l'interpretazione più conservativa rispetto ai principi della costituzione e segnalarla esplicitamente, invece di procedere silenziosamente con un'assunzione.
6. Ogni fase include i propri task di test: non vanno rimandati a una fase finale unica, per poter verificare la fase appena chiusa prima di costruirci sopra.

---

## Dipendenze tra fasi

```
FASE 0 (Setup)
   │
   ├──► FASE 1 (Backend: Convex Hull) ──┐
   │                                     │
   └──► FASE 2 (Backend: API/File statici) ──► FASE 3 (Backend: errori/logging)
                                          │
                                          ▼
                                   FASE 4 (Frontend: scaffold scena)
                                          │
                                          ▼
                                   FASE 5 (Frontend: Rapier/fisica) ◄── richiede collider da FASE 1/2
                                          │
                                          ▼
                                   FASE 6 (Frontend: interazione prese)
                                          │
                                          ▼
                                   FASE 7 (Frontend: catalogo/UI)
                                          │
                    ┌─────────────────────┼─────────────────────┐
                    ▼                     ▼                     ▼
             FASE 8 (Immagine)   FASE 9 (Errori frontend)  FASE 10 (E2E/prestazioni)
                    │                     │                     │
                    └─────────────────────┴─────────────────────┘
                                          ▼
                                   FASE 11 (Documentazione)
                                          ▼
                                   FASE 12 (Pipeline CI)
```

Le fasi 1 e 2 possono procedere in parallelo (entrambe dipendono solo da FASE 0). La fase 3 (errori/logging backend) può iniziare in parallelo a FASE 1/2 se si preferisce, ma va completata prima di considerare chiuso il backend. Le fasi 8, 9 e 10 possono procedere in parallelo una volta chiusa la fase 7.

---

## FASE 0 — Setup del progetto

**Obiettivo**: scheletro di backend e frontend pronti a ricevere le implementazioni successive.

- **T0.1** — Scaffold del progetto backend ASP.NET Core Web API, con struttura cartelle come da `02-design-tecnico.md` §2, configurazione Swagger/OpenAPI. *(REQ-ARC-001, REQ-ARC-002)*
- **T0.2** — Scaffold del progetto frontend (TypeScript + Vite secondo l'assunzione di design §2, o stack alternativo se corretto in fase di revisione del design). *(REQ-ARC-003)*
- **T0.3** — Predisposizione delle cartelle `Data/Pareti` e `Data/Prese` con almeno un modello GLB di parete e 2-3 modelli GLB di presa di test (anche placeholder), per poter sviluppare e testare le fasi successive. *(REQ-MOD-001)*

**Definition of Done**: backend avviabile con Swagger UI raggiungibile; frontend avviabile con hot-reload; cartelle dati di test popolate.

---

## FASE 1 — Backend: generazione dei Convex Hull

- **T1.1** — Servizio di lettura dei vertici di una mesh da file GLB (es. tramite SharpGLTF). *(design §4)*
- **T1.2** — Servizio di generazione dell'inviluppo convesso a partire dai vertici, tramite libreria .NET indipendente da Rapier. *(REQ-HUL-001)*
- **T1.3** — Salvataggio del collider generato (vertici, indici, hash del GLB sorgente) in `collider.json` nella cartella della presa. *(REQ-HUL-002, REQ-MOD-003)*
- **T1.4** — Logica di invalidazione: confronto tra hash salvato e hash corrente del GLB; rigenerazione solo se diversi o se il file manca. *(REQ-HUL-003)*
- **T1.5** — Esecuzione della scansione/generazione come task in background all'avvio del backend, non bloccante. *(REQ-HUL-004)*
- **T1.6** — Test xUnit: collider mancante → generato; collider presente e coerente → non rigenerato; GLB modificato → rigenerato. *(REQ-TST-003)*

**Definition of Done**: avviando il backend su una cartella "Prese" senza collider precalcolati, al termine dell'avvio ogni presa con GLB valido ha un `collider.json` associato; i test T1.6 passano tutti.

---

## FASE 2 — Backend: API REST e file statici

- **T2.1** — Endpoint `GET /api/wall` + serving statico della cartella "Pareti". *(REQ-SCN-004, REQ-MOD-005)*
- **T2.2** — Endpoint `GET /api/holds` (manifest catalogo) + serving statico di GLB/PREV_/collider dalla cartella "Prese", tollerante ai file mancanti (solo GLB obbligatorio). *(REQ-CAT-003, REQ-MOD-002, REQ-MOD-004)*
- **T2.3** — Endpoint `GET /api/holds/{id}/model` e `GET /api/holds/{id}/collider` per il caricamento on-demand. *(REQ-CAT-004, REQ-HUL-005)*
- **T2.4** — Endpoint `POST /api/logs`. *(REQ-LOG-002, verrà completato dal punto di vista del contenuto in FASE 3)*
- **T2.5** — Documentazione Swagger di tutti gli endpoint. *(REQ-ARC-002)*
- **T2.6** — Test di integrazione REST per ciascun endpoint. *(REQ-TST-004)*

**Definition of Done**: tutti gli endpoint rispondono correttamente su dati di test; una presa senza texture viene comunque inclusa nel manifest senza errori; i test T2.6 passano tutti.

---

## FASE 3 — Backend: gestione errori e logging

**Obiettivo generale**: implementazione della gestione centralizzata degli errori sia lato backend che lato frontend *(REQ-ERR-001)* — la parte frontend è completata in FASE 9.

- **T3.1** — Middleware centralizzato di gestione eccezioni: ID univoco, log server-side completo, risposta generica al client. *(REQ-ERR-002, REQ-ERR-003)*
- **T3.2** — Configurazione del logging strutturato (JSON), livelli standard `LogLevel`, soglia minima configurabile via `appsettings.json`. *(REQ-LOG-004, REQ-LOG-005)*
- **T3.3** — Sanitizzazione dei dati di contesto prima della scrittura nel log. *(REQ-LOG-003)*
- **T3.4** — Rotazione giornaliera dei file di log con retention 7 giorni. *(REQ-LOG-006)*
- **T3.5** — Implementazione asincrona/non bloccante del logging. *(REQ-LOG-001)*
- **T3.6** — Completamento dell'endpoint `POST /api/logs` per ricevere e registrare eventi dal frontend. *(REQ-LOG-002)*
- **T3.7** — Test unitari xUnit del middleware di gestione eccezioni e dei servizi di logging (a completamento dei test già previsti in T1.6/T2.6 per catalogo e Convex Hull). *(REQ-TST-002)*

**Definition of Done**: un'eccezione simulata in un endpoint di test produce una risposta generica con ID errore e una voce di log corrispondente completa; nessun dato sensibile presente in un campione di log generato durante i test; dopo la simulazione di più giorni, i file di log oltre 7 giorni risultano eliminati.

---

## FASE 4 — Frontend: scaffold della scena 3D

- **T4.1** — Bootstrap three.js: scena, renderer, camera prospettica, `OrbitControls` centrati sulla parete. *(REQ-UI-003)*
- **T4.2** — Caricamento della parete da `GET /api/wall` all'avvio. *(REQ-SCN-004)*
- **T4.3** — Impostazione del sistema di coordinate/unità (1 unità three.js = 1 metro). *(REQ-FIS-001)*

**Definition of Done**: all'apertura dell'applicazione la parete è visibile e navigabile con mouse (orbit, zoom, pan) senza ulteriori azioni dell'utente.

---

## FASE 5 — Frontend: integrazione Rapier e fisica

- **T5.1** — Inizializzazione del mondo fisico Rapier (WASM) lato client, gravità nulla. *(REQ-FIS-002, REQ-ARC-004)*
- **T5.2** — Collider TriMesh della parete generato lato client al caricamento, come entità indipendente dalla mesh grafica. *(REQ-FIS-012, REQ-FIS-011)*
- **T5.3** — Recupero del collider precalcolato di una presa da `GET /api/holds/{id}/collider` e costruzione del collider Rapier corrispondente, mantenuto separato dalla mesh grafica della presa. *(REQ-HUL-005, REQ-FIS-011)*
- **T5.4** — Configurazione del `KinematicCharacterController` per il movimento vincolato, con autostep/snap-to-ground disattivati. *(REQ-FIS-004)*
- **T5.5** — Test headless (Vitest/Jest, Rapier senza rendering): presa non attraversa la parete; due prese non si compenetrano; posizionamento corretto in assenza di collisioni; rilascio corretto dello spazio alla rimozione; determinismo dei risultati. *(REQ-TST-006, REQ-TST-007)*

**Definition of Done**: in ambiente di test headless, tutti gli scenari di T5.5 passano in modo deterministico su esecuzioni ripetute.

---

## FASE 6 — Frontend: interazione con le prese

- **T6.1** — Selezione di un'istanza in scena tramite click del mouse, con evidenziazione visiva. *(REQ-SCN-002)*
- **T6.2** — Origine locale delle prese nel centro della superficie posteriore; calcolo della normale nel punto di contatto. *(REQ-FIS-005)*
- **T6.3** — Snap automatico entro 5 cm dalla parete, con orientamento secondo la normale locale. *(REQ-FIS-006, REQ-FIS-008)*
- **T6.4** — Bottoni + shortcut da tastiera per la rotazione (1°/click, continua a pressione). *(REQ-FIS-009)*
- **T6.5** — Bottoni + shortcut da tastiera per la traslazione (1 cm/click, continua a pressione), con proiezione degli assi vista→piano tangente. *(REQ-FIS-010)*
- **T6.6** — Applicazione del movimento vincolato tramite `KinematicCharacterController` (move-and-slide) per traslazione e verifica di non-compenetrazione, garantendo reattività percepita come immediata e nessuna chiamata di rete durante il movimento continuo. *(REQ-FIS-003, REQ-FIS-007, REQ-PRF-003, REQ-LOG-007)*

**Definition of Done**: una presa selezionata si aggancia automaticamente entro 5 cm dalla parete; si sposta con i bottoni/tastiera restando aderente, con feedback visivo percepito come immediato e senza generare traffico di rete durante il movimento continuo; incontrando un'altra presa o il bordo della parete, continua a scorrere sulle componenti di movimento libere invece di bloccarsi bruscamente; la rotazione avviene attorno al punto di contatto.

---

## FASE 7 — Frontend: catalogo e struttura UI

- **T7.1** — Caricamento del manifest e delle anteprime all'avvio, con cache di sessione. *(REQ-CAT-003)*
- **T7.2** — Rendering del pannello catalogo: box per presa con anteprima PREV_, bottone "Utilizza", bottone "Dettagli". *(REQ-UI-001, REQ-CAT-001, REQ-CAT-002)*
- **T7.3** — Pannello modale "Dettagli": caricamento on-demand del GLB al click, rilascio del modello e del contesto grafico alla chiusura. *(REQ-CAT-005)*
- **T7.4** — Flusso "Utilizza": caricamento on-demand del GLB, creazione istanza in scena, rimozione dal catalogo, blocco della riselezione della stessa presa. *(REQ-CAT-004, REQ-CAT-006, REQ-CAT-007, REQ-SCN-001)*
- **T7.5** — Bottone "Rimuovi presa": rimozione dell'istanza selezionata dalla scena e ripristino nel catalogo. *(REQ-SCN-003, REQ-CAT-006)*
- **T7.6** — Menu superiore con i bottoni "Genera immagine" e "Rimuovi presa" (quest'ultimo attivo solo con istanza selezionata). *(REQ-UI-002)*
- **T7.7** — Verifica che non esista alcun meccanismo di salvataggio: ricaricando la pagina, la scena riparte da zero (parete vuota, catalogo completo). *(REQ-ARC-006)*

**Definition of Done**: flusso manuale completo eseguibile senza errori: apertura applicazione → dettagli di una presa → utilizzo di una presa → posizionamento sulla parete → rimozione → ricomparsa nel catalogo.

---

## FASE 8 — Generazione dell'immagine guida

- **T8.1** — Vista ortografica frontale dedicata, dimensionata sulla parete. *(REQ-IMG-001)*
- **T8.2** — Occultamento della UI e sfondo bianco durante la generazione. *(REQ-IMG-002)*
- **T8.3** — Esportazione JPG ad alta risoluzione e download del file. *(REQ-IMG-003)*

**Definition of Done**: cliccando "Genera immagine" con alcune prese posizionate si ottiene un file JPG scaricabile, privo di elementi UI, su sfondo bianco, con le proporzioni della parete rispettate.

---

## FASE 9 — Gestione errori lato frontend

**Obiettivo generale**: completamento della gestione centralizzata degli errori *(REQ-ERR-001)*, lato frontend (la parte backend è stata completata in FASE 3).

- **T9.1** — Intercettazione di tutte le categorie di errore elencate in `REQ-ERR-004` (JS non gestiti, REST, HTTP, caricamento/parsing modelli, three.js, Rapier, generazione immagine, funzionalità UI principali).
- **T9.2** — Notifica al backend (`POST /api/logs`) degli errori intercettati, quando tecnicamente possibile.
- **T9.3** — Verifica che un errore isolato non comprometta le funzionalità non coinvolte. *(REQ-ERR-005)*

**Definition of Done**: simulando un errore in ciascuna categoria elencata (es. GLB corrotto, endpoint non raggiungibile), l'applicazione mostra un messaggio comprensibile, registra l'evento lato server quando possibile, e resta utilizzabile per le funzioni non coinvolte.

---

## FASE 10 — Test end-to-end e verifica prestazioni

- **T10.1** — Suite Playwright sui flussi principali: caricamento parete, utilizzo presa dal catalogo, spostamento/rotazione, rimozione, generazione immagine, eseguita sulle ultime versioni stabili di Chrome, Edge e Firefox. *(REQ-TST-005, REQ-ARC-005)*
- **T10.2** — Scenario di prova con almeno 40 prese posizionate in scena, misurazione del framerate durante l'interazione su hardware con GPU integrata (non dedicata), rappresentativo del target descritto nella specifica. *(REQ-PRF-002, REQ-PRF-004, REQ-PRF-001)*
- **T10.3** — Verifica dell'assenza di blocchi dell'interfaccia durante caricamento e posizionamento. *(REQ-PRF-005)*
- **T10.4** — Eventuale ottimizzazione (istanziazione geometrie, riduzione draw call, ecc.) se il target prestazionale di FASE 10.2 non è raggiunto.

**Definition of Done**: suite Playwright interamente verde; framerate ≥ 30 FPS con 40 prese in scena su hardware di riferimento con GPU integrata; nessun blocco percepibile dell'interfaccia durante i test manuali di caricamento.

---

## FASE 11 — Documentazione

- **T11.1** — Completamento della documentazione inline in italiano su tutte le classi e i metodi pubblici, backend e frontend. *(REQ-DOC-001)*
- **T11.2** — Documento markdown generale: funzionamento, architettura, componenti, istruzioni di avvio live e debug. *(REQ-DOC-002)*
- **T11.3** — Sei diagrammi richiesti: architetturale, cartelle sorgenti, API, ciclo di vita delle prese, flusso UI, gestione compiti backend/frontend. *(REQ-DOC-003)*
- **T11.4** — Documento sui test automatici: scopi, risultati attesi, framework utilizzati. *(REQ-DOC-004)*
- **T11.5** — Organizzazione di tutta la documentazione (esclusa quella inline) nella cartella `/docs`. *(REQ-DOC-005)*

**Definition of Done**: seguendo esclusivamente le istruzioni del documento generale, è possibile avviare l'applicazione da zero sia in debug che in modalità live; tutti i sei diagrammi sono presenti e coerenti con il codice effettivo.

---

## FASE 12 — Pipeline di integrazione continua

- **T12.1** — Configurazione della pipeline CI per eseguire, ad ogni build: test xUnit (unitari + integrazione), test Vitest/Jest della fisica, test Playwright e2e. *(REQ-TST-001)*
- **T12.2** — Configurazione della pipeline affinché fallisca se uno qualsiasi dei test fallisce.

**Definition of Done**: un fallimento intenzionale introdotto in un qualsiasi test (backend, fisica o e2e) fa fallire l'intera pipeline.

---

## Tracciabilità

Ogni requisito di `01-specifica-requisiti.md` è coperto da almeno un task di questo piano. In caso di modifica futura ai requisiti, verificare quali task delle fasi elencate vanno rivisti prima di procedere con ulteriori sviluppi, per evitare derive tra specifica e implementazione.
