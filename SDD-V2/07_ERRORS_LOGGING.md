# 07 — Error Handling and Logging

## 1. Centralized error handling

Backend e frontend devono avere gestione centralizzata degli errori.

Quando tecnicamente possibile, gli errori frontend devono essere notificati al server.

## 2. Backend errors

Per ogni errore:
1. intercettare;
2. generare ErrorId univoco;
3. registrare il dettaglio completo lato server;
4. restituire status HTTP appropriato;
5. restituire un messaggio utente comprensibile;
6. non esporre dettagli interni.

Vietato restituire:
- stack trace;
- path locali;
- dettagli eccezioni;
- configurazioni;
- informazioni sensibili.

## 3. Frontend errors

Devono essere gestiti almeno:
- JavaScript non gestito;
- REST;
- HTTP;
- caricamento modelli;
- parsing modelli;
- Three.js;
- Rapier;
- generazione immagine;
- funzionalità UI.

## 4. Error isolation

Un errore relativo a una singola presa non deve bloccare l'intera applicazione se è possibile mantenere uno stato coerente.

## 5. Logging architecture

Tutti i log applicativi devono essere prodotti/conservati lato server.

Il frontend non deve:
- mantenere log persistenti locali;
- scrivere direttamente file di log.

## 6. Structured logging

Preferenza: JSON strutturato.

Ogni evento deve contenere quando disponibile:
- timestamp;
- livello;
- categoria;
- messaggio;
- RequestId;
- ErrorId;
- componente;
- contesto necessario al troubleshooting.

Livelli richiesti, coerenti con .NET `LogLevel`:
- Trace;
- Debug;
- Information;
- Warning;
- Error;
- Critical.

Default minimo di logging: `Information`.

Configurazione tramite `appsettings.json`.

## 7. Sanitization

Non devono essere registrati:
- credenziali;
- token;
- dati personali;
- informazioni riservate.

I dati provenienti dal frontend devono essere sanitizzati prima della registrazione.

## 8. Performance logging

Il logging deve:
- essere asincrono;
- evitare I/O sincrono durante le operazioni normali;
- non bloccare il thread principale backend;
- privilegiare continuità applicativa rispetto a log non essenziali.

## 9. High-frequency events

Non devono essere registrati:
- ogni frame;
- ogni movimento mouse;
- ogni aggiornamento continuo della posizione.

Devono essere registrati solo eventi utili al troubleshooting.

## 10. Request correlation

Quando applicabile, gli eventi devono essere correlabili tramite RequestId.

Gli errori devono avere ErrorId.

La fonte non prescrive un SessionId nel logging finale: non introdurlo come requisito obbligatorio senza una decisione esplicita.

## 11. Log rotation

Requisiti:
- un file per giornata;
- eliminazione dei file più vecchi di 7 giorni;
- eliminazione effettuata alla data di salvataggio del nuovo file.

La tecnologia concreta di rolling file non è prescritta.

## 12. Asset errors

Per errori di caricamento/generazione:
- log tecnico completo lato server;
- modello/risorsa identificata;
- messaggio utente generico;
- nessun dettaglio tecnico al client;
- altri modelli devono continuare a funzionare quando possibile.
