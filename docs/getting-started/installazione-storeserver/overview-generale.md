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
Il presente documento descrive i nuovi requisiti minimi e le procedure necessarie all'installazione e alla configurazione dello *StoreServer*, il nuovo sistema centralizzato per la gestione e l'analisi del punto vendita introdotto da Posware :material-tag:`4.3` e superiori.

!!! warning "Ordine di installazione obbligatorio"
    Negli scenari standard di installazione ex novo e migrazione (si esclude quindi lo scenario ibrido), è necessario installare Posware :material-tag:`4.3` prima lato **cassa** e poi sullo **StoreServer**.

## Requisiti minimi
### Hardware
!!! warning "Considerazioni importanti"

    Tali requisiti sono richiesti da un sistema usato **esclusivamente per lo *StoreServer* **. <br> Qualora il software dovesse condividere le risorse hardware con altri software di terze parti, rimane a carico del tecnico installatore adeguare tali risorse in base alle richieste hardware di quest'ultimi aumentando le risorse di quanto necessario. 
    
    Si raccomanda di acquistare un disco rigido SSD (tipo SATA o NVE) ad uso esclusivo per lo *StoreServer*.

- Un processore compatibile con la versione Windows 10 **x64** (**64bit**) :material-tag:**`1809`** (**Redstone 5**) o superiore oppure compatibile con Windows 11
- 8 Gb di RAM per installazioni fino a 4 casse
- 12 Gb di RAM per installazioni da 5 a 10 casse
- 16 Gb di RAM per installazioni da 11 a 16 casse
- 32 Gb di RAM per installazioni con più di 16 casse
- Disco rigido SSD (tipo SATA o NVME) da almeno 256 Gb fino a 10 casse
- Disco rigido SSD (tipo SATA o NVME) da almeno 512 Gb per più di 10 casse

!!! info "Linee guida"
    Le informazioni relative alla capacità dei dischi rigidi sono da considerarsi linee guida e possono variare in base al fatturato prodotto dal punto vendita. <br> Qualora ci fossero dei dubbi sullo spazio necessario, considerare una contingenza dal **25% fino al 50%** .

### Software
!!! failure "Sistemi a 32 bit non supportati"
    Non sono supportate versioni a 32bit di Windows 10.

!!! warning "Obbligo di tutte le dipendenze in Posware :material-tag:`4.3`"
    Le dipendenze sottostanti sono da considerarsi **TUTTE** obbligatorie per poter utilizzare Posware :material-tag:`4.3`.
    
