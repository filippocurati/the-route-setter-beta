# SDD — Applicazione per tracciature vie climbing indoor

## Scopo

Questo pacchetto contiene la specifica **Spec-Driven Development (SDD)** derivata dal documento sorgente `Istruzioni.md`, insieme al piano di implementazione incrementale e verificabile.

Il documento sorgente rimane la fonte dei requisiti di prodotto. I documenti SDD trasformano quei requisiti in:

- requisiti identificati e tracciabili;
- invarianti architetturali;
- specifiche di dominio e fisica;
- specifiche frontend, backend e asset;
- specifiche di error handling e logging;
- specifiche di test;
- criteri di accettazione;
- piano di implementazione a step verificabili.

## Regola fondamentale per l'agente

L'agente deve trattare le specifiche SDD come vincoli di implementazione.

**Non deve introdurre comportamenti in contrasto con gli invarianti o con i requisiti MUST.**

Quando una decisione tecnica non è determinata dai requisiti, l'agente può scegliere una soluzione coerente solo se questa non modifica il comportamento richiesto. Quando la scelta può modificare il comportamento osservabile, deve essere trattata come decisione da confermare.

## Documenti

| File | Contenuto |
|---|---|
| `01_CONSTITUTION.md` | principi e vincoli non negoziabili |
| `02_REQUIREMENTS.md` | requisiti funzionali e non funzionali |
| `03_ARCHITECTURE.md` | architettura software e responsabilità |
| `04_DOMAIN_AND_PHYSICS.md` | modello di dominio, geometria, collisioni e movimento |
| `05_FRONTEND_UI_ASSETS.md` | frontend, scena 3D, UI e gestione asset |
| `06_BACKEND_API.md` | backend, catalogo, collider e API |
| `07_ERRORS_LOGGING.md` | error handling e logging |
| `08_TESTING.md` | strategia e requisiti dei test |
| `09_TRACEABILITY.md` | matrice requisito → specifica → test |
| `10_OPEN_DECISIONS.md` | punti che il documento sorgente non determina completamente |
| `IMPLEMENTATION_PLAN.md` | piano di implementazione incrementale |

## Criterio di interpretazione

I requisiti sono classificati come:

- **MUST**: vincolo obbligatorio;
- **SHOULD**: requisito raccomandato, da non derogare senza motivazione;
- **OPEN**: informazione non determinata dal documento sorgente.

Le parti marcate come OPEN non devono essere trasformate dall'agente in requisiti arbitrari.

