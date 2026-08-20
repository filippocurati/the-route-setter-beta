# 04 — Domain, Geometry and Physics Specification

## 1. Domain entities

### WallModel

Rappresenta il modello 3D della parete.

Responsabilità:
- mesh visuale;
- geometria da cui generare il TriMesh Rapier;
- riferimento all'asset statico.

### HoldModel

Rappresenta il modello disponibile nel catalogo.

Attributi concettuali:
- identificativo;
- percorso/cartella;
- GLB;
- eventuali texture/asset;
- anteprima;
- collider Convex Hull;
- stato di disponibilità del collider.

### HoldInstance

Rappresenta una presa presente nella scena.

Attributi concettuali:
- identificativo istanza;
- riferimento al HoldModel;
- posizione;
- orientamento;
- stato di aggancio;
- riferimenti alla mesh Three.js;
- riferimenti ai componenti Rapier.

## 2. Stati concettuali della hold

```text
CATALOG
   │
   │ Utilizza
   ▼
LOADING
   │
   ├── errore → CATALOG / errore locale
   │
   ▼
SCENE_UNATTACHED
   │
   │ snap entro 5 cm
   ▼
ATTACHED
   │
   ├── movimento
   ├── rotazione
   │
   └── rimozione → CATALOG
```

La nomenclatura concreta degli stati può variare, ma il comportamento deve rimanere equivalente.

## 3. Coordinate

- sistema destrorso;
- coordinate coerenti con Three.js;
- unità in metri;
- 1 unità = 1 metro.

## 4. Origine del modello hold

L'origine locale del modello deve essere nel centro geometrico della superficie posteriore della presa, corrispondente al punto di contatto con la parete.

Questo requisito deve essere rispettato dagli asset oppure gestito tramite una trasformazione di importazione coerente.

## 5. Base della presa

La base è la parte piatta posteriore che aderisce alla parete.

La parte superiore può avere forma arbitraria.

## 6. Collider hold

Il collider è un Convex Hull.

Proprietà:
- indipendente dalla mesh grafica;
- utilizzato esclusivamente dalla fisica;
- ottimizzato per compromesso accuratezza/prestazioni;
- non deve necessariamente rappresentare ogni dettaglio della mesh.

## 7. Generazione Convex Hull

Input:
- vertici della mesh GLB.

Output:
- vertici dell'hull;
- indici delle facce se disponibili;
- metadati di provenienza del GLB.

La generazione è eseguita dal backend tramite una libreria geometrica .NET indipendente da Rapier.

## 8. Cache del Convex Hull

Il file del collider deve contenere un identificatore del contenuto GLB sorgente, tramite hash o timestamp.

Regole:

```text
Hull assente
    → genera

Hull presente + sorgente invariata
    → riusa

Hull presente + sorgente modificata
    → rigenera
```

## 9. Collider parete

La parete utilizza un TriMesh derivato direttamente da vertici e triangoli della mesh.

Il TriMesh:
- è generato lato client;
- è statico;
- non partecipa a simulazioni dinamiche.

## 10. Configurazione fisica

Obbligatoriamente:
- gravità disabilitata;
- niente simulazione dinamica;
- niente inerzia;
- niente rimbalzo;
- niente attrito;
- collision detection continua;
- movimento cinematico;
- KinematicCharacterController Rapier.

## 11. Snap

Condizione:
- distanza di attivazione: 5 cm = 0,05 m.

Comportamento:
1. individuare il punto di contatto più vicino;
2. determinare la normale nel punto esatto;
3. orientare la base della presa alla superficie;
4. eliminare l'inclinazione manuale rispetto alla normale;
5. consentire successivamente traslazione tangenziale e rotazione attorno alla normale.

## 12. Movimento tangenziale

Dopo lo snap, il movimento deve rimanere sulla superficie.

Per la direzione:

```text
view vertical axis
        ↓
projection onto tangent plane
        ↓
tangent "up/down"

view horizontal axis
        ↓
projection onto tangent plane
        ↓
tangent "left/right"
```

La proiezione deve essere effettuata rispetto alla normale della superficie nel punto corrente.

La normalizzazione e la gestione di eventuali vettori degeneri devono essere implementate in modo robusto.

## 13. Incrementi

- traslazione: 0,01 m per click;
- rotazione: 1° per click.

Il movimento continuo con pulsante mantenuto premuto deve essere implementato tramite aggiornamenti locali del frontend e non tramite REST.

Il metodo concreto di ripetizione temporale è una decisione implementativa, purché il click singolo mantenga l'incremento richiesto.

## 14. Collision response

La risposta richiesta è:

```text
desired movement
       ↓
KinematicCharacterController
       ↓
collision detection
       ↓
movement with blocked components removed
       ↓
final permitted movement
```

Non implementare una risposta custom finché il comportamento nativo del controller soddisfa i requisiti.

## 15. Compenetrazione

È vietato che una hold:
- attraversi la parete;
- attraversi un'altra hold.

Se una direzione è bloccata, le componenti non bloccate devono continuare a essere disponibili.

## 16. Sincronizzazione rendering/fisica

La trasformazione risultante dalla simulazione deve essere applicata alla corrispondente mesh Three.js.

La mesh non deve diventare la fonte della verità per la collisione.

La rappresentazione fisica deve rimanere indipendente dalla mesh visuale.

## 17. Determinismo

I test fisici devono usare simulazione headless e non devono dipendere dal frame rate o dal timing reale del browser.

Il sistema di test deve usare input e stato iniziale controllati.
