---
title: Getting started - Scenari di installazione
description: Panoramica completa dei scenari di installazione di Posware. Scopri quale opzione è adatta al tuo punto vendita.
icon: simple/readme
---

# Getting started - Scenari di installazione

**Prima revisione documento: 27 agosto 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**

---

Questa guida introduttiva vi orienterà nella scelta del percorso di installazione più appropriato al vostro contesto operativo.

Gli scenari elencati fanno riferimento sia i software da installare sul server di barriera che Posware Frontend cassa.
Per praticità, in questo documento si farà riferimento a Posware come unico ecosistema software backend e frontend cassa.

## **Prima di iniziare**

Prima di procedere con l'installazione di Posware, assicuratevi di:

- Aver verificato i **requisiti hardware e software** minimi richiesti
- Aver installato tutte le **dipendenze esterne** necessarie
- Aver effettuato i **backup necessari** dei sistemi esistenti

Per informazioni dettagliate su questi prerequisiti, consultate le pagine:

- [Requisiti di Sistema Posware](./installazione-posware-frontend/overview-generale.md#requisiti-minimi)
- [Dipendenze esterne Posware](./installazione-posware-frontend/dipendenze-aggiornamenti-necessari.md)
- [Requisiti di Sistema StoreServer](./installazione-storeserver/overview-generale.md#requisiti-minimi)

## **I tre scenari principali di installazione**

Posware supporta tre scenari di installazione distinti, ciascuno con specifiche modalità operative e vincoli tecnici. La scelta dello scenario dipende dalla vostra situazione attuale.

### Scenario 1: Installazione ex-novo

**Situazione**: Dovete installare Posware per la prima volta su terminali casse completamente nuovi o su un ambiente privo di versioni precedenti.

**Caratteristiche**:

- Creazione di database completamente nuovo e vuoto
- Configurazione iniziale del sistema da zero
- Nessun dato preesistente da migrare
- Setup più veloce e lineare

**Versioni di riferimento**:

- Posware Frontend 4.3.x
- MySQL 5.6 (obbligatorio per nuove installazioni)

!!! tip "Database di partenza pre-configurato"
    In caso di nuova installazione, è possibile utilizzare un database di una versione Posware 4.2.306 o superiore come base, a condizione che il database sia dumpato e ripristinato su MySQL 5.6 **prima** di lanciare il setup.
    [Consulta la procedura dettagliata per maggiori dettagli](./installazione-posware-frontend/setup-posware.md)..

**Procedura consigliata**:

1. Verificate i requisiti di sistema
2. Installate tutte le dipendenze (.NET Framework 4.8, MySQL 5.6)
3. Eseguite SetupPosware
4. Installate lo StoreServer, associate la cassa ed inviate l'anagrafica iniziale

Per la procedura dettagliata, consultate:

- [Posware - Overview](./installazione-posware-frontend/overview-generale.md)
- [SetupPosware - Installazione ex-novo](./installazione-posware-frontend/setup-posware.md)
- [Storeserver - Overview](./installazione-storeserver/overview-generale.md)
- [Storeserver - Installazione ex-novo](./installazione-storeserver/installazione-nuovo-storeserver.md)

---

### Scenario 2: Migrazione standard da Posware 4.2 a Posware 4.3

**Situazione**: Possedete già un'installazione funzionante di Posware 4.2 e desiderate eseguire l'upgrade a Posware 4.3 su **tutti i terminali cassa del punto vendita**.

**Caratteristiche**:

- Migrazione dei dati e della configurazione esistente
- Database migrato e aggiornato allo schema 4.3
- **Il database una volta aggiornato non sarà più compatibile con Posware 4.2. Il backup è cruciale in caso di rollback.**

**Versioni supportate**:

- Versione minima richiesta: Posware 4.2.306
- MySQL 4.1 o 5.6 (dipende dalla versione in uso)

!!! danger "Punto Critico"
    Questa migrazione è **non reversibile**. Una volta aggiornato allo schema 4.3, il database non potrà tornare a Posware 4.2. Eseguite backup completi prima di procedere.

**Prerequisiti obbligatori**:

- Backup integrale del database `cassa` e di tutta la cartella di Posware
- Versione di Posware installata non inferiore a 4.2.306
- **Tutti i record della tabella `logcassa` devono essere stati inviati al server** (nessun dato pendente)
- MySQL avviato e accessibile con il database `cassa` presente

!!! warning "Dati Pendenti"
    Se nella tabella `logcassa` sono presenti record non ancora inviati al server, il setup non eseguirà proprio l'installazione. In tal caso, riavviate Posware e attendete il completamento della trasmissione prima di ritentare.

**Procedura consigliata**:

1. Effettuate backup completi (database + cartella di installazione)
2. Verificate la versione di Posware installata
3. Controllate che non vi siano dati pendenti in `logcassa`
4. Installate le dipendenze aggiornate
5. Eseguite SetupPosware con modalità migrazione
6. Verificate il corretto funzionamento post-upgrade
7. Installate Posware StoreServer ed associate la cassa

Per le procedure dettagliate, consultate:

- [Posware - Overview](./installazione-posware-frontend/overview-generale.md)
- [SetupPosware - Migrazione Standard](./installazione-posware-frontend/setup-posware.md#migrazione-dalla-versione-42-alla-43).
- [Storeserver - Overview](./installazione-storeserver/overview-generale.md)
- [Storeserver - Installazione ex-novo](./installazione-storeserver/installazione-nuovo-storeserver.md)

---

### Scenario 3: Migrazione ibrida Posware 4.2 a Posware 4.3

**Situazione**: Dovete migrare progressivamente da Posware `4.2` a `4.3`, mantenendo **contemporaneamente operative casse con versione 4.2 e casse con versione 4.3**.

Questo scenario è tipico di punti vendita ove si riscontrano una o più esigenze quali:

- Si vuole rispettare un periodo di collaudo su un numero limitato di casse prima di procedere all'aggiornamento completo
- Si hanno un numero elevato di casse da migrare tale per cui non si concluderebbe la migrazione in un singolo giorno.
- Si deve ridurre al minimo l'impatto sull'operatività durante la migrazione

**Caratteristiche**:

- Coesistenza controllata di Posware `4.2` e `4.3` nello stesso ambiente
- Migrazione graduale e pianificata terminale per terminale
- Mantenimento della continuità operativa
- StoreServer e Pos_Log per la coesistenza dei dati

**Architettura Supportata**:

- **Casse locali 4.2**: Continuano a operare con database locale esistente e versione invariata di MySql.
- **Casse locali 4.3**: Operano con database locale migrato su MySQL 5.6. Qualora non si voglia aggiornare l'istanza database, **è possibile rimanere sul MySQL esistente, compreso il 4.1.**
- **StoreServer**: Gestisce tutte le casse versione `4.3+`
- **Pos_Log**: Continua a ricevere dati da tutte le casse versione `4.2`

**Prerequisiti obbligatori**:

- Backup completi di tutti i sistemi interessati
- Connettività di rete tra casse e server centrale
- Versione di Posware 4.2 installata non inferiore a **`4.2.1483` se il database del server di barriera è MySql**.

**Procedura di migrazione**:

=== "**Fase 1**: Casse"

    1. Individuate le casse da migrare alla v4.3
    - Effettuate backup completi (database + cartella di installazione) di queste casse
    - Escludete le casse dalla vendita (fate usare agli operatori altre casse) in modo da non generare altri dati log
    - Assicuratevi di aver trasmesso tutto il log di cassa (tabella logcassa) per tutte le casse da migrare
    - Verificate la versione installata di Posware 4.2 sulle casse da migrare. Deve essere almeno la minima supportata per la migrazione.
    - Aggiornate a Posware 4.3 utilizzando SetupPosware, verrà effettuata la migrazione. 
    - Verificate il corretto funzionamento post-upgrade

    !!! tip "Aggiornate una cassa per volta"
        **Eseguite i punti 6 e 7 una cassa per volta.**
        Prima di aggiornare la cassa successiva, associatela allo StoreServer (vedi fase 2 e 3) e verificate il corretto funzionamento.

=== "**Fase 2**: Server di barriera"
    1. Eseguite i backup del database del server di barriera.
    - Stoppate tutti i servizi sul server di barriera (pos_log, pos_fidelity ecc..)
    - Aggiornate il MySql o SqlServer alla versione supportata
    - Migrate il database usando DatabaseUpgrader
    - Installate lo StoreServer con Winget usando la [**modalità di installazione migrazione**](./installazione-storeserver/migrazione-servizi-legacy.md).

    !!! warning "Non avviate ancora i servizi legacy"
        Prima di far ripartire i servizi legacy, **è necessario aggiornarli per renderli compatibili con la nuova versione di MySql o SqlServer**. 
        
        Continuate con la fase 3.

=== "**Fase 3**: Associate la cassa"
    1. Associate la cassa allo StoreServer dalla Dashboard casse. **La cassa deve essere aperta e Posware avviato sulla schermata di login**.
    - Una volta associata, riavviate la cassa
    - Verificate la trasmissione dei dati nella tabella log, troverete i record di apertura programma. È possibile effettuare una transazione fiscale da annullare e monitorare i record ricevuti in tabella log.
    - Se la cassa inizia subito la vendita, è possibile anche verificare l'andamento tramite le statistiche dello StoreServer

=== "**Fase 4**: Servizi legacy"
    Aggiornate i servizi legacy (pos_log, pos_fidelity ecc..) all'ultima versione così da poter ripristinare l'operatività completa delle casse `v4.2`

    1. Effettuate un backup della cartella C:\Posware
    - Avviate il setup dei servizi legacy
    - Al termine dell'installazione, non tutti i servizi stoppati vengono avviati. Avviare manualmente quelli utili.

=== "**Fase 5**: Adeguamento casse 4.2"
    Dopo l'installazione dello StoreServer e dei servizi legacy aggiornati, Pos_Log risponderà sulla porta **6951**

    - **Se il database del server è MySQL, sarà necessario aggiornare le casse v4.2 alla versione `4.2.1483` o superiore che consente la lettura ed invio dei dati con MySql 8.4.**. Non è necessario aggiornare le casse v.4.2 se il database del server è SqlServer
    - Riconfigurate la trasmissione del log delle casse `v4.2`. Nella tabella `config_log` cambiate la porta a **`6951`**
    - Riavviate le casse v4.2 per applicare le modifiche alla trasmissione log

=== "Fase 6: Migrazioni rimanenti"

    - Selezionate le casse in sequenza pianificata
    - Effettuate backup prima di ogni migrazione
    - Testate dopo ogni aggiornamento
    - Monitorate l'impatto operativo

---

Per le procedure dettagliate, consultate:

- [Posware - Overview](./installazione-posware-frontend/overview-generale.md)
- [SetupPosware - Migrazione Standard](./installazione-posware-frontend/setup-posware.md#migrazione-dalla-versione-42-alla-43).
- [Storeserver - Overview](./installazione-storeserver/overview-generale.md)
- [Storeserver - Installazione ex-novo](./installazione-storeserver/installazione-nuovo-storeserver.md)

---

#### Gestione della Continuità Operativa

Durante la migrazione ibrida:

- Le casse 4.2 continueranno a operare normalmente e indipendentemente
- Le casse 4.3 trasmetteranno dati allo StoreServer
- Non sarà necessario alcun intervento di sincronizzazione dei dati tra versioni diverse
- Sarà possibile operare in modalità completamente autonoma per giorni finché la migrazione non è completa
- Le casse SCO continueranno ad operare sulla versione 4.2 fino al rilascio di una nuova release del produttore NCR o Diebold

---

#### Limitazioni e considerazioni sulla migrazione ibrida

- Le casse 4.2 **non possono associarsi allo StoreServer** e continueranno a trasmettere ai servizi legacy
- È necessario mantenere attivo il sistema di **barriera legacy** per le casse 4.2
- Le statistiche dello StoreServer non mostreranno i dati delle casse 4.2.
- **Per continua a consultare i dati di vendita delle casse 4.2, utilizzare Pos_Insta**

---

## Tabella Comparativa dei Tre Scenari

| Aspetto | Installazione Ex-novo | Migrazione Standard | Migrazione Ibrida |
|--------|---------|-------------------|------------------|
| **Downtime** | N/A | Medio (migrazione DB server di barriera e cassa) | Medio (migrazione DB server di barriera e casse selezionate) |
| **Complessità** | Bassa | Media | Alta |
| **Tempo realizzazione** | In giornata | In giornata | Giorni o settimane |
| **Posware 4.2 attive** | N/A | No | Sì (temporaneamente) |
| **Posware 4.3 attive** | Sì | Sì | Sì |
| **StoreServer** | Sì (obbligatorio) | Sì (obbligatorio) | Sì (obbligatorio) |
| **Reversibilità** | N/A | No (backup necessario) | Limitata (in base alle casse aggiornate alla 4.3) |
| **Produttività operativa** | N/A | Compromessa durante migrazione del database server | Compromessa durante migrazione del database server |

---

## Decisione: Quale Scenario Scegliere?

### Scegliete **Installazione ex-novo** se:
✅ State installando Posware per la prima volta  
✅ Non avete dati preesistenti da migrare  
✅ State installando un nuovo punto vendita  
✅ Desiderate partire con una configurazione pulita

### Scegliete **Migrazione Standard** se:
✅ Avete già una versione Posware 4.2 funzionante  
✅ Volete eseguire l'aggiornamento a 4.3 su tutto il punto vendita  
✅ Potete programmare un downtime medio (alcune ore)  
✅ Desiderate preservare tutta la configurazione e i dati esistenti

### Scegliete **Migrazione Ibrida** se:
✅ State gestendo un punto vendita con molte casse  
✅ Non potete permettervi downtime prolungati  
✅ Volete testare Posware 4.3 su pochi terminali prima di un rollout completo o desiderate una transizione graduale e controllata  
✅ Nel punto vendita sono presenti casse automatiche Self-Checkout

---

## Prossimi Passi

Una volta identificato lo scenario appropriato:

1. **Consultate la pagina Requisiti**: Verificate hardware e software minimi per il vostro scenario
2. **Leggete la guida specifica**: Approfondite i dettagli della procedura scelta
3. **Preparate l'ambiente**: Installate le dipendenze e i prerequisiti
4. **Eseguite l'installazione**: Seguite passo-passo le istruzioni fornite

### Link Rapidi

| Scenario | Link |
|----------|------|
| Requisiti di Sistema Posware | Vedi [requisiti](./installazione-posware-frontend/overview-generale.md#requisiti-minimi) e [Dipendenze e aggiornamenti](./installazione-posware-frontend/dipendenze-aggiornamenti-necessari.md)|
| Installazione ex-novo | [SetupPosware - Nuova Installazione](./installazione-posware-frontend/setup-posware.md)|
| Migrazione Standard | [SetupPosware - Migrazione](./installazione-posware-frontend/setup-posware.md#migrazione-dalla-versione-42-alla-43) |
| StoreServer | [Installazione StoreServer](./installazione-storeserver/overview-generale.md) |

---

## Supporto Tecnico

Se durante il processo di installazione riscontrate difficoltà:

1. Consultate i troubleshooting nelle procedure dettagliate per i problemi più frequenti
2. Esaminate la sezione FAQ nelle procedure dettagliate per domande comuni
3. Contattate il supporto tecnico con i file di log disponibili in `C:\Programmi (x86)\Posware\` (per SetupPosware) o nella cartella di installazione dello StoreServer

---

## Glossario Rapido

**Posware Frontend**: L'applicazione principale che opera sui terminali cassa  
**StoreServer**: Sistema centralizzato per gestione e analisi dei dati (v. 4.3+)  
**SetupPosware**: Installer per nuove installazioni e migrazioni da 4.2 a 4.3  
**Database Provider**: Sistema di database (MySQL, SQL Server) che archivia i dati  
**Scenario Ibrido**: Coesistenza di Posware 4.2 e 4.3 nello stesso ambiente  

---

*Ultima modifica: {git-revision-date-localized}*