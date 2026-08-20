# 01 — Project Constitution

## 1. Obiettivo

L'applicazione assiste i tracciatori di vie di climbing indoor nel posizionamento di prese 3D su una parete 3D.

## 2. Invarianti architetturali

### ARCH-INV-001 — Rendering client-side

Il rendering della scena 3D deve essere eseguito interamente nel browser tramite Three.js.

**MUST:** nessun rendering 3D lato server.

### ARCH-INV-002 — Separazione Three.js / Rapier

Three.js è responsabile esclusivamente del rendering della scena 3D.

Rapier è responsabile lato client di collisioni, contatti e vincoli di movimento.

**MUST:** non utilizzare Three.js per la simulazione fisica.

### ARCH-INV-003 — Fisica esclusivamente client-side

La simulazione fisica interattiva deve essere eseguita interamente nel browser tramite `@dimforge/rapier3d-compat`.

**MUST:** il backend non deve contenere Rapier né logica di simulazione fisica.

La generazione statica dei Convex Hull è esclusa da questo vincolo: è calcolo geometrico lato backend e deve rimanere indipendente da Rapier.

### ARCH-INV-004 — Comunicazione REST

Frontend e backend comunicano tramite REST API e JSON.

Le API devono essere documentate tramite OpenAPI/Swagger e implementate con separazione Controller/Service.

### ARCH-INV-005 — Nessuna chiamata REST durante il movimento continuo

Il frontend non deve effettuare chiamate REST per frame, movimento del mouse o aggiornamenti continui della posizione.

### ARCH-INV-006 — Separazione mesh/collider

Ogni oggetto della scena deve avere:

1. mesh grafica Three.js;
2. collider fisico Rapier.

La mesh grafica non deve essere utilizzata direttamente per il calcolo delle collisioni.

### ARCH-INV-007 — Sistema di unità

- sistema cartesiano destrorso coerente con Three.js;
- metri come unità di misura;
- 1 unità Three.js = 1 metro.

### ARCH-INV-008 — Nessuna persistenza della tracciatura

La posizione delle prese nella sessione non deve essere salvata o storicizzata.

### ARCH-INV-009 — Compatibilità browser

L'applicazione deve funzionare tramite browser, senza installazioni lato utente, sulle ultime versioni stabili di Chrome, Edge e Firefox con WebGL 2.0.

### ARCH-INV-010 — Prestazioni

Il progetto deve essere compatibile con normali PC desktop/notebook dotati di GPU integrata compatibile con WebGL 2.0.

Target minimo dichiarato: almeno 40 prese simultanee oltre alla parete e almeno circa 30 FPS durante l'interazione.

## 3. Principi di implementazione

1. Preferire le funzionalità native delle librerie richieste.
2. Non duplicare nel codice una funzionalità già prevista dal motore fisico quando quella funzionalità soddisfa il requisito.
3. Evitare comunicazioni server non necessarie durante l'interazione.
4. Mantenere separati stato del catalogo e stato delle istanze in scena.
5. Progettare i test contemporaneamente alle funzionalità.
6. Gli errori di una singola risorsa non devono compromettere l'intera applicazione quando è possibile mantenere uno stato coerente.
7. Non sacrificare i requisiti fisici per semplificare la UI.
8. Non introdurre persistenza della tracciatura nella versione corrente.

## 4. Regola per le decisioni non specificate

Quando il documento sorgente non determina una scelta tecnica:

- l'agente può scegliere l'implementazione più semplice e coerente;
- la scelta non deve modificare il comportamento richiesto;
- se la scelta modifica un comportamento osservabile o un vincolo architetturale, deve essere registrata come decisione da confermare.
