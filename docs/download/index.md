---
title: Download
description: Pagina da cui poter scaricare le ultime versioni dei software rilasciati
icon: material/download
---

Questa pagina raccoglie in un unico punto i pacchetti di installazione utilizzati nell’ecosistema Posware per la gestione di casse, e servizi legacy di Front Office.
Tutti i link riportati si riferiscono alle **ultime versioni stabili** rilasciate e devono essere utilizzati esclusivamente da personale tecnico autorizzato.

!!! warning "Prima di scaricare e installare"
    Prima di utilizzare qualsiasi strumento, assicurarsi di aver letto le sezioni *Prerequisiti*, *Procedure comuni di installazione* e *Guida alla migrazione* nella documentazione di Posware.

---

## Riepilogo strumenti disponibili

| Software             | Componente principale         | Utilizzo principale                                                                 |
|----------------------|------------------------------|--------------------------------------------------------------------------------------|
| Database Upgrader    | StoreServer / database Posware | Aggiornamento strutture e dati del database posware negli scenari di migrazione. |
| SetupPosware         | Posware Frontend (casse)     | Installazione ex-novo o migrazione da Posware 4.2.x a 4.3.x sui terminali cassa. |
| PosUpdate            | Posware Frontend (casse)     | Aggiornamento di Posware 4.3.x all’ultima build disponibile sulla stessa major/minor. |
| Setup Servizi legacy | Servizi legacy Front Office  | Installazione, migrazione e aggiornamento dei servizi legacy lato server di barriera.|

---

## SetupPosware

**SetupPosware** è il nuovo installer riprogettato per l’uscita di Posware Frontend 4.3, utilizzato sia per nuove installazioni su casse che per la migrazione da `v4.2`.

**Non viene utilizzato per gli aggiornamenti di build all’interno della stessa versione 4.3.x, che rimangono a carico di PosUpdate.**

[Download SetupPosware](https://winget.isid.it/assets/download/latest?tag=setupposware){:target="_blank" .md-button .md-button--primary }

Scenari supportati:

- Nuova installazione di Posware Frontend sui terminali cassa.
- Migrazione da Posware 4.2.306 (o superiore) a Posware 4.3.x.

---

## PosUpdate

**PosUpdate** è l’applicativo dedicato esclusivamente all’aggiornamento di Posware Frontend all’ultima build disponibile, mantenendo invariata la major/minor version (es. da 4.3.x a una build più recente di 4.3.x).

È utilizzato sia per aggiornare il Frontend, sia per installare o reinstallare componenti di supporto come il servizio *PosSbv* sulle casse.

[Download PosUpdate](https://winget.isid.it/assets/download/latest?tag=posupdate){:target="_blank" .md-button .md-button--primary }

Funzioni principali:

- Aggiornamento delle build di Posware 4.3.x senza cambiare major/minor.
- Installazione e reinstallazione del servizio *PosSbv Client* per la ricezione delle variazioni sulle casse.

!!! tip "Uso congiunto con SetupPosware"
    SetupPosware si occupa delle installazioni ex-novo e delle migrazioni 4.2 → 4.3, mentre PosUpdate gestisce gli aggiornamenti successivi di manutenzione sulla 4.3.x.
    Nello scenario di installazione standard, SetupPosware viene utilizzato una volta per portare le casse a 4.3.x, quindi gli aggiornamenti ricorrenti vengono delegati a PosUpdate.

---

## Setup Servizi legacy Front Office

Il **setup dei servizi legacy** del Front Office installa, migra e aggiorna l’intero parco servizi legacy (PosLog, PosSbv, PosFidelity, PosTxLog, PosTxScontr, ecc.) necessari alla comunicazione e integrazione tra casse, server di barriera e StoreServer.

Questo setup supporta tre scenari: nuova installazione dei servizi, migrazione dalla versione 4.2 alla 4.3 e aggiornamento alla versione più recente in linea con la major/minor installata.

[Download Setup Servizi legacy](https://winget.isid.it/assets/download/static/Setup_Posware_Servizi_Legacy_4.3.27-1.exe){:target="_blank" .md-button .md-button--primary }

Esempi di componenti installati/migrati:

- Servizi Windows: PosFidelity, PosLog, PosSbv, PosTxLog, PosTxScontr.
- Eseguibili di Front Office: PosAna*, PosInv*, PosUtility, PoswareTxLog, vari batch di aggiornamento e trasmissione.

!!! warning "Prerequisiti e ordine delle installazioni"
    L’installazione dei servizi legacy richiede che lo StoreServer sia già installato e configurato correttamente, inclusi database provider MySQL 8.4 o SQL Server e invio massivo dell’anagrafica.

---

## Database Upgrader

Il **Database Upgrader** è il tool utilizzato per aggiornare il database `posware` quando si migra da Posware 4.2 a Posware 4.3 nello scenario StoreServer, dopo aver importato il database su MySQL 8.4 o su una nuova istanza di SQL Server.

Rappresenta uno step centrale nelle procedure di migrazione, in quanto applica le migrazioni strutturali e di dati necessarie al corretto funzionamento del nuovo StoreServer.

[Download Database Upgrader](https://winget.isid.it/assets/download/latest?tag=databaseupgrader){:target="_blank" .md-button .md-button--primary }

Ambito d’uso tipico:  

- Migrazione da Posware 4.2 a 4.3 lato StoreServer.
- Migrazione di un database `posware` importato in MySQL 8.4 o SQL Server 2016/2017/2019/2022 alla struttura aggiornata.

!!! danger "Uso solo in scenari di migrazione"
    Il Database Upgrader va eseguito solo su database di produzione di cui è stato effettuato un backup completo e verificato.
    Qualsiasi errore o interruzione durante le migrazioni potrebbe rendere necessario un rollback, possibile solo in presenza di backup integri.

---

## StoreServer, versioni dei pacchetti ed installazioni con Package Manager

- Le versioni effettive dei pacchetti (es. *4.3.55*) sono riportate nel nome file e nella pagina [*Release Notes*](../release-notes/index.md).
- Per ambienti gestiti tramite **Winget** package manager, in particolare lo StoreServer, possono essere installati o aggiornati **==*unicamente*== usando i comandi documentati nella [guida di installazione](../getting-started/installazione-storeserver/overview-generale.md).**
