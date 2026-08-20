# 06 — Backend and API Specification

## 1. Technology

Backend:
- C#;
- ASP.NET Core Web API;
- REST;
- JSON;
- OpenAPI/Swagger;
- Controller/Service.

## 2. Backend responsibilities

Il backend deve:
- analizzare gli asset;
- esporre il manifest/catalogo;
- esporre le risorse statiche;
- generare e aggiornare i Convex Hull;
- ricevere eventi di troubleshooting;
- gestire errori;
- produrre log.

Non deve:
- eseguire la simulazione fisica interattiva;
- dipendere da Rapier;
- aggiornare la posizione delle prese durante l'interazione.

## 3. Asset discovery

All'avvio:
1. individuare `Pareti`;
2. individuare `Prese`;
3. individuare cartelle `Hold...`;
4. identificare GLB e risorse presenti;
5. identificare `PREV_`;
6. verificare collider;
7. avviare generazione dei collider mancanti/invalidi.

## 4. Manifest catalogo

Il backend deve fornire un manifest sufficiente al frontend per popolare il catalogo senza scaricare tutti i modelli 3D.

Il manifest deve consentire almeno di identificare:
- hold;
- anteprima;
- disponibilità del modello;
- disponibilità del collider;
- riferimenti alle risorse necessarie.

La forma JSON precisa del DTO è una decisione progettuale da definire durante l'implementazione e documentare in OpenAPI.

## 5. API

Il documento sorgente impone REST/OpenAPI ma non prescrive nomi e URI specifici.

Pertanto l'agente deve progettare API coerenti, documentarle in Swagger/OpenAPI e mantenere il contratto stabile durante l'implementazione.

Le capacità minime richieste sono:

### API-CAP-001 — Catalogo

Recupero dell'elenco delle prese e delle relative anteprime/riferimenti.

### API-CAP-002 — Asset

Recupero dei modelli e degli asset statici necessari.

### API-CAP-003 — Collider

Recupero del collider Convex Hull pre-calcolato.

### API-CAP-004 — Logging frontend

Ricezione asincrona degli eventi frontend necessari al troubleshooting.

## 6. Controller/Service

I Controller devono:
- validare la richiesta;
- delegare ai Service;
- restituire response HTTP coerenti.

I Service devono contenere la logica applicativa.

La logica di generazione dei collider deve essere isolata in un servizio dedicato.

## 7. Convex Hull service

Responsabilità:
- leggere il GLB;
- estrarre i vertici necessari;
- calcolare l'hull;
- serializzarlo;
- salvare metadati del GLB;
- verificare validità del file esistente.

Il servizio non deve importare Rapier.

## 8. Asynchronous generation

La generazione iniziale dei collider deve essere eseguita in background.

Il sistema deve distinguere almeno:
- collider pronto;
- collider in generazione;
- errore generazione.

Il catalogo deve poter riflettere la disponibilità progressiva.

## 9. Error contract

Le risposte di errore devono:
- essere coerenti con lo status HTTP;
- non esporre dettagli tecnici;
- contenere un ErrorId quando appropriato;
- consentire la correlazione con il log server.

Non deve essere restituito uno stack trace al client.

## 10. Logging endpoint

Il frontend deve poter inviare eventi applicativi necessari al troubleshooting.

La trasmissione:
- deve essere asincrona;
- non deve bloccare l'interfaccia;
- non deve impedire il normale funzionamento se il backend di logging non è disponibile.

## 11. API documentation

Swagger/OpenAPI deve documentare:
- endpoint;
- request;
- response;
- status code;
- error contract;
- DTO.

## 12. Static files

I modelli e gli asset descritti dalla fonte devono essere serviti come file statici.

Il backend non deve trasformarli in render server-side.
