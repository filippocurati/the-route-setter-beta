# 03 — Architecture Specification

## 1. Architettura logica

```text
Browser
├── UI
├── Scene/Application State
├── Three.js Renderer
├── OrbitControls
├── Asset Loader
├── Rapier WASM
│   ├── Wall TriMesh
│   └── Hold Convex Hull colliders
└── REST Client
        │
        │ JSON / REST
        ▼
ASP.NET Core Web API
├── Controllers
├── Services
├── Catalog / Asset discovery
├── Convex Hull generation
├── Error handling
└── Structured logging
        │
        └── Static asset folders
            ├── Pareti
            └── Prese
```

## 2. Frontend responsibilities

Il frontend è responsabile di:

- rendering;
- UI;
- stato della sessione;
- caricamento dei modelli;
- cache catalogo e anteprime;
- creazione del collider parete;
- utilizzo dei collider hold forniti dal backend;
- integrazione Rapier;
- collisioni;
- snap;
- orientamento;
- movimento;
- selezione;
- generazione dell'immagine;
- invio degli eventi di troubleshooting al backend.

## 3. Backend responsibilities

Il backend è responsabile di:

- esposizione delle risorse statiche;
- analisi delle cartelle delle prese;
- costruzione del manifest/catalogo;
- gestione dei collider Convex Hull pre-calcolati;
- generazione/invalidation dei Convex Hull;
- API REST;
- OpenAPI/Swagger;
- gestione errori;
- logging centralizzato;
- ricezione degli eventi frontend necessari al troubleshooting.

Il backend non è responsabile della simulazione fisica interattiva.

## 4. Asset pipeline

```text
GLB Hold
   │
   ├── frontend → mesh Three.js
   │
   └── backend → Convex Hull
                    │
                    ▼
                 file hull
                    │
                    ▼
             frontend → Rapier
```

## 5. Stato applicativo

Lo stato deve distinguere almeno:

- catalogo;
- modelli disponibili;
- anteprime;
- istanze in scena;
- selezione corrente;
- stato di caricamento;
- stato del collider;
- stato della scena fisica.

La struttura concreta delle classi/interfacce è una decisione implementativa, purché mantenga queste separazioni.

## 6. Separazione modello/istanza

Un modello di presa rappresenta una risorsa disponibile.

Una `HoldInstance` rappresenta una specifica presenza nella scena e deve mantenere almeno:
- riferimento al modello;
- posizione;
- orientamento;
- stato di posizionamento;
- riferimento/identità del collider utilizzato;
- altri dati necessari al funzionamento.

La struttura deve consentire in futuro più istanze dello stesso modello, anche se la versione corrente ne consente una sola.

## 7. Flusso di caricamento

### Startup

1. inizializzazione backend;
2. scansione asset;
3. avvio generazione collider mancanti;
4. esposizione del catalogo disponibile;
5. inizializzazione frontend;
6. caricamento manifest catalogo;
7. cache anteprime;
8. caricamento parete;
9. generazione TriMesh parete lato client;
10. disponibilità della scena.

### Utilizzo hold

```text
Catalog item
   ↓ Utilizza
Load GLB + assets presenti
   ↓
Load precomputed hull
   ↓
Create HoldInstance
   ↓
Create Three.js representation
   ↓
Create Rapier representation
   ↓
Scene
```

## 8. Modello di comunicazione

Le operazioni interattive ad alta frequenza restano locali al browser.

Le REST API sono utilizzate per:
- manifest/catalogo;
- risorse statiche;
- collider pre-calcolati;
- logging/troubleshooting;
- altre operazioni necessarie non interattive.

Non devono essere usate per aggiornare la posizione ad ogni frame.

## 9. Layering

### Backend

```text
API / Controller
       ↓
Application Services
       ↓
Asset / Geometry Services
       ↓
File System
```

### Frontend

```text
UI
 ↓
Application / Scene State
 ↓
Interaction / Placement
 ├── Three.js
 └── Rapier
 ↓
REST Client
```

## 10. Regola di dipendenza

Il codice frontend può dipendere da Three.js e Rapier.

Il backend può dipendere da librerie .NET per parsing GLB/calcolo geometrico.

Il backend non deve dipendere da Rapier.

Il modulo di generazione Convex Hull deve essere indipendente dal motore fisico client.
