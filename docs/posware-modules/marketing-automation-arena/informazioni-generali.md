---
tags:
    - Posware
    - Gruppo Arena
    - Marketing Automation
    - Epipoli
---

# Posware - Modulo Marketing Automation Gruppo Arena

**Prima revisione documento: 19 marzo 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**

!!! warning "DOCUMENTO PRELIMINARE"
    Le informazioni e i dettagli contenuti in questo documento possono essere soggetti a modifiche e non costituiscono un protocollo definitivo.

## Introduzione

Questo modulo introduce una nuova integrazione tra il sistema di cassa **Posware** e la piattaforma di loyalty gestita da **Epipoli**, nell'ambito del nuovo programma fedeltà del cliente **Gruppo Arena**.

Il modulo è specifico per il cliente **Gruppo Arena** e si inserisce nel loro piano di rinnovamento interno relativo alla **Marketing Automation**, che prevede la gestione congiunta di **promozioni**, **coupon**, **sincronizzazioni giornaliere** e **vendite**, assicurando un flusso operativo coerente e affidabile tra i diversi sistemi coinvolti.

Il modulo consente alle casse Posware di:

- Ricevere ed elaborare **promozioni associate alle loyalty card**, rese disponibili tramite sincronizzazione periodica dei dati.
- Gestire **coupon digitali** in modalità ***online real-time***, tramite chiamate ai servizi esposti da Epipoli.
- Applicare automaticamente regole promozionali e operazioni di “**burning**” (bruciatura) dei coupon tramite meccanismi di **reserve/commit/rollback/unfreeze**.
- Garantire continuità operativa tramite dati offline precaricati dai server di barriera.
- Trasmettere a Epipoli i dati di vendita completi, includendo il dettaglio delle promozioni applicate e dei coupon utilizzati.

---

## Ambito del progetto

Il presente modulo copre le attività di integrazione lato **Posware** e **server di barriera** (inclusi i servizi *PosPromo*, *PosFidelity*, *PosSbv* e *PosEpipoliHelper*). Nello specifico, l'ambito include:

1. **Sincronizzazione ibrida dei dati (S3 e Fallback API)** per il caricamento quotidiano di loyalty card e regole promozionali dal bucket S3 di Epipoli al server di barriera fino ad arrivare ai database locali delle casse, con interrogazione online real-time di emergenza per le tessere non presenti in locale.
2. **Motore promozionale offline** per la valutazione autonoma a scontrino e l'applicazione automatica degli sconti legati alla carta fidelity, basato sugli *ExtRuleCode* delle promozioni precedentemente sincronizzate.
3. **Integrazione API real-time per la gestione coupon** per la lettura e validazione dei voucher digitali verso i sistemi Epipoli, comprensiva di logiche transazionali antifrode (*reserve*, *commit*, *rollback*, *unfreeze*) e gestione strutturata delle cadute di linea e degli altri scenari di *fault-back*.
4. **Estensione dei flussi di vendita** per l'arricchimento dei dati di transazione con le informazioni dettagliate sui coupon e le promozioni applicate, integrandoli nei sistemi di salvataggio ordini esistenti per il successivo invio a Epipoli.
5. **Processi di riconciliazione notturna** su bucket S3 per assicurare il perfetto allineamento contabile e operativo tra i dati emessi dalle casse e la piattaforma Epipoli, garantendo continuità con i flussi di catalogo prodotti gestiti via *SFTP*.

---

## Componenti applicativi e loro interazione

L’architettura dell’integrazione tra il sistema di cassa **Posware** e la piattaforma di loyalty gestita da **Epipoli** è distribuita su tre livelli gerarchici (cassa, server di barriera e server centralizzato), ognuno dei quali ospita componenti specifici che cooperano per garantire la continuità operativa e la sincronizzazione dei dati.

### Distribuzione dei componenti

L'infrastruttura si articola come segue:

