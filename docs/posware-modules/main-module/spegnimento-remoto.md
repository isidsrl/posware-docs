---
tags:
    - Posware
    - StoreServer
---

# Posware Module - Spegnimento remoto casse

**Prima revisione documento: 29 settembre 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Glossario

- **WebApp:** applicazione web che permette di gestire in modo centralizzato le funzionalità disponibili nello *StoreServer*;
- **Cassa** (o *terminale*): postazione Posware del punto vendita, raggiungibile in rete dallo *StoreServer*;
- **Schermata di login:** schermata iniziale del programma di cassa, senza operatore collegato e senza scontrino in corso;
- **Barriera casse:** insieme delle casse del punto vendita;
- **Cassa scollegata:** cassa non abilitata alla trasmissione (`casse.Trasmissione ≠ 1`) o senza indirizzo IP.

## Cenni preliminari

La funzione **Spegnimento remoto casse** permette di **spegnere il PC di una o più casse** direttamente dalla WebApp, senza raggiungere fisicamente la barriera. Tipici casi d'uso: lo spegnimento della barriera a fine giornata e lo spegnimento delle casse subito dopo la [Chiusura fiscale](chiusura-fiscale.md).

La funzione:

- agisce su **una cassa** oppure su **più casse insieme** (barriera);
- mostra l'**esito per ogni cassa** (spenta, non spenta, non raggiungibile...);
- prosegue anche se la pagina o la finestra vengono chiuse: lo spegnimento è eseguito dallo *StoreServer*;
- registra ogni richiesta e ogni esito nel **log di audit** dello *StoreServer*.

!!! info "Regola della cassa"
    La cassa accetta lo spegnimento **solo se il programma di cassa è nella schermata di login** (nessun operatore collegato e nessuno scontrino in corso). In quel caso risponde *Spenta* e subito dopo spegne il PC; in ogni altro caso risponde *Non spenta* e **resta accesa**. La WebApp anticipa la regola in base allo stato noto della cassa, ma **la decisione finale è sempre della cassa**.

!!! info "Sostituzione di un programma di back-office"
    Questa funzione sostituisce l'utility desktop `Pos_Spegni.exe`. **Non è richiesto alcun aggiornamento del software di cassa**: il comando inviato alla cassa è identico. Le differenze sono elencate nel capitolo [Differenze rispetto a Pos_Spegni](#differenze-rispetto-a-pos_spegni).

### Versioni software

La funzione è inclusa nel **PoswareModule** dello *StoreServer*. È necessario che il modulo sia licenziato e attivo.

## Installazione

Nessuna installazione aggiuntiva. Lo *StoreServer* deve poter raggiungere le casse sulla **porta TCP 6855** (la stessa usata dalla [Chiusura fiscale](chiusura-fiscale.md) e dal [Cambio posizione chiave](cambio-chiave-remoto.md)): verificare firewall e VLAN di barriera.

### Configurazione

I parametri si trovano in `appsettings.json`, sezione `Posware:RemotePosShutdown`. I valori vengono verificati all'avvio: con valori non validi lo *StoreServer* non parte e lo segnala nel log.

| Parametro | Default | Descrizione |
|---|---|---|
| `ConnectTimeout` | `00:00:02` | timeout di connessione alla cassa (come il programma legacy) |
| `FirstByteTimeout` | `00:00:02` | attesa massima del primo byte di risposta |
| `CompletionTimeout` | `00:00:01` | attesa massima del completamento della risposta |
| `MaxDegreeOfParallelism` | `8` | numero di casse contattate contemporaneamente (da 1 a 32) |
| `AutomaticRetries` | `2` | tentativi automatici aggiuntivi sulle casse non spente, **solo** per lo spegnimento avviato dal rapporto della chiusura fiscale (da 0 a 5) |
| `RetryDelay` | `00:00:30` | attesa tra un tentativo automatico e il successivo (minimo 1 secondo) |
| `CompletedOperationRetention` | `00:30:00` | per quanto tempo l'esito di uno spegnimento concluso resta consultabile |

La porta TCP è quella della sezione `Posware:LegacyTcpMessenger` (`Port`, default `6855`), condivisa con le altre funzioni di cassa. I timeout della sezione `Posware:LegacyTcpMessenger` **non** si applicano allo spegnimento, che usa quelli brevi indicati sopra.

### Permessi

| Permesso | Descrizione |
|---|---|
| `store.devices.pos.remoteshutdown.execute` | mostra le voci *Spegni* e *Spegni barriera casse* nella lista casse e il pannello *Spegnimento casse chiuse* nel rapporto della chiusura fiscale; avvia lo spegnimento |

