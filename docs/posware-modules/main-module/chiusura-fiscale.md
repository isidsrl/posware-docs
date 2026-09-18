---
tags:
    - Posware
    - StoreServer
---

# Posware Module - Chiusura fiscale

**Prima revisione documento: 18 settembre 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Glossario

- **WebApp:** applicazione web che permette di gestire in modo centralizzato le funzionalità disponibili nello *StoreServer*;
- **Chiusura fiscale** (o *rapporto Z*, *ZReport*): chiusura giornaliera della stampante fiscale (RT) collegata a una cassa;
- **Terminale** (o *cassa*): postazione Posware abilitata alla chiusura centralizzata (`casse.Trasmissione = 1`);
- **Sessione:** singola esecuzione della chiusura fiscale su tutti i terminali del punto vendita;
- **Operatore chiuso / attivo:** stato di login dell'operatore sulla cassa. La chiusura parte solo con operatore chiuso.

## Cenni preliminari

La funzione **Chiusura fiscale** permette all'utente di:

- **verificare lo stato degli operatori** su tutte le casse abilitate (*Controllo operatori*);
- **avviare la chiusura fiscale** su tutte le casse, escludendo quelle non raggiungibili o con chiusura già effettuata di recente;
- **seguire l'avanzamento** di ogni cassa in tempo reale, anche riaprendo la pagina o dopo aver chiuso il browser;
- **consultare il rapporto finale** e **ripetere la chiusura sulla singola cassa** che ha dato errore;
- **avviare la chiusura da programmi esterni** tramite API REST sincrona.

La funzione si trova nella sezione ***Casse*** della **WebApp** dello *StoreServer* (voce *Chiusura fiscale*) ed è raggiungibile anche dal menu della lista casse.

!!! info "Sostituzione di un programma di back-office"
    Questa funzione sostituisce l'utility desktop `Pos_CFisc.exe`. Il protocollo di comunicazione con le casse è lo stesso (TCP porta 6855, comandi `ST`/`CF`/`PC`): **non è richiesto alcun aggiornamento del software di cassa**. Le differenze sono elencate nel capitolo [Differenze rispetto a Pos_CFisc](#differenze-rispetto-a-pos_cfisc).

### Versioni software

La funzione è inclusa nel **PoswareModule** dello *StoreServer*. È necessario che il modulo sia licenziato e attivo.

## Installazione

Nessuna installazione aggiuntiva. All'avvio lo *StoreServer* crea la tabella `zreport_session` (e rimuove la vecchia `zreport_history`) tramite le migrazioni del modulo.

### Configurazione

I parametri si trovano in `appsettings.json`, sezione `Posware:ZReport`:

| Parametro | Default | Descrizione |
|---|---|---|
| `Port` | `6855` | porta TCP su cui è in ascolto il software di cassa |
| `ConnectTimeout` | `00:00:10` | timeout di connessione alla cassa |
| `FirstByteTimeout` | `00:00:10` | attesa massima del primo byte di risposta |
| `CompletionTimeout` | `00:00:10` | attesa massima del completamento della risposta |
| `PollInterval` | `00:00:03` | intervallo tra due interrogazioni di avanzamento (`PC`) |
| `SessionTimeout` | `00:30:00` | durata massima di una sessione: allo scadere le casse ancora in corso vengono marcate *Timeout* |
| `RecentZReportThreshold` | `00:10:00` | una chiusura completata entro questo intervallo è considerata "recente" |
| `MaxDegreeOfParallelism` | `1` | numero di casse contattate contemporaneamente (1 = sequenziale, come il programma legacy) |

### Permessi

| Permesso | Descrizione |
|---|---|
| `store.zreport.view` | visualizza la pagina, i terminali e lo stato delle sessioni |
| `store.zreport.execute` | avvia, annulla e ripete la chiusura fiscale |

I ruoli *Amministratore* e *Responsabile turno* dispongono di entrambi i permessi.

## Utilizzo

### Lista terminali

All'apertura la pagina mostra tutte le casse abilitate e interroga subito lo stato dell'operatore su ognuna. Per ogni cassa vengono mostrati:

- il numero del terminale (es. `1`);
- lo stato: *Operatore chiuso*, *Operatore attivo* oppure *Terminale non raggiungibile*;
- la data dell'ultima chiusura completata dallo *StoreServer*, con l'indicazione *recente* se avvenuta da meno di 10 minuti.

Il pulsante **Controllo operatori** ripete la verifica senza avviare nulla.

### Avvio della chiusura

Premendo **Esegui chiusura fiscale** la WebApp esegue in sequenza:

1. una nuova verifica degli operatori;
2. se su qualche cassa l'operatore è ancora **attivo**: la chiusura non parte e viene chiesto di *Riprovare* (dopo il logout in cassa) o *Annullare*;
3. se qualche cassa **non risponde**: viene chiesto se avviare la chiusura escludendola (*Sì*) oppure rinunciare (*No*);
4. se qualche cassa ha una chiusura **recente**: viene mostrato l'elenco per scegliere quali escludere;
5. la sessione viene avviata.

!!! warning "Una sola sessione per volta"
    Se una chiusura è già in corso (anche avviata da un altro utente o da un programma esterno) la WebApp mostra il suo avanzamento invece di avviarne una nuova.

### Avanzamento

