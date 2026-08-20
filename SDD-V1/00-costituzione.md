# Costituzione del progetto
## Applicazione per tracciatura vie climbing indoor

Questo documento elenca i principi non negoziabili del progetto. Sono vincoli trasversali che restano validi indipendentemente dal task specifico su cui si sta lavorando in un dato momento del piano di implementazione (`03-piano-implementazione.md`).

**Regola d'uso per l'agente AI**: questo documento va letto per intero prima di iniziare qualsiasi fase del piano, e va tenuto presente durante l'intera implementazione. Se un dettaglio di un singolo task sembrasse in conflitto con uno di questi principi, il principio qui descritto prevale e il task va reinterpretato di conseguenza, segnalando l'eventuale conflitto.

---

**C1 — Rendering esclusivamente client-side.**
Nessun rendering 3D deve avvenire lato server. Tutto il rendering avviene nel browser tramite three.js.

**C2 — Fisica esclusivamente client-side.**
Rapier gira esclusivamente lato client (binding JavaScript/WebAssembly). Il backend non deve avere alcuna dipendenza da Rapier né eseguire simulazione fisica. Unica eccezione: la generazione dei collider Convex Hull delle prese (vedi C11) è calcolo geometrico statico, non simulazione, e per questo è ammessa lato backend.

**C3 — Nessuna chiamata di rete per interazione continua.**
Il frontend non deve effettuare chiamate REST al server per ogni frame, movimento del mouse o aggiornamento continuo della scena. Ogni interazione di trascinamento, rotazione o movimento delle prese deve essere gestita interamente lato client, con feedback percepito come immediato.

**C4 — Nessuna persistenza.**
In questa versione dell'applicazione nessuna tracciatura, posizione di presa o sessione utente viene salvata, storicizzata o recuperata tra sessioni diverse, né lato server né lato client.

**C5 — Nessun dato sensibile nei log.**
Ogni evento di log deve essere sanitizzato prima della registrazione. Credenziali, token o altre informazioni riservate non devono mai comparire nei log, nemmeno nei dati di contesto.

**C6 — Nessuna installazione lato utente.**
L'applicazione deve essere utilizzabile esclusivamente tramite browser (ultime versioni stabili di Chrome, Edge, Firefox, con supporto WebGL 2.0), senza installazione di software o librerie da parte dell'utente finale.

**C7 — Separazione netta mesh/collider.**
Ogni oggetto presente nella scena (parete, prese) deve sempre essere composto da due elementi indipendenti: una mesh grafica per il rendering e un collider fisico dedicato per Rapier. La mesh grafica non deve mai essere usata per il calcolo delle collisioni.

**C8 — Unicità d'uso delle prese.**
Ogni presa del catalogo può essere utilizzata una sola volta contemporaneamente nella scena. Quando è in uso, non è più disponibile nel catalogo.

**C9 — Documentazione inline in italiano.**
Tutta la documentazione all'interno del codice sorgente (commenti su classi e metodi, sia backend che frontend) deve essere scritta in lingua italiana.

**C10 — Nessun dettaglio tecnico esposto all'utente.**
Nessun errore mostrato all'utente finale deve contenere stack trace, percorsi di file, dettagli di implementazione o configurazioni del server. Ogni errore deve però essere associato a un identificativo univoco tracciabile nel log server-side.

**C11 — Generazione dei Convex Hull lato backend.**
I collider Convex Hull delle prese sono precalcolati e cachati lato backend (non generati a runtime lato client), poiché il catalogo delle prese è statico durante una sessione applicativa.

**C12 — Nessuna regressione silenziosa sulle regole fisiche.**
Qualsiasi modifica al codice relativo alla fisica (collisioni, snap, movimento) deve continuare a rispettare il comportamento definito nei requisiti del dominio FIS (`01-specifica-requisiti.md`), verificato dalla relativa suite di test automatici headless.
