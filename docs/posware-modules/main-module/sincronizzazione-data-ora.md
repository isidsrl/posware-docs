---
tags:
    - Posware
    - StoreServer
---

# Posware Module - Sincronizzazione data/ora casse

**Prima revisione documento: 2 ottobre 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Glossario

- **WebApp:** applicazione web che permette di gestire in modo centralizzato le funzionalità disponibili nello *StoreServer*;
- **Cassa** (o *terminale*): postazione Posware del punto vendita, raggiungibile in rete dallo *StoreServer*;
- **ApiGateway:** servizio di Posware installato su ogni cassa (`PoswareApiGatewayWatchdog`) che riceve le richieste dello *StoreServer* e imposta l'orologio di Windows;
- **Schermata di login:** schermata iniziale del programma di cassa, senza operatore collegato, senza scontrino e senza altre operazioni in corso;
- **Ora ufficiale:** data e ora correnti del fuso orario italiano (*Europe/Rome*, ora legale compresa) lette dal servizio esterno [timeapi.io](https://timeapi.io);
- **Sincronizzazione automatica:** sincronizzazione eseguita ogni giorno dallo *StoreServer* all'orario stabilito, senza intervento dell'utente;
- **Cassa scollegata:** cassa non abilitata alla trasmissione (`casse.Trasmissione ≠ 1`) o senza indirizzo IP.

## Cenni preliminari

La funzione **Sincronizzazione data/ora casse** imposta sulle casse la **data e l'ora ufficiali, al secondo**. Un orologio corretto è importante perché data e ora finiscono sugli scontrini, sui documenti fiscali e sui log.

La funzione:

- legge l'ora ufficiale dal servizio **timeapi.io** con il fuso *Europe/Rome*: **l'orologio del server non viene mai usato**, perché anch'esso potrebbe essere sbagliato;
- imposta l'orologio tramite l'**ApiGateway** della cassa, con precisione al secondo e indipendentemente dal fuso orario di Windows;
- agisce su **una cassa**, su **più casse insieme** oppure **in automatico ogni giorno** all'orario stabilito;
- mostra l'**esito per ogni cassa** con la data e l'ora impostate e la correzione applicata all'orologio;
- nella scheda della cassa mostra l'**orologio della cassa** confrontato con l'ora ufficiale e permette di scegliere come allineare l'orario della **stampante fiscale**;
- prosegue anche se la pagina o la finestra vengono chiuse: la sincronizzazione è eseguita dallo *StoreServer*;
- registra ogni richiesta e ogni esito nel **log di audit** dello *StoreServer*.

!!! warning "Versione minima di Posware: 4.3.1012.0"
    La funzione è disponibile solo sulle casse con **Posware 4.3.1012.0 o successiva**. Sulle casse con una versione precedente, o di cui lo *StoreServer* non conosce la versione, la voce e il pulsante *Sincronizza data/ora* sono **disabilitati**: passando sopra con il mouse compare il motivo, ad esempio *Funzione non disponibile: richiesta versione Posware 4.3.1012.0 o successiva (installata 4.3.1000.0)*. La versione è quella comunicata dalla cassa (riquadro *Informazioni di sistema* della [scheda cassa](scheda-cassa.md)). Quando lo *StoreServer* non ha la versione registrata, o ne ha una precedente alla 4.3.1012.0, prima di procedere la chiede direttamente alla cassa: una cassa appena aggiornata viene quindi sincronizzata anche se la versione registrata non è ancora quella nuova.

!!! info "Regola della cassa"
    La cassa accetta data e ora **solo se il programma di cassa è nella schermata di login**: nessun operatore collegato, nessuno scontrino o transazione in corso, nessuna chiusura fiscale o altra operazione in corso. In ogni altro caso la sincronizzazione non viene eseguita e l'esito ne indica il motivo. La WebApp anticipa la regola in base allo stato noto della cassa, ma **la decisione finale è sempre della cassa**.

!!! info "Sostituzione di un programma di back-office"
    Questa funzione sostituisce la scheda *Impostazione ORA* dell'utility desktop `Pos_Utility.exe`. Le differenze sono elencate nel capitolo [Differenze rispetto a Pos_Utility](#differenze-rispetto-a-pos_utility).

### Versioni software

- **StoreServer:** la funzione è inclusa nel **PoswareModule**. È necessario che il modulo sia licenziato e attivo;
- **Casse:** **Posware 4.3.1012.0** o successiva, con il servizio **ApiGateway** installato e avviato.

## Installazione

Nessuna installazione aggiuntiva sullo *StoreServer*. Prerequisiti:

- accesso di rete dallo *StoreServer* verso le casse sulla **porta TCP 6857** (ApiGateway). La porta è la stessa su tutte le casse e non è configurabile;
- accesso di rete dallo *StoreServer* verso **internet**, in HTTPS (porta 443) all'indirizzo `timeapi.io`. Se il negozio usa un proxy, lo *StoreServer* utilizza il proxy di sistema;
- su **ogni cassa**, il servizio `PoswareApiGatewayWatchdog` avviato con l'account **LocalSystem** (vedi [Servizio ApiGateway sulle casse](#servizio-apigateway-sulle-casse)).

!!! warning "Senza accesso a timeapi.io la sincronizzazione non parte"
    Se l'ora ufficiale non è disponibile nessuna cassa viene contattata e compare il messaggio *Ora ufficiale non disponibile*. È una scelta voluta: non si usa mai un orario di ripiego.

### Servizio ApiGateway sulle casse

L'orologio di Windows viene cambiato dall'**ApiGateway**, non dal programma di cassa. Windows consente di cambiare l'ora solo a chi ha il diritto **Modifica dell'ora di sistema** (`SeSystemtimePrivilege`): l'ApiGateway lo possiede perché il servizio `PoswareApiGatewayWatchdog` è eseguito con l'account **LocalSystem**. Il programma di cassa, invece, **non** deve essere avviato come amministratore e non ha bisogno di diritti particolari.

Se il servizio viene eseguito con un altro account, o se l'ApiGateway viene avviato a mano senza elevazione, l'esito della sincronizzazione è *Non sincronizzata: il servizio ApiGateway della cassa non ha il permesso di modificare l'ora*.

**Verifica**, su ogni cassa, da un *Prompt dei comandi*:

```text
sc qc PoswareApiGatewayWatchdog
```

Il risultato deve contenere `SERVICE_START_NAME : LocalSystem`.

**Correzione**, con un utente amministratore:

=== "Servizi di Windows"

    1. premere ++win+r++, digitare `services.msc` e premere **OK**;
    2. fare doppio clic sul servizio **PoswareApiGatewayWatchdog**;
    3. nella scheda **Connessione** scegliere **Account di sistema locale** e premere **OK**;
    4. fare clic con il tasto destro sul servizio → **Riavvia**.

=== "Prompt dei comandi"

    Aprire il *Prompt dei comandi* **come amministratore** ed eseguire:

    ```text
    sc config PoswareApiGatewayWatchdog obj= LocalSystem
    sc stop PoswareApiGatewayWatchdog
    sc start PoswareApiGatewayWatchdog
    ```

!!! note "Fuso orario di Windows"
    L'ApiGateway imposta l'orologio in UTC, quindi l'ora è corretta qualunque sia il fuso orario di Windows. Posware però mostra e stampa l'**ora locale**: il fuso orario della cassa deve essere *(UTC+01:00) Amsterdam, Berlino, Berna, Roma, Stoccolma, Vienna*. Un fuso diverso viene segnalato nell'esito della sincronizzazione e nella scheda della cassa.

### Configurazione

I parametri tecnici si trovano in `appsettings.json`. I valori vengono verificati all'avvio: con valori non validi lo *StoreServer* non parte e lo segnala nel log.

Sezione `Posware:PosDateTimeSync`:

| Parametro | Default | Descrizione |
|---|---|---|
| `MaxDegreeOfParallelism` | `8` | numero di casse contattate contemporaneamente (da 1 a 32) |
| `CompletedOperationRetention` | `00:30:00` | per quanto tempo l'esito di una sincronizzazione conclusa resta consultabile |
| `SchedulerTickInterval` | `00:00:30` | ogni quanto la sincronizzazione automatica controlla se è il momento di partire |
| `ToleranceMs` | `1000` | differenza in millisecondi (da 0 a 60000) sotto la quale la cassa non cambia l'orologio: l'esito è *Orologio già allineato* |

Sezione `Posware:PosApiGateway` (comunicazione con le casse):

| Parametro | Default | Descrizione |
|---|---|---|
| `RequestTimeout` | `00:00:10` | attesa massima della risposta di una cassa (massimo 1 minuto) |

Sezione `Posware:TimeReference` (lettura dell'ora ufficiale):

| Parametro | Default | Descrizione |
|---|---|---|
| `AttemptTimeout` | `00:00:05` | attesa massima di una singola lettura |
| `Retries` | `2` | nuovi tentativi in caso di errore temporaneo (da 0 a 5) |
| `RetryDelay` | `00:00:00.500` | attesa tra un tentativo e il successivo |
| `TotalTimeout` | `00:00:12` | durata massima complessiva della lettura |
| `ServerSkewWarningThreshold` | `00:01:00` | oltre questo scostamento dell'orologio dello *StoreServer* viene scritto un avviso nel log |

L'indirizzo di timeapi.io, il fuso orario *Europe/Rome* e la porta **6857** delle casse **non sono configurabili**.

Le impostazioni della **sincronizzazione automatica** (attivazione, orario, nuovi tentativi) **non** si trovano in `appsettings.json`: si modificano dalla WebApp e sono salvate nel database (vedi [Sincronizzazione automatica](#sincronizzazione-automatica)). La **modalità della stampante fiscale** è un parametro di ciascuna cassa e si modifica dalla scheda della cassa (vedi [Orologio cassa](#orologio-cassa)).

### Permessi

| Permesso | Descrizione |
|---|---|
| `store.devices.pos.datetimesync.execute` | mostra la voce *Sincronizza data/ora* nel menu della cassa, il pulsante e il riquadro *Orologio cassa* nella scheda cassa; nella finestra *Sincronizzazione data/ora casse* mostra la sezione *Sincronizza ora* |
| `store.devices.pos.datetimesync.manage` | nella finestra *Sincronizzazione data/ora casse* mostra e permette di modificare la sezione *Sincronizzazione automatica*; nel riquadro *Orologio cassa* permette di cambiare la modalità della stampante fiscale, il server NTP, il passaggio automatico ora legale/solare e la disattivazione della sincronizzazione automatica della stampante; nella stessa finestra mostra il riquadro *Orologio stampante fiscale — imposta su più casse* |

La voce *Sincronizzazione data/ora casse* del menu **Operazioni** è visibile con almeno uno dei due permessi. I permessi sono assegnati ai ruoli `SystemAdmin`, `Technician` e `StoreManager`; possono essere aggiunti ai ruoli personalizzati (vedi [Ruoli e permessi](../ruoli-permessi.md)).

## Utilizzo

La funzione si trova nella sezione ***Casse*** della **WebApp**: nella lista *Barriera casse* e nella [scheda della cassa](scheda-cassa.md).

<!-- TODO: aggiungere screenshot della finestra Sincronizzazione data/ora casse e del riquadro Orologio cassa -->

### Sincronizzare una cassa

1. aprire il menu della **singola cassa** (pulsante a destra della riga) → **Sincronizza data/ora**, oppure premere **Sincronizza data/ora** nella scheda della cassa;
2. confermare la domanda *Sincronizzare data e ora della cassa N con l'ora ufficiale?*;
3. durante la sincronizzazione la riga della cassa mostra *Sincronizzazione…*;
4. al termine compare una notifica: *Cassa N: data e ora impostate (02/10/2024 15:31:20, correzione +1 min 12 s)*, *Cassa N: orologio già allineato* oppure il motivo per cui la cassa non è stata sincronizzata.

La voce è visibile solo per le casse con trasmissione attiva ed è **disabilitata** (con il motivo mostrato passando sopra con il mouse) quando:

| Situazione | Motivo mostrato |
|---|---|
| sincronizzazione già in corso sulla cassa | *Sincronizzazione in corso* |
| Posware precedente alla 4.3.1012.0 | *Funzione non disponibile: richiesta versione Posware 4.3.1012.0 o successiva (installata …)* |
| versione di Posware non conosciuta | *Funzione non disponibile: richiesta versione Posware 4.3.1012.0 o successiva (versione Posware non rilevata)* |
| operatore collegato in cassa | *Operatore collegato: la cassa non è nella schermata di login* |

Le casse *Offline*, *Online - StandBy* o con stato sconosciuto possono essere sincronizzate: se la cassa non risponde l'esito sarà *Cassa non raggiungibile*.

### Finestra Sincronizzazione data/ora casse

Si apre dal menu **Operazioni** della barriera → **Sincronizzazione data/ora casse**. In alto c'è la sezione **Sincronizza ora**; sotto, due riquadri **chiusi** che si aprono facendo clic sull'intestazione: **Sincronizzazione automatica** (l'intestazione ne riporta lo stato, per esempio *Attiva, ogni giorno alle 07:30*) e **Orologio stampante fiscale — imposta su più casse**.

#### Sincronizza ora

Visibile con il permesso `store.devices.pos.datetimesync.execute`.

1. in alto è mostrata l'**ora ufficiale** (*Europe/Rome*, con l'indicazione di ora legale o solare) che avanza ogni secondo. Se timeapi.io non risponde compare l'avviso *Ora ufficiale non disponibile*, il pulsante di sincronizzazione è disabilitato ed è possibile premere **Riprova**;
2. sono elencate le casse con trasmissione attiva; quelle utilizzabili sono **tutte selezionate**:
    - le casse con **operatore collegato** sono evidenziate con l'avviso *verrà rifiutata se non torna alla schermata di login*, ma restano selezionabili;
    - le casse già **in sincronizzazione** non sono selezionabili;
    - le casse con **Posware precedente alla 4.3.1012.0** (o con versione non rilevata) non sono selezionabili e mostrano il motivo;
    - **Seleziona tutte** seleziona o deseleziona l'intero elenco;
3. premere **Sincronizza N casse**;
4. la finestra mostra l'avanzamento e poi l'esito di ogni cassa con la data e l'ora impostate e la correzione applicata; al termine il riepilogo *N sincronizzate / M non sincronizzate*;
5. se qualche cassa non è stata sincronizzata è disponibile **Riprova casse non sincronizzate**, che ripete l'operazione **solo** su quelle casse con una nuova lettura dell'ora ufficiale.

La finestra si può chiudere in qualsiasi momento: la sincronizzazione prosegue e le righe della lista continuano a mostrare *Sincronizzazione…* fino al termine.

#### Sincronizzazione automatica

Visibile con il permesso `store.devices.pos.datetimesync.manage`, in un riquadro chiuso.

| Campo | Descrizione |
|---|---|
| **Attiva / Disattiva** | accende o spegne la sincronizzazione automatica. **Alla prima installazione è disattiva** |
| **Orario** | orario italiano della sincronizzazione giornaliera (default **07:30**) |
| **Riprova per** | per quanto tempo, dopo l'orario, vengono ritentate le casse non sincronizzate: da *Nessun nuovo tentativo* a **2 ore**, a passi di 15 minuti (default 2 ore) |
| **Ogni** | intervallo tra un tentativo e il successivo: da **5** a 60 minuti, a passi di 5, mai più lungo del periodo *Riprova per* (default 15 minuti) |

Premere **Salva impostazioni** per applicarle: lo *StoreServer* le usa dal controllo successivo, entro pochi secondi.

Funzionamento:

1. ogni giorno all'**orario stabilito** lo *StoreServer* sincronizza **tutte le casse con trasmissione attiva**;
2. le casse non sincronizzate (ad esempio con un operatore collegato o spente) vengono **ritentate ogni intervallo** fino alla fine del periodo *Riprova per*; le casse già sincronizzate non vengono più contattate;
3. le casse con **Posware precedente alla 4.3.1012.0** non vengono contattate né ritentate: compaiono nel riepilogo con l'esito *Versione Posware non supportata*;
4. a ogni tentativo l'ora ufficiale viene letta di nuovo; se timeapi.io non è disponibile il tentativo viene ripetuto all'intervallo successivo;
5. la sincronizzazione automatica viene eseguita **una sola volta al giorno**. Se lo *StoreServer* si riavvia durante il periodo dei tentativi, riprende dalle casse non ancora sincronizzate.

Solo la sincronizzazione automatica può portare, dopo la chiusura fiscale, all'allineamento dell'orario della **stampante fiscale** (vedi [Orologio cassa](#orologio-cassa)); la sincronizzazione su richiesta cambia solo l'orologio di Windows.

Nella sezione sono indicate anche:

- la **prossima esecuzione** prevista;
- l'**ultima esecuzione**: data e ora dell'ultimo tentativo, stato (*In corso*, *Completata*, *Completata con errori*), numero di casse sincronizzate e non sincronizzate, numero di tentativi ed elenco delle casse non sincronizzate con il motivo.

!!! tip "Scegliere l'orario"
    Scegliere un orario in cui le casse sono **accese** ma **nessun operatore è collegato**, ad esempio prima dell'apertura del negozio. Una cassa spenta all'orario stabilito risulterà *Cassa non raggiungibile* fino a quando non viene accesa, entro il periodo dei tentativi.

!!! warning "Prima di attivarla"
    Verificare su **tutte le casse** la versione di Posware e il [servizio ApiGateway](#servizio-apigateway-sulle-casse), ad esempio con una sincronizzazione manuale. Una cassa con il servizio non configurato risulterà *Non sincronizzata* a ogni tentativo, tutti i giorni.

!!! note "Cambio ora legale / solare"
    L'orario è sempre l'orario italiano: la sincronizzazione segue automaticamente il passaggio tra ora legale e ora solare. Se l'orario stabilito non esiste (ad esempio le 02:30 del giorno in cui si passa all'ora legale) la sincronizzazione parte un'ora dopo.

#### Orologio stampante fiscale su più casse

Visibile con il permesso `store.devices.pos.datetimesync.manage`, in un riquadro chiuso. Permette di applicare la stessa [modalità della stampante fiscale](#modalita-della-stampante-fiscale) a più casse insieme, invece di impostarla cassa per cassa nel riquadro [Orologio cassa](#orologio-cassa).

1. scegliere la **modalità**; con *Data e ora automatica sulla stampante* compaiono anche:
    - **Server NTP**: se lasciato **vuoto** ogni cassa **mantiene** il server NTP già impostato; se indicato viene impostato su tutte le casse selezionate;
    - **Passaggio automatico ora legale/solare**: viene impostato su tutte le casse selezionate;

    con *Usa la data di Posware* compare l'interruttore **Se configurata, disattiva la sincronizzazione automatica della stampante** (vedi [Usa la data di Posware](#usa-la-data-di-posware)), impostato su tutte le casse selezionate; con le altre modalità viene disattivato;
2. scegliere le casse: sono elencate le casse con trasmissione attiva e quelle con **Posware 4.3.1012.0 o successiva** sono **tutte selezionate**; le casse con Posware precedente (o con versione non rilevata) non sono selezionabili e mostrano il motivo. Le casse con operatore collegato possono essere impostate;
3. premere **Applica a N casse**: le impostazioni vengono inviate a ogni cassa e sostituiscono quelle presenti;
4. accanto a ogni cassa compare l'esito: *Impostazioni salvate*, un avviso se la stampante della cassa non supporta la data e ora automatica o il passaggio automatico ora legale/solare, oppure il motivo del rifiuto (per esempio *La stampante fiscale della cassa si sincronizza tramite un server NTP: indicare il server NTP*, *Cassa non raggiungibile*). Al termine compare il riepilogo *N salvate / M non salvate*;
5. se qualche cassa non è stata impostata è disponibile **Riprova casse non salvate**, che invia le stesse impostazioni **solo** a quelle casse.

Ogni cassa impostata viene registrata nell'[audit](#audit) come una modifica dal riquadro *Orologio cassa*. Le stampanti ricevono la nuova configurazione alla **chiusura fiscale** successiva.

!!! tip "Stampanti con server NTP"
    Se alcune casse hanno stampanti che si sincronizzano tramite NTP, indicare il server NTP: lasciandolo vuoto quelle casse vengono rifiutate se non hanno già un server impostato.

### Orologio cassa

Nella [scheda della cassa](scheda-cassa.md), con il permesso `store.devices.pos.datetimesync.execute`, il riquadro **Orologio cassa** legge le informazioni direttamente dalla cassa (pulsante **Aggiorna** per rileggerle):

| Informazione | Descrizione |
|---|---|
| **Ora della cassa** | orologio della cassa al momento della lettura |
| **Ora ufficiale** | ora ufficiale nello stesso momento (*Non disponibile* se timeapi.io non risponde) |
| **Fuso orario di Windows** | fuso della cassa, con ora legale o solare; un fuso diverso da quello italiano è evidenziato con un avviso |
| scostamento | *La cassa è avanti di …*, *La cassa è indietro di …* oppure *Orologio allineato all'ora ufficiale* (meno di un secondo) |
| **Orario della stampante fiscale dopo la chiusura** | modalità di allineamento della stampante fiscale (vedi sotto) |
| **Ultima sincronizzazione automatica** / **manuale** | data e ora registrate dalla cassa all'ultima sincronizzazione |
| **Ultimo allineamento stampante** | data, esito e motivo dell'ultimo allineamento dopo la chiusura fiscale |

Se la cassa non ha la trasmissione attiva o ha un Posware precedente alla 4.3.1012.0 il riquadro ne indica il motivo e la cassa non viene contattata.

#### Modalità della stampante fiscale

La modalità stabilisce come viene mantenuto corretto l'orario della stampante fiscale dopo la chiusura fiscale. Si sceglie per ogni cassa, con il permesso `store.devices.pos.datetimesync.manage`, e si applica con **Salva impostazioni**; per impostarla su più casse insieme usare la [finestra di sincronizzazione](#orologio-stampante-fiscale-su-piu-casse):

| Modalità | Comportamento |
|---|---|
| **Data e ora automatica sulla stampante** | la stampante allinea da sola il proprio orologio, dall'Agenzia delle Entrate o da un server NTP a seconda del modello. A ogni chiusura fiscale Posware legge la configurazione della stampante e la reinvia solo se è cambiata (per esempio se la data e ora automatica è stata disattivata a mano) |
| **Usa la data di Posware** | dopo una **sincronizzazione automatica**, alla chiusura fiscale Posware verifica che l'orologio di Windows sia coerente con quella sincronizzazione e, solo se la data della stampante coincide, ne allinea l'**orario** (ore e minuti). La **data** della stampante non viene mai cambiata |
| **Nessun allineamento** | impostazione predefinita: l'orario della stampante fiscale non viene mai modificato |

Le due prime modalità non possono essere attive insieme. Se la cassa le riporta entrambe attive compare un avviso e nessun allineamento viene eseguito finché non si salvano le impostazioni.

##### Data e ora automatica sulla stampante

Selezionando la modalità *Data e ora automatica sulla stampante* il riquadro mostra:

| Campo | Descrizione |
|---|---|
| **Data e ora automatica della stampante** | come si sincronizza il modello di stampante della cassa: *dall'Agenzia delle Entrate*, *da server NTP*, *non supportata dalla stampante* oppure *non rilevata* se la stampante non è disponibile. Con una stampante che non la supporta compare un avviso: l'orario della stampante non viene allineato |
| **Server NTP** | nome host o indirizzo IP (senza porta) del server NTP. È **obbligatorio** per le stampanti che si sincronizzano tramite NTP; per le altre viene ignorato. Lasciarlo vuoto per cancellarlo |
| **Passaggio automatico ora legale/solare** | se attivo, viene inviato alla stampante insieme alla data e ora automatica: la stampante passa da sola tra ora legale e ora solare. Se la stampante non lo supporta l'impostazione viene salvata ma non inviata, e il riquadro lo segnala. Predefinito: disattivo |

Server NTP e passaggio automatico ora legale/solare vengono inviati alla cassa solo con questa modalità; scegliendo un'altra modalità restano memorizzati sulla cassa senza modifiche. La stampante riceve la nuova configurazione alla **chiusura fiscale** successiva.

Il salvataggio viene rifiutato con un messaggio quando:

| Messaggio | Significato |
|---|---|
| *Server NTP non valido: indicare un nome host o un indirizzo IP, senza porta* | il server indicato non è un nome host o un indirizzo IP valido |
| *La stampante fiscale della cassa si sincronizza tramite un server NTP: indicare il server NTP* | la stampante richiede un server NTP e il campo è vuoto |

Dopo la chiusura fiscale *Ultimo allineamento stampante* riporta:

| Esito | Significato |
|---|---|
| *Data e ora automatica configurata sulla stampante* | la configurazione è stata inviata alla stampante |
| *Nessuna modifica necessaria: data e ora automatica già configurata sulla stampante* | la stampante era già configurata correttamente: non è stato inviato nulla |
| *Orario della stampante non modificato: la stampante non supporta la data e ora automatica* | il modello di stampante non la supporta: scegliere un'altra modalità |
| *Orario della stampante non modificato: server NTP non configurato, obbligatorio per la stampante della cassa* | indicare il server NTP |
| *Orario della stampante non modificato: configurazione della data e ora automatica non riuscita* | la stampante ha rifiutato la configurazione: verificare la stampante |
| *Orario della stampante non modificato: stampante fiscale non abilitata* | la stampante fiscale non è abilitata in Posware |
| *Orario della stampante non modificato: giornata fiscale ancora aperta* | la stampante non ha confermato la chiusura |

##### Usa la data di Posware

Selezionando la modalità *Usa la data di Posware* il riquadro mostra l'interruttore **Se configurata, disattiva la sincronizzazione automatica della stampante** (predefinito: disattivo):

- **attivo**: alla chiusura fiscale, se i controlli sull'orario sono superati e sulla stampante la data e ora automatica è attiva (o non leggibile), Posware la **disattiva** e poi allinea l'orario. Così la stampante non sovrascrive più l'orario impostato da Posware. Se l'allineamento viene rifiutato (vedi sotto) la data e ora automatica resta attiva;
- **disattivo**: la data e ora automatica della stampante resta com'è; se è attiva la stampante può sincronizzarsi da sola in altri momenti, e nel log della cassa viene scritto un avviso.

Se la stampante della cassa non ha la data e ora automatica il riquadro indica che l'impostazione non ha effetto. Con le altre modalità l'interruttore viene salvato **disattivo**.

Se la disattivazione non riesce, l'orario viene comunque allineato e *Ultimo allineamento stampante* riporta, in arancione, *Orario della stampante allineato: disattivazione della data e ora automatica della stampante non riuscita*; Posware riprova alla chiusura fiscale successiva.

L'orario della stampante **non** viene modificato, e il motivo compare in *Ultimo allineamento stampante*, quando:

| Motivo | Significato |
|---|---|
| *nessuna sincronizzazione automatica memorizzata* | la cassa non ha ancora ricevuto una sincronizzazione automatica |
| *fuso orario di Windows cambiato dopo la sincronizzazione* | il fuso di Windows è diverso da quello registrato alla sincronizzazione |
| *l'orologio di Windows è precedente all'ultima sincronizzazione* | l'orologio è stato spostato indietro a mano |
| *chiusura fiscale eseguita il giorno successivo alla sincronizzazione* | la chiusura è avvenuta dopo la mezzanotte: l'allineamento avverrà dopo la sincronizzazione del giorno |
| *giornata fiscale ancora aperta* | la stampante non ha confermato la chiusura |
| *ora della stampante non leggibile* | la stampante non ha restituito data e ora |
| *data della stampante diversa da quella di Windows: cambio di data non ammesso* | per sicurezza la data della stampante non viene mai cambiata: verificare la stampante |
| *impostazione dell'orario della stampante non riuscita* | la stampante ha rifiutato il comando |

### Esiti

| Esito | Colore | Significato |
|---|---|---|
| *Data e ora impostate — 02/10/2024 15:31:20 (+1 min 12 s)* | verde | l'orologio della cassa è stato impostato all'ora ufficiale; tra parentesi la correzione applicata (positiva se la cassa era indietro) |
| *Orologio già allineato* | verde | la differenza era inferiore alla tolleranza (`ToleranceMs`): l'orologio non è stato modificato |
| *Non sincronizzata: operatore collegato in cassa* | arancione | effettuare il logout in cassa e riprovare |
| *Non sincronizzata: scontrino o transazione in corso* | arancione | chiudere lo scontrino e riprovare |
| *Non sincronizzata: chiusura fiscale o altra operazione in corso sulla cassa* | arancione | attendere la fine dell'operazione e riprovare |
| *Non sincronizzata: un altro cambio orario è in corso sulla cassa* | arancione | riprovare tra qualche secondo |
| *Non sincronizzata: il programma di cassa non è avviato o non risponde* | rosso | avviare Posware sulla cassa |
| *Non sincronizzata: il servizio ApiGateway della cassa non ha il permesso di modificare l'ora* | rosso | configurare il [servizio ApiGateway](#servizio-apigateway-sulle-casse) come LocalSystem |
| *Non sincronizzata: impostazione dell'ora non riuscita sulla cassa* | rosso | Windows ha rifiutato il cambio o l'orologio riletto non corrisponde: verificare il log della cassa |
| *Cassa non raggiungibile (timeout)* | rosso | nessuna risposta entro il tempo previsto |
| *Cassa non raggiungibile: servizio ApiGateway non attivo o rete non disponibile* | rosso | cassa spenta, servizio ApiGateway fermo o porta 6857 bloccata |
| *Risposta non riconosciuta* | rosso | la cassa ha risposto con un errore non previsto |
| *Versione Posware non supportata* | grigio | Posware precedente alla 4.3.1012.0 o versione non rilevata: la cassa non è stata contattata |
| *Cassa scollegata* | grigio | la cassa non è abilitata alla trasmissione o non ha indirizzo IP: la cassa non è stata contattata |
| *Operazione interrotta (riavvio server)* | grigio | lo *StoreServer* è stato riavviato durante la sincronizzazione: ripetere l'operazione |

Un esito verde può riportare in coda un avviso, ad esempio *attenzione: fuso orario della cassa diverso dall'ora italiana*.

Possono inoltre comparire questi messaggi:

| Messaggio | Causa |
|---|---|
| *Ora ufficiale non disponibile (timeapi.io): impossibile sincronizzare le casse. Riprovare tra qualche minuto.* | lo *StoreServer* non riesce a leggere l'ora ufficiale: verificare l'accesso a internet e al sito timeapi.io |
| *Alcune casse sono già in sincronizzazione: N, M* | un altro utente, un'altra finestra o la sincronizzazione automatica stanno già sincronizzando quelle casse: attendere l'esito |
| *Cassa non trovata* | la cassa non esiste più nell'anagrafica casse |
| *Permessi insufficienti per sincronizzare le casse* | l'utente non dispone del permesso |

### Audit

Ogni operazione viene registrata nel log di audit dello *StoreServer*:

- azione `pos.datetimesync.requested`: utente (oppure `system` per la sincronizzazione automatica), origine, elenco delle casse, ora ufficiale di riferimento e scostamento dell'orologio dello *StoreServer*;
- azione `pos.datetimesync`, una per cassa al termine: utente richiedente, origine (`Gui` o `Scheduled`), esito, istante UTC inviato, correzione applicata, durata, identificativo di correlazione (riportato anche nel log della cassa) e dettaglio della risposta;
- azione `pos.datetimesync.autosync.updated`: utente e nuove impostazioni della sincronizzazione automatica;
- azione `pos.datetimesync.clocksettings.updated`: utente, cassa, nuova modalità della stampante fiscale, disattivazione della sincronizzazione automatica della stampante e, se inviati, server NTP e passaggio automatico ora legale/solare.

??? question "Domande frequenti"
    **La voce *Sincronizza data/ora* è disabilitata con il messaggio *richiesta versione Posware 4.3.1012.0*.**
    La cassa ha una versione di Posware precedente: aggiornarla. Se la versione mostrata è già corretta, premere **Aggiorna status** nella scheda cassa per rileggerla dalla cassa. In ogni caso, all'avvio della sincronizzazione lo *StoreServer* chiede la versione alla cassa quando quella registrata manca o non è supportata.

    **La cassa risulta *Non sincronizzata: operatore collegato in cassa* anche se non c'è nessuno alla cassa.**
    Un operatore è rimasto collegato: effettuare il logout in cassa e ripetere la sincronizzazione.

    **L'esito è *il servizio ApiGateway della cassa non ha il permesso di modificare l'ora*.**
    Il servizio `PoswareApiGatewayWatchdog` non gira come *LocalSystem*: correggerlo come descritto in [Servizio ApiGateway sulle casse](#servizio-apigateway-sulle-casse).

    **L'orologio del PC dello *StoreServer* è sbagliato: è un problema?**
    No. L'orario inviato alle casse è sempre quello ufficiale letto da timeapi.io. L'orologio dello *StoreServer* serve solo a decidere quando far partire la sincronizzazione automatica; lo scostamento viene segnalato nel log.

    **Perché la sincronizzazione manuale non allinea la stampante fiscale?**
    Per sicurezza solo la sincronizzazione automatica, eseguita dallo *StoreServer* all'orario stabilito, può portare a un cambio dell'orario della stampante fiscale dopo la chiusura.

    **Con *Data e ora automatica sulla stampante* devo indicare il server NTP?**
    Solo se il riquadro indica che la stampante si sincronizza *da server NTP*: in quel caso il server è obbligatorio. Le stampanti che si sincronizzano dall'Agenzia delle Entrate non ne hanno bisogno.

    **Ho attivato il passaggio automatico ora legale/solare ma la stampante non cambia ora.**
    Verificare nel riquadro che la stampante lo supporti. La configurazione viene inviata alla stampante alla chiusura fiscale successiva al salvataggio.

    **La sincronizzazione automatica non è partita.**
    Verificare che sia *Attiva* e che lo *StoreServer* fosse in funzione all'orario stabilito. Se lo *StoreServer* viene avviato dopo l'orario, la sincronizzazione parte comunque se si è ancora nel periodo *Riprova per* (o nei 10 minuti successivi all'orario, se il periodo è più breve); altrimenti viene eseguita il giorno successivo.

    **Ho cambiato l'orario dopo che la sincronizzazione del giorno è già stata eseguita.**
    Il nuovo orario vale dal giorno successivo: la sincronizzazione automatica viene eseguita una sola volta al giorno.

## Differenze rispetto a Pos_Utility

- data e ora **non si digitano più**: si usa sempre l'ora ufficiale italiana, mai l'orologio di un PC;
- l'orologio viene impostato **al secondo** e indipendentemente dal fuso orario della cassa;
- **sincronizzazione automatica giornaliera** con nuovi tentativi sulle casse non sincronizzate;
- sincronizzazione della **singola cassa** dal menu della cassa e dalla scheda cassa, oltre che di più casse insieme;
- casse selezionabili (il programma legacy inviava sempre a tutte);
- esito **per ogni cassa** con il motivo preciso di un eventuale rifiuto;
- lettura dell'orologio della cassa e scelta della modalità di allineamento della stampante fiscale, con server NTP e passaggio automatico ora legale/solare;
- casse contattate in parallelo e operazione eseguita dallo *StoreServer*: la WebApp non si blocca;
- accesso tramite utente *StoreServer* e permessi dedicati; ogni richiesta e ogni esito sono tracciati nel log di audit;
- richiede **Posware 4.3.1012.0** o successiva sulle casse.
