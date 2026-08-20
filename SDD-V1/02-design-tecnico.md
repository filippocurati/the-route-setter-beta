# Design tecnico
## Applicazione per tracciatura vie climbing indoor

Questo documento descrive **come** vengono realizzati i requisiti elencati in `01-specifica-requisiti.md` (referenziati tra parentesi quadre, es. `[REQ-ARC-004]`). Dove la specifica originale lasciava un margine di scelta implementativa, questo documento propone una decisione esplicita, etichettata come **Assunzione**: è una proposta ragionevole, non un vincolo imposto dall'utente — può essere modificata prima di iniziare l'implementazione senza impatto sugli altri documenti.

---

## 1. Vista d'insieme dell'architettura

```
┌─────────────────────────────┐         ┌──────────────────────────────┐
│           BROWSER            │  REST   │            BACKEND            │
│  ┌─────────────────────────┐ │  JSON   │  ASP.NET Core Web API         │
│  │  three.js (rendering)   │ │◄───────►│  Controller / Service         │
│  │  Rapier WASM (fisica)   │ │         │  - Manifest catalogo prese    │
│  │  UI (catalogo, comandi) │ │         │  - File statici (GLB/PREV_)   │
│  └─────────────────────────┘ │         │  - Generazione Convex Hull    │
│  Nessuna persistenza locale  │         │  - Logging centralizzato      │
└───────────────────────────────┘         │  Nessuna persistenza          │
                                            └──────────────────────────────┘
```

Principio cardine [C1, C2, C3]: il backend non renderizza, non simula fisica interattiva e non riceve chiamate per ogni frame. Il suo ruolo è: servire dati e file statici, precalcolare i collider delle prese (operazione di sola geometria, non fisica interattiva), gestire logging ed errori centralizzati.

---

## 2. Struttura delle cartelle di progetto

**Assunzione tecnica**: la specifica originale non indica il framework/linguaggio del frontend oltre a three.js e Rapier. Si propone **TypeScript** con build tool **Vite**, senza framework UI reattivo (React/Vue), dato che l'interfaccia descritta (catalogo, pannelli, bottoni, modale) è composta da elementi relativamente statici gestibili con DOM/event handler diretti — questo minimizza le dipendenze e la superficie di apprendimento, mantenendo comunque una base tipizzata per la complessità di three.js + Rapier + gestione dello stato delle istanze. Se si preferisce un framework reattivo, la struttura sottostante resta valida a meno della sostituzione dello strato UI.

```
/backend
  /src
    /Controllers        → endpoint REST (thin, delegano ai Service)
    /Services
      /Catalog           → lettura cartelle "Prese", costruzione manifest
      /Wall               → lettura cartella "Pareti"
      /ConvexHull         → generazione/cache/invalidazione collider [HUL]
      /Logging            → wrapper logging strutturato [LOG]
    /Middleware
      /ExceptionHandling  → gestione centralizzata eccezioni [ERR-002, ERR-003]
    /Models              → DTO di scambio con il frontend
  /Data
    /Pareti               → modello GLB della parete
    /Prese
      /Hold_1
        model.glb
        PREV_thumb.png
        collider.json     → generato dal backend [HUL-002]
      /Hold_2
        ...
  /tests
    /UnitTests            → xUnit [TST-002, TST-003]
    /IntegrationTests      → xUnit + WebApplicationFactory [TST-004]

/frontend
  /src
    /scene                → bootstrap three.js, camera, OrbitControls
    /physics               → inizializzazione Rapier, KinematicCharacterController
    /holds                 → logica di istanza presa (snap, movimento, rotazione)
    /catalog                → pannello catalogo, box presa, pannello Dettagli
    /api                   → client REST tipizzato verso il backend
    /logging                → invio eventi al backend [LOG-002]
    /errors                 → gestione centralizzata errori frontend [ERR-004]
    /export                 → generazione immagine guida [IMG]
  /tests
    /physics                → Vitest/Jest + Rapier headless [TST-006, TST-007]
    /e2e                    → Playwright [TST-005]

/docs
  architettura.md
  cartelle-sorgenti.md
  api.md
  ciclo-vita-prese.md
  flusso-ui.md
  gestione-compiti-backend-frontend.md
  test-automatici.md
  README.md               → avvio live e debug [DOC-002]
```

---

## 3. Contratti API (REST)

**Assunzione tecnica**: la specifica non elenca gli endpoint minimi; sono derivati dai requisiti funzionali. I nomi/percorsi sono indicativi e possono essere adattati in fase di implementazione mantenendo le responsabilità descritte.

| Metodo | Percorso | Scopo | Requisiti coperti |
|---|---|---|---|
| `GET` | `/api/wall` | Restituisce metadati della parete (percorso del modello GLB) | `SCN-004` |
| `GET` | `/api/holds` | Restituisce il manifest del catalogo: per ogni presa, ID, nome, percorso GLB, percorso anteprima PREV_, percorso collider | `CAT-003`, `MOD-004` |
| `GET` | `/api/holds/{id}/model` | Serve il file GLB completo di una presa (richiesto on-demand) | `CAT-004` |
| `GET` | `/api/holds/{id}/collider` | Restituisce il collider Convex Hull precalcolato (vertici + facce) | `HUL-005` |
| `POST` | `/api/logs` | Riceve un evento applicativo dal frontend da registrare nel log server-side | `LOG-002`, `ERR-004` |

