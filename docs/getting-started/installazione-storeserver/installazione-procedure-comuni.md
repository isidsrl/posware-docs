---
tags:
    - Posware
    - StoreServer
---

# Procedure comuni di installazione

**Prima revisione documento: 13 maggio 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Cenni preliminari
Il presente documento descrive la procedure comuni ad entrambi gli scenari di installazione dello *StoreServer*. Vengono illustrati i backup preliminari da effettuare e le configurazioni necessarie per i database provider.

## Backup preliminari
!!! danger "Backup necessari"
    Prima di iniziare tutta la procedura di installazione dello *StoreServer*, è **assolutamente mandatorio** eseguire i backup da usare in caso di **malfunzionamenti** e per l'eventuale procedura di **rollback**.<br>**Non eseguendo i backup, qualora si manifestassero problematiche di varia gravità che necessitino di rollback, quest'ultimo non sarà più possibile.**
    
    Eseguire il backup di quanto segue:

    1. Backup dei database in uso quali **`cassamaster`** e **`posware`** (**solo in caso di migrazione da Posware :material-tag:`4.2` a :material-tag:`4.3`**)
    2. Eventuali **database e directory non direttamente collegati a Posware** che possono essere influenzati dall'installazione di una o più procedure/dipendenze usate dalla nuova versione (MySQL, SQL Server, runtime .NET, ecc...)

!!! tip "Backup generale di sistema" 
    Valutare se è possibile effettuare un **backup generale di sistema**.<br>**Quest'ultimo backup non è obbligatorio**, ma rimane una precauzione ulteriore da adoperare per ripristinare eventuali sistemi compromessi a causa di eventuali errori umani.

## Installazione e configurazione del database provider
!!! warning "Le credenziali standard del database centralizzato sono cambiate"
    Ai fini di compatibilità con le nuove restrizioni dei criteri password complesse delle recenti versioni di MySQL e SQL Server, la nuova password di default è stata variata in **Vbhg4132!**

    **Si consiglia di cambiare la password in tutti i sistemi di produzione.**  

### MySQL
In caso il database provider scelto sia **MySQL**, è necessario installare MySQL :material-tag:`8.4`, l'unica versione attualmente compatibile con lo *StoreServer*. 

!!! info "Tool di gestione del database provider MySQL"
    A partire da Posware :material-tag:`4.3` si consiglia di utilizzare come tool di gestione del database provider **MySQL**:

    - MySQL Workbench all'ultima versione disponibile (**lo strumento messo a disposizione da Oracle**), oppure
    - Navicat :material-tag:`12.0.28` o superiore, oppure
    - HeidiSQL all'ultima versione disponibile

!!! danger "Obsolescenza di versioni di Navicat precedenti alla :material-tag:`12.0.28`"
    **A partire da MySQL :material-tag:`8.4`, in uso dallo *StoreServer*, NON sono assolutamente più supportate versioni di Navicat precedenti alla :material-tag:`12.0.28`.**

    **Usare versioni obsolete di Navicat comporta la CORRUZIONE IRRIMEDIABILE del database del server centrale!**

#### Installazione e configurazione
Per l'installazione e la configurazione fare riferimento alla **documentazione di Oracle** riguardante MySQL :material-tag:`8.4`.

Le seguenti configurazioni sono obbligatorie:

- Nello step in cui bisogna scegliere la configurazione del server impostare la voce *Config Type* pari a **Dedicated Computer**
- Nello step in cui è necessario configurare le credenziali di accesso al database provider, impostare la password dell'utente **root** di sistema pari a **Vbhg4132!**

Si consiglia infine di abilitare lo "*Slow Query Log*".

!!! danger "Installazione di MySQL :material-tag:`8.4` in presenza di un'altra istanza già installata"
    **In caso sul computer sia installata un'altra istanza di MySQL**, oltre alla versione :material-tag:`8.4`, si consiglia di eseguire una installazione side-by-side della nuova istanza.<br>**In caso contrario, l'istanza precedentemente installata potrebbe venir sovrascritta rendendo così impossibile eseguire la procedura di rollback, nell'eventualità in cui dovesse rendersi necessaria.**

!!! warning "Abilitare le connessioni esterne"
    Per una corretta trasmissione dei dati tra lo *StoreServer* e le casse, **è assolutamente necessario configurare MySQL per accettare connessioni esterne.**<br>Per abilitare le connessioni esterne, consultare la **documentazione di Oracle** riguardante MySQL.

