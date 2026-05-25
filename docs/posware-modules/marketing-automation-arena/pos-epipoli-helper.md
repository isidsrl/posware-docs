---
tags:
    - PosEpipoliHelper
    - Gruppo Arena
    - Marketing Automation
    - Epipoli
---

# PosEpipoliHelper - Servizio di ETL e di invio del venduto

**Prima revisione documento: 24 marzo 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**

!!! warning "DOCUMENTO PRELIMINARE"
    Le informazioni e i dettagli contenuti in questo documento possono essere soggetti a modifiche e non costituiscono un protocollo definitivo.

## Introduzione 

Il **PosEpipoliHelper** è un servizio dell'ecosistema Posware progettato per gestire l'integrazione dati con la piattaforma loyalty Epipoli, per il rinnovamento relativo alla **Marketing Automation** del cliente **Gruppo Arena**.

Si tratta di un applicativo sviluppato in **.NET 10** che agisce come un Windows Service in background.<br>
La sua architettura è progettata per eseguire due funzionalità distinte, a seconda del parametro passato in fase di avvio:

1. **Modalità ETL (`--etl`):** Si occupa del download periodico, della validazione e dell'importazione locale delle promozioni e delle fidelity card dal bucket AWS S3 di Epipoli.
2. **Modalità Kafka Producer (`--kafka-producer`):** Agisce come intermediario (tipicamente dal server centralizzato CEDI), ricevendo i dati del venduto dalle casse tramite API REST e inoltrandoli ai sistemi Epipoli tramite una coda Kafka REST Proxy.

Le due modalità possono essere attive contemporaneamente sullo stesso nodo (server CEDI) oppure separatamente (server PDV).

Di seguito è illustrato il diagramma logico dei flussi di dati:

```
AWS S3 ──NDJSON──► ETL ──► SQL Server ◄── POS (PUT /api/extsell)
                                │
                                └──► Kafka Producer ──► Epipoli REST Proxy
```

Lo schema evidenzia come il database SQL Server locale funga da crocevia centrale: da un lato riceve le anagrafiche loyalty scaricate da AWS S3 tramite il **modulo ETL**, dall'altro raccoglie le transazioni inviate dalle casse, per poi inoltrarle a Epipoli tramite il modulo **Kafka Producer**.

---

## Requisiti di sistema

Per garantire il corretto funzionamento del servizio, l'infrastruttura ospitante deve rispettare i seguenti requisiti minimi:

| Requisito | Dettaglio |
|-----------|-----------|
| Sistema operativo | Windows 10 x64 Build 17763+ (Redstone 5) o Windows 11 qualsiasi versione |
| Runtime .NET | .NET 10 Hosting Bundle (installato automaticamente dal setup) |
| Database | SQL Server 2014 o superiore |
| Connettività uscente | AWS S3 `eu-central-1`, Kafka REST Proxy (URL configurabile), SQL Server |
| Privilegi | Amministratore locale per installazione ex-novo; non richiesto per l'aggiornamento |

---

## PosEpipoliHelperSetup

### Nome file e scenari d'uso

L'eseguibile ha il formato `PosEpipoliHelperSetup_{Versione}.exe`.<br>
Il programma supporta **sia l'installazione ex-novo sia lo scenario di aggiornamento** (il servizio rileva in automatico la presenza del servizio).

Nello scenario di installazione ex-novo è obbligatorio avviare l'applicativo tramite shell (prompt dei comandi o PowerShell è indifferente), fornendo alcuni parametri specifici di questo servizio.<br>
Nello scenario di aggiornamento, invece, l'utente può decidere se lanciare il setup facendo ordinario doppio click sull'eseguibile oppure avviare l'applicativo tramite shell.

#### Parametri da riga di comando

I parametri specifici del setup del **PosEpipoliHelper** sono:

| Parametro | Esempio | Note |
|-----------|---------|------|
| `/SERVICEARGS` | `--etl --kafka-producer` | Argomenti del Windows Service. Default: `--etl --kafka-producer` |
| `/CONNECTIONSTRING` | `Server=localhost;Database=pos;...` | **Obbligatoria** per installazione ex-novo. Connection string SQL Server scritta in `appsettings.Production.json`. Per l'aggiornamento viene letta automaticamente dal file esistente — il parametro è ignorato. |

Oltre a quelli del servizio, inoltre, è possibile fornire anche alcuni dei parametri standard previsti da **Inno Setup**:

- `/SILENT`, `/VERYSILENT`
- `/LOG="filename"`
- `/NOCANCEL`

Per la lista completa dei parametri standard previsti da **Inno Setup**, così come per sapere cosa fanno quelli soprastanti, consultare la guida ufficiale di **Inno Setup**.

Di seguito un esempio completo di comando di installazione con tutti i parametri specifici del setup, ma senza parametri di **Inno Setup**:

```cmd title="Esempio installazione completa"
PosEpipoliHelperSetup_1.0.exe /SERVICEARGS="--etl" /CONNECTIONSTRING="Server=sql01;Database=posdb;Integrated Security=True"
```

### Flusso di installazione

Il processo di installazione tramite **Inno Setup** è progettato per essere completamente automatizzato e sicuro, gestendo in autonomia i pre-requisiti di sistema, la validazione delle configurazioni critiche e il ciclo di vita del servizio. Di seguito vengono descritte le fasi sequenziali eseguite dal setup durante l'installazione:

