---
tags:
    - Posware
    - StoreServer
---

# Posware Module - Cambio posizione chiave remoto

**Prima revisione documento: 28 settembre 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Glossario

- **WebApp:** applicazione web che permette di gestire in modo centralizzato le funzionalità disponibili nello *StoreServer*;
- **Cassa** (o *terminale*): postazione Posware del punto vendita, raggiungibile in rete dallo *StoreServer*;
- **Posizione chiave** (o *livello chiave*): livello di autorizzazione attivo in cassa. Riprende il concetto della chiave meccanica delle vecchie casse: a ogni livello corrispondono le operazioni riservate abilitate (storni, annulli, resi, apertura cassetto...);
- **Operatore Posware:** codice e password dell'operatore di cassa che autorizza il cambio chiave;
- **Cassa scollegata:** cassa non abilitata alla trasmissione (`casse.Trasmissione ≠ 1`) o senza indirizzo IP.

## Cenni preliminari

La funzione **Cambio posizione chiave** permette al direttore o al responsabile di turno di **alzare (o riportare) da remoto la posizione chiave di una cassa**, senza raggiungere fisicamente la barriera. Tipico caso d'uso: un cassiere deve eseguire un'operazione riservata e il responsabile la autorizza dalla WebApp.

La funzione:

- agisce su **una cassa per volta**;
- chiede sempre **codice e password di un operatore Posware**: è la cassa a verificare le credenziali e a decidere se accettare il cambio;
- mostra l'**esito** restituito dalla cassa (chiave impostata, credenziali rifiutate, cassa non raggiungibile...);
- registra ogni tentativo nel **log di audit** dello *StoreServer*.

!!! info "Sostituzione di un programma di back-office"
    Questa funzione sostituisce l'utility desktop `Pos_Chiave.exe`. **Non è richiesto alcun aggiornamento del software di cassa**: il comando inviato alla cassa è identico. Le differenze sono elencate nel capitolo [Differenze rispetto a Pos_Chiave](#differenze-rispetto-a-pos_chiave).

### Posizioni chiave

| Posizione | Descrizione |
|---|---|
| **Operatore** (1) | livello standard del cassiere |
| **Amministratore (livello 2)** | operazioni configurate in Posware per il livello 2 |
| **Amministratore (livello 3)** | operazioni configurate in Posware per il livello 3 |
| **Amministratore (livello 4)** | operazioni configurate in Posware per il livello 4 |

Le operazioni abilitate da ciascun livello si configurano in Posware. Il livello 5 della vecchia utility non esiste più e non è selezionabile.

!!! note
    La posizione impostata resta attiva finché non viene cambiata di nuovo: per riportare la cassa al livello standard eseguire un nuovo cambio chiave con posizione **Operatore**.

### Versioni software

La funzione è inclusa nel **PoswareModule** dello *StoreServer*. È necessario che il modulo sia licenziato e attivo.

## Installazione

Nessuna installazione aggiuntiva. Lo *StoreServer* deve poter raggiungere le casse sulla **porta TCP 6855** (la stessa usata dalla [Chiusura fiscale](chiusura-fiscale.md)): verificare firewall e VLAN di barriera.

### Configurazione

I parametri si trovano in `appsettings.json`.

Sezione `Posware:RemotePosKeyChange`:

| Parametro | Default | Descrizione |
|---|---|---|
| `PreValidateCredentials` | `false` | se `true`, prima di contattare la cassa lo *StoreServer* verifica operatore e password sulla tabella `Operatori` del database Posware. Lasciare `false` se le credenziali del database centrale possono essere disallineate da quelle di cassa: **fa fede la cassa** |
| `RateLimitPermitLimit` | `10` | numero massimo di richieste di cambio chiave per utente nella finestra `RateLimitWindow` (protezione contro tentativi ripetuti di indovinare la password) |
| `RateLimitWindow` | `00:01:00` | durata della finestra del limite di richieste |

La modifica di `PreValidateCredentials` ha effetto senza riavviare lo *StoreServer*.

Sezione `Posware:LegacyTcpMessenger` (comunicazione con le casse, **condivisa con la Chiusura fiscale**):

| Parametro | Default | Descrizione |
|---|---|---|
| `Port` | `6855` | porta TCP su cui è in ascolto il software di cassa |
| `ConnectTimeout` | `00:00:10` | timeout di connessione alla cassa |
| `FirstByteTimeout` | `00:00:10` | attesa massima del primo byte di risposta |
| `CompletionTimeout` | `00:00:10` | attesa massima del completamento della risposta |

### Permessi

| Permesso | Descrizione |
|---|---|
| `store.devices.pos.remotekeychange.view` | mostra le voci *Cambio posizione chiave* nella lista casse e apre la finestra |
| `store.devices.pos.remotekeychange.execute` | invia il cambio chiave alla cassa |

I ruoli *Amministratore* e *Responsabile turno* dispongono di entrambi i permessi. Con il solo permesso di visualizzazione la finestra si apre ma il pulsante di invio resta disabilitato.

