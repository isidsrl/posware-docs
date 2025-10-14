---
tags:
    - Posware
    - Celiachia
---

!!! warning "DOCUMENTO PRELIMINARE"
    Le informazioni e i dettagli contenuti in questo documento possono essere soggetti a modifiche e non costituiscono un protocollo definitivo.

# Documentazione introduttiva integrazione celiachia

**Prima revisione documento: 30 settembre 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**

## Introduzione
Questo progetto nasce dall'esigenza di integrare nei sistemi di cassa **Posware** la gestione dei pagamenti e della rendicontazione legati al bonus statale per i soggetti celiaci, erogato tramite il **provider Celiachia/ASL di riferimento**.<br>
L’integrazione permette di gestire in maniera automatica e tracciata tutte le fasi del processo: dal riconoscimento del cliente tramite tessera sanitaria, all’autorizzazione del pagamento, fino al salvataggio dei dati sul **server di barriera** ai fini della rendicontazione centralizzata. 

L'obiettivo del progetto è garantire:

- una **gestione fluida e standardizzata** del bonus celiachia,
- la **conformità normativa** con le regole stabilite dalle ASL,
- la possibilità di **rendicontare in maniera unificata** i dati per analisi e controlli futuri.

!!! info "Generalizzazione dell'integrazione bonus celiachia in Posware"
    Sebbene questo progetto sia stato sviluppato per l'integrazione specifica con il provider **Argentea**, con **ARIA** come gestore del bonus celiachia, il sistema è scalabile per poter gestire facilmente in futuro anche altri provider Celiachia, vista la sua generalizzazione.

## Ambito del progetto
Lo sviluppo riguarda:

1. **Integrazione lato frontend sulla cassa Posware** per la gestione operativa del bonus celiachia durante le transazioni.
2. **Definizione di un modello dati dedicato** per la rendicontazione delle operazioni in cui sono stati effettuati pagamenti con bonus celiachia.
3. **Implementazione di API REST** per l’invio dei dati necessari alla rendicontazione celiachia da Posware al server di barriera, con relativi **meccanismi di controllo e di validazione** per garantire coerenza e qualità dei dati trasmessi.
4. **Supporto alla configurazione e gestione operativa** per i punti vendita che attivano il servizio.

## Attivazione del servizio e requisiti burocratici
Per poter utilizzare il bonus celiachia in un punto vendita è necessario:

1. **Contattare la ASL di riferimento della zona** per formalizzare l’adesione al servizio.
2. **Ottenere dalla ASL la lista dei prodotti autorizzati** acquistabili con il bonus.
3. Configurare all’interno di Posware e del server di barriera le anagrafiche prodotti e i parametri operativi coerenti con quanto stabilito dall’ente sanitario.

Solo al termine di questo iter burocratico il punto vendita può essere abilitato all’erogazione del bonus tramite le casse Posware.

## Struttura della documentazione tecnica
Il presente documento ha lo scopo di fornire un quadro introduttivo.<br>
La documentazione tecnica completa del progetto si articola in due capitoli principali:

1. **Comportamenti e configurazioni lato frontend cassa Posware**<br>
Documento che raccoglie tutte le informazioni operative e di configurazione necessarie al corretto utilizzo del bonus celiachia in cassa, incluse le regole di funzionamento e i casi particolari gestiti dal sistema.
2. **Struttura della tabella `celiac_evoucher_settlement` e del payload JSON `CeliacSettlementRecord`**<br>
Documento che descrive in dettaglio il modello dati utilizzato per la rendicontazione, utile a chi deve leggere e interpretare i dati presenti nel database MySQL/SQL Server del server di barriera.

## Conclusione
Questo progetto rappresenta un passo importante nella digitalizzazione e nella tracciabilità dei pagamenti tramite bonus sanitari.<br>
Grazie a questa integrazione, i punti vendita possono gestire il processo in modo automatizzato, sicuro e conforme alle normative, riducendo gli oneri manuali e garantendo un servizio migliore al cliente finale.