- Posware Frontend :material-tag:`4.3.96` o superiore
- Windows 10 **x64** (**64bit**) :material-tag:**`1809`** (**Redstone 5**) o superiore, oppure
- Windows 11
- Winget :material-tag:`1.8.1911`
- SQL Server `2016`, `2017`, `2019` o `2022`
- MySQL :material-tag:`8.4`
- .NET Hosting Bundle :material-tag:`8.x`
- Servizi legacy (per la lista completa, consultare la relativa [sezione](./servizi-legacy.md#file-installati))

## Breaking changes
Vengono elencati in questa sezione i diversi **breaking changes** introdotti a partire dalla versione :material-tag:`4.3` di Posware.

Lo *StoreServer* è un nuovo applicativo introdotto per la prima volta con Posware :material-tag:`4.3`, quindi non ha esso stesso dei **breaking changes**. Questi saranno, quindi, tutti relativi alle **operazioni che ora non sono più possibili** con il passaggio alla versione :material-tag:`4.3` di Posware e l'introduzione dello *StoreServer*.

!!! warning "Le credenziali standard del database centralizzato sono cambiate"
    Ai fini di compatibilità con le nuove restrizioni dei criteri password complesse delle recenti versioni di MySQL e SQL Server, la nuova password di default è stata variata in **Vbhg4132!**

    **Si consiglia di cambiare la password in tutti i sistemi di produzione.** 

!!! danger "Obsolescenza di versioni di Navicat precedenti alla :material-tag:`12.0.28`"
    **A partire da MySQL :material-tag:`8.4`, in uso dallo *StoreServer*, NON sono assolutamente più supportate versioni di Navicat precedenti alla :material-tag:`12.0.28`.**

    **Usare versioni obsolete di Navicat comporta la CORRUZIONE IRRIMEDIABILE del database del server centrale!**

!!! warning "Obbligo di utilizzo di mysqldump per importare/esportare i dump dei database in MySQL"
    Da Posware :material-tag:`4.3` in avanti **è obbligatorio** usare il tool **mysqldump** di MySQL per eseguire l'esportazione e l'importazione dei dump dei database in MySQL. 

    Non è più possibile usare **Navicat o altri strumenti simili** per fare queste operazioni, altrimenti non verrà garantita **la completa integrità del database** durante gli spostamenti tra versioni diverse di MySQL.

!!! info "Obsolescenza del database CASSAMASTER"
    Lo *StoreServer* usa esclusivamente il database **POSWARE**.

    Tutte le strutture presenti nel vecchio database **`cassamaster`** sono state integrate nel database **`posware`**. Il vecchio database **`cassamaster`** non viene più usato da nessuna procedura interna ISiD.  

!!! warning "Campi custom nelle tabelle del database `posware`"
    **Con l'introduzione dello *StoreServer*, non è più possibile inserire nuovi campi custom alle tabelle esistenti del database `posware`.**

    Qualora fosse necessario estendere le informazioni esistenti con altre di terze parti, consultare il documento ["Aggiunta dati custom al database dello StoreServer"](../scenari-avanzati/dati-custom-database-storeserver.md). 

!!! danger "Non copiare dati e/o strutture tra database diversi!"
    Al fine di popolare il database dello *StoreServer*, **NON copiare dati e/o strutture delle tabelle del database `cassamaster`** o dai database delle **casse**, in quanto differenti e non compatibili tra loro!
    
    **Operazioni di questo tipo NON sono assolutamente più supportate e comportano l'immediata CORRUZIONE IRRIMEDIABILE del database del server centrale!**

!!! warning "Computazione automatica del codice assortimento 0"
    A partire dalla versione `4.3.0` del Pos_Sbv, **il codice assortimento 0 equivale ad un codice di default, riservato e non variabile da nessuno, che rappresenta l'invio a tutte le casse (o a tutti i punti vendita in caso di installazione in sede).**
    
    Quindi, a differenza della situazione precedente, dove il codice assortimento 0 doveva essere manutenuto manualmente, ora viene gestito automaticamente e **non deve quindi essere inserito nella tabella `assortimenti` del database dello *StoreServer***.<br>In tal modo, da standard, **il codice assortimento 0 invia a tutte le casse e punti vendita che hanno il flag di trasmissione pari ad 1 nella tabella `casse`** dello *StoreServer*.

    **Qualora si volesse escludere dall'invio delle variazione alcuni punti vendita o casse, si crea un assortimento dedicato personalizzato diverso da 0.**

!!! warning "Cambiamento del tipo di dato dei campi `Id` e `Pdv` della tabella `isi_txscontr`"
    **A partire dalla versione `4.3.0.0` del *Pos_TxScontr*, i campi `Id` e `Pdv` della tabella `isi_txscontr` del database dello *StoreServer* hanno subito un cambiamento del tipo di dato, passando da stringhe ad interi.**

## Installazione e configurazione
!!! abstract "Prima di iniziare"
    Prima di proseguire con lo scenario di installazione desiderato, completare le **[procedure di installazione comuni](./installazione-procedure-comuni.md)**.

Scenari di installazione previsti:

- [Installazione di un nuovo sistema](./installazione-nuovo-storeserver.md)
- [Migrazione da servizi legacy](./migrazione-servizi-legacy.md)

I dettagli specifici di ogni scenario e delle procedure comuni sono consultabili nelle rispettive sezioni.