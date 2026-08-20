# 02 — Requirements

## 1. Functional Requirements

### FR-001 — Avvio e caricamento parete

All'avvio deve essere caricata automaticamente la parete disponibile nella cartella `Pareti`.

La versione corrente gestisce una sola parete per sessione.

**Acceptance criteria**
- la scena viene inizializzata con la parete disponibile;
- non viene richiesta una selezione manuale della parete nella versione corrente;
- la parete è disponibile prima dell'uso normale della scena.

### FR-002 — Catalogo prese

Il frontend deve mostrare a sinistra il catalogo delle prese disponibili.

Il catalogo deve inizialmente contenere l'elenco delle prese e le relative anteprime.

### FR-003 — Cache catalogo

Elenco e anteprime devono essere mantenuti in cache lato frontend per l'intera sessione.

Il modello 3D completo non deve essere caricato per il solo caricamento del catalogo.

### FR-004 — Utilizzo presa

Premendo `Utilizza`:
- il modello 3D della presa viene caricato;
- viene creata una relativa istanza in scena;
- la presa viene rimossa dal catalogo.

Ogni modello può essere utilizzato una sola volta nella sessione corrente.

### FR-005 — Rimozione presa

Una presa selezionata nella scena può essere rimossa.

Dopo la rimozione:
- l'istanza viene eliminata dalla scena;
- la presa torna disponibile nel catalogo;
- lo spazio precedentemente occupato viene liberato dal sistema fisico.

### FR-006 — Selezione

L'utente deve poter selezionare una presa nella scena tramite click del mouse.

### FR-007 — Rotazione

La presa agganciata alla parete può ruotare liberamente attorno alla normale locale della superficie.

Comandi:
- rotazione oraria;
- rotazione antioraria;
- 1 grado per click;
- ripetizione continua con pulsante mantenuto premuto.

### FR-008 — Traslazione

La presa agganciata alla parete può essere spostata lungo la superficie tramite:
- su;
- giù;
- sinistra;
- destra.

Ogni click equivale a 1 cm.

Il comando mantenuto premuto produce movimento continuo.

### FR-009 — Direzioni relative alla vista

Le direzioni su/giù/sinistra/destra devono essere ottenute proiettando gli assi verticale e orizzontale della vista corrente sul piano tangente alla parete nel punto di contatto.

Il comportamento deve quindi essere intuitivo rispetto alla vista corrente, indipendentemente dall'orientamento della parete o della camera.

### FR-010 — Shortcut

I comandi disponibili tramite pulsanti devono essere disponibili anche tramite shortcut da tastiera.

L'associazione delle shortcut è una decisione di implementazione libera purché:
- sia coerente;
- sia documentata;
- non interferisca con i normali controlli dell'interfaccia.

### FR-011 — Snap alla parete

Quando una presa viene avvicinata alla parete entro 5 cm, deve essere agganciata automaticamente alla superficie usando il punto di contatto più vicino.

### FR-012 — Orientamento allo snap

Al momento dello snap:
- la base della presa deve aderire alla parete;
- la presa deve orientarsi secondo la normale della superficie;
- l'inclinazione rispetto alla superficie non è modificabile manualmente.

La normale deve essere calcolata nel punto esatto di contatto.

### FR-013 — Rotazione post-snap

Dopo lo snap, l'utente può modificare esclusivamente la rotazione attorno alla normale locale.

### FR-014 — Movimento vincolato

Una volta agganciata, la presa può essere traslata esclusivamente lungo la superficie della parete. 

### FR-015 — Collisioni

Una presa non deve poter compenetrare:
- la parete;
- un'altra presa.

Quando una componente del movimento genera compenetrazione, quella componente deve essere bloccata.

Le componenti non in collisione devono rimanere disponibili.

### FR-016 — Movimento cinematico

Il movimento deve utilizzare il `KinematicCharacterController` di Rapier con comportamento move-and-slide.

Non deve essere introdotta una collision-response custom salvo insufficienza comprovata della funzionalità nativa.

### FR-017 — Modelli delle prese

Ogni cartella `HoldN` può contenere:
- GLB;
- texture;
- altri file necessari al modello;
- immagine `PREV_`;
- collider Convex Hull pre-calcolato.

Il caricamento deve utilizzare esclusivamente le risorse effettivamente presenti.

### FR-018 — Dettagli presa

