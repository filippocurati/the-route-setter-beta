# Implementation Plan — SDD

## Regole generali

L'agente deve procedere in ordine.

Per ogni step:

1. leggere i requisiti SDD coinvolti;
2. implementare esclusivamente lo scope dello step;
3. eseguire tutti i test richiesti;
4. verificare gli Acceptance Criteria;
5. correggere eventuali regressioni;
6. solo dopo considerare lo step completato.

Non implementare funzionalità degli step successivi in anticipo se questo aumenta la complessità o rende difficile verificare il risultato.

---

## STEP-001 — Solution skeleton

### Obiettivo
Creare la struttura base frontend/backend/test.

### Attività
- creare soluzione ASP.NET Core;
- creare progetto frontend;
- predisporre progetto test backend;
- predisporre progetto test frontend/physics;
- predisporre Playwright;
- configurare build;
- predisporre cartelle asset;
- predisporre documentazione.

### Requisiti
ARCH-INV-001..010, NFR-007.

### Verifica
- build backend;
- build frontend;
- esecuzione suite iniziale;
- nessun test configurato deve fallire.

### Exit criteria
La soluzione compila e i test skeleton sono eseguibili automaticamente.

---

## STEP-002 — Backend asset discovery e manifest

### Obiettivo
Implementare scansione di `Pareti` e `Prese` e manifest.

### Attività
- discovery cartelle;
- identificazione GLB;
- identificazione PREV_;
- identificazione asset opzionali;
- DTO manifest;
- service;
- controller;
- OpenAPI;
- static file serving.

### Test
- unit test discovery;
- integration test API.

### Exit criteria
Il frontend può ottenere un manifest completo senza caricare i GLB delle prese.

---

## STEP-003 — Convex Hull generation

### Obiettivo
Implementare generazione server-side dei collider hold.

### Attività
- parser geometrico GLB;
- estrazione vertici;
- Convex Hull;
- serializzazione;
- hash/timestamp;
- cache;
- invalidazione;
- background generation.

### Test
TEST-HULL-001..004.

### Exit criteria
Collider mancanti vengono creati; collider validi vengono riutilizzati; GLB modificato provoca rigenerazione; Rapier non è una dipendenza backend.

---

## STEP-004 — Frontend base e caricamento parete

### Obiettivo
Visualizzare la parete nel browser.

### Attività
- Three.js;
- renderer;
- scene;
- camera;
- OrbitControls;
- caricamento parete;
- TriMesh Rapier.

### Test
- E2E startup;
- caricamento parete;
- camera.

### Exit criteria
La parete è visibile, navigabile e possiede il collider fisico.

---

## STEP-005 — Rapier integration foundation

### Obiettivo
Creare l'infrastruttura Rapier client.

### Attività
- inizializzazione WASM;
- world;
- configurazione senza gravità;
- collider wall;
- lifecycle fisico;
- sincronizzazione con scena.

### Test
- physics smoke tests;
- nessuna dipendenza REST nel loop di simulazione.

### Exit criteria
Rapier funziona headless nei test e nel frontend.

---

## STEP-006 — Hold asset loading

### Obiettivo
Caricare una presa solo quando richiesta.

### Attività
- loader GLB;
- gestione asset opzionali;
- caricamento collider;
- HoldModel;
- HoldInstance;
- separazione mesh/collider.

### Test
- unit/integration asset;
- E2E Utilizza.

### Exit criteria
Una presa utilizzata appare in scena con mesh e collider separati.

---

## STEP-007 — Catalog UI

### Obiettivo
Implementare catalogo e anteprime.

### Attività
- pannello sinistro;
- item;
- PREV_;
- Utilizza;
- Dettagli;
- lazy loading;
- cache catalogo/anteprime.

### Test
E2E catalogo e modale.

### Exit criteria
Il catalogo funziona senza caricare anticipatamente i GLB delle prese.

---

## STEP-008 — Selezione e rimozione

### Obiettivo
Gestire il lifecycle della hold in scena.

### Attività
- raycasting/selezione;
- selected hold;
- Rimuovi presa;
- ritorno al catalogo;
- rimozione collider;
- rilascio risorse.

### Test
E2E selezione/rimozione;
TEST-PHYS-004.

### Exit criteria
Una presa rimossa libera realmente il relativo spazio fisico.

---

## STEP-009 — Snap alla parete

### Obiettivo
Implementare l'aggancio automatico.

### Attività
- soglia 0,05 m;
- punto di contatto;
- normale locale;
- orientamento;
- base aderente.