1. **Controllo dei privilegi:** il setup richiede diritti di amministratore in caso di installazione ex-novo, mentre per gli aggiornamenti non sono richiesti privilegi elevati.
2. **Stop del servizio esistente:** se il servizio `PosEpipoliHelper` è in esecuzione, viene fermato prima di procedere per evitare blocchi sui file.
3. **Verifica della Connection String (pre-estrazione file):** per installazione ex-novo controlla che `/CONNECTIONSTRING` sia presente; per l'aggiornamento verifica che sia valorizzata nel file `appsettings.Production.json` esistente. Se assente o vuota, in entrambi i casi, il setup si interrompe immediatamente con una modal di errore **prima di scrivere qualsiasi file**, per prevenire installazioni corrotte.
4. **Verifica/installazione .NET 10 Hosting Bundle:** controlla la presenza del runtime e se è assente, lo scarica e installa automaticamente. In caso di riavvio richiesto dal runtime, il setup lo gestisce.
5. **Copia file:** vengono copiati i vari file del servizio all'interno della directory `%LOCALAPPDATA%\isid\PosEpipoliHelper`.
6. **Scrittura connection string:** solo ed esclusivamente in caso di installazione ex-novo, scrive la connection string in `appsettings.Production.json`. In caso di aggiornamento il file esistente viene preservato.
7. **Registrazione e avvio del servizio Windows:** il sistema esegue i comandi per la creazione del servizio Windows (`sc.exe create PosEpipoliHelper`) e ne forza l'avvio automatico con privilegi di `LocalSystem` (`sc.exe start PosEpipoliHelper`).

!!! note "Comportamento nello scenario di aggiornamento"
    Nello scenario di aggiornamento, i file binari (`.exe`, `.dll`) vengono sempre sovrascritti, mentre il file `appsettings.Production.json` viene **preservato**.<br>
    Inoltre, anche la cartella `etl\archive\` e i log sono mantenuti.

### Directory di installazione

Al termine del processo di setup, i file binari, le configurazioni e le cartelle operative del servizio vengono posizionati nel percorso `%LOCALAPPDATA%\isid\PosEpipoliHelper`.

Di seguito è illustrata la struttura gerarchica della directory, utile ai tecnici per l'individuazione rapida dei file di log, degli archivi e dei parametri di configurazione:

```
%LOCALAPPDATA%\isid\PosEpipoliHelper\
├── PosEpipoliHelper.Api.exe      ← Eseguibile principale del servizio Windows
├── appsettings.Production.json   ← File di configurazione principale (richiede valorizzazione manuale della ConnectionString)
├── *.dll                         ← Librerie e dipendenze di sistema (.NET 10)
├── etl\
│   └── archive\                  ← Archivio dei file NDJSON scaricati da S3 e compressi (.zip)
└── logs\                          ← Cartella root per i file di log
    ├── general-log.log               
    ├── etl-yyyy-mm-dd.log            
    ├── etl-session_{ID}_{Data}_{Ora}.log 
    ├── kafka-yyyy-mm-dd.log          
    └── kafka-ext_sell_{INSEGNA}-yyyy-mm-dd.log