##### Configurazione dei parametri del file my.ini e del metodo di autenticazione 
Terminata la configurazione iniziale del provider, sarà necessario impostare alcuni parametri all'interno del file *my.ini* di MySQL, situato nella directory di MySQL :material-tag:`8.4` (se non cambiata, la default è *C:\ProgramData\MySQL\MySQL Server 8.4*).<br>**Prima di procedere con la modifica del file assicurarsi di stoppare il servizio *MySQL84* di Windows.**

I parametri da impostare sono i seguenti:

- `max_connections` = 500
- `lower_case_table_names` = 1
- `log_error_suppression_list` = MY-013360 
- `key_buffer_size` = 64M
- `innodb_dedicated_server` = ON

!!! info "Inserimento dei parametri nel file *my.ini*"
    Se il parametro è già presente nel file *my.ini* è necessario cambiare il suo valore.<br>Se non presenti devono essere aggiunti al di sotto della sezione **[mysqld]** del file sia il parametro che il rispettivo valore (possono essere copiati e incollati così come sono riportati nell'elenco puntato).

###### Server non dedicati
**Lo *StoreServer* è progettato per funzionare su server dedicati.**<br>In caso contrario, bisognerà agire manualmente sulla memoria riservata a MySQL per evitare che utilizzi tutta la memoria esistente.

Per farlo, impostare ad `OFF` la variabile `innodb_dedicated_server` nel file *my.ini* e fare riferimento alla guida di MySQL per impostare la quantità di RAM da dedicare con questi valori minimi:

- **1 Gigabyte**: fino a 2 casse oppure fino a un massimo di 1,5 milioni di incasso. 
- **Almeno il 50% di RAM**: in tutti gli altri casi.

#### Importare un database migrato da Posware :material-tag:`4.2` in MySQL :material-tag:`8.4`
!!! warning "Scenario di migrazione"
    Le informazioni contenute in questo paragrafo sono da considerarsi valide **UNICAMENTE** per lo scenario di **migrazione da Posware :material-tag:`4.2` a :material-tag:`4.3`**.

Dopo aver completato l'installazione e la configurazione di MySQL :material-tag:`8.4`, è necessario esportare il database **`posware`** dalla vecchia versione di MySQL ed importarlo nella :material-tag:`8.4`.<br>**Questo è un prerequisito fondamentale per prepararsi alla migrazione con il Database Upgrader durante la procedura di installazione dello *StoreServer*.**

Durante l'importazione del database in MySQL :material-tag:`8.4` si raccomanda vivamente di rinominare il database con un nome diverso da **`posware`** (ad esempio **`posware42`**), così da non dover svolgere questa operazione successivamente.

!!! warning "Obbligo di utilizzo di mysqldump per importare/esportare i dump dei database in MySQL"
    Da Posware :material-tag:`4.3` in avanti **è obbligatorio** usare il tool **mysqldump** di MySQL per eseguire l'esportazione e l'importazione dei dump dei database in MySQL. 

    Non è più possibile usare **Navicat o altri strumenti simili** per fare queste operazioni, altrimenti non verrà garantita **la completa integrità del database** durante gli spostamenti tra versioni diverse di MySQL.

!!! warning "Effettuare il dump binario nell'esportazione dei database con mysqldump"
    Nell'esportazione del database con **mysqldump** è assolutamente mandatorio usare nel comando apposito "`-r`" invece del maggiore (`>`), così da effettuare il dump binario, che evita possibili errori che possono avvenire nel ripristino successivo del dump a causa di conflitti con la codifica. 

    Si ricorda che il comando generico di esportazione di un database con **mysqldump** senza alcun parametro aggiuntivo è `mysqldump <dbname> > <filename>`, dove "*dbname*" è il nome del database da esportare e "*filename*" è il nome del file in cui verrà salvato il dump del database.

    ``` bat title="Esempio di comando generico di esportazione di un database posware con mysqldump"
    mysqldump posware -r posware42.dump
    ``` 

### SQL Server
In caso il database provider scelto sia **SQL Server**, è possibile scegliere una tra le seguenti versioni, tutte compatibili con lo *StoreServer*: SQL Server `2016`, `2017`, `2019` o `2022`. 

#### Installazione e configurazione
Per la scelta della versione di riferimento, l'installazione e la configurazione fare riferimento alla **documentazione di Microsoft** riguardante SQL Server. 

Le seguenti configurazioni sono obbligatorie:

- Scegliere come modalità di installazione *Download Media* col package *Express Core*, perché così si potrà effettuare un'installazione in cui sarà poi possibile configurare il provider step per step, rispetto alle installazioni di default. **Questo serve per configurare le impostazioni successive**
- Nello step in cui bisogna scegliere la configurazione dell'istanza di SQL Server scegliere *Default instance*
- Nello step della configurazione dell'engine del database impostare l'*Authentication Mode* pari a *Mixed Mode (SQL Server authentication and Windows authentication)* e settare la password dell'utente **sa** di sistema pari a **Vbhg4132!**

!!! warning "Abilitare le connessioni esterne"
    Per una corretta trasmissione dei dati tra lo *StoreServer* e le casse, **è assolutamente necessario configurare SQL Server per accettare connessioni esterne.**<br>Per abilitare le connessioni esterne, consultare la **documentazione di Microsoft** riguardante SQL Server.

#### Importare un database migrato da Posware :material-tag:`4.2` in SQL Server
!!! warning "Scenario di migrazione"
    Le informazioni contenute in questo paragrafo sono da considerarsi valide **UNICAMENTE** per lo scenario di **migrazione da Posware :material-tag:`4.2` a :material-tag:`4.3`**.

Dopo aver completato l'installazione e la configurazione della versione scelta di SQL Server, è necessario esportare il database **`posware`** dalla vecchia versione di SQL Server ed importarlo nella nuova.<br>**Questo è un prerequisito fondamentale per prepararsi alla migrazione con il Database Upgrader durante la procedura di installazione dello *StoreServer*.**

### Troubleshooting per i tecnici
#### Errore "Access denied for user [...]" usando MySQL :material-tag:`8.4` sul server di barriera
Dopo aver associato le casse allo *StoreServer* nel relativo step dell'installazione, potrebbe verificarsi l'errore "***Access denied for user 'NOME_UTENTE'@'NOME_MACCHINA_CASSA' (using password: YES)***" nell'invio dei dati relativi alle tabelle legacy (carscar, ristampa_scontrini, tabdivise, ecc...) dalle casse allo *StoreServer*. 

Viene riportato di seguito un esempio del tracciato di log in cui è stato riscontrato tale errore:

``` bat title="Esempio di log con errore"
2024-09-03 16:09:05.6328|LEGACY|ERROR|Tabella:dett_msg_carscar -> Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.6328|LEGACY|ERROR|Impossibile copiare dett_msg_carscar su server :Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.6484|LEGACY|ERROR|Tabella:dettagli_carscar -> Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.6484|LEGACY|ERROR|Impossibile copiare dettagli_carscar su server :Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.6796|LEGACY|ERROR|Tabella:ristampa_scontrini -> Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.6796|LEGACY|ERROR|Impossibile copiare ristampa_scontrini su server :Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.6953|LEGACY|ERROR|Tabella:tabdivise -> Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.6953|LEGACY|ERROR|Impossibile copiare TabDivise su server :Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.7109|LEGACY|ERROR|Tabella:carscar_articoli -> Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:05.7109|LEGACY|ERROR|Impossibile copiare Carscar_articoli su server :Access denied for user 'root'@'CASSA1' (using password: YES)
2024-09-03 16:09:10.3515|LEGACY|ERROR|Access denied for user 'root'@'CASSA1' (using password: YES)
Connessione in uso: server=192.170.5.10;user id=root;password=Vbhg4132!;database=posware;connection timeout=3;default command timeout=3;provider=MySqlConnector
2024-09-03 16:09:10.3515|LEGACY|ERROR|Access denied for user 'root'@'CASSA1' (using password: YES)
``` 

Per diagnosticare questo errore eseguire le seguenti verifiche:

1. Verificare che nel file *ConnectionStrings.config* di *Posware Frontend*, il campo **ConnessioneServer** contenga al suo interno la *Connection string* esatta del database del server centrale.
2. Verificare che i campi `ip_server_barriera` e `db_server_barriera` nella tabella `config_cassa` del database `cassa` di *Posware Frontend* contengano, rispettivamente, i valori esatti dell'indirizzo IP e del nome del database del server di barriera.
3. Verificare che dal computer cassa **sia possibile collegarsi** sull'istanza MySQL del server di barriera. In caso contrario, **abilitare sul server di barriera le connessioni esterne al database**.
4. Dopo aver effettuato **tutti gli step precedenti con successo**, riavviare *Posware Frontend* sulle casse in cui si è verificato l'errore e controllare se viene registrato ancora nel log o è stato risolto.