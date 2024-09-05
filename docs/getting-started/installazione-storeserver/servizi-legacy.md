---
tags:
    - Posware
    - StoreServer
---

# Front Office - Aggiornamento [DRAFT DA COMPLETARE]

**Prima revisione documento: 3 settembre 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---
## Cenni preliminari
Il presente documento, come introdotto nell'overview generale, descrive nel dettaglio la procedure comuni allo scenario di aggiornamento del *Front-office* dando particolare enfasi ai vari passaggi necessari per portare a compimento queste operazioni.

## Backup preliminari
Prima di iniziare tutta la procedura di aggiornamento del *Front-office*, è assolutamente mandatorio eseguire i backup da usare in caso di malfunzionamenti e per l'eventuale procedura di rollback. Questa attività è complementare a tutte quelle svolte per l'installazione dello *StoreServer*.
 
Eseguire il backup di quanto segue:

1. Backup dei programmi da aggiornare

## Installazione e configurazione del database
### MySQL
Come introdotto nell'overview generale, in caso il provider del database scelto sia MySQL, è necessario installare MySQL v. `8.4`, l'unica versione attualmente compatibile con lo *StoreServer*. 

## Installazione dei moduli
Per prima cosa, da gestione servizi, arrestare tutti i servizi oggetto di questo aggiornamento.

| Nome servizio | Descrizione | Presenza |
| ------------- | ----------- | ------------|
| Pos_Log       | Servizio di ricezione log | Necessario |
| Pos_Fidelity  | Servizio di gestione fidelity | Opzionale |
| Pos_TxLog     | Servizio di invio log a server | Opzionale |
| Pos_TxScontr  | Servizio di invio scontrini a server | Opzionale |
| Pos_Sbv       | Servizio di invio manutenzioni | Necessario |

Copiare tutti programmi aggiornati comprensivi delle dll sovrascrivendo quanto già in essere.

| Nome programma | Descrizione | Presenza |
| ---------------- | ----------- | ------------|
| Pos_Anarep.exe   | Analisi venduto per reparto | Opzionale |
| Pos_Casse.exe    | Utility di attivazione/disattivazione terminali | Opzionale |
| Pos_CFisc.exe    | Utility di invio comando di chiusura fiscale | Necessario |
| Pos_Chiave.exe   | Utility di gestione posizione chiave su front-end | Opzionale |
| Pos_CtrCas.exe   | Utility di controllo stato operatori | Opzionale |
| Pos_Eod.exe      | Utility di creazione file EOD | Opzionale |
| Pos_Insta.exe    | Analisi statistiche | Necessario |
| Pos_Spegni.exe   | Utility di invio comando di spegnimento casse | Opzionale |
| Pos_Utility.exe  | Utility di gestione casse | Opzionale |
| ElaborazioneFile.exe  | Utility di conversione manutenzioni | Opzionale |
| Pos_SbvGest.exe  | Utility di gestione servizio Pos Sbv | Opzionale |

#### Configurazione
Una volta aggiornati i programmi occorre modificare la configurazione dei seguenti files:

| Nome file | Descrizione | Presenza |
| ---------------- | ----------- | ------------|
| config.ini   | File di configurazione dei servizi e dei moduli di front-end | Necessario |
| appsettings.json | File di configurazione per il servizio Pos_Sbv| Necessario |
| ElaborazioneFile.exe.config | File di configurazione per il modulo ElaborazioneFile.exe | Opzionale |
| Pos_SbvGest.exe.config | File di configurazione per il modulo Pos _SbvGest | Necessario |

Nel nuovo database convertito occorre modificare nella tabella Isi_Config l'accesso al database e la porta di configurazione del Pos_Log.
I parametri da modificare sono:

| Sezione | Chiave | Valore |
| ---------------- | ----------- | ------------|
| Log           | Porta     | 6951 |
| Loyalty       | Anagrafica | Posware |
| Loyalty       | Scontrini | Posware |