```

La struttura e il contenuto dei singoli file di log sono documentati in dettaglio nell'apposita [sezione](#log).

### Disinstallazione

Il programma di disinstallazione propone due opzioni:

- **Mantieni i file modificati (default):** rimuove binari e deregistra il servizio, ma lascia `appsettings.Production.json` e i log.
- **Rimuovi tutto:** elimina anche i file con le impostazioni e i log.

Il servizio Windows viene sempre fermato e deregistrato prima della rimozione dei file.

---

## Configurazione

Le impostazioni principali del PosEpipoliHelper si trovano nel file `appsettings.Production.json` nella directory di installazione.

Al suo interno sono definiti in modo strutturato tutti i parametri operativi che governano sia il ciclo di vita del servizio di **ETL** (come le frequenze di aggiornamento o le logiche di archiviazione), sia le direttive di inoltro del servizio **Kafka Producer** (come l'endpoint REST, le dimensioni dei batch e le politiche di retry).

È fondamentale che i tecnici familiarizzino con queste sezioni: sebbene la maggior parte dei valori sia preimpostata con default ottimali per l'ambiente di produzione, alcuni parametri critici richiedono una verifica o una valorizzazione manuale rigorosa per garantire il corretto avvio e funzionamento dell'applicativo.

Di seguito vengono descritte nel dettaglio le sezioni chiave del file e i relativi parametri.

### ConnectionStrings

Parametro in cui è indicata la connection string al database:

| Chiave | Tipo | Descrizione |
|--------|------|-------------|
| `Default` | string | Connection string SQL Server principale |

### PosEpipoliHelper.Etl

Parametri fondamentali per il servizio di ETL:

| Chiave | Tipo | Default | Descrizione |
|--------|------|---------|-------------|
| `UpdateFrequencyInMinutes` | int | `120` | Intervallo tra cicli ETL, in minuti |
| `Insegna` | string | `"*"` | Fondamentale per il processo di ETL. Può assumere i valori: `deco`, `superconveniente` o `*` (tutte le insegne, tipico del CEDI).<br> *Nota:* Se il parametro non viene letto correttamente, il programma lancia un'eccezione e termina |
| `MaxRetryCount` | int | `3` | Tentativi di download da S3 in caso di errore |
| `BucketName` | string | `arena-isid-techgroove` | Nome del bucket AWS S3 |
| `S3Folder` | string | `output/cards/` | Prefisso (cartella) S3 dei file NDJSON |
| `AwsAccessKeyId` | string | — | Credenziale AWS Access Key ID |
| `AwsSecretKey` | string | — | Credenziale AWS Secret Access Key |
| `AwsRegionEndpoint` | string | `eu-central-1` | Regione AWS del bucket |
| `ArchivePath` | string | `etl\archive` | Percorso relativo alla directory di installazione per i file archiviati |
| `MaxArchiveDays` | int | `7` | Giorni di mantenimento dei file nell'archivio |
| `MaxArchiveFileCount` | int | `168` | Numero massimo di file NDJSON nell'archivio locale (168 = 7 giorni × 24 ore) |
| `MaxSessionLogFiles` | int | `480` | Soglia oltre la quale scatta il cleanup dei log di sessione ETL |
| `SessionLogArchiveInPercent` | int | `10` | Percentuale dei log di sessione più vecchi da comprimere in ZIP al cleanup |

### PosEpipoliHelper.KafkaProducer

Parametri fondamentali per il servizio Kafka Producer:

| Chiave | Tipo | Default | Descrizione |
|--------|------|---------|-------------|
| `Insegna` | string | `"*"` | Fondamentale per il servizio Kafka Producer. Può assumere i valori: `deco`, `superconveniente` o `*` (tutte le insegne, tipico del CEDI).<br> *Nota:* Se il parametro non viene letto correttamente, il programma lancia un'eccezione e termina |
| `RestProxyBaseUrl` | string | `""` | URL base del Kafka REST Proxy |
| `BasicAuthCredentials` | string | `""` | Credenziali in formato Base64 |
| `PollingIntervalInSeconds` | int | `30` | Intervallo di polling della tabella `epipoli_ext_sell`, in secondi |
| `BatchSize` | int | `100` | Numero di record per batch inviato a Kafka |
| `MaxRetryCount` | int | `3` | Numero massimo di tentativi di invio dati a Kafka in caso di errore |

---

## Modalità operative del Servizio Windows

Il **PosEpipoliHelper** è stato progettato per ospitare al suo interno due *Background Service* distinti.<br>
Il comportamento dell'applicativo non è statico, ma viene determinato dinamicamente in base ai parametri che gli vengono passati nel momento in cui il servizio di Windows viene creato.

### Argomenti di avvio

L'attivazione dei moduli interni è governata dai seguenti argomenti:

| Argomento | Effetto |
|-----------|---------|
| `--etl` | Attiva il background service ETL |
| `--kafka-producer` | Attiva il background service Kafka Producer |
| (entrambi) | Attiva entrambe le modalità |
| (nessuno) | Il servizio si avvia e si arresta immediatamente con exit code 1 |

Questi argomenti sono esattamente gli stessi che l'installer **Inno Setup** accetta tramite il parametro custom `/SERVICEARGS` in fase di deploy iniziale.<br>
Quindi, gli argomenti del servizio Windows sono impostati al momento dell'installazione. 

Tuttavia, qualora si rendesse necessario modificare il comportamento del servizio in un secondo momento dopo l'installazione, è necessario deregistrare e riscrivere il servizio, impostando gli argomenti di avvio necessari nel `binPath` del comando di creazione del servizio tramite shell.

```cmd title="Esempio comando creazione"
sc.exe create PosEpipoliHelper binPath="\"C:\Users\...\PosEpipoliHelper.Api.exe\" --etl --kafka-producer" start=auto obj=LocalSystem DisplayName="PosEpipoliHelper"
```

Se vengono specificati entrambi i parametri separati da uno spazio, il servizio istanzierà e farà girare parallelamente entrambi i moduli sulla stessa macchina.

---

## Funzionamento ETL

Il modulo ETL (Extract, Transform, Load) è responsabile dell'allineamento costante tra il repository centralizzato di Epipoli su AWS S3 e i database locali del Gruppo Arena. Il servizio opera in modo asincrono e resiliente, seguendo un workflow strutturato per garantire l'integrità del dato e la continuità operativa anche in caso di instabilità della rete.

### Flusso di una sessione ETL

Per facilitare l'attività di monitoraggio, il processo di sincronizzazione segue un flusso logico rigoroso. Lo schema riassuntivo di seguito descrive il percorso standard di una sessione che si conclude con successo, dalla verifica dell'ultimo ETag S3 salvato fino alla storicizzazione finale:

``` title="Workflow tipico di una sessione ETL"
1. Controllo del DB: ultimo ETag S3 salvato in epipoli_sync_log
2. Consultazione della lista di file su S3 (BucketName / S3Folder)
3. Se l'ETag è invariato → skip (sessione skippata, nessuna scrittura)
4. Download NDJSON + verifica del checksum (algoritmi CRC64/CRC32)
   └─ In caso di errore: retry fino a MaxRetryCount, poi fallimento
5. Parse NDJSON riga per riga
   └─ Verifica del campo "source" del NDJSON == Insegna configurata
   └─ Insegna non corrispondente → ETL si ferma senza retry
6. TRUNCATE tabella epipoli_card_promo_temp
7. Bulk insert nella tabella epipoli_card_promo_temp
8. MERGE da epipoli_card_promo_temp a epipoli_card_promo
9. Scrittura in epipoli_sync_log (Status, ETag, contatori)
10. Archiviazione dei file in etl\archive\ con meccanismo di rotazione
```

### Tabelle coinvolte

L'architettura del database è stata progettata per garantire alte performance di scrittura e una separazione netta tra i dati in fase di elaborazione e quelli pronti per la produzione. Di seguito sono riportate le tabelle coinvolte nel processo ETL con i relativi ruoli:

| Tabella | Ruolo |
|---------|-------|
| `epipoli_card_promo_temp` | Tabella temporanea di appoggio in cui salvare i dati in attesa di mergiarli in `epipoli_card_promo` |
| `epipoli_card_promo` | Dati delle promozioni attive per carta fedeltà |
| `epipoli_loyalty_cards_temp` | Tabella temporanea di appoggio in cui salvare i dati in attesa di mergiarli in `epipoli_loyalty_cards` |
| `epipoli_loyalty_cards` | Dati delle carte fedeltà |
| `epipoli_sync_log` | Storico delle sessioni ETL. È la tabella in cui vengono salvati gli ETag dell'ultimo file NDJSON scaricato |

La struttura e i campi delle singole tabelle sono documentati in dettaglio nell'apposita [sezione](#struttura-tabelle-del-database).

### Trigger e Scheduling

L'attivazione del processo di sincronizzazione può avvenire secondo due diverse modalità, garantendo sia la regolarità operativa che la flessibilità necessaria per interventi immediati:

- **Trigger Automatico:** Il servizio utilizza un timer interno che avvia ciclicamente il processo in base all'intervallo definito dal parametro `UpdateFrequencyInMinutes`.
- **Trigger Manuale:** È possibile forzare un ciclo di sincronizzazione immediato tramite:

    - il relativo pulsante nell'interfaccia grafica consultabile all'indirizzo `http://localhost:5298`
    - una chiamata all'endpoint dedicato `POST /api/etl/trigger`. 
 
    Questa funzionalità è essenziale per i test, in fase di debug o per forzare l'elaborazione delle promozioni in caso di aggiornamenti urgenti da parte di Epipoli.