Nota: i file GLB/texture/anteprime possono anche essere serviti tramite file statici standard di ASP.NET Core (`app.UseStaticFiles()`) su un percorso dedicato, invece che tramite controller espliciti — scelta implementativa libera, purché il manifest restituito da `GET /api/holds` contenga URL coerenti e risolvibili dal frontend.

### Forma indicativa del manifest (`GET /api/holds`)

```json
[
  {
    "id": "Hold_1",
    "previewUrl": "/static/prese/Hold_1/PREV_thumb.png",
    "modelUrl": "/api/holds/Hold_1/model",
    "colliderUrl": "/api/holds/Hold_1/collider"
  }
]
```

### Forma indicativa del collider (`GET /api/holds/{id}/collider`)

```json
{
  "sourceHash": "sha256:...",
  "vertices": [0.01, 0.02, 0.00, ...],
  "indices": [0, 1, 2, ...]
}
```

`sourceHash` è l'hash del GLB sorgente al momento della generazione, usato per l'invalidazione `[HUL-003]`.

---

## 4. Generazione dei Convex Hull [HUL]

Flusso all'avvio del backend:

1. Scansione di tutte le sottocartelle di "Prese".
2. Per ciascuna, verifica se esiste già `collider.json` e se il campo `sourceHash` corrisponde all'hash corrente del GLB.
3. Se manca o non corrisponde: lettura dei vertici della mesh dal GLB (es. tramite `SharpGLTF`), calcolo dell'inviluppo convesso tramite una libreria di geometria .NET indipendente da Rapier (es. `MIConvexHull`), salvataggio di `collider.json` con l'hash aggiornato `[HUL-001, HUL-002, HUL-003]`.
4. L'intero processo gira su un task in background, non blocca l'avvio del server; il manifest del catalogo espone una presa come disponibile solo quando il relativo collider è pronto `[HUL-004]`.

Questo passaggio è **calcolo geometrico statico**, non simulazione fisica: non richiede l'esecuzione di Rapier né del motore fisico, e per questo è coerente con il vincolo che vieta simulazione fisica lato backend `[C2, C11]`.

Il collider della parete segue invece un percorso diverso: è un TriMesh costruito lato client, direttamente dalla geometria del modello GLB della parete, al momento del suo caricamento — nessun precalcolo lato server, poiché non richiede riduzione a inviluppo convesso `[FIS-012]`.

---

## 5. Fisica e interazione [FIS]

### 5.1 Inizializzazione

Il mondo fisico Rapier viene creato lato client con gravità nulla `[FIS-002]`. La parete riceve un collider statico TriMesh; ogni istanza di presa riceve un collider Convex Hull costruito dai dati ottenuti da `GET /api/holds/{id}/collider` `[HUL-005]`.

### 5.2 Movimento vincolato (move-and-slide)

Il movimento di una presa selezionata (da trascinamento implicito nello snap iniziale, o dai comandi di traslazione/rotazione) è calcolato tramite il `KinematicCharacterController` di Rapier `[FIS-004]`:

- si definisce lo spostamento desiderato (delta di posizione sul piano tangente alla parete);
- si invoca il metodo di calcolo del movimento vincolato del controller, che restituisce lo spostamento effettivo consentito senza compenetrazione;
- si applica lo spostamento effettivo alla posizione cinematica della presa.

Vanno esclusi in configurazione i comportamenti pensati per personaggi soggetti a gravità (autostep, snap-to-ground, gestione di pendii) poiché non pertinenti in un mondo senza gravità `[FIS-004]`. Si raccomanda di verificare la sintassi esatta dell'API sulla documentazione ufficiale aggiornata di Rapier.js, poiché può variare tra versioni.

### 5.3 Snap alla parete

Al posizionamento iniziale di una presa (dopo il click su "Utilizza", prima che sia agganciata), il sistema effettua un raycast/query di prossimità continua contro il collider della parete; quando la distanza minima scende sotto 5 cm, la presa viene agganciata al punto di contatto più vicino, orientata secondo la normale locale della parete in quel punto `[FIS-006]`.

### 5.4 Traslazione guidata dalla vista

Per calcolare le direzioni "su/giù/sinistra/destra" dei pulsanti di traslazione `[FIS-010]`: si proiettano i vettori "up" e "right" della camera corrente sul piano tangente alla parete nel punto di contatto della presa selezionata (sottraendo la componente lungo la normale), normalizzando il risultato. Questo garantisce che il movimento risulti intuitivo indipendentemente dall'angolazione della camera.

---

## 6. Catalogo e ciclo di vita delle istanze [CAT, SCN]