- **Posware in cassa:** I punti vendita utilizzano una versione aggiornata di **Posware** equipaggiata con il **plugin Arena 2026**, che abilita le nuove logiche di comunicazione verso Epipoli per i coupon e verso il server di barriera e centralizzato per le promozioni.
- **Server di barriera (punto vendita):** Presidia l'operatività del singolo negozio. Ospita:

    - Una istanza del servizio **PosEpipoliHelper** configurata in modalità **ETL** (specifica per l'insegna del punto vendita).
    - Il servizio **PosFidelity** per le interrogazioni riguardanti le loyalty card e le promozioni.
    - Il servizio **PosSbv** per l'invio delle variazioni dal punto vendita alle singole casse.

- **Server centralizzato (CEDI):** Coordina l'intera rete e garantisce i servizi di fallback. Ospita:

    - **PosPromo:** Il modulo centrale per la creazione delle promozioni e la gestione della tabella di equipollenza tra i codici promo interni Posware e quelli esterni Epipoli.
    - Il servizio **PosFidelity** per le interrogazioni di **fallback** riguardanti le carte fedeltà e le promozioni, in caso le casse debbano contattare il server centralizzato, quando non riescono a collegarsi con il **PosFidelity** sul server di barriera.
    - Il servizio **PosSbv** per l'invio delle variazioni dal server CEDI ai singoli punti vendita.
    - **Due istanze distinte di PosEpipoliHelper:**

        - **Modalità ETL:** configurata con parametro insegna `*` per l'elaborazione globale di tutte le loyalty card, sia quelle di **Decò** che di **Superconveniente**.
        - **Modalità Kafka-Producer:** dedicata alla ricezione dei flussi di vendita dalle casse e all'inoltro verso Epipoli.

### Interazione tra i componenti applicativi

Il funzionamento del sistema si basa su uno scambio costante di informazioni tra i componenti sopra elencati.

#### Gestione Vendite e Coupon

Le casse Posware comunicano in tempo reale con i sistemi esterni e il server centralizzato:

1. **Invio venduto:** Il dato dello scontrino viene inviato al **PosEpipoliHelper (Kafka-Producer)** sul server centralizzato, che ne garantisce la persistenza e l'inoltro definitivo a Epipoli.
2. **Validazione coupon:** L'attivazione dei coupon digitali avviene tramite chiamata diretta della cassa verso gli endpoint **Epipoli**.

#### Logica Promozionale e Loyalty

Il cuore della logica promozionale risiede nell'interazione tra i dati sincronizzati e i motori di calcolo:

1. **Sincronizzazione (Mantemp):** Il **PosPromo** (CEDI) scrive i record nella tabella `mantemp` del server centralizzato. Il servizio **PosSbv** si occupa di distribuire questi dati ai server di barriera per popolare le tabelle `promozioni` e `promo_thirdparty`.
2. **Identificazione loyalty card:** Al passaggio della carta fedeltà, **Posware** interroga il **PosFidelity** locale sul server di barriera. Questo risponde verificando l'esistenza e lo stato della tessera tramite i dati scaricati dal **PosEpipoliHelper (ETL)**.
3. **Attivazione promozioni:** Il **PosFidelity** decide se una promozione deve scattare incrociando i dati della tabella `promozioni` con quelli ricevuti da Epipoli, utilizzando la tabella `promo_thirdparty` come "ponte" di traduzione tra i codici.
4. **Resilienza (Fallback):** In caso di indisponibilità del server di barriera locale, la cassa è configurata per reindirizzare le interrogazioni delle loyalty card e delle promozioni verso il **PosFidelity del server centralizzato**.

#### Ruolo del PosEpipoliHelper

Il servizio agisce come il principale orchestratore di dati dell'integrazione:

- **Modalità ETL:** Popola costantemente le tabelle delle anagrafiche riguardanti le loyalty card e le promozioni attive per ogni tessera.
- **Modalità Kafka-Producer:** Esegue l'invio del venduto, ricevendo i payload JSON dalle casse, salvandoli nella tabella `epipoli_ext_sell` e gestendo l'invio batch verso Epipoli.

---

## Prossimi passi

- Per la creazione della nuova tipologia di promozioni dal PosPromo, consultare [questo documento](./pos-promo.md)
<!-- - Per l'aggiornamento e la configurazione del servizio PosFidelity, consultare [questo documento]() -->
- Per l'installazione e la configurazione del servizio PosEpipoliHelper, consultare [questo documento](./pos-epipoli-helper.md)
<!-- - Per l'installazione e la configurazione del modulo su Posware frontend, consultare [questo documento]() -->
- Per le configurazioni relative al PosFidelity e del nuovo plugin Arena 2026 su Posware frontend, consultare il documento dei test UAT.