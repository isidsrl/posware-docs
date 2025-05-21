---
tags:
    - Posware
    - StoreServer
---

# Aggiornamento StoreServer

**Prima revisione documento: 20 maggio 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Cenni preliminari
Il presente documento descrive la procedura di aggiornamento del componente ***StoreServer*** attraverso il package manager **Winget**. L’aggiornamento tramite **Winget** consente di installare l’ultima versione disponibile del software, mantenendo eventuali file di configurazione personalizzati già presenti nel sistema.

## Prerequisiti
Prima di procedere con l'aggiornamento dello *StoreServer* **assicurarsi che lo *StoreServer* sia già installato e perfettamente funzionante**.

Il comando di update di **Winget** funziona **solo se lo *StoreServer* è già installato** nel sistema.<br>In caso contrario, verrà avviato automaticamente il processo di **installazione della nuova versione**, ma **senza argomenti custom**.

!!! warning "Versione minima necessaria"
    Assicurarsi di avere installata la versione minima di **Winget** supportata per l'installazione e l'aggiornamento dello *StoreServer*, consultabile nell'[overview generale](./overview-generale.md). 
        
    In caso di dubbi, procedere in ogni caso all'aggiornamento di **Winget** usando il Microsoft Store.

## Aggiornamento dello StoreServer tramite Winget
Per aggiornare lo *StoreServer* all’ultima versione disponibile è necessario utilizzare il package manager **Winget**.

### Avvio dell'aggiornamento
Avviare un nuovo terminale **con i diritti di amministratore** (*cmd.exe*) ed eseguire il comando:

``` bat title="Comando"
winget update StoreServer 
```

L’aggiornamento sovrascrive i file esistenti del programma, **ma mantiene intatti tutti i file di configurazione modificati manualmente dall’utente**.

!!! warning "Aggiornamento e argomenti custom"
    Il comando `winget update` **non supporta argomenti custom** (come connection string personalizzate o parametri di configurazione).<br>Se lo *StoreServer* **non è già installato**, il comando avvierà comunque l’installazione dell’ultima versione disponibile, **ma con impostazioni standard**. Questo è il comportamento predefinito del package manager **Winget** di Windows.

    Per mantenere una configurazione personalizzata, **verificare sempre che lo *StoreServer* sia già installato prima di avviare l'aggiornamento**.

#### Parametri proprietari di Winget
**Winget** mette a disposizione ulteriori parametri la cui trattazione esula da questa documentazione. 

I parametri più utili allo scopo sono:

- `--vebose`: abilita un output dettagliato del processo di aggiornamento.
- `--open-logs`: apre automaticamente la cartella dei log a fine aggiornamento. 

Per abilitari tali comportamenti, aggiungere i parametri al comando riportato in precedenza.

``` bat title="Comando con parametri proprietari di Winget"
winget update StoreServer --verbose --open-logs
```