---
tags:
    - Posware
    - Celiachia
---

# Posware - Modulo E-Voucher celiachia

**Prima revisione documento: 30 settembre 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**

!!! warning "DOCUMENTO PRELIMINARE"
    Le informazioni e i dettagli contenuti in questo documento possono essere soggetti a modifiche e non costituiscono un protocollo definitivo.

## Introduzione

Questo modulo permette ai consumatori di pagare prodotti coinvolti dall'iniziativa, usando la tessera sanitaria.

Il modulo è concepito per supportare i provider di servizi presenti sul mercato e garantire  la tracciabilità delle transazioni ai fini della **rendicontazione**.

Il modulo garantisce:

- una **gestione fluida e standardizzata** del bonus celiachia,
- la **conformità normativa** con le regole stabilite dalle ASL,
- la possibilità di **rendicontare in maniera unificata** i dati per analisi e controlli futuri.

## Ambito del progetto

Il modulo include:

1. **Integrazione lato frontend sulla cassa Posware** per la gestione operativa del bonus celiachia durante le transazioni.
2. **Un modello dati dedicato** per la rendicontazione delle operazioni in cui sono stati effettuati pagamenti con bonus celiachia.
3. **API REST di consultazione dati** per l’invio dei dati necessari alla rendicontazione celiachia dallo StoreServer alla propria ASL di competenza.

## Attivazione del servizio e requisiti burocratici

Per poter utilizzare il bonus celiachia in un punto vendita è necessario:

1. **Contattare la ASL di riferimento della zona** per formalizzare l’adesione al servizio.
2. **Ottenere dalla ASL la lista dei prodotti autorizzati** acquistabili con il bonus.
3. Configurare all’interno di Posware e del server di barriera le anagrafiche prodotti e i parametri operativi coerenti con quanto stabilito dall’ente sanitario.

Solo al termine di questo iter burocratico il punto vendita può essere abilitato all’erogazione del bonus tramite le casse Posware.

## Prossimi passi

- Per l'installazione e la configurazione del modulo su Posware frontend, consultare [questo documento](./integrazione-bonus-celiachia-posware.md)
- Per informazioni su come effettuare la fase di rendicontazione, consultare [questo documento](./rendicontazione-celiachia.md)