Durante l'esecuzione ogni cassa mostra uno spinner, la barra di avanzamento restituita dalla cassa, l'operazione in corso (es. *Passo 2 di 5 · Chiusura fiscale sulla stampante RT*, vedi [Rapporto](#rapporto)) e lo stato corrente:

| Stato | Significato |
|---|---|
| *Chiusura fiscale richiesta...* | la cassa ha accettato il comando ed è in coda |
| *Chiusura fiscale in corso...* | la cassa sta eseguendo i passi della chiusura |
| *Chiusura fiscale terminata* | tutti i passi completati |
| *Chiusura fiscale abortita* | la cassa ha terminato senza completare tutti i passi |
| *Chiusura fiscale rifiutata* | la cassa ha risposto `KO` al comando di chiusura |
| *Timeout invio chiusura* / *Timeout scaduto* | la cassa non ha risposto nei tempi previsti |
| *Terminale in errore* | risposta non valida o connessione interrotta |
| *Chiusura saltata* | cassa esclusa dall'utente |

Il pulsante **Annulla** interrompe il monitoraggio: le chiusure già inviate alle casse proseguono comunque, la sessione viene marcata *annullata*.

Lo stato della sessione è salvato sul server: chiudendo e riaprendo la pagina (o il browser) il monitoraggio riprende da dove era.

### Rapporto

Al termine il pulsante **Continua** apre il rapporto con l'esito di ogni cassa. Se tutte le casse sono *terminate* (o *saltate*) compare il messaggio *Operazione completata senza errori o avvisi*; altrimenti, per ogni cassa in errore, è disponibile **Riprova chiusura** che ripete l'intera sequenza (controllo operatore, invio chiusura, avanzamento) solo su quella cassa. Le casse con avvisi sono già espanse all'apertura del rapporto.

Per ogni cassa il rapporto elenca le **operazioni eseguite in cassa** con il loro esito (completata, non completata, in attesa):

| # | Operazione | Descrizione |
|---|---|---|
| 1 | Avvio procedura di chiusura | la cassa ha registrato l'avvio della chiusura fiscale |
| 2 | Chiusura fiscale sulla stampante RT | rapporto Z, scrittura del log `06`, registrazione della chiusura, chiusura giornata buoni pasto e chiusura del POS di pagamento elettronico |
| 3 | Invio dati fidelity sospesi | trasmissione al server dei messaggi fidelity rimasti in sospeso |
| 4 | Invio log sospesi | trasmissione al server dei log di vendita non ancora inviati |
| 5 | Conferma completamento chiusura | la cassa ha registrato il completamento di tutte le operazioni |

L'elenco corrisponde alla configurazione delle operazioni automatiche della cassa (tabella `passioperazioni_auto`, operazione `CF`): la cassa comunica l'avanzamento come sequenza di cifre (`1` = eseguita, `0` = non eseguita) e lo **aggiorna solo sui passi di controllo** (1 e 5). Di conseguenza una cassa che si ferma dopo l'avvio riporta un solo passo completato su cinque, anche se la chiusura sulla stampante potrebbe essere stata eseguita: in quel caso verificare lo scontrino di chiusura sulla cassa prima di ripetere l'operazione.

Per le casse che non hanno mai avviato la chiusura (chiusura rifiutata, terminale non raggiungibile, timeout di invio) non viene mostrato alcun elenco: la cassa non ha comunicato nessun avanzamento.

**Termina operazione** riporta alla lista terminali.

!!! tip "Ripresa di una sessione"
    Se l'ultima sessione della giornata è terminata con avvisi o è stata interrotta, all'apertura della pagina viene proposto di riaprire direttamente il rapporto.

## API REST per programmi esterni

Base: `/api/poswareModule/zreport` (autenticazione JWT, permessi come sopra).

| Metodo | Route | Descrizione |
|---|---|---|
| `GET` | `terminals` | casse abilitate con data ultima chiusura |
| `POST` | `terminals/check` | stato operatore di ogni cassa (`Closed`, `Active`, `Unreachable`) |
| `POST` | `sessions` | avvia la sessione in background: `202` con `sessionId`, `409` se già in corso |
| `POST` | `sessions/sync` | avvia la sessione e **attende il completamento** restituendo lo stato finale |
| `GET` | `sessions/current` | sessione in corso o ultima eseguita (`204` se nessuna) |
| `GET` | `sessions/{id}` | stato di una sessione |
| `GET` | `sessions?from=&to=` | storico sessioni per intervallo di date |
| `POST` | `sessions/{id}/terminals/{pos}/retry` | ripete la chiusura sulla singola cassa |
| `POST` | `sessions/{id}/cancel` | annulla la sessione in corso |

Body di `POST sessions` e `sessions/sync`:

```json
{ "excludedPosNumbers": [3, 7] }
```

In modalità sincrona non ci sono decisioni interattive: se un operatore risulta attivo o una cassa (non esclusa) non risponde, la sessione termina con stato `Aborted` e motivo `OperatorCheckFailed`.

Stati della sessione: `OperatorCheck`, `SendingCf`, `Polling` (in corso), `Completed`, `CompletedWithErrors`, `Aborted` (motivi: `OperatorCheckFailed`, `UserCancelled`, `Timeout`, `Interrupted`).

## Differenze rispetto a Pos_CFisc

- le decisioni (escludi cassa, riprova) vengono prese **prima** dell'avvio e non durante, così una chiusura lanciata da API non si blocca mai in attesa di una risposta;
- gli stati *abortita* ed *errore* sono terminali: non esiste più il polling infinito, ed è disponibile la riprova per singola cassa;
- la sessione ha una durata massima (`SessionTimeout`);
- ogni sessione è tracciata nella tabella `zreport_session` e nel log di audit (`zreport.start`, `zreport.retry`, `zreport.cancel`);
- non vengono più scritti il file semaforo `Pos_Cfisc.Ok` e la tabella `Log.Pos_Eod`.
