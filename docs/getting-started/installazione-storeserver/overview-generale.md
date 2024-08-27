---
tags:
    - Posware
    - StoreServer
---

# Overview generale

**Prima revisione documento: 24 aprile 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Cenni preliminari
Il presente documento descrive i nuovi requisiti minimi e le procedure necessarie all'installazione e alla configurazione dello *StoreServer*, il nuovo sistema centralizzato per la gestione e l'analisi del punto vendita introdotto da Posware 4.3 e superiori.

## Requisiti minimi
### Hardware
- Un processore compatibile con la versione Windows 10 **x64** (**64bit**) v. `1809` o superiore oppure compatibile con Windows 11
- 8 Gb di RAM per installazioni fino a 4 casse
- 12 Gb di RAM per installazioni da 5 a 10 casse
- 16 Gb di RAM per installazioni da 11 a 16 casse
- 32 Gb di RAM per installazioni con più di 16 casse
- Disco rigido SSD (tipo SATA o NVE) da almeno 256 Gb fino a 10 casse
- Disco rigido SSD (tipo SATA o NVE) da almeno 512 Gb per più di 10 casse

> Le informazioni relative alla capacità dei dischi rigidi sono da considerarsi linee guida e possono variare in base al fatturato prodotto dal punto vendita. <br> Qualora ci fossero dei dubbi sullo spazio necessario, considerare una contingenza dal 25% fino al 50%.

**Tali requisiti sono richiesti da un sistema usato esclusivamente per lo *StoreServer*. <br> Qualora il software dovesse condividere le risorse hardware con altri software di terze parti, rimane a carico del tecnico installatore adeguare tali risorse in base alle richieste hardware di quest'ultimi aggiungendo quanto necessario. In tal caso, si raccomanda di acquistare un disco rigido SSD (tipo SATA o NVE) esclusivo solo per lo *StoreServer*.**

### Software
- Posware Frontend v. `4.3.x`
- Windows 10 **x64** (**64bit**) v. `1809` o superiore
- Windows 11
- Winget v. `1.8.1911`
- SQL Server 2016, 2017, 2019 o 2022
- MySQL v. `8.4`
- .NET SDK v. `8.x`
- Servizi legacy:
    
    - Pos_Eod v. `1.9.0`
    - Pos_Log v. `1.3.23`
    - Pos_Insta v. `4.7.0`
    - Pos_Fidelity v. `1.3.19`
    - Pos_Chiave v. `1.1.4`
    - Pos_Cfisc v. `2.2.0`
    - Pos_RxVen v. `2.2.1`
    - Pos_RxLog v. `2.5.0`
    - Pos_Open v. `1.4.1`
    - Pos_Rxscon v. `2.4.1`
    - Pos_Anafatt v. `1.7.0`
    - Pos_AnaRep v. `1.3.2`
    - Pos_Anacli v. `1.9.0`
    - Pos_FatturaPA v. `1.0.27`
    - Pos_Utility v. `1.0.2`
    - Pos_Spegni v. `1.7`
    - Pos_RecEod v. `1.5`
    - Pos_RielabScontr v. `2.4.1`
    - Pos_AnaArt v. `2.1.3`
    - Pos_TxScontr v. `2.1.0`
    - Pos_Casse v. `1.2.0`
    - Pos_CtrCas v. `1.1.0`
    - Pos_Library

**Non sono supportate le versioni 32bit di Windows 10**

## Breaking changes (?)


## Installazione e configurazione
Sono previsti questi scenari di installazione:

- [Installazione ex novo](./installazione-scenario-ex-novo.md)
- [Migrazione da Posware 4.2.x](./installazione-scenario-migrazione.md)

Prima di proseguire con lo scenario di installazione desiderato, completare le [procedure di installazione comuni](./installazione-procedure-comuni.md).

I dettagli specifici di ogni scenario e delle procedure comuni sono consultabili nelle rispettive sezioni.