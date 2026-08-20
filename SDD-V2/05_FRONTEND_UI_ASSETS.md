# 05 — Frontend, UI and Asset Specification

## 1. Frontend technology

MUST:
- Three.js;
- `@dimforge/rapier3d-compat`;
- WebGL 2.0;
- browser-only execution;
- nessun rendering server-side.

## 2. Scene

La scena contiene:
- parete;
- prese presenti;
- illuminazione/rendering necessario alla visualizzazione;
- camera;
- controlli.

L'implementazione concreta dell'illuminazione non è determinata dal documento sorgente e non deve alterare la leggibilità della scena.

## 3. Catalog UI

Posizione: lato sinistro.

Ogni elemento contiene:
- anteprima `PREV_`;
- pulsante `Utilizza`;
- pulsante `Dettagli`.

Il catalogo deve distinguere tra:
- metadati/anteprime cached;
- modello 3D completo lazy-loaded.

## 4. Modale Dettagli

Apertura:
- caricare il modello 3D della presa.

Chiusura:
- rimuovere la rappresentazione del modello;
- liberare le risorse non più necessarie, per quanto tecnicamente possibile.

Il modello aperto nel modale non deve diventare automaticamente un'istanza della scena.

## 5. Scena e selezione

La selezione avviene tramite click.

Deve esistere un solo concetto di "presa selezionata" coerente con i comandi:
- movimento;
- rotazione;
- rimozione.

## 6. Comandi UI

Comandi richiesti:

| Comando | Click |
|---|---|
| Rotazione oraria | +1° |
| Rotazione antioraria | -1° |
| Su | +1 cm |
| Giù | -1 cm |
| Sinistra | +1 cm |
| Destra | -1 cm |

La ripetizione continua deve essere disponibile tenendo premuto il pulsante.

## 7. Shortcut

Tutti i comandi devono essere accessibili da tastiera.

L'associazione specifica è lasciata all'implementazione ma deve essere:
- intuitiva;
- documentata;
- non conflittuale con i controlli del browser quando possibile.

## 8. OrbitControls

Devono essere usati `OrbitControls`.

Sono richiesti:
- orbit;
- zoom;
- pan.

Il target deve essere centrato sulla parete.

## 9. Generazione immagine

Procedura concettuale:

```text
User click "Genera immagine"
        ↓
preparazione camera ortografica frontale
        ↓
UI nascosta
        ↓
sfondo bianco
        ↓
render parete + prese
        ↓
export JPG alta risoluzione
        ↓
ripristino vista/UI
```

La procedura non deve modificare permanentemente la scena interattiva.

## 10. Gestione asset

Struttura attesa:

```text
Pareti/
    <wall model>

Prese/
    Hold1/
        <GLB and optional assets>
        PREV_<image>
        <precomputed collider>
    Hold2/
        ...
```

Il sistema non deve assumere la presenza di texture o file opzionali.

Deve caricare solo le risorse effettivamente disponibili.

## 11. Errori asset

Un errore relativo a una singola presa non deve bloccare gli altri modelli quando l'applicazione può continuare coerentemente.

La parete è una risorsa fondamentale: un suo errore può essere bloccante per la scena, ma deve comunque essere gestito tramite il sistema centralizzato di error handling.

## 12. Performance frontend

L'implementazione deve:
- evitare caricamenti inutili;
- caricare i GLB delle prese solo quando servono;
- mantenere catalogo e anteprime in cache;
- evitare chiamate REST durante il movimento;
- evitare blocchi UI;
- sostenere almeno 40 prese oltre alla parete come target funzionale.

## 13. Memory management

Quando una risorsa 3D non è più necessaria:
- deve essere rimossa dalla scena;
- le risorse GPU/CPU associate devono essere liberate quando appropriato;
- il sistema non deve conservare inutilmente modelli del modale chiuso.

La strategia concreta di caching dei modelli già utilizzati non è completamente specificata dalla fonte e deve essere scelta senza compromettere il requisito di caricamento lazy.