### Test
TEST-SNAP-001..003.

### Exit criteria
Lo snap è deterministico e conforme ai criteri.

---

## STEP-010 — Kinematic movement

### Obiettivo
Implementare movimento cinematico con collisioni.

### Attività
- KinematicCharacterController;
- collision detection;
- move-and-slide;
- blocco componenti in collisione;
- mantenimento componenti libere;
- nessuna dinamica.

### Test
TEST-PHYS-001..006.

### Exit criteria
Nessuna compenetrazione e comportamento di slide conforme alla specifica.

---

## STEP-011 — Movimento tangenziale e rotazione

### Obiettivo
Implementare i sei comandi di trasformazione.

### Attività
- proiezione assi camera sul piano tangente;
- ±1 cm;
- ±1°;
- rotazione attorno alla normale;
- ripetizione continua.

### Test
input tests + physics tests.

### Exit criteria
I movimenti sono relativi alla vista e coerenti con la normale locale.

---

## STEP-012 — Shortcut

### Obiettivo
Associare shortcut ai comandi.

### Attività
- definizione tasti;
- gestione keydown/keyup;
- prevenzione conflitti;
- documentazione.

### Test
E2E keyboard.

### Exit criteria
Tutti i comandi sono disponibili sia da UI sia da tastiera.

---

## STEP-013 — Error handling

### Obiettivo
Implementare gestione centralizzata errori.

### Attività
- backend exception middleware/handler;
- ErrorId;
- error response;
- frontend error boundary/handlers;
- isolamento errori non bloccanti.

### Test
- backend error tests;
- REST error tests;
- E2E error scenarios.

### Exit criteria
Nessun dettaglio tecnico viene esposto all'utente.

---

## STEP-014 — Logging

### Obiettivo
Implementare logging centralizzato.

### Attività
- structured logging;
- JSON;
- livelli;
- `appsettings.json`;
- sanitization;
- logging asincrono;
- endpoint frontend;
- daily files;
- retention 7 giorni.

### Test
- log structure;
- sanitization;
- rotation;
- logging endpoint;
- failure logging non bloccante.

### Exit criteria
Il logging non blocca il normale funzionamento e non contiene dati sensibili.

---

## STEP-015 — Generazione immagine

### Obiettivo
Implementare tavola JPG per il tracciatore.

### Attività
- camera ortografica frontale;
- UI hidden;
- background white;
- proporzioni corrette;
- alta risoluzione;
- export JPG;
- ripristino scena.

### Test
E2E export e verifica file.

### Exit criteria
La generazione non altera permanentemente la scena interattiva.

---

## STEP-016 — Performance optimization

### Obiettivo
Verificare il target di almeno 40 prese.

### Attività
- profiling;
- verifica memoria;
- verifica rendering;
- caricamento asincrono;
- eliminazione allocazioni inutili;
- ottimizzazione asset se necessaria.

### Test
benchmark ripetibile.

### Exit criteria
Target funzionale raggiunto sul benchmark scelto e documentato.

---

## STEP-017 — Complete automated test suite

### Obiettivo
Chiudere la copertura automatica.

### Attività
- completare xUnit;
- integration;
- Playwright;
- Rapier headless;
- regressione;
- CI fail-on-test-failure.

### Exit criteria
Una suite completa esegue automaticamente senza intervento manuale.

---

## STEP-018 — Documentation

### Obiettivo
Produrre tutta la documentazione richiesta.

### Attività
- documentazione applicazione;
- struttura software;
- avvio live/debug;
- diagramma architettura;
- cartelle;
- API;
- lifecycle prese;
- UI flow;
- responsabilità backend/frontend;
- documentazione test.

### Exit criteria
La documentazione è presente nell'apposita cartella ed è coerente con il codice effettivamente implementato.

---

## STEP-019 — Final verification

### Obiettivo
Verifica finale rispetto alla SDD.

### Attività
- eseguire tutta la suite;
- controllare traceability;
- controllare invarianti architetturali;
- controllare assenza di REST nel loop fisico;
- controllare assenza Rapier backend;
- controllare separazione mesh/collider;
- controllare assenza persistenza tracciatura;
- controllare error handling;
- controllare logging;
- controllare documentazione.

### Exit criteria

Tutti gli Acceptance Criteria risultano soddisfatti.

La pipeline completa è verde.

Nessun requisito MUST della SDD risulta non implementato senza una decisione esplicita e documentata.
