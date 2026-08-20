# 08 — Testing Specification

## 1. General rule

La pipeline di build/CI deve fallire se uno qualsiasi dei test automatici fallisce.

Tutti i test devono essere eseguibili senza intervento manuale.

## 2. Backend unit tests

Framework: xUnit.

Devono essere testati almeno:
- servizi;
- controller;
- caricamento catalogo;
- logging;
- gestione dei collider;
- invalidazione collider.

## 3. Collider tests

### TEST-HULL-001 — Generazione mancante

Dato un GLB senza collider:
- avviare il servizio;
- verificare che il collider venga generato;
- verificare che il file venga salvato.

### TEST-HULL-002 — Riutilizzo

Dato un collider coerente con il GLB:
- avviare il processo;
- verificare che non venga rigenerato inutilmente.

### TEST-HULL-003 — Invalidazione

Dato un GLB modificato:
- verificare che il collider venga rigenerato.

### TEST-HULL-004 — Indipendenza Rapier

Il servizio di generazione deve essere testabile senza dipendenza dal runtime Rapier.

## 4. API integration tests

Devono verificare:
- richieste HTTP;
- status code;
- JSON;
- DTO;
- error contract;
- catalogo;
- asset/collider;
- logging endpoint.

## 5. Frontend E2E

Framework: Playwright.

Devono essere verificate almeno:
1. avvio applicazione;
2. caricamento parete;
3. caricamento catalogo;
4. visualizzazione anteprime;
5. apertura Dettagli;
6. chiusura Dettagli;
7. Utilizza;
8. selezione presa;
9. movimento;
10. rotazione;
11. snap;
12. rimozione;
13. ritorno nel catalogo;
14. Genera immagine;
15. gestione di errori non bloccanti.

## 6. Physics tests

Ambiente:
- JavaScript/Node;
- Rapier headless;
- Vitest o Jest.

I test devono essere deterministici.

### TEST-PHYS-001

Una presa non attraversa la parete.

### TEST-PHYS-002

Due prese non compenetrano.

### TEST-PHYS-003

Una presa può muoversi quando non esistono collisioni.

### TEST-PHYS-004

La rimozione di una presa libera lo spazio occupato.

### TEST-PHYS-005

Una componente bloccata non impedisce le componenti di movimento non bloccate, nei casi previsti dal move-and-slide.

### TEST-PHYS-006

Lo stesso stato iniziale e lo stesso input producono un risultato deterministico.

### TEST-PHYS-007

Il comportamento delle collisioni rimane invariato dopo modifiche al codice tramite regressione automatica.

## 7. Acceptance test di snap

### TEST-SNAP-001

Una presa a distanza > 5 cm non viene automaticamente agganciata.

### TEST-SNAP-002

Una presa entro 5 cm viene agganciata secondo il punto di contatto più vicino.

### TEST-SNAP-003

La normale usata per l'orientamento è quella del punto di contatto.

### TEST-SNAP-004

Dopo lo snap la presa può ruotare attorno alla normale.

### TEST-SNAP-005

Dopo lo snap il movimento è tangenziale alla superficie.

## 8. Input tests

Verificare:
- click singolo = 1 cm / 1°;
- pressione continua = ripetizione;
- shortcut equivalenti ai pulsanti.

## 9. Performance validation

Il progetto deve includere una procedura automatizzata o ripetibile per verificare almeno il target di 40 prese.

Il documento sorgente non definisce un hardware benchmark specifico né un metodo numerico di misura FPS; pertanto il metodo esatto deve essere definito nel piano di implementazione senza trasformare un valore indicativo in una garanzia hardware universale.

## 10. Test documentation

Deve essere generato un documento dedicato che descriva:
- test implementati;
- scopo;
- risultato atteso;
- framework;
- strumenti.