Ad ogni ciclo, il servizio interroga il bucket S3 per identificare il file `arena_cards_coupons_yyyymmdd.ndjson` più recente basandosi sull'attributo "**Last Modified**" di AWS S3. **Il file può essere aggiornato più volte al giorno, con una frequenza che può arrivare ad un massimo di ogni 15 minuti.**

Per ottimizzare le risorse di rete, il download effettivo avviene solo se viene soddisfatta una delle seguenti condizioni:

- **Variazione ETag:** L'header HTTP `ETag` (hash del contenuto del file) restituito da S3 è differente rispetto a quello dell'ultima sincronizzazione completata con successo.
- **Assenza di dati locali:** Non è presente alcuna cronologia di sincronizzazione nel database locale.

### Download e Verifica di Integrità

Una volta avviato il download, il file `NDJSON` viene memorizzato temporaneamente in una cartella di transito. Data la criticità dei dati (fidelity card e coupon), il sistema esegue una **validazione di integrità obbligatoria** ricalcolando il checksum del file e confrontandolo con i metadati forniti da AWS:

- **Verifica Primaria:** Il sistema cerca l'header `x-amz-checksum-crc64nvme` e confronta il valore (codificato in `base64`) con il calcolo locale CRC64NVME.
- **Fallback:** In assenza dell'header precedente, il sistema utilizza l'header `x-amz-checksum-crc32`.

#### Gestione Errori di Integrità

Se il checksum non coincide, il file viene rimosso e il download viene ripetuto per un numero massimo di tentativi definito in `MaxRetryCount` della sezione `PosEpipoliHelper.Etl`. **Ogni volta che la verifica di integrità fallisce, deve essere inserito un log di livello `ERROR` nel rispettivo file di log.**

Superato tale limite, la sessione viene marcata come `failed` nella tabella `epipoli_sync_log`, registrando nel campo `ErrorMessage` il dettaglio della discrepanza tra i checksum. Inoltre, viene sempre inserito un log di livello `ERROR` nel rispettivo file di log.

### Validazione dell'Insegna

Durante il parsing del file `NDJSON`, il servizio esegue un controllo di coerenza fondamentale: il campo `source` contenuto nel `JSON` deve corrispondere all'**Insegna** configurata nel file `appsettings.json` (*deco*, *superconveniente* o `*`).

- Se l'insegna è configurata come `*` (caso tipico del CEDI), tutte le tessere di tutte le insegne vengono elaborate. Altrimenti vengono elaborate solo le tessere di una o dell'altra insegna. In ogni caso viene sempre riportato nei log il valore letto così da poter aiutare i tecnici in fase di eventuale diagnosi.
- Se viene riscontrata una discrepanza tra la `source` del file e l'insegna del punto vendita, il processo viene interrotto immediatamente con una **Fatal Exception** per prevenire l'importazione di anagrafiche errate. Inoltre, viene loggata una voce dedicata nei file di log con il motivo esplicito che ha causato l'arresto del programma. 

### Elaborazione e Persistenza su Database

Per garantire la massima efficienza e minimizzare il blocco delle tabelle durante l'importazione di migliaia di record, il servizio adotta una strategia a due stadi:

1. **Caricamento in Tabelle Temporanee:** I dati vengono inizialmente inseriti nelle tabelle `epipoli_card_promo_temp` e `epipoli_loyalty_cards_temp`.
2. **Sincronizzazione (Upsert):** Tramite una logica di `MERGE`, i dati vengono trasferiti nelle tabelle principali `epipoli_card_promo` e `epipoli_loyalty_cards`. I record esistenti vengono aggiornati, i nuovi vengono inseriti e quelli non più presenti vengono eliminati.
3. **Auditing:** Ad ogni ciclo viene prodotta una riga nella tabella `epipoli_sync_log` che traccia l'inizio, la fine, l'ID della sessione e il numero di loyalty card e promozioni elaborate.

### Archiviazione e Retention Policy

Al termine di una sessione completata con successo, il file `NDJSON` originale viene compresso in formato `.zip` all'interno del path `etl\archive\` nella directory di progetto con un alto livello di compressione per risparmiare spazio su disco.