Il pulsante `Dettagli` deve aprire un pannello modale con una visualizzazione 3D della presa.

Il modello deve essere caricato al momento dell'apertura.

Alla chiusura, il modello del pannello deve essere liberato per non mantenere memoria non necessaria.

### FR-019 — Collider hold

Per ogni modello deve esistere un collider Convex Hull pre-calcolato.

Il collider è indipendente dalla mesh grafica.

### FR-020 — Generazione collider backend

All'avvio del backend:
- devono essere analizzate le cartelle in `Prese`;
- i collider mancanti devono essere generati;
- il risultato deve essere salvato nella cartella della presa;
- la generazione deve essere indipendente da Rapier.

### FR-021 — Cache/invalidation collider

Il file del collider deve contenere informazioni sufficienti a verificare la corrispondenza con il GLB sorgente, tramite hash o timestamp.

Se il GLB non è cambiato, il collider deve essere riutilizzato.

Se il GLB è cambiato, deve essere rigenerato.

### FR-022 — Generazione asincrona collider

La generazione dei collider mancanti deve essere asincrona rispetto all'avvio del backend e non deve impedire la disponibilità dell'applicazione.

Le prese devono diventare utilizzabili quando il rispettivo collider è pronto.

### FR-023 — Collider parete

Il collider della parete deve essere un TriMesh derivato dalla geometria della parete.

Può essere generato lato client al caricamento della parete.

Deve essere statico.

### FR-024 — Camera

La navigazione deve usare `OrbitControls`.

Sono richiesti:
- orbit;
- zoom;
- pan;
- parete come punto centrale di osservazione.

### FR-025 — Generazione immagine

Il comando `Genera immagine` deve:
1. creare una vista dedicata;
2. utilizzare una camera ortografica frontale;
3. nascondere la UI;
4. usare sfondo bianco;
5. mantenere le proporzioni della parete;
6. includere la disposizione delle prese;
7. esportare un JPG ad alta risoluzione.

### FR-026 — Nessun salvataggio

La tracciatura non deve essere persistita.

### FR-027 — Error handling

Backend e frontend devono implementare gestione centralizzata degli errori.

### FR-028 — Logging

Il logging deve essere centralizzato lato server, strutturato e asincrono.

### FR-029 — Documentazione

Devono essere prodotti:
- documentazione completa dell'applicazione;
- diagramma architetturale;
- diagramma cartelle sorgenti;
- diagramma API;
- diagramma ciclo di vita prese;
- diagramma flusso UI;
- diagramma responsabilità backend/frontend;
- documentazione dei test automatici.

## 2. Physical Requirements

### PHY-001

Gravità disabilitata.

### PHY-002

Nessuna simulazione dinamica.

### PHY-003

Nessuna inerzia.

### PHY-004

Nessun rimbalzo.

### PHY-005

Nessun attrito.

### PHY-006

Collision detection continua.

### PHY-007

Le prese sono trattate come corpi rigidi statici/cinematici durante il posizionamento, secondo il modello previsto dal KinematicCharacterController.

### PHY-008

Il risultato del movimento non deve consentire compenetrazioni.

### PHY-009

Il movimento deve mantenere le componenti libere quando una sola componente è bloccata dalla collisione.

## 3. Non-Functional Requirements

### NFR-001 — Browser

Chrome, Edge e Firefox nelle ultime versioni stabili con WebGL 2.0.

### NFR-002 — Hardware

Funzionamento su normali desktop/notebook con GPU integrata WebGL 2.0.

### NFR-003 — Capacity

Almeno 40 prese contemporaneamente oltre alla parete.

### NFR-004 — Responsiveness

Selezione, movimento e rotazione percepiti come immediati.

### NFR-005 — Frame rate

Target indicativo minimo: circa 30 FPS durante l'interazione.

### NFR-006 — Caricamento

Nessun blocco dell'interfaccia durante caricamento o posizionamento.

### NFR-007 — Test

Tutti i test devono essere eseguibili automaticamente.

### NFR-008 — Determinismo fisico

I test fisici non devono dipendere da frame rate o temporizzazione reale del browser.

### NFR-009 — Sicurezza log

I log non devono contenere dati sensibili o riservati.

### NFR-010 — Rotazione log

Un file di log per giornata; i file più vecchi di 7 giorni devono essere eliminati alla data di salvataggio del nuovo file.
