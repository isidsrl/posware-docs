---
tags:
    - DatabaseUpgrader
    - StoreServer
    - Migrazione
title: DatabaseUpgrader - Guida utente
description: Guida per la migrazione del database Posware dalla versione 4.0/4.2 alla versione 4.3+
---

# DatabaseUpgrader - Guida utente

**Prima revisione documento: 6 maggio 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**

---

## Introduzione

Il **Database Upgrader** è un'applicazione Windows che permette di migrare il database `posware` dalla versione :material-tag:`4.0`/:material-tag:`4.2` alla versione :material-tag:`4.3`+, rendendolo compatibile con la nuova architettura dello *StoreServer*.

Il tool supporta entrambi i provider di database:

- **MySQL** :material-tag:`8.4`
- **SQL Server** :material-tag:`2016` / :material-tag:`2017` / :material-tag:`2019` / :material-tag:`2022`

!!! danger "Backup obbligatorio prima di procedere"
    Prima di avviare qualsiasi operazione con il Database Upgrader, è **obbligatorio** effettuare un backup completo e verificato del database di produzione.

    **Nessuna operazione deve essere eseguita senza un backup integro disponibile.**

---

## Prerequisiti e preparazione

Prima di usare il Database Upgrader, verificare di disporre di:

- **Backup completo e verificato** del database `posware` corrente
- **Accesso al filesystem** della directory in cui è installato il Database Upgrader
- **Permessi sul database**: l'utente nella connection string deve avere permessi di lettura, scrittura, creazione e rinomina dei database