- **Naming:** `arena_cards_coupons_yyyymmdd_{ETLID}.zip` (dove ETLID è l'ID univoco della sessione ETL che ha elaborato il file ed è lo stesso ID memorizzato nella tabella `epipoli_sync_log`).
- **Retention per data (`MaxArchiveDays`):** Tutti i file più vecchi del numero di giorni prestabilito (default 7) vengono rimossi.
- **Retention per quantità (`MaxArchiveFileCount`):** Se il numero totale di file compressi supera il limite (default 168), il sistema elimina il file più vecchio ad ogni nuovo inserimento.

Tutte le operazioni di eliminazione vengono tracciate nel log generale con livello `INFO`.

---

## Funzionamento Kafka Producer

Il servizio, quando avviato con il flag `--kafka-producer`, opera come un concentratore di dati (tipicamente installato sul server CEDI, ma anche sui server di barriera). La sua architettura interna è suddivisa in due componenti indipendenti che comunicano tramite il database locale per garantire che nessun dato venga perso in caso di disconnessione.


### Flusso del Kafka Producer

Il cuore operativo dell'invio del venduto è rappresentato dal componente **Sender**, un processo in background che agisce come un motore di svuotamento della coda locale.<br>
Il workflow è progettato per operare in modo asincrono rispetto alla ricezione dei dati, garantendo che le transazioni vengano elaborate e trasmesse con una logica di "almeno una volta" (*at-least-once delivery*).

Attraverso una scansione ciclica (*polling*) del database, il **Sender** identifica i dati pendenti, risolve le anagrafiche necessarie per il corretto instradamento e gestisce la resilienza tramite politiche di retry. Di seguito viene descritta la sequenza logica delle operazioni:

``` title="Workflow tipico dell'invio del venduto"
1. Legge da epipoli_ext_sell i record con SentAt IS NULL (non ancora inviati)
2. Raggruppa per insegna (via tabella pdv.StoreId → insegna)
3. Invia batch al topic Kafka: ext_sell_{insegna}_in
   └─ Successo: aggiorna SentAt = NOW()
   └─ Errore: incrementa RetryCount, scrive LastError
4. Record con RetryCount > MaxRetryCount: non ritenta
```

### Logica dei processi

Per massimizzare le performance e la scalabilità, il modulo gestisce parallelamente due processi distinti:

- **Receiver (Ricezione):** Espone un endpoint REST che rimane in ascolto delle chiamate provenienti dalle casse Posware. Il suo unico compito è validare sintatticamente il JSON ricevuto e persistere il dato nel database il più velocemente possibile.
- **Sender (Inoltro):** È un worker ciclico che scansiona il database alla ricerca di transazioni non ancora inviate. Recupera i record, determina l'insegna di appartenenza e gestisce l'inoltro verso la piattaforma Epipoli tramite protocollo Kafka REST.

### Ricezione dati dalle casse

Le casse Posware trasmettono le informazioni di vendita in tempo reale o in differita tramite una richiesta API.<br>
Il servizio è configurato per accettare payload complessi che includono:

- Dati di testata (Data, ora, codice punto vendita, terminale cassa)
- Dati Loyalty (Codice tessera, totale scontrino)
- Dettaglio carrello (Articoli, EAN, quantità, sottocategorie)
- Dettaglio promozioni e coupon utilizzati (codici univoci ed *ExtRuleCode*)

### Determinazione dell'Insegna e Logica di Invio

Il processo di invio del venduto avviene in modalità esclusivamente **automatica** tramite un timer interno che avvia ciclicamente l'invio dei dati in base all'intervallo definito dal parametro `PollingIntervalInSeconds` nel file `appsettings.Production.json`.

Prima di procedere all'invio, il sistema deve stabilire l'endpoint di destinazione, poiché Epipoli prevede canali separati per le diverse insegne (*deco*, *superconveniente* o `*`).<br>
Poiché il payload inviato dalla cassa contiene solo lo `StoreId` numerico, viene eseguita una **JOIN dinamica** tra la tabella delle vendite `epipoli_ext__sell` e la tabella anagrafica `pdv` del database `posware` del server di barriera.

- Se l'insegna viene identificata correttamente, il dato viene accodato per l'invio verso il topic specifico.
- In assenza di una corrispondenza nella tabella `pdv`, il sistema solleva un errore di configurazione e sospende l'invio per quel punto vendita fino alla risoluzione del disallineamento.

Per quanto riguarda i log:

- Viene sempre riportato nei log il valore letto per il parametro **Insegna** con le informazioni su dove è stato letto e il valore recuperato, così da poter aiutare i tecnici in fase di eventuale diagnosi.
- In caso non sia possibile leggere l'insegna dalla tabella `pdv`, il programma scrive una riga di log `ERROR` seguita da una riga di `WARNING`:

    - La riga di errore evidenzia che per lo `StoreId` *XXXX* non è stata trovata un'insegna nella tabella `pdv`.
    - La riga di warning aggiunge nel log esplicitamente che l'invio del dato per quel punto vendita è stato saltato. Per riprovare l'invio deve venir compilato il campo `Insegna` della tabella `pdv`.

### Resilienza e Politiche di Retry

L'integrazione utilizza la libreria **Polly** per gestire i fallimenti temporanei della rete o l'eventuale indisponibilità del **Kafka REST Proxy** di Epipoli.

- **Exponential Backoff:** In caso di errore di rete (es. *HTTP 500* o *Timeout*), il sistema non scarta il dato, ma applica una strategia di attesa esponenziale prima di ritentare l'invio.
- **Batching:** I record vengono inviati in blocchi di dimensioni predefinite (`BatchSize`, default 100), ottimizzando il numero di chiamate HTTP e riducendo il carico sui server.

---

## Struttura tabelle del database

### Tabella `epipoli_card_promo`

La tabella `epipoli_card_promo` raccoglie i dati relativi alle promozioni attive associate alle relative carte fedeltà.

#### Struttura tabella `epipoli_card_promo`

``` sql title="Query CREATE TABLE"
    CREATE TABLE [epipoli_card_promo] (
        [Card] nvarchar(25) NOT NULL,
        [ExtRuleCode] nvarchar(25) NOT NULL,
        [EpipoliCode] nvarchar(100) NOT NULL,
        [ExpirationDate] date NOT NULL,
        [InsertedAt] datetime NOT NULL DEFAULT (GETDATE()),
        CONSTRAINT [PK_epipoli_card_promo] PRIMARY KEY ([Card], [ExtRuleCode], [EpipoliCode])
    );

    CREATE INDEX [idx_ExpirationDate] ON [epipoli_card_promo] ([ExpirationDate]);

    CREATE INDEX [idx_ExtRuleCode] ON [epipoli_card_promo] ([ExtRuleCode]) INCLUDE ([EpipoliCode]);

    CREATE INDEX [idx_InsertedAt] ON [epipoli_card_promo] ([InsertedAt]);
```

#### Descrizione dei campi tabella `epipoli_card_promo`

|Campo|Tipo|Descrizione|
|-----|----|-----------|
|**Card**|`nvarchar(25)`|Codice EAN della carta fedeltà associata alla promozione|
|**ExtRuleCode**|`nvarchar(25)`|Codice identificativo della regola di cassa. Viene concordato tra Gruppo Arena ed Epipoli per fare il matching con le promozioni di Posware|
|**EpipoliCode**|`nvarchar(100)`|Codice identificativo della promozione sulla piattaforma TechGroove di Epipoli|
|**ExpirationDate**|`date`|Data di scadenza della promozione|
|**InsertedAt**|`datetime`|Timestamp di inserimento del record|

!!! note "Chiave primaria"
    La chiave primaria è composta da `(Card, ExtRuleCode, EpipoliCode)`.

### Tabella `epipoli_card_promo_temp`

La tabella `epipoli_card_promo_temp` è una tabella temporanea di appoggio in cui salvare i dati in attesa di mergiarli all'interno di `epipoli_card_promo`.<br>Viene svuotata ad ogni ciclo ETL.

#### Struttura tabella `epipoli_card_promo_temp`

``` sql title="Query CREATE TABLE"
    CREATE TABLE [epipoli_card_promo_temp] (
        [Card] nvarchar(25) NOT NULL,
        [ExtRuleCode] nvarchar(25) NOT NULL,
        [EpipoliCode] nvarchar(100) NOT NULL,
        [ExpirationDate] date NOT NULL,
        CONSTRAINT [PK_epipoli_card_promo_temp] PRIMARY KEY ([Card], [ExtRuleCode], [EpipoliCode])
    );
```

#### Descrizione dei campi tabella `epipoli_card_promo_temp`

|Campo|Tipo|Descrizione|
|-----|----|-----------|
|**Card**|`nvarchar(25)`|Codice EAN della carta fedeltà associata alla promozione|
|**ExtRuleCode**|`nvarchar(25)`|Codice identificativo della regola di cassa. Viene concordato tra Gruppo Arena ed Epipoli per fare il matching con le promozioni di Posware|
|**EpipoliCode**|`nvarchar(25)`|Codice identificativo della promozione sulla piattaforma TechGroove di Epipoli|
|**ExpirationDate**|`date`|Data di scadenza della promozione|

!!! note "Chiave primaria"
    La chiave primaria è composta da `(Card, ExtRuleCode, EpipoliCode)`.

### Tabella `epipoli_ext_sell`

La tabella `epipoli_ext_sell` raccoglie i dati di vendita delle varie transazioni effettuate in cassa che contengono carte fedeltà e coupon o promozioni del nuovo programma fedeltà del Gruppo Arena, che andranno poi inviate ad Epipoli.

#### Struttura tabella `epipoli_ext_sell`

``` sql title="Query CREATE TABLE"
    CREATE TABLE [epipoli_ext_sell] (
        [Date] date NOT NULL,
        [StoreId] int NOT NULL,
        [TerminalId] int NOT NULL,
        [TransactionId] int NOT NULL,
        [JsonData] nvarchar(max) NOT NULL,
        [UpdatedAt] datetime2 NOT NULL,
        [SentAt] datetime2 NULL,
        [RetryCount] int NOT NULL DEFAULT 0,
        [LastError] nvarchar(max) NULL,
        CONSTRAINT [PK_epipoli_ext_sell] PRIMARY KEY ([Date], [StoreId], [TerminalId], [TransactionId])
    );

    CREATE INDEX [ix_retry_count] ON [epipoli_ext_sell] ([RetryCount]);

    CREATE INDEX [ix_sent_at] ON [epipoli_ext_sell] ([SentAt]);
```

#### Descrizione dei campi tabella `epipoli_ext_sell`

|Campo|Tipo|Descrizione|
|-----|----|-----------|
|**Date**|`date`|Data della transazione (corrisponde al giorno di emissione dello scontrino)|
|**StoreId**|`int`|Identificativo univoco del punto vendita|
|**TerminalId**|`int`|Identificativo univoco della cassa|
|**TransactionId**|`int`|Identificativo univoco della transazione lato Posware|
|**JsonData**|`nvarchar(MAX)`|Dato transazionale in formato JSON conforme allo standard richiesto da Epipoli per l'invio del venduto|
|**UpdateAt**|`datetime2`|Timestamp di ultima modifica/inserimento del record|
|**SentAt**|`datetime2`|Timestap di invio del venduto ad Epipoli|
|**RetryCount**|`int`|Numero di tentativi di rinvio del venduto in caso di errori nell'invio|
|**LastError**|`nvarchar(MAX)`|Ultimo messaggio di errore ricevuto in risposta da Epipoli in caso di fallimento nell'invio dei dati|

!!! note "Chiave primaria"
    La chiave primaria è composta da `(Date, StoreId, TerminalId, TransactionId)`.

### Tabella `epipoli_loyalty_cards`

La tabella `epipoli_loyalty_cards` contiene al suo interno l'anagrafica delle carte fedeltà registrate in Epipoli.

#### Struttura tabella `epipoli_loyalty_cards`

``` sql title="Query CREATE TABLE"
    CREATE TABLE [epipoli_loyalty_cards] (
        [Card] nvarchar(50) NOT NULL,
        [Source] nvarchar(50) NOT NULL,
        [Status] nvarchar(50) NOT NULL,
        [Username] nvarchar(100) NOT NULL,
        [ExpiringDate] datetime2 NOT NULL,
        CONSTRAINT [PK_epipoli_loyalty_cards] PRIMARY KEY ([Card])
    );
```

#### Descrizione dei campi tabella `epipoli_loyalty_cards`

|Campo|Tipo|Descrizione|
|-----|----|-----------|
|**Card**|`nvarchar(50)`|Codice EAN della carta fedeltà considerata|
|**Source**|`nvarchar(50)`|Insegna del punto vendita. Può assumere i valori: `deco` o `superconveniente`|
|**Status**|`nvarchar(50)`|Stato della carta fedeltà|
|**Username**|`nvarchar(100)`|Username del cliente a cui è associata la carta fedeltà|
|**ExpiringDate**|`datetime2`|Timestamp di scadenza della carta fedeltà|

!!! note "Chiave primaria"
    La chiave primaria è il campo `Card`.

### Tabella `epipoli_loyalty_cards_temp`

La tabella `epipoli_loyalty_cards_temp` è una tabella temporanea di appoggio in cui salvare i dati in attesa di mergiarli in `epipoli_loyalty_cards`.<br>Viene svuotata ad ogni ciclo ETL.

#### Struttura tabella `epipoli_loyalty_cards_temp`

``` sql title="Query CREATE TABLE"
    CREATE TABLE [epipoli_loyalty_cards_temp] (
        [Card] nvarchar(50) NOT NULL,
        [Source] nvarchar(50) NOT NULL,
        [Status] nvarchar(50) NOT NULL,
        [Username] nvarchar(100) NOT NULL,
        [ExpiringDate] datetime2 NOT NULL
    );
```

#### Descrizione dei campi tabella `epipoli_loyalty_cards_temp`

|Campo|Tipo|Descrizione|
|-----|----|-----------|
|**Card**|`nvarchar(50)`|Codice EAN della carta fedeltà considerata|
|**Source**|`nvarchar(50)`|Insegna del punto vendita. Può assumere i valori: `deco` o `superconveniente`|
|**Status**|`nvarchar(50)`|Stato della carta fedeltà|
|**Username**|`nvarchar(100)`|Username del cliente a cui è associata la carta fedeltà|
|**ExpiringDate**|`datetime2`|Timestamp di scadenza della carta fedeltà|

!!! note "Chiave primaria"
    Questa tabella non ha una chiave primaria e nessun vincolo.

### Tabella `epipoli_sync_log`

La tabella `epipoli_sync_log` raccoglie i dati relativi allo storico delle sessioni ETL.<br>È la tabella in cui vengono salvati gli ETag dell'ultimo file NDJSON scaricato dal bucket AWS S3 di Epipoli.

#### Struttura tabella `epipoli_sync_log`

``` sql title="Query CREATE TABLE"
    CREATE TABLE [epipoli_sync_log] (
        [Id] int NOT NULL IDENTITY,
        [StartedAt] datetime2 NOT NULL,
        [CompletedAt] datetime2 NULL,
        [Status] varchar(50) NOT NULL,
        [FileName] varchar(500) NULL,
        [RecordsSynced] int NULL,
        [PromotionsSynced] int NULL,
        [LastETag] varchar(255) NULL,
        [CardParseErrors] int NULL,
        [PromotionParseErrors] int NULL,
        [ErrorMessage] varchar(max) NULL,
        CONSTRAINT [PK_epipoli_sync_log] PRIMARY KEY ([Id])
    );

    CREATE INDEX [ix_status] ON [epipoli_sync_log] ([Status]);
```

#### Descrizione dei campi tabella `epipoli_sync_log`

|Campo|Tipo|Descrizione|
|-----|----|-----------|
|**Id**|`int`|Identificativo del record contenente lo storico della sessione ETL considerata|
|**StartedAt**|`datetime2`|Timestamp di inizio del download del file NDJSON|
|**CompletedAt**|`datetime2`|Timestamp di fine del download del file NDJSON|
|**Status**|`varchar(50)`|Stato del download del file NDJSON|
|**FileName**|`varchar(500)`|Nome del file NDJSON scaricato|
|**RecordsSynced**|`int`|Numero di record sincronizzati a fronte dell'operazione di ETL|
|**PromotionsSynced**|`int`|Numero di promozioni sincronizzate a fronte dell'operazione di ETL|
|**LastETag**|`varchar(255)`|ETag dell'ultimo file NDJSON scaricato|
|**CardParseErrors**|`int`|Numero di errori nel parsing dei dati delle carte fedeltà|
|**PromotionParseErrors**|`int`|Numero di errori nel parsing dei dati delle promozioni|
|**ErrorMessage**|`varchar(MAX)`|Messaggio di errore riscontrato in caso in caso di fallimento nel download del file NDJSON o nell'operazione di ETL|

!!! note "Chiave primaria"
    La chiave primaria è il campo `Id`.

### Script SQL delle tabelle

Il **PosEpipoliHelper** crea in automatico le tabelle di cui ha bisogno per il suo funzionamento, ma in presenza di database di grandi dimensioni, soprattutto sui server centralizzati (dimensioni anche superiori ai 3 TB), potrebbe essere necessario applicare manualmente gli script SQL.

Tali script si trovano all'interno del file `PosEpipoliHelper_InitialScript.sql` nella directory `db\` situata nella directory di installazione del **PosEpipoliHelper** e vanno eseguiti sul database SQL Server configurato nella connection string, consultabile nel file `appsettings.Production.json` dell'applicativo.

!!! warning "Esecuzione degli script SQL"
    Gli script sono **idempotenti rispetto all'ordine**, ma devono essere eseguiti sequenzialmente. Se il database è già **parzialmente inizializzato**, verificare quali tabelle esistono prima di eseguire gli script mancanti.

---

## Log

Il sistema di logging è un componente critico del **PosEpipoliHelper**, essenziale per monitorare l'andamento dei processi asincroni e per fornire ai tecnici gli strumenti necessari alla diagnosi. Tutti i file di log generati dall'applicativo convergono in un'unica directory centralizzata, `logs\`, situata nella directory di installazione.

### Dettaglio dei file di log

Poiché tutti i tracciamenti convergono nell'unica cartella `logs\`, il sistema utilizza una *naming convention* specifica per separare i flussi informativi. In fase di troubleshooting, i tecnici dovranno consultare i seguenti file a seconda dell'anomalia riscontrata:

- `general-log_yyyy-mm-dd.log`: Log applicativo generale giornaliero. Contiene tutti gli eventi di sistema e gli avvisi globali, escludendo il dettaglio granulare delle singole sessioni operative.
- `etl-yyyy-mm-dd.log`: Log generale giornaliero del ciclo di vita del processo ETL (es. avvio del timer, controlli preliminari su AWS S3), al netto del dettaglio della singola elaborazione.
- `etl-session_{ETLSessionId}_yyyy-mm-dd_HHmmss.log`: Log diagnostico giornaliero, completo e granulare relativo a una singola sessione ETL (identificata da un ID univoco e dal timestamp). È il file fondamentale per diagnosticare errori specifici, come il fallimento del check CRC64NVME, anomalie nel parsing del file JSON o problemi durante l'inserimento nel database locale.
- `kafka-yyyy-mm-dd.log`: Log generale giornaliero del processo Kafka Producer contenente le informazioni di stato del *Background Service* (avvii, arresti, inizializzazione code), escludendo i dettagli specifici delle singole code di invio.
- `kafka-ext_sell_{Insegna}-yyyy-mm-dd`: Log di dettaglio contenente il tracciato esatto degli invii verso le code Kafka per una specifica insegna (Deco o Superconveniente) nel giorno considerato. È il primo file da consultare in caso di mancato recapito degli scontrini per un determinato punto vendita, in quanto riporta l'intervallo di `SequenceId` elaborato e gli eventuali dump completi delle eccezioni di rete. 

### Formato riga

Per facilitare la lettura automatizzata e manuale, ogni riga all'interno dei file di log segue un tracciato standardizzato e predicibile:

```
{Timestamp} {LEVEL} {Logger} {Messaggio} {Eventuale eccezione}
```

*Esempio:*

```
2026-03-13 10:05:01.234 INFO PosEpipoliHelper.Api.BackgroundServices.EtlBackgroundService ETL session started
```

Il **Livello** (`INFO`, `WARNING`, `ERROR`, `FATAL`) determina la gravità dell'evento. I messaggi di livello `ERROR` includono sempre lo Stack Trace completo dell'eccezione per consentire un debug profondo.

### Rotazione dei log della sessione di ETL

Per prevenire l'esaurimento dello spazio su disco del server, il servizio applica politiche di rotazione automatica dei log della sessione di ETL.<br>
Infatti, poichè il sistema può generare numerosi file di sessione in una singola giornata (fino a uno ogni 15 minuti in caso di aggiornamenti continui sul bucket S3), è stato necessario implementare un algoritmo di pulizia dedicato che monitora questo tipo di log.

Il workflow dell'algoritmo è il seguente:

- Ogni sessione ETL genera un file `etl-session_{ETLSessionId}_yyyy-mm-dd_HHmmss.log`.
- Il servizio `SessionLogCleanupService` gira ogni `UpdateFrequencyInMinutes` minuti.
- Se i file superano `MaxSessionLogFiles`: i più vecchi (`SessionLogArchiveInPercent`%) vengono compressi in `etl-session-archive_{yyyyMMdd_HHmmss}.zip` e cancellati.

---

## Troubleshooting per i tecnici

Questa sezione fornisce un supporto rapido per l'identificazione e la risoluzione delle anomalie operative più frequenti che possono interessare i servizi ETL e Kafka Producer.

La tabella seguente è strutturata per guidare i tecnici, permettendo di distinguere immediatamente tra errori di configurazione locale, problemi di connettività verso i sistemi esterni o incoerenze anagrafiche nel database.

!!! warning "Nota metodologica per i tecnici"
    Prima di procedere con qualsiasi azione correttiva, è imperativo consultare i file di log descritti nell'apposita [sezione](#log). Mentre la tabella fornisce soluzioni a problemi noti, i log di sessione e di dettaglio contengono lo Stack Trace completo dell'errore, fondamentale per isolare casi limite o eventuali bug non censiti.

| Sintomo | Possibile causa | Azione |
|---------|----------------|--------|
| Servizio si avvia e si ferma subito | Nessun argomento `--etl`/`--kafka-producer` | Verificare `binPath` del servizio con `sc.exe qc PosEpipoliHelper` |
| ETL non scarica da S3 | Credenziali AWS errate o assenti | Verificare `AwsAccessKeyId` e `AwsSecretKey` in `appsettings.Production.json` |
| ETL si ferma con `InsegnaMismatchException` | Campo `source` nel file NDJSON non corrisponde all'`Insegna` configurata | Correggere il parametro `Insegna` o verificare il file su S3 |
| Kafka Producer non invia | `RestProxyBaseUrl` vuoto o non raggiungibile | Verificare URL e credenziali in sezione `KafkaProducer` |
| Log di sessione ETL crescono senza limiti | `MaxSessionLogFiles` troppo alto | Ridurre il valore o aumentare `SessionLogArchiveInPercent` |
| Servizio non si registra (errore setup) | Installazione ex-novo senza diritti amministratore | Eseguire il setup come amministratore |
| Setup interrotto con errore "Stringa di connessione non fornita" | Installazione ex-novo senza `/CONNECTIONSTRING` | Aggiungere `/CONNECTIONSTRING="Server=...;Database=...;..."` alla riga di comando del setup |
| Setup interrotto con errore "Impossibile leggere la stringa di connessione" | Aggiornamento del servizio con `appsettings.Production.json` assente o chiave `Default` vuota | Verificare che il file esista nella directory di installazione e che `ConnectionStrings.Default` non sia vuoto |