---
tags:
    - Posware
    - StoreServer
---

# StoreServer - Disinstallazione

**Prima revisione documento: 10 novembre 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Cenni preliminari
La disinstallazione dello *StoreServer* può rendersi necessaria in diversi scenari, in particolare nel caso di errori durante una **installazione da zero** o durante una **migrazione da versioni precedenti**.<br>
Per garantire una corretta reinstallazione del sistema ed evitare problemi con il motore di gestione dei pacchetti, è consigliato **non utilizzare il Pannello di Controllo di Windows come primo approccio**, ma procedere sempre tramite **Winget**, il sistema di gestione dei pacchetti utilizzato dal processo di installazione dello *StoreServer*.

L'obiettivo di questo documento è illustrare la procedura corretta di disinstallazione, mostrando anche i passaggi da seguire per la diagnosi dei problemi e i log da consultare in caso di errore.

## Prerequisiti
Prima di procedere con la disinstallazione dello *StoreServer* **assicurarsi che lo *StoreServer* sia già installato o sia registrato almeno nei "Programmi e funzionalità" del Pannello di Controllo di Windows**.

Il comando di uninstall di **Winget** funziona **solo se lo *StoreServer* è già registrato** nel sistema.

!!! warning "Versione minima necessaria"
    Assicurarsi di avere installata la versione minima di **Winget** supportata per l'installazione e l'aggiornamento dello *StoreServer*, consultabile nell'[overview generale](./overview-generale.md). 
        
    In caso di dubbi, procedere in ogni caso all'aggiornamento di **Winget** usando il Microsoft Store.

## Disinstallazione dello StoreServer tramite Winget
Per disinstallare lo *StoreServer* è necessario utilizzare primariamente il package manager **Winget**.

### Avvio della disinstallazione
Avviare un nuovo terminale **con i diritti di amministratore** (*cmd.exe*) ed eseguire il comando:

``` bat title="Comando"
winget uninstall StoreServer 
```

Questa modalità garantisce la rimozione corretta del pacchetto, evitando che rimangano **registrazioni incomplete** all’interno del database locale di **Winget**.

!!! warning "Priorità nei tentativi di disinstallazione"
    Se si utilizza il Pannello di Controllo per la disinstallazione **prima** di aver tentato quella via **Winget**, **Winget** potrebbe continuare a credere che il pacchetto sia ancora installato.<br>
    Ciò costringerebbe a reinstallare lo *StoreServer* utilizzando l’opzione con il **parametro `force`**:
    ``` bat title="Comando"
    winget install --id isid.storeserver --source posware --force
    ```
    opzione da evitare se non strettamente necessaria.

#### Parametri proprietari di Winget
**Winget** mette a disposizione ulteriori parametri la cui trattazione esula da questa documentazione. 

I parametri più utili allo scopo sono:

- `--verbose`: abilita un output dettagliato del processo di disinstallazione.
- `--open-logs`: apre automaticamente la cartella dei log a fine disinstallazione. 

Per abilitare tali comportamenti, aggiungere i parametri al comando riportato in precedenza.

``` bat title="Comando con parametri proprietari di Winget"
winget uninstall StoreServer --verbose --open-logs
```

## Disinstallazione alternativa tramite Pannello di Controllo
Se la disinstallazione via **Winget** dovesse fallire per un qualsiasi motivo, è possibile procedere alternativamente tramite il **Pannello di Controllo di Windows** facendo click sulla voce ***Posware StoreServer*** e scegliendo l'opzione ***Disinstalla***.

Questa procedura esegue l’*uninstaller* generato da **InnoSetup**, che rimuove fisicamente i file installati.

!!! warning "Disinstallare lo StoreServer dal Pannello di Controllo"
    Questa modalità potrebbe non aggiornare il database interno di **Winget**, pertanto è consigliabile disinstallare da Pannello di Controllo **solo come fallback**, dopo alcuni tentativi falliti da **Winget**.

## Troubleshooting per i tecnici
### Log da consultare
Per diagnosticare correttamente eventuali problemi riscontrati durante le fasi di installazione, migrazione o aggiornamento dello *StoreServer*, è necessario consultare i log prodotti dai diversi componenti coinvolti nel processo.<br>
Di seguito sono riportati i log principali da analizzare e il loro contenuto.

#### Log di Winget
Questo log è presente nella cartella che viene aperta alla fine di un'installazione, migrazione o aggiornamento dello *StoreServer*, se si inserisce nel comando lanciato il parametro proprietario di **Winget** `--open-logs`. La cartella viene aperta anche in caso l'operazione non dovesse concludersi con successo restituendo un qualche tipo di errore.

Si raccomanda sempre di inserire il parametro proprietario `--open-logs` quando si lanciano certi tipi di operazioni con **Winget**, così da avere la cartella aperta automaticamente a fine installazione.

In caso si lanciasse un comando con **Winget** senza il suddetto parametro, il percorso dove trovare i log dovrebbe essere il seguente:
``` bat title="Percorso log Winget"
%LOCALAPPDATA%\Packages\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe\LocalState\DiagOutputDir
```