=== "MySQL"

    !!! warning "Operazione preliminare obbligatoria"
        Rinominare il database sorgente con un nome diverso da `posware` **prima** di avviare il Database Upgrader.

        Poiché MySQL non permette la rinomina diretta dei database, la procedura è:

        1. Eseguire un dump del database `posware`
        2. Creare un nuovo database con il nome desiderato (es. `posware_v42`)
        3. Importare il dump nel nuovo database
        4. Eliminare il database originale `posware`

        Se il database sorgente mantiene il nome `posware` al momento della migrazione, il Database Upgrader assegnerà al database migrato un nome casuale. In tal caso sarà necessario un intervento manuale aggiuntivo al termine — vedere [Cosa succede al database dopo la migrazione](#cosa-succede-al-database-dopo-la-migrazione).

=== "SQL Server"

    !!! info "Rinomina automatica"
        Con SQL Server la rinomina del database sorgente è gestita automaticamente dal tool al termine della migrazione. Non è richiesta alcuna operazione preliminare di rinomina.

---

## Configurazione

Il Database Upgrader legge due file di configurazione nella sua directory di installazione.

### connectionStrings.Production.json

Contiene la stringa di connessione al database sorgente da migrare.

=== "MySQL"

    ```json title="connectionStrings.Production.json"
    {
      "ConnectionStrings": {
        "posware_mysql": "Server=localhost;Port=3306;Database=nome_database_sorgente;User Id=utente;Password=password;",
        "posware_sqlserver": ""
      }
    }
    ```

=== "SQL Server"

    ```json title="connectionStrings.Production.json"
    {
      "ConnectionStrings": {
        "posware_mysql": "",
        "posware_sqlserver": "Server=localhost;Database=nome_database_sorgente;User Id=utente;Password=password;"
      }
    }
    ```

!!! info "Quale provider viene utilizzato"
    Il tool usa la prima stringa di connessione non vuota. Se entrambe sono valorizzate, viene usata MySQL. Lasciare **vuota** la voce del provider non utilizzato.

### databaseUpgraderSettings.json

Controlla la quantità di dati storici da importare nel nuovo database.

```json title="databaseUpgraderSettings.json"
{
  "MigrationOptions": {
    "MonthsToImport": 6,
    "MonthsToImportForRistampaScontrini": 1
  }
}
```

| Campo | Descrizione | Valore default |
|-------|-------------|----------------|
| `MonthsToImport` | Mesi di storico da importare per log, scontrini e dati di vendita. Il mese corrente è incluso nel conteggio. | `6` |
| `MonthsToImportForRistampaScontrini` | Mesi di storico da importare per le ristampe degli scontrini. Il mese corrente è incluso nel conteggio. | `1` |

!!! tip "Come scegliere i valori"
    Un valore più alto significa più dati migrati e tempi di migrazione più lunghi. Scegliere in base alle esigenze operative del punto vendita, tenendo in considerazione che i dati al di fuori della finestra temporale impostata non verranno migrati.

### Esclusione tabelle dalla migrazione dati

Per escludere determinate tabelle dalla migrazione dei dati (la struttura viene comunque creata nel nuovo database), aggiungere la lista dei nomi tabella in `databaseUpgraderSettings.json`:

```json title="databaseUpgraderSettings.json"
{
  "MigrationOptions": {
    "MonthsToImport": 6,
    "MonthsToImportForRistampaScontrini": 1,
    "TablesToExcludeFromDataMigration": ["articoli", "log", "scontrini"]
  }
}
```

| Campo | Descrizione |
|-------|-------------|
| `TablesToExcludeFromDataMigration` | Array di nomi tabella da escludere dalla migrazione dati. La struttura delle tabelle viene creata, ma i dati non vengono copiati. |

!!! tip "Quando usare questa opzione"
    Utile quando alcune tabelle sono troppo grandi per essere migrate automaticamente dal tool.
    I dati potranno essere migrati manualmente con script SQL ottimizzati dopo il completamento della migrazione.

!!! info "Comportamento"
    - I nomi tabella sono case-insensitive (`articoli`, `ARTICOLI`, `Articoli` sono equivalenti)
    - Tabelle inesistenti nel database sorgente vengono ignorate silenziosamente
    - L'esclusione via configurazione si aggiunge alle esclusioni predefinite del tool, non le sostituisce

---

## Avvio e procedura passo per passo

### Passo 1 – Avvio dell'applicazione

Avviare il file `DatabaseUpgrader.exe` dalla directory di installazione.

Si aprirà la finestra **"Posware Database Upgrader"**.

!!! danger "Non chiudere la finestra durante la migrazione"
    Una volta avviata la migrazione, **non è possibile chiudere l'applicazione fino al completamento**. Il tentativo di chiusura viene bloccato dal sistema con il messaggio:

    > *"Migrazione in corso, non è possibile chiudere l'applicazione prima del termine."*

### Passo 2 – Rilevamento versione

All'avvio, il tool rileva automaticamente la versione del database configurato nella connection string (attesa di circa 2 secondi).

**Messaggi attesi se il database deve essere aggiornato:**

=== "MySQL"

    > *Upgrade database MySql da versione 4.0 a 4.3 necessario.*  
    > *Effettuare la migrazione.*

=== "SQL Server"

    > *Upgrade database SqlServer da versione 4.0 a 4.3 necessario.*  
    > *Effettuare la migrazione.*

Se appare invece un messaggio di errore, consultare la sezione [Risoluzione problemi](#risoluzione-problemi).

### Passo 3 – Preparazione migrazione

Premere il pulsante **"Prepara migrazione"**.

Il tool esegue automaticamente le seguenti operazioni:

1. Creazione di un database temporaneo con la struttura della versione :material-tag:`4.3`
2. Analisi delle tabelle presenti nel database sorgente
3. Costruzione della lista di tabelle da migrare e di quelle escluse dalla migrazione

Al termine dell'analisi, viene mostrata una griglia con l'elenco delle tabelle individuate.

### Passo 4 – Revisione dell'analisi

La griglia mostra le tabelle con quattro colonne:

| Colonna | Descrizione |
|---------|-------------|
| **Stato** | `Da importare` — tabella in attesa di migrazione |
| **Tabella** | Nome della tabella nel database sorgente |
| **Warning** | Eventuali avvertimenti pre-migrazione |
| **Risultato migrazione** | Vuoto in questa fase; si popola durante la migrazione |

Verificare che le tabelle attese siano presenti nell'elenco, quindi procedere al passo successivo.

### Passo 5 – Avvio migrazione

Premere il pulsante **"Avvia migrazione"**.

Il tool migra le tabelle in sequenza aggiornando in tempo reale le colonne **Stato** e **Risultato migrazione** nella griglia.

!!! warning "Attendere il completamento"
    Non interrompere l'operazione. La durata dipende dalla quantità di dati da trasferire e dalla finestra temporale configurata in `databaseUpgraderSettings.json`.

### Passo 6 – Completamento

Al termine della migrazione compare il pulsante **"Chiudi"**.

- **Senza warning:** premere **"Chiudi"** per uscire dall'applicazione.
- **Con warning:** premere **"Chiudi"** apre automaticamente il file di log per la revisione.

Per interpretare i messaggi nella griglia, consultare la sezione [Interpretare i risultati](#interpretare-i-risultati-della-migrazione).

---

## Cosa succede al database dopo la migrazione

=== "SQL Server"

    Il tool gestisce automaticamente la sostituzione del database al termine della migrazione:

    1. Il database sorgente viene messo in modalità `SINGLE_USER`
    2. Il database sorgente viene rinominato in **`{nome_originale}_v4`** (backup automatico)
    3. Il database migrato viene rinominato con il nome originale
    4. Il database viene riportato in modalità `MULTI_USER`

    **Il database `{nome}_v4` rimane come backup** e può essere eliminato manualmente dopo aver verificato il corretto funzionamento dello *StoreServer*.

    In caso di rollback, è sufficiente rinominare manualmente i due database invertendo i nomi.

=== "MySQL"

    Con MySQL il tool **non rinomina automaticamente il database**. Al termine della migrazione:

    - Il database sorgente rimane invariato con il suo nome
    - Il database migrato viene creato con il nome `posware` se il database sorgente era stato precedentemente rinominato (vedere [Prerequisiti e preparazione](#prerequisiti-e-preparazione))

    !!! danger "Se il database migrato ha un nome casuale"
        Se al termine della migrazione il database migrato ha un nome casuale (es. `posware_a3f2b1`), significa che il database sorgente non era stato rinominato prima dell'avvio.

        In tal caso è **obbligatorio** rinominare il database migrato con il nome `posware` prima di procedere con l'installazione dello *StoreServer*:

        1. Eseguire un dump del database con il nome casuale
        2. Creare un nuovo database chiamato `posware`
        3. Importare il dump nel database `posware`
        4. Eliminare il database con il nome casuale

---

## Interpretare i risultati della migrazione

Al termine della migrazione, ogni riga nella griglia riporta lo stato finale nella colonna **Risultato migrazione**.

### Messaggi di successo

| Messaggio | Significato | Azione |
|-----------|-------------|--------|
| `Migrazione completata. Record migrati N.` | Tabella migrata con successo; N record importati. | Nessuna |
| `Migrazione completata. Tutti i record migrati.` | Tutti i record della tabella sono stati importati. | Nessuna |
| `Migrazione completata. Migrazione record non necessaria.` | La tabella è esclusa dalla migrazione dei dati (struttura migrata, dati no). Comportamento normale per alcune tabelle di sistema. | Nessuna |
| `Migrazione completata. Tabella vuota, nessun record da importare.` | La tabella sorgente era vuota. | Nessuna |
| `Migrazione completata. Dati non migrati (tabella inserita dal tecnico in lista esclusioni).` | Tabella esclusa dalla migrazione dati tramite configurazione `TablesToExcludeFromDataMigration`. La struttura è stata creata ma i dati non sono stati copiati. | Migrare i dati manualmente se necessario |

### Warning

!!! warning "I campi fuori standard non sono stati migrati"
    Il database sorgente contiene colonne aggiuntive non presenti nello schema standard :material-tag:`4.3`. Queste colonne non vengono migrate.

    **Cause possibili:**

    - Colonne aggiunte da personalizzazioni di terze parti
    - Colonne presenti in versioni precedenti di Posware e successivamente rimosse

    **Azione richiesta:** Consultare il log per identificare le colonne interessate e verificare che nessuna colonna fondamentale sia stata esclusa per errore. In caso contrario, aprire una segnalazione.

!!! warning "Alcuni record duplicati o con errori non sono stati migrati"
    Durante la migrazione sono stati trovati record duplicati o con errori di integrità referenziale che non è stato possibile importare.

    **Causa comune:** La tabella sorgente non aveva chiavi primarie; la versione :material-tag:`4.3` le introduce, rendendo alcuni record incompatibili.

    **Azione richiesta:** Consultare il log per individuare i record saltati. Valutare se migrarli manualmente oppure se il record eliminato fosse già errato tra i duplicati.

    !!! example "Tabella frequentemente interessata"
        La tabella `isi_menu` è una delle più soggette a questo problema.

---

## File di log

Il Database Upgrader produce un file di log nella propria directory di installazione.

**Nome del file:** `dbupgrader-log_YYYY-MM-DD.log`

**Come accedere al log:**

- Al termine della migrazione con warning, il log si apre automaticamente al click su **"Chiudi"**
- In alternativa, aprire manualmente il file dalla directory di installazione del tool

**Formato delle righe di log:**

```
timestamp|logger|LEVEL|messaggio
```

**Esempio di riga:**

```
2026-05-06 10:23:14.5678|DatabaseUpgrader.Domain.Db40To43MigratorBase|INFO|Starting migration of table scontrini
```

!!! info "Archivio automatico dei log"
    I log vengono archiviati automaticamente con compressione. Vengono conservati fino a 30 file. Se il log del giorno supera i 10 MB, viene archiviato automaticamente e ne viene creato uno nuovo.

---

## Risoluzione problemi

### Nessuna stringa di connessione valida trovata

**Messaggio:** *"Nessuna stringa di connessione valida trovata nel file connectionStrings.Production.json."*

!!! failure "Causa e soluzione"
    Il file `connectionStrings.Production.json` è assente, vuoto o contiene entrambe le voci vuote.

    **Verificare:**

    1. Che il file `connectionStrings.Production.json` esista nella directory del Database Upgrader
    2. Che la connection string del provider in uso (MySQL o SQL Server) sia valorizzata
    3. Che il nome della chiave sia esattamente `posware_mysql` oppure `posware_sqlserver`

### Impossibile rilevare la versione del database

**Messaggio:** *"Impossibile rilevare la versione del database."*

!!! failure "Causa e soluzione"
    Il tool non riesce a connettersi al database o la versione rilevata non è supportata.

    **Verificare:**

    1. Che il server database sia avviato e raggiungibile dall'host in cui gira il Database Upgrader
    2. Che i parametri della connection string (host, porta, nome database, credenziali) siano corretti
    3. Che l'utente del database abbia i permessi necessari per leggere lo schema (`INFORMATION_SCHEMA` / tabelle di sistema)
    4. Che il database non sia già alla versione :material-tag:`4.3`+ (in tal caso non è necessaria alcuna migrazione)

### Errore nel processo durante l'analisi

**Messaggio:** *"Errore nel processo. Consultare il log per maggiori dettagli."*

!!! failure "Causa e soluzione"
    Si è verificato un errore durante la fase di analisi o durante la creazione del database temporaneo.

    **Azioni:**

    1. Aprire il file di log (`dbupgrader-log_YYYY-MM-DD.log`) nella directory del Database Upgrader
    2. Cercare le righe con livello `ERROR` per identificare la causa specifica
    3. Cause comuni: permessi insufficienti per creare nuovi database, spazio disco insufficiente, connessione al database interrotta durante l'operazione

### Impossibile chiudere l'applicazione durante la migrazione

**Comportamento:** La finestra non si chiude; appare il messaggio *"Migrazione in corso, non è possibile chiudere l'applicazione prima del termine."*

!!! info "Comportamento previsto"
    Questo è il comportamento corretto del tool. La chiusura viene bloccata per evitare che la migrazione venga interrotta in uno stato inconsistente, che potrebbe corrompere il database temporaneo.

    Attendere il completamento della migrazione e premere **"Chiudi"** quando il pulsante diventa disponibile.

### MySQL: database migrato con nome casuale

**Situazione:** Al termine della migrazione, il database creato dal tool ha un nome casuale anziché `posware`.

!!! failure "Causa e soluzione"
    Il database sorgente aveva ancora il nome `posware` al momento dell'avvio del tool, pertanto il database migrato ha ricevuto un nome temporaneo.

    **Soluzione obbligatoria:**

    1. Eseguire un dump del database con nome casuale
    2. Creare un nuovo database con nome `posware`
    3. Importare il dump nel database `posware`
    4. Eliminare il database con il nome casuale

---

## Riferimenti

- [Migrazione da servizi legacy – procedura completa di installazione StoreServer](../getting-started/installazione-storeserver/migrazione-servizi-legacy.md)
- [Download Database Upgrader](../download/index.md)
