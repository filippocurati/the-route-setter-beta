# Specifica dei requisiti
## Applicazione per tracciatura vie climbing indoor

## Come leggere questo documento

Ogni requisito ha un ID univoco nella forma `REQ-<DOMINIO>-<NUMERO>`, una descrizione sintetica e uno o più criteri di accettazione verificabili (ciò che deve essere vero perché il requisito si consideri soddisfatto). Il piano di implementazione (`03-piano-implementazione.md`) referenzia questi ID per garantire tracciabilità tra task di sviluppo e requisito coperto. Il documento di design tecnico (`02-design-tecnico.md`) referenzia gli stessi ID per spiegare **come** ciascun requisito viene realizzato.

Domini: `ARC` architettura, `MOD` gestione file dei modelli, `CAT` catalogo prese, `SCN` scena e istanze, `FIS` fisica e movimento, `HUL` generazione Convex Hull, `UI` interfaccia utente, `IMG` generazione immagine guida, `PRF` prestazioni, `ERR` gestione errori, `LOG` logging, `TST` test automatici, `DOC` documentazione.

---

## ARC — Architettura generale

**REQ-ARC-001 — Architettura a due livelli.**
Il sistema è composto da un backend (C# ASP.NET Core Web API) e un frontend (three.js), comunicanti tramite REST API in formato JSON.
- Criteri: il backend espone endpoint REST documentati; nessuna logica di rendering o di stato di scena risiede nel backend.

**REQ-ARC-002 — Standard delle API.**
Le API REST devono seguire lo standard OpenAPI, con separazione tra Controller e Service, e documentazione Swagger accessibile.
- Criteri: esiste un endpoint Swagger UI funzionante; ogni controller delega la logica di dominio a un service dedicato, senza logica di business nei controller.

**REQ-ARC-003 — Rendering esclusivamente frontend.**
Nessun rendering 3D deve avvenire lato server.
- Criteri: il backend non referenzia librerie di rendering grafico; l'unico output binario prodotto dal backend sono i file statici dei modelli e i collider precalcolati.

**REQ-ARC-004 — Rapier esclusivamente client-side.**
Il motore fisico Rapier deve essere integrato lato frontend tramite il binding ufficiale JavaScript/WebAssembly (`@dimforge/rapier3d-compat`), eseguito interamente nel browser. Il backend non deve avere alcuna dipendenza da Rapier né eseguire simulazione fisica.
- Criteri: il progetto backend non referenzia pacchetti Rapier; ogni interazione di spostamento/rotazione di una presa produce feedback visivo senza alcuna chiamata di rete.

**REQ-ARC-005 — Compatibilità browser.**
L'applicazione deve essere utilizzabile dagli utenti esclusivamente via browser, senza installazione di software, sulle ultime versioni stabili di Chrome, Edge e Firefox con supporto WebGL 2.0.
- Criteri: l'applicazione avvia la scena 3D e resta pienamente funzionante sui tre browser indicati; nessun passaggio del flusso utente richiede plugin o installazioni.

**REQ-ARC-006 — Nessuna persistenza.**
In questa versione non deve esistere alcun meccanismo di salvataggio o storicizzazione delle tracciature, delle sessioni o dello stato utente.
- Criteri: nessun endpoint di scrittura persistente è esposto; ricaricando la pagina, la scena riparte da zero (parete vuota, catalogo completo).

---

## MOD — Gestione dei file dei modelli

**REQ-MOD-001 — Struttura delle cartelle.**
Il backend gestisce una cartella "Pareti" (un solo modello di parete) e una cartella "Prese" contenente una sottocartella "Hold" + suffisso numerico per ciascun modello di presa.
- Criteri: aggiungendo una nuova cartella "Hold_N" con un GLB valido, questa compare nel catalogo al successivo avvio senza modifiche al codice.

**REQ-MOD-002 — Caricamento tollerante ai file mancanti.**
Ogni cartella "Hold" può contenere solo il file GLB oppure anche texture o altri file accessori; l'applicazione carica solo i file effettivamente presenti.
- Criteri: una presa priva di texture viene comunque caricata e visualizzata correttamente (con materiale di default se necessario), senza errori bloccanti.

**REQ-MOD-003 — File del collider precalcolato.**
Ogni cartella "Hold" contiene (dopo la prima generazione) un file addizionale con il collider Convex Hull precalcolato per quella presa. Vedi dominio `HUL` per la logica di generazione.
- Criteri: dopo il primo avvio dell'applicazione, ogni cartella "Hold" con un GLB valido contiene il relativo file collider.

**REQ-MOD-004 — Immagine di anteprima.**
Ogni cartella "Hold" contiene un file immagine con prefisso "PREV_", usato come anteprima nel catalogo.
- Criteri: il catalogo mostra l'immagine PREV_ corretta per ciascuna presa; una presa priva di questo file non deve causare il fallimento del caricamento dell'intero catalogo (v. REQ-ERR-004).

**REQ-MOD-005 — File statici.**
Tutti i file dei modelli, delle texture, delle anteprime e dei collider sono serviti come file statici locali dal backend.
- Criteri: ogni file è raggiungibile tramite un URL statico stabile servito dal backend.

---

## CAT — Catalogo delle prese

**REQ-CAT-001 — Posizione e struttura del catalogo.**
Il catalogo delle prese è visualizzato nel pannello sinistro dell'interfaccia, con un elenco verticale di box, uno per presa.
- Criteri: tutte le prese non ancora utilizzate sono visibili nel pannello, scorrevole se necessario.

**REQ-CAT-002 — Contenuto del box presa.**
Ogni box mostra: l'immagine di anteprima (PREV_), un bottone "Utilizza" e un bottone "Dettagli".
- Criteri: entrambi i bottoni sono sempre visibili e attivi per ogni presa non ancora in uso.

**REQ-CAT-003 — Caricamento e cache del catalogo.**
All'avvio, l'elenco delle prese e le relative anteprime (dati leggeri) sono caricati e mantenuti in cache lato frontend per l'intera sessione.
- Criteri: il manifest e le anteprime vengono richiesti al backend una sola volta per sessione, anche se l'utente naviga ripetutamente il catalogo.

**REQ-CAT-004 — Caricamento on-demand del modello 3D completo.**
Il modello GLB completo di una presa è caricato solo quando effettivamente necessario: al click su "Utilizza" o all'apertura del pannello "Dettagli".
- Criteri: nessuna richiesta del file GLB completo viene effettuata per una presa non ancora selezionata dall'utente.

**REQ-CAT-005 — Pannello Dettagli.**
Al click su "Dettagli" si apre un pannello modale con il modello 3D della presa caricato al momento. Alla chiusura del pannello, il modello viene rilasciato dalla memoria/contesto grafico.
- Criteri: dopo la chiusura del pannello, il modello non risulta più referenziato in memoria; aprendo e chiudendo il pannello ripetutamente per prese diverse non si verifica un incremento progressivo del numero di contesti WebGL attivi.

**REQ-CAT-006 — Transizione catalogo ↔ scena.**
Quando una presa viene aggiunta alla scena (bottone "Utilizza"), scompare dal catalogo. Quando viene rimossa dalla scena, ritorna nel catalogo.
- Criteri: in ogni istante, l'insieme "prese nel catalogo" e "prese in scena" sono disgiunti e la loro unione corrisponde al totale delle prese disponibili.

**REQ-CAT-007 — Unicità d'uso.**
Ogni presa può essere aggiunta alla scena una sola volta alla volta.
- Criteri: il bottone "Utilizza" non è disponibile per una presa già presente in scena.

---

## SCN — Scena e istanze

**REQ-SCN-001 — Separazione modello/istanza.**
I modelli del catalogo sono distinti dalle istanze posizionate in scena. Un'istanza mantiene un riferimento al modello originale e memorizza solo il proprio stato (posizione, orientamento, altri attributi di posizionamento).
- Criteri: l'architettura dati consente, in linea di principio, più istanze dello stesso modello anche se la versione corrente ne vincola l'uso a una sola (REQ-CAT-007).

**REQ-SCN-002 — Selezione istanza.**
L'utente seleziona un'istanza in scena tramite click del mouse, per poi spostarla o rimuoverla.
- Criteri: cliccando su una presa in scena, questa risulta visivamente riconoscibile come selezionata e i comandi di movimento/rotazione/rimozione agiscono su di essa.

**REQ-SCN-003 — Rimozione istanza.**
L'utente rimuove un'istanza selezionata tramite un apposito bottone, che la riporta nel catalogo (v. REQ-CAT-006).
- Criteri: dopo la rimozione, l'istanza non è più presente in scena e la relativa voce ricompare nel catalogo.

**REQ-SCN-004 — Caricamento della parete.**
La parete viene caricata automaticamente all'avvio dell'applicazione dal modello disponibile nella cartella "Pareti". La versione corrente gestisce una sola parete per sessione.
- Criteri: all'apertura dell'applicazione, la parete è visibile senza alcuna azione dell'utente.

---

## FIS — Fisica e movimento

**REQ-FIS-001 — Sistema di coordinate e unità.**
Il sistema usa coordinate cartesiane destrorse coerenti con three.js; tutti i modelli sono espressi in metri (1 unità three.js = 1 metro).
- Criteri: le distanze configurate nei requisiti (es. 5 cm di snap, 1 cm di incremento movimento) risultano coerenti tra modelli di scala diversa.

**REQ-FIS-002 — Regole fisiche di base.**
Gravità disabilitata; prese come corpi rigidi statici durante il posizionamento; collision detection continua (CCD); nessuna simulazione dinamica, inerzia, rimbalzo o attrito; solo rilevazione delle collisioni.
- Criteri: una presa lasciata "libera" non cade né si muove autonomamente; un movimento rapido del mouse non causa attraversamento di collider (tunneling).

**REQ-FIS-003 — Blocco della compenetrazione con scorrimento libero (move-and-slide).**
Quando il movimento desiderato genera una collisione, il sistema calcola la massima posizione consentita nella direzione di movimento senza compenetrazione, mantenendo libere le componenti di movimento che non generano collisione.
- Criteri: spostando una presa verso un'altra già posizionata con movimento non perpendicolare al contatto, la presa continua a scorrere lungo la componente libera invece di fermarsi bruscamente.

**REQ-FIS-004 — Uso del KinematicCharacterController di Rapier.**
Il comportamento di REQ-FIS-003 è implementato tramite il `KinematicCharacterController` nativo di Rapier, configurato escludendo funzionalità pensate per personaggi soggetti a gravità (autostep, snap-to-ground). Non deve essere scritta una collision-response custom, salvo insufficienza documentata delle funzionalità native.
- Criteri: il codice sorgente frontend referenzia esplicitamente l'API del controller nativo di Rapier per il calcolo del movimento vincolato; autostep e snap-to-ground risultano disattivati in configurazione.

**REQ-FIS-005 — Origine locale delle prese.**
Ogni modello di presa ha l'origine del proprio sistema di coordinate locale nel centro geometrico della superficie posteriore (punto di contatto con la parete).
- Criteri: applicando una rotazione attorno alla normale, la presa ruota attorno al proprio punto di contatto, non attorno al proprio baricentro visivo.

**REQ-FIS-006 — Snap automatico alla parete.**
Quando una presa viene avvicinata alla parete entro 5 cm, si aggancia automaticamente al punto di contatto più vicino, orientandosi secondo la normale della superficie in quel punto, con la base perfettamente aderente.
- Criteri: avvicinando una presa alla parete a una distanza ≤ 5 cm, questa scatta automaticamente in aderenza; a distanza > 5 cm nessuno snap si verifica.

**REQ-FIS-007 — Movimento post-aggancio.**
Dopo l'aggancio, l'utente può modificare liberamente la rotazione attorno alla normale e la posizione lungo la superficie della parete (vincolata dalle collisioni, v. REQ-FIS-003).
- Criteri: una presa agganciata non può essere staccata dalla parete tramite i normali comandi di movimento; resta sempre aderente durante lo spostamento.

**REQ-FIS-008 — Inclinazione non modificabile manualmente.**
L'inclinazione della presa rispetto alla superficie non è controllabile dall'utente: è determinata automaticamente dalla normale nel punto di contatto.
- Criteri: nessun controllo dell'interfaccia consente di modificare l'inclinazione della presa indipendentemente dalla normale della parete.

**REQ-FIS-009 — Controlli di rotazione.**
Due pulsanti (orario/antiorario) ruotano la presa selezionata attorno alla normale, con incremento di 1° per click e rotazione continua a pulsante premuto. Comandi equivalenti disponibili da tastiera.
- Criteri: un singolo click ruota la presa esattamente di 1°; tenendo premuto il pulsante, la rotazione prosegue in modo continuo fino al rilascio.

**REQ-FIS-010 — Controlli di traslazione.**
Quattro pulsanti direzionali (su/giù/sinistra/destra) muovono la presa selezionata lungo gli assi tangenti alla superficie della parete nel punto di contatto corrente, con incremento di 1 cm per click e movimento continuo a pulsante premuto. Le direzioni "su/giù" e "sinistra/destra" sono calcolate proiettando gli assi verticale e orizzontale della vista corrente sul piano tangente alla parete, così che il movimento risulti sempre intuitivo rispetto a ciò che l'utente vede a schermo. Comandi equivalenti disponibili da tastiera.
- Criteri: un singolo click sposta la presa esattamente di 1 cm nella direzione attesa rispetto alla vista corrente; ruotando la camera attorno alla parete, "su" continua a corrispondere visivamente all'alto dello schermo.

**REQ-FIS-011 — Separazione mesh/collider.**
Ogni oggetto in scena ha una mesh grafica (rendering three.js) e un collider fisico (Rapier) come entità indipendenti; la mesh non è mai usata per il calcolo delle collisioni.
- Criteri: è possibile ispezionare/debuggare il collider di un oggetto indipendentemente dalla sua rappresentazione grafica.

**REQ-FIS-012 — Collider della parete.**
La parete usa un collider TriMesh derivato direttamente dalla geometria del modello 3D così com'è, generato lato client al caricamento della parete, usato esclusivamente come corpo statico.
- Criteri: la forma di collisione della parete corrisponde fedelmente alla sua geometria visibile, incluse eventuali irregolarità della superficie.

---

## HUL — Generazione dei Convex Hull (backend)

**REQ-HUL-001 — Generazione del collider dal GLB.**
Per ogni modello di presa, il backend genera un collider Convex Hull a partire dai vertici della mesh GLB, tramite una libreria di calcolo geometrico .NET indipendente da Rapier (es. MIConvexHull).
- Criteri: dato un GLB di test con geometria nota, l'hull generato ha come vertici un sottoinsieme dei vertici originali che ne costituisce l'inviluppo convesso.

**REQ-HUL-002 — Salvataggio del collider.**
Il risultato (vertici e, se disponibili, indici delle facce) è salvato in un file dedicato nella cartella della presa.
- Criteri: dopo la generazione, il file del collider è leggibile e referenzia correttamente il modello di origine.

**REQ-HUL-003 — Invalidazione basata sul GLB sorgente.**
Il file del collider include un hash o timestamp del GLB sorgente usato per la generazione. Se il GLB risulta modificato rispetto all'ultima generazione, il collider viene rigenerato automaticamente; altrimenti viene riutilizzato.
- Criteri: sostituendo il GLB di una presa esistente con una geometria diversa, il collider viene rigenerato al successivo avvio; lasciando il GLB invariato, il collider non viene ricalcolato.

**REQ-HUL-004 — Generazione non bloccante.**
La generazione dei collider mancanti all'avvio è asincrona rispetto alla disponibilità del backend; le prese diventano disponibili nel catalogo man mano che il rispettivo collider è pronto.
- Criteri: con un numero elevato di prese senza collider precalcolato, l'applicazione risulta comunque utilizzabile (anche se con catalogo parziale) prima che tutti i collider siano generati.

**REQ-HUL-005 — Consumo del collider precalcolato dal frontend.**
Il frontend richiede e utilizza il collider così come fornito dal backend, senza calcolare l'hull lato client.
- Criteri: il codice frontend non contiene logica di calcolo dell'inviluppo convesso; il collider Rapier di una presa è costruito direttamente dai dati ricevuti dal backend.

---

## UI — Struttura dell'interfaccia

**REQ-UI-001 — Layout generale.**
Pannello "Catalogo prese" a sinistra; tutto lo spazio restante mostra il modello 3D della parete.
- Criteri: il viewport 3D occupa lo spazio non coperto dal catalogo, senza sovrapposizioni.

**REQ-UI-002 — Menu superiore.**
Un menu nella parte superiore contiene i bottoni "Genera immagine" e "Rimuovi presa".
- Criteri: entrambi i bottoni sono sempre visibili; "Rimuovi presa" è attivo solo quando un'istanza è selezionata.

**REQ-UI-003 — Navigazione camera.**
La navigazione della scena usa `OrbitControls` di three.js: rotazione libera, zoom e pan, con la parete come punto centrale di osservazione.
- Criteri: l'utente può orbitare, zoomare e pannare senza mai perdere la parete dal campo visivo di riferimento.

---

## IMG — Generazione dell'immagine guida

**REQ-IMG-001 — Vista dedicata alla stampa.**
Al click su "Genera immagine" viene generata una vista ortografica frontale dedicata.
- Criteri: la vista generata usa una proiezione ortografica (non prospettica) inquadrando frontalmente la parete.

**REQ-IMG-002 — Presentazione pulita.**
L'interfaccia utente viene nascosta e la parete è renderizzata su sfondo bianco, mantenendo le proporzioni corrette.
- Criteri: l'immagine risultante non contiene elementi di UI (pannelli, bottoni, cursori) e lo sfondo è uniformemente bianco.

**REQ-IMG-003 — Esportazione.**
L'immagine è esportata ad alta risoluzione in formato JPG.
- Criteri: il file scaricato è un JPG leggibile a piena risoluzione, con dettaglio sufficiente a distinguere chiaramente ogni presa.

---

## PRF — Prestazioni

**REQ-PRF-001 — Hardware target.**
L'applicazione funziona su PC desktop e notebook domestici con GPU integrata compatibile WebGL 2.0, senza richiedere GPU dedicate.
- Criteri: la suite di verifica prestazionale viene eseguita (o simulata) su un profilo hardware di fascia integrata.

**REQ-PRF-002 — Numero di prese gestite.**
Gestione contemporanea di almeno 40 prese 3D oltre alla parete.
- Criteri: con 40 istanze in scena, l'applicazione resta pienamente interattiva.

**REQ-PRF-003 — Reattività percepita.**
Le operazioni di selezione, spostamento e rotazione hanno un tempo di risposta percepito come immediato.
- Criteri: nessun ritardo visibile tra input dell'utente e aggiornamento visivo della presa selezionata.

**REQ-PRF-004 — Framerate.**
Frequenza di rendering di almeno 30 FPS durante l'interazione, con target 60 FPS su hardware più performante.
- Criteri: misurando il framerate durante l'interazione con 40 prese in scena, il valore non scende sotto i 30 FPS su hardware di riferimento.

**REQ-PRF-005 — Nessun blocco dell'interfaccia.**
Nessun blocco della UI durante il caricamento o il posizionamento delle prese.
- Criteri: durante il caricamento di un modello o la generazione di un collider, l'interfaccia resta responsiva ad altri input.

---

## ERR — Gestione degli errori

**REQ-ERR-001 — Gestione centralizzata.**
Gestione centralizzata degli errori sia lato backend che lato frontend; tutti gli errori intercettabili sono registrati nel log server-side.
- Criteri: non esistono percorsi di codice che generano eccezioni non intercettate senza che venga prodotto un log.

**REQ-ERR-002 — Gestione delle eccezioni backend.**
Meccanismo centralizzato di gestione delle eccezioni; nessuna eccezione non gestita viene restituita direttamente al client; nessun dettaglio tecnico interno viene esposto.
- Criteri: forzando un'eccezione non gestita in un endpoint di test, la risposta HTTP non contiene stack trace né dettagli implementativi.

**REQ-ERR-003 — Tracciabilità dell'errore.**
Per ogni errore: eccezione intercettata, ID univoco generato, dettaglio completo registrato nel log server-side, risposta HTTP coerente con la natura dell'errore, messaggio utente comprensibile e privo di informazioni tecniche/sensibili. L'ID univoco è restituito al frontend quando appropriato.
- Criteri: dato un errore riprodotto in test, l'ID mostrato all'utente corrisponde esattamente a una voce nel log server-side.

**REQ-ERR-004 — Gestione degli errori frontend.**
Gli errori frontend sono intercettati e, quando tecnicamente possibile, notificati al backend per la registrazione. Devono essere gestiti almeno: errori JS non gestiti, errori nelle chiamate REST, errori HTTP dal backend, errori di caricamento/parsing dei modelli 3D, errori relativi a three.js, errori relativi a Rapier, errori nella generazione dell'immagine, errori nelle funzionalità principali dell'interfaccia.
- Criteri: ciascuna delle categorie elencate ha un punto di intercettazione dedicato nel codice, verificabile tramite test o ispezione.

**REQ-ERR-005 — Continuità applicativa.**
Il frontend evita di interrompere l'intera applicazione per un errore relativo a una singola operazione, quando sia possibile continuare in uno stato coerente.
- Criteri: un errore nel caricamento di una singola presa non impedisce l'uso del resto dell'applicazione (catalogo, altre prese già posizionate, navigazione camera).

---

## LOG — Logging

**REQ-LOG-001 — Logging centralizzato e non bloccante.**
Logging centralizzato lato server, asincrono, senza I/O sincrono durante le normali operazioni, senza blocco del thread principale del backend.
- Criteri: sotto carico di eventi elevato, il tempo di risposta delle API non degrada in modo significativo a causa del logging.

**REQ-LOG-002 — Nessun log persistente lato client.**
Il frontend non mantiene log persistenti locali né scrive su file di log del client; comunica al backend solo gli eventi necessari al troubleshooting non rilevabili direttamente dal server.
- Criteri: non esiste alcun meccanismo di scrittura file o storage persistente lato browser dedicato al logging.

**REQ-LOG-003 — Sanitizzazione.**
Nessun dato sensibile (credenziali, token, dati personali, informazioni riservate) compare nei log; ogni evento è sanitizzato prima della registrazione.
- Criteri: ispezionando un campione di log prodotti durante i test, non è presente alcun dato sensibile.

**REQ-LOG-004 — Formato strutturato.**
Logging strutturato, preferibilmente JSON. Ogni evento contiene, quando disponibili: timestamp, livello di severità secondo i `LogLevel` standard di .NET (Trace, Debug, Information, Warning, Error, Critical), categoria, messaggio, ID della richiesta HTTP associata, ID dell'errore se applicabile, componente applicativo di origine, informazioni di contesto utili al troubleshooting.
- Criteri: ogni voce di log è un oggetto JSON valido contenente almeno i campi elencati quando pertinenti.

**REQ-LOG-005 — Soglia di severità configurabile.**
Il livello minimo di severità da loggare è configurabile lato server (default `Information`) tramite il meccanismo di filtraggio nativo di .NET (`appsettings.json`).
- Criteri: modificando la configurazione, eventi sotto la soglia impostata non vengono più scritti nel log.

**REQ-LOG-006 — Rotazione e conservazione.**
Un file di log per giornata; i file più vecchi di 7 giorni dalla data di salvataggio del nuovo file vengono eliminati.
- Criteri: dopo l'esecuzione dell'applicazione per più di 7 giorni consecutivi (o in un test che simula il passaggio del tempo), i file più vecchi risultano rimossi.

**REQ-LOG-007 — Nessuna sovraccarico da interazione continua.**
Coerentemente con C3, nessuna chiamata REST per ogni frame o movimento continuo genera eventi di log durante l'interazione, per non introdurre un volume di eventi eccessivo.
- Criteri: spostando una presa in modo continuo per diversi secondi, il volume di eventi di log prodotti resta contenuto e proporzionato (es. un evento all'inizio/fine dell'operazione, non uno per frame).

---

## TST — Test automatici

**REQ-TST-001 — Suite completa in CI.**
Suite di test automatici eseguibile ad ogni build o pipeline di integrazione continua; la build fallisce se un test fallisce.
- Criteri: un fallimento intenzionale introdotto in un test fa fallire la pipeline.

**REQ-TST-002 — Test unitari backend.**
Test xUnit per servizi, controller, logica di caricamento del catalogo, logging.
- Criteri: copertura dei principali percorsi di successo ed errore dei service backend.

**REQ-TST-003 — Test della generazione/invalidazione dei Convex Hull.**
Verificano: generazione corretta di un collider mancante; non rigenerazione di un collider già presente e coerente; rigenerazione automatica a seguito di modifica del GLB sorgente.
- Criteri: i tre scenari sono coperti da altrettanti test automatici distinti.

**REQ-TST-004 — Test di integrazione REST.**
Verificano richieste HTTP e dati restituiti dalle API.
- Criteri: ogni endpoint pubblico ha almeno un test di integrazione che ne verifica lo status code e la forma della risposta.

**REQ-TST-005 — Test end-to-end.**
Test Playwright sulle principali funzionalità dell'interfaccia utente.
- Criteri: i flussi principali (caricamento parete, aggiunta presa dal catalogo, spostamento/rotazione, rimozione, generazione immagine) sono coperti da scenari e2e automatici.

**REQ-TST-006 — Test delle regole fisiche.**
Test headless (Vitest o Jest) con Rapier senza rendering, dato che la fisica è interamente lato frontend. Verificano: una presa non attraversa la parete; due prese non si compenetrano; una presa si posiziona correttamente in assenza di collisioni; la rimozione di una presa libera correttamente lo spazio occupato; il comportamento delle collisioni resta invariato dopo modifiche al codice.
- Criteri: ciascuno dei cinque scenari è coperto da un test automatico dedicato, eseguibile senza browser.

**REQ-TST-007 — Determinismo dei test fisici.**
I test relativi alla fisica sono deterministici, indipendenti dal frame rate o dalla temporizzazione reale del browser.
- Criteri: eseguendo lo stesso test fisico più volte consecutive, il risultato è sempre identico.

---

## DOC — Documentazione

**REQ-DOC-001 — Documentazione inline.**
Codice backend e frontend interamente documentato nei metodi e nelle classi, in lingua italiana.
- Criteri: ogni classe e metodo pubblico ha un commento di documentazione in italiano.

**REQ-DOC-002 — Documento markdown generale.**
Documento markdown che spiega funzionamento, logica e struttura software dell'intera applicazione, incluse le istruzioni di avvio ed esecuzione (live e debug).
- Criteri: seguendo le istruzioni del documento, è possibile avviare l'applicazione da zero sia in modalità debug che live.

**REQ-DOC-003 — Diagrammi.**
Diagramma architetturale, diagramma delle cartelle dei sorgenti, diagramma delle API, diagramma del ciclo di vita delle prese, diagramma del flusso della UI, diagramma della gestione dei compiti tra backend e frontend.
- Criteri: tutti e sei i diagrammi sono presenti e coerenti con lo stato effettivo del codice.

**REQ-DOC-004 — Documentazione dei test.**
File di documentazione che spiega i test automatici implementati, i loro scopi, i risultati attesi e i framework/strumenti utilizzati.
- Criteri: ogni categoria di test del dominio TST è descritta nel documento.

**REQ-DOC-005 — Collocazione della documentazione.**
Tutta la documentazione, esclusa quella inline nel codice, è salvata in un'apposita cartella dedicata.
- Criteri: la cartella di documentazione contiene il documento generale, i diagrammi e il documento sui test.