Il permesso è assegnato ai ruoli `SystemAdmin`, `Technician`, `StoreManager` e `ShiftManager` (*Responsabile turno*). Nel rapporto della chiusura fiscale è richiesto anche il permesso `store.zreport.execute`.

Per disattivare la funzione per un ruolo è sufficiente togliere il permesso.

## Utilizzo

La funzione si trova nella sezione ***Casse*** della **WebApp**, nella lista *Barriera casse*, e nel rapporto della [Chiusura fiscale](chiusura-fiscale.md#spegnimento-delle-casse-dopo-la-chiusura).

<!-- TODO: aggiungere screenshot della voce Spegni e della finestra Spegni barriera casse -->

### Spegnere una cassa

1. aprire il menu della **singola cassa** (pulsante a destra della riga) → **Spegni**;
2. confermare la domanda *Spegnere la cassa N?* (il pulsante predefinito è *Annulla*);
3. durante lo spegnimento la riga della cassa mostra *Spegnimento…*;
4. al termine compare una notifica: *Cassa N spenta* oppure *Cassa N non spenta* con il motivo.

Dopo uno spegnimento riuscito lo stato della cassa nella lista viene aggiornato automaticamente.

La voce **Spegni** è visibile solo per le casse con trasmissione attiva ed è **disabilitata** (con il motivo mostrato passando sopra con il mouse) quando:

| Situazione | Motivo mostrato |
|---|---|
| operatore collegato in cassa | *Operatore collegato: la cassa non è nella schermata di login* |
| spegnimento già in corso sulla cassa | *Spegnimento in corso* |

Le casse *Offline*, *Online - StandBy* o con stato sconosciuto possono essere spente: se la cassa non risponde l'esito sarà *Cassa non raggiungibile (timeout)*.

### Spegnere la barriera casse

1. aprire il menu **Operazioni** della barriera → **Spegni barriera casse**;
2. nella finestra sono elencate le casse con trasmissione attiva, **tutte selezionate**:
    - le casse con **operatore collegato** sono evidenziate con l'avviso *verrà rifiutata se non torna alla schermata di login*, ma restano selezionabili: l'operatore potrebbe uscire nel frattempo;
    - le casse già **in spegnimento** non sono selezionabili;
    - **Seleziona tutte** seleziona o deseleziona l'intero elenco;
3. premere **Spegni N casse**;
4. la finestra mostra l'esito di ogni cassa e, al termine, il riepilogo *N spente / M non spente*;
5. se qualche cassa non si è spenta è disponibile **Riprova casse non spente**, che ripete lo spegnimento **solo** su quelle casse.

La finestra si può chiudere in qualsiasi momento: lo spegnimento prosegue e le righe della lista continuano a mostrare *Spegnimento…* fino al termine.

!!! note
    Lo stato mostrato nella finestra è quello noto alla lista casse al momento dell'apertura ed è solo indicativo: fa fede la risposta della cassa.

### Dopo la chiusura fiscale

Nel rapporto della chiusura fiscale è possibile spegnere le casse che hanno completato la chiusura, anche tutte insieme, con tentativi automatici se la cassa non è ancora tornata alla schermata di login. Vedi [Chiusura fiscale - Spegnimento delle casse dopo la chiusura](chiusura-fiscale.md#spegnimento-delle-casse-dopo-la-chiusura).

### Esiti

| Esito | Colore | Significato |
|---|---|---|
| *Spenta* | verde | la cassa ha accettato il comando e si sta spegnendo |
| *Non spenta: la cassa non è nella schermata di login (operatore collegato o scontrino in corso)* | arancione | la cassa ha rifiutato il comando e resta accesa |
| *Cassa non raggiungibile (timeout)* | rosso | nessuna risposta entro i tempi previsti (cassa già spenta, rete o programma di cassa non attivo) |
| *Errore di comunicazione* | rosso | connessione rifiutata o interrotta |
| *Risposta non riconosciuta* | rosso | la cassa ha risposto con un codice diverso da quelli previsti |
| *Cassa scollegata* | grigio | la cassa non è abilitata alla trasmissione o non ha indirizzo IP: il comando non è stato inviato |
| *Operazione interrotta (riavvio server)* | grigio | lo *StoreServer* è stato riavviato durante lo spegnimento: verificare lo stato della cassa e, se serve, ripetere |

Il messaggio di successo compare solo se **tutte** le casse richieste risultano *Spenta*.

Possono inoltre comparire questi messaggi:

| Messaggio | Causa |
|---|---|
| *Alcune casse sono già in spegnimento: N, M* | un altro utente (o un'altra finestra) sta già spegnendo quelle casse: attendere l'esito |
| *Cassa non trovata* | la cassa non esiste più nell'anagrafica casse |
| *Permessi insufficienti per spegnere le casse* | l'utente non dispone del permesso |

### Audit

Ogni richiesta viene registrata nel log di audit dello *StoreServer*:

- azione `pos.remoteshutdown.requested`: utente, data e ora, origine (*lista casse* o *chiusura fiscale*), elenco delle casse;
- azione `pos.remoteshutdown`, una per cassa al termine: utente che ha richiesto lo spegnimento, esito, numero di tentativi, durata e risposta della cassa.

??? question "Domande frequenti"
    **La cassa risulta *Non spenta* anche se non c'è nessuno alla cassa. Perché?**
    Il programma di cassa non è nella schermata di login: un operatore è rimasto collegato oppure c'è uno scontrino aperto. Effettuare il logout in cassa e ripetere lo spegnimento.

    **Ho riavviato lo *StoreServer* durante uno spegnimento: cosa succede?**
    Le casse che avevano già risposto restano spente; quelle ancora in attesa vengono mostrate come *Operazione interrotta (riavvio server)*. Verificare lo stato delle casse nella lista e ripetere lo spegnimento se necessario.

    **Nella finestra *Spegni barriera casse* manca una cassa.**
    Le casse scollegate (trasmissione disattivata o senza indirizzo IP) non vengono elencate perché non possono essere contattate.

    **Chiudendo la finestra lo spegnimento si interrompe?**
    No, prosegue sullo *StoreServer*. Riaprendo la lista casse le righe mostrano *Spegnimento…* fino al termine.

## API REST

Base: `/api/poswareModule/pos/remote-shutdown` (autenticazione JWT, permesso `store.devices.pos.remoteshutdown.execute`).

| Metodo | Route | Descrizione |
|---|---|---|
| `POST` | `operations` | avvia lo spegnimento e restituisce subito l'identificativo dell'operazione |
| `GET` | `operations/{operationId}` | stato dell'operazione e esito per cassa (interrogato dalla WebApp ogni secondo) |
| `GET` | `operations/running` | operazioni in corso (ripresa dopo l'aggiornamento della pagina) |

Body di `POST operations`:

```json
{ "posNumbers": [3, 5], "origin": "Gui" }
```

`origin` vale `Gui` (default, lista casse, nessun tentativo automatico) oppure `FiscalClosure` (rapporto della chiusura fiscale, tentativi automatici).

Risposta `202`:

```json
{ "operationId": "7f1e2c9a-..." }
```

Risposta di `GET operations/{operationId}`:

```json
{
  "operationId": "7f1e2c9a-...",
  "origin": "FiscalClosure",
  "requestedBy": "mario.rossi@negozio.it",
  "status": "Running",
  "isRunning": true,
  "allSucceeded": false,
  "maxAttempts": 3,
  "startedAtUtc": "2026-09-29T21:00:00Z",
  "completedAtUtc": null,
  "terminals": [
    { "posNumber": 3, "status": "Done", "outcome": "Ok", "message": "Spenta", "attempts": 1,
      "nextAttemptAtUtc": null, "elapsedMs": 312, "updatedAtUtc": "2026-09-29T21:00:01Z" }
  ]
}
```

Valori di `outcome`: `Ok`, `NotAtLogin`, `Timeout`, `CommunicationError`, `UnexpectedResponse`, `TerminalDisabled`. Valori di `status` dell'operazione: `Running`, `Completed`, `CompletedWithErrors`, `Interrupted`.

Altri codici: `400` dati non validi (elenco vuoto, casse duplicate, più di 100 casse, origine sconosciuta), `404` cassa inesistente oppure operazione sconosciuta, scaduta o persa per riavvio, `409` casse già in spegnimento (elenco in `busyPosNumbers`).

## Differenze rispetto a Pos_Spegni

- accesso tramite utente *StoreServer* e permesso dedicato, invece dell'accesso libero al PC server di barriera;
- esito **per ogni cassa** e messaggio di successo solo se tutte le casse si sono spente;
- **Riprova casse non spente** ripete lo spegnimento solo sulle casse rimaste accese (il programma legacy ripeteva su tutte);
- casse contattate in parallelo: lo spegnimento della barriera richiede pochi secondi;
- dopo la chiusura fiscale lo spegnimento si avvia dal rapporto, con tentativi automatici, al posto del lancio di `Pos_Spegni.exe BATCH` nella procedura di chiusura;
- una cassa non può essere spenta contemporaneamente da due utenti;
- ogni richiesta e ogni esito sono tracciati nel log di audit.