```
Catalogo (modello disponibile)
        │  click "Utilizza"
        ▼
Caricamento on-demand del GLB completo [CAT-004]
        │
        ▼
Istanza creata in scena, in stato "non agganciata"
        │  utente posiziona vicino alla parete
        ▼
Snap automatico entro 5cm [FIS-006] → istanza "agganciata"
        │  utente muove/ruota tramite bottoni o tastiera
        ▼
Istanza agganciata, posizionabile lungo la parete [FIS-007]
        │  selezione + bottone "Rimuovi presa"
        ▼
Istanza rimossa dalla scena → modello ritorna nel catalogo [CAT-006]
```

Un'istanza è rappresentata a runtime (lato frontend, in memoria, nessuna persistenza `[C4]`) come un oggetto leggero: riferimento al modello di catalogo, posizione, orientamento (quaternion), stato di aggancio. Il modello del catalogo (mesh, collider) resta separato e viene caricato una sola volta anche se, nella versione corrente, ogni modello genera al più un'istanza `[SCN-001, CAT-007]`.

---

## 7. Gestione errori e logging [ERR, LOG]

### 7.1 Backend

Middleware di gestione eccezioni globale (`IExceptionHandler` o middleware custom) che intercetta ogni eccezione non gestita, genera un ID univoco (es. GUID), registra il dettaglio completo nel log strutturato, e restituisce al client una risposta JSON generica con lo stesso ID, senza stack trace né dettagli implementativi `[ERR-002, ERR-003]`.

### 7.2 Frontend

Un modulo centrale di gestione errori intercetta: eccezioni JS non gestite (`window.onerror`, `unhandledrejection`), errori delle chiamate REST, errori HTTP dal backend, errori di caricamento/parsing GLB, errori specifici di three.js e di Rapier, errori nella generazione dell'immagine, errori nelle funzionalità principali della UI `[ERR-004]`. Ogni errore intercettato, quando possibile, viene inviato a `POST /api/logs`; l'applicazione continua a funzionare per le parti non coinvolte nell'errore `[ERR-005]`.

### 7.3 Logging

Impostato su `Microsoft.Extensions.Logging` (`ILogger<T>`), con i livelli standard `LogLevel` `[LOG-004]`, configurazione della soglia minima via `appsettings.json` (default `Information`) `[LOG-005]`, sink su file con rotazione giornaliera e retention di 7 giorni `[LOG-006]` (es. tramite Serilog con `File` sink configurato a rotazione giornaliera). Ogni evento è scritto in formato JSON strutturato, con sanitizzazione dei campi di contesto prima della scrittura per rimuovere eventuali dati sensibili `[LOG-003]`.

---

## 8. Generazione dell'immagine guida [IMG]

Al click su "Genera immagine": creazione temporanea di una camera ortografica frontale, dimensionata sul bounding box della parete; occultamento di tutti gli elementi DOM della UI; impostazione dello sfondo della scena a bianco; render a risoluzione elevata su un canvas offscreen; esportazione del contenuto del canvas in formato JPG tramite `canvas.toBlob('image/jpeg', qualità_alta)` e download del file; ripristino della UI e della camera precedente al termine dell'operazione.

---

## 9. Test automatici — mapping framework/dominio [TST]

| Dominio testato | Framework | Ambiente | Requisiti |
|---|---|---|---|
| Servizi/controller backend | xUnit | .NET | `TST-002` |
| Generazione/invalidazione Convex Hull | xUnit | .NET | `TST-003` |
| API REST | xUnit + `WebApplicationFactory` | .NET (in-memory server) | `TST-004` |
| Fisica (Rapier) | Vitest o Jest | Node, Rapier headless (no rendering) | `TST-006`, `TST-007` |
| Flussi UI end-to-end | Playwright | Browser reale | `TST-005` |

Punto di attenzione: la fisica gira lato client `[C2]`, quindi i suoi test **non** possono essere scritti in xUnit — richiedono un ambiente Node con il binding JS/WASM di Rapier eseguito senza three.js e senza browser, per restare deterministici e veloci `[TST-007]`.

---

## 10. Decisioni tecniche aperte — da confermare prima dell'implementazione

Queste sono le uniche aree dove il documento propone una scelta non esplicitamente richiesta nella specifica originale:

1. **Stack frontend**: TypeScript + Vite, nessun framework UI reattivo (proposta, sezione 2).
2. **Libreria .NET per il Convex Hull**: MIConvexHull o equivalente (proposta, sezione 4).
3. **Libreria di lettura GLB lato backend**: SharpGLTF o equivalente (proposta, sezione 4).
4. **Libreria di logging strutturato**: Serilog con sink su file a rotazione giornaliera, oppure il provider di logging nativo di .NET con un sink custom (proposta, sezione 7.3).
5. **Percorsi/nomi esatti degli endpoint REST**: indicativi, sezione 3.

Se una di queste scelte non è gradita, va corretta qui prima di avviare il piano di implementazione, poiché diversi task del piano vi fanno riferimento diretto.