Il file di log ha un nome con la seguente struttura: `WinGet-YYYY-MM-DD-hh-mm-ss.ms.log`.

Contiene:

- L'esito delle operazioni *install*/*uninstall*, compresi eventuali errori interni di **Winget**.
- Eventuali messaggi di fallback o retry del motore di installazione.

#### Log di InnoSetup
Questo log è presente nella stessa cartella del **log di Winget**, ossia quella che si apre tramite il parametro `--open-logs`, se quest'ultimo viene utilizzato.

In alternativa, può essere recuperato manualmente nel percorso:
``` bat title="Percorso log InnoSetup"
%LOCALAPPDATA%\Packages\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe\LocalState\DiagOutputDir
```

Il file di log ha un nome con la seguente struttura: `WinGet-isid.storeserver.<StoreServerVersion>-YYYY-MM-DD-hh-mm-ss.ms.log`.

Contiene:

- Log dettagliati dell'*installer*/*uninstaller* **InnoSetup**.
- Informazioni utili per comprendere eventuali interruzioni nel setup.
- Errori nelle fasi preliminari dell'installazione o nella rimozione dei componenti

#### Log del SetupCompanion
Questo tipo di log è presente nella cartella di installazione dello *StoreServer* e il percorso preciso per consultarlo è il seguente:
``` bat title="Percorso log SetupCompanion"
%LOCALAPPDATA%\isid\StoreServer\SetupCompanionLogs
```

I file di log hanno un nome con la seguente struttura: `setupcompanion-log_YYYY-MM-DD_<GenericId>.log`.

Nel percorso indicato ci sono diversi file di log di questo tipo e ognuno contiene tipologie di informazioni differenti.<br>
Per la diagnosi quello più importante è quello delle **migrations** che contiene informazioni critiche, tra cui:

- Output grezzo delle EF Core (*Entity Framework Core*) migrations
- Errori del motore database
- Query SQL eseguite
- Nome della migration fallita
- Stacktrace delle eccezioni generate

!!! warning "Rimozione cartella `SetupCompanionLogs`"
    La cartella **`SetupCompanionLogs`** viene **rimossa durante la disinstallazione**.<br>
    È quindi essenziale consultare i log **subito dopo il fallimento dell’installazione e prima di disinstallare**.

### Diagnostica in caso di errore durante le migrations
Alcuni errori di installazione dello *StoreServer* derivano da problemi durante l’esecuzione delle **migrations di EF Core**, utilizzate per configurare o aggiornare il database.<br>
In caso di errore è necessario eseguire alcune verifiche preliminari per identificare **esattamente quale migration è fallita** e quali condizioni potrebbero aver causato il problema.

#### Passaggi per la diagnosi
1. **Identificare la migration fallita**<br>
Il nome della migration viene riportato nei log del *SetupCompanion*.<br>
Cercare la voce contenente:
``` bat title="Applicazione della migration"
Applying migration <MigrationName>
```
2. **Consultare eventuali errori del database provider**<br>
Gli errori restituiti dal database (timeout, violazioni di vincoli, problemi di memoria, blocchi, ecc...) sono riportati **dopo** il nome della migration.
3. **Verifiche consigliate**<br>
    - **Spazio e memoria del database:**<br>
    Assicurarsi che il database abbia sufficiente spazio disponibile e che non siano presenti limiti di memoria troppo restrittivi.
    - **Dimensione della tabella coinvolta:**<br>
    Alcune migrations potrebbero agire su tabelle con milioni di record, generando timeout o tempi di elaborazione troppo lunghi.
    - **Blocchi o transazioni pendenti:**<br>
    Verificare che non siano presenti blocchi a livello di database che impediscano a EF Core di applicare la migration.
    - **Permessi dell’utente di connessione:**<br>
    L’utente utilizzato per le migrations deve disporre delle autorizzazioni necessarie per creare/modificare tabelle e indici.

### Segnalazione del problema agli sviluppatori
In presenza di errori gravi non riconducibili a cause comuni (timeout, spazio insufficiente, permessi mancanti), è necessario inviare una segnalazione dettagliata al team di sviluppo.

La segnalazione deve includere i seguenti elementi:

1. **Tutti i log raccolti**, in particolare:
    - Log completi del ***SetupCompanion***
    - Log di **InnoSetup**
    - Log di **Winget**
2. **La versione dello *StoreServer*** installata o tentata
3. **Scenario operativo:**
    - Installazione da zero
    - Migrazione
    - Aggiornamento
4. **Versione del database** ed eventuali informazioni sulla dimensione delle tabelle coinvolte
5. **Eventuali modifiche custom effettuate manualmente sul database** (se presenti)

Una volta ricevute queste informazioni, il team di sviluppo potrà valutare:

- la causa esatta del problema
- la strategia di recovery più sicura
- se sia necessaria una patch o un intervento manuale