## Utilizzo

La funzione si trova nella sezione ***Casse*** della **WebApp**, nella lista *Barriera casse*, ed è raggiungibile in due modi:

- menu **Operazioni** della barriera → **Cambio posizione chiave**: si sceglie la cassa dall'elenco;
- menu della **singola cassa** (pulsante a destra della riga) → **Cambio posizione chiave**: la cassa è già selezionata.

<!-- TODO: aggiungere screenshot della finestra di cambio chiave -->

### Invio del cambio chiave

1. selezionare la **cassa** (le casse scollegate sono visibili ma non selezionabili);
2. inserire il **codice operatore** (solo cifre, massimo 10) e la **password** (massimo 6 caratteri, senza lettere accentate);
3. scegliere la **posizione chiave**;
4. premere **Invia alla cassa** (oppure *Invio* dalla tastiera);
5. per i livelli *Amministratore* viene chiesta una **conferma**: *Impostare la chiave Amministratore (livello N) sulla cassa X?*

Durante l'invio il pulsante è disabilitato e viene mostrato l'avanzamento: se la cassa non risponde l'attesa può durare alcuni secondi (fino ai timeout configurati).

!!! warning "Password"
    La password non viene mai salvata: dopo ogni invio il campo viene svuotato e va reinserito per un nuovo tentativo.

### Esiti

| Esito | Colore | Significato |
|---|---|---|
| *Chiave impostata* | verde | la cassa ha accettato il comando (`OK`) |
| *Credenziali errate o impossibile effettuare il cambio chiave in questo momento* | arancione | la cassa ha rifiutato il comando (`KO`): credenziali errate o cassa in uno stato che non consente il cambio |
| *Cassa non raggiungibile (timeout)* | rosso | nessuna risposta entro i tempi previsti |
| *Errore di comunicazione* | rosso | connessione rifiutata o interrotta |
| *Risposta non riconosciuta dalla cassa* | rosso | la cassa ha risposto con un codice diverso da `OK`/`KO` |
| *Cassa scollegata* | grigio | la cassa non è abilitata alla trasmissione: il comando non è stato inviato |

Per gli esiti in rosso è possibile riprovare: reinserire la password e premere di nuovo **Invia alla cassa**.

La finestra può mostrare anche questi messaggi:

| Messaggio | Causa |
|---|---|
| *Un cambio chiave è già in corso sulla cassa X* | un altro utente sta inviando un cambio chiave alla stessa cassa: attendere e riprovare |
| *Operatore o password non validi* | solo con `PreValidateCredentials = true`: credenziali non presenti nel database Posware, la cassa non è stata contattata |
| *Troppi tentativi di cambio chiave* | superato il limite di richieste per utente: attendere qualche minuto |

### Audit

Ogni tentativo viene registrato nel log di audit dello *StoreServer* con azione `pos.remotekeychange`: utente, data e ora, cassa, codice operatore Posware, posizione chiave, esito, durata e risposta della cassa. **La password non viene mai registrata.**

## API REST

Base: `/api/poswareModule/remotePosKeyChange` (autenticazione JWT, permessi come sopra).

| Metodo | Route | Permesso | Descrizione |
|---|---|---|---|
| `GET` | `terminals` | `...remotekeychange.view` | elenco casse con indicazione di selezionabilità (`isLinkEnabled`) |
| `POST` | `terminals/{posNumber}` | `...remotekeychange.execute` | invia il cambio chiave alla cassa e restituisce l'esito |

Body di `POST terminals/{posNumber}`:

```json
{ "operatorId": "12", "password": "123456", "keyLevel": 2 }
```

Risposta `200`:

```json
{ "posNumber": 3, "keyLevel": 2, "outcome": "Ok", "message": "Chiave impostata", "elapsedMs": 180 }
```

Valori di `outcome`: `Ok`, `Rejected`, `Timeout`, `CommunicationError`, `UnexpectedResponse`, `TerminalDisabled`.

Altri codici: `400` dati non validi, `404` cassa inesistente, `409` cambio chiave già in corso sulla cassa, `422` credenziali rifiutate dalla verifica preliminare, `429` limite di richieste superato.

## Differenze rispetto a Pos_Chiave

- **una cassa per volta**: l'invio contemporaneo su più casse non è più disponibile;
- accesso tramite utente *StoreServer* e permessi dedicati, invece dell'accesso libero al PC server di barriera;
- ogni tentativo è tracciato nel log di audit;
- posizioni chiave da 1 a 4 (il livello 5 non esiste più);
- esiti tipizzati: una risposta non riconosciuta non lascia più la riga in *Invio in corso...*;
- la verifica delle credenziali sul database (tasto *Invio* della vecchia utility, di fatto non funzionante) è ora opzionale e disattivata di default;
- conferma obbligatoria per i livelli *Amministratore* e limite di richieste per utente;
- non viene più scritto il file `Versioni.INI`.
