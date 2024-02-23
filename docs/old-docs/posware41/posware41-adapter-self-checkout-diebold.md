# Posware 4.1 - Adapter Self-CheckOut Diebold
#### Note di attivazione ed utilizzo
Data aggiornamento documento: **31/05/2021** <br>
Ultima versione rilasciata: `2.0.0.17`

---
> ## Versioni software di riferimento
>
>
> - Posware **4.1.404.0** con Adapter Diebold SCO **2.0.0.15**
> - Posware **4.1.432.0** con Adapter Diebold SCO **2.0.0.16**
> - Posware **4.1.516.0** con Adapter Diebold SCO **2.0.0.17** o superiore
>

## Cenni preliminari

### Posware
La versione minima di posware richiesta è la **4.1.404.0**. E' richiesta l'installazione del **.net Framework 4.8** <br>
Consultare le note di installazione della versione 4.1.x per info approfondite.

### Stampanti
A far data **05/01/2021** le stampanti RT certificate con la SCO Diebold sono:

* Epson - tutti i modelli

### Breaking Changes
A partire dalla release **2.0.0.15** con Posware **4.1**, le connection string di Posware vengono elaborate solo dal file `Posware.exe.config` e non più dal file config dell'adattatore `Posware_Sco.exe.config` . <br>
Per evitare errori di invio dei dati al server centrale, verificare l'allineamento dei parametri `ConnessioneServer` e `Connessione` del file `Posware_Sco.exe.config` e riportarli eventualmente nel file `Posware.exe.config`

## Nuove funzionalità
### Lotteria degli scontrini
Dalla versione **2.0.0.15** è possibile inserire il codice lotteria tramite SCO. E' abilitato l'inserimento sia manuale che tramite scanner. <br>
Il pulsante `Codice Lotteria` appare nella schermata principale (in base alla configurazione del cliente potrebbe essere altrove).

* Una volta accettato il codice lotteria, viene confermata l'operazione tramite messaggio.
* E' possibile sovrascrivere il codice lotteria con un'altro ripetendo l'operazione. L'ultimo codice inserito sarà quello inviato all'Ade.
* Una volta inserito il codice lotteria non è previsto poterlo rimuovere
* Se il cliente digita un codice lotteria **vuoto**, viene rifiutato e segnalato con un messaggio
* Se il cliente digita un codice lotteria inesistente, viene comunque accettato e seguirà le regole previste dall'AdE (non parteciperà alla lotteria)

> E' necessario un adeguamento dei workflow della SCO per poter utilizzare la feature. Contattare Diebold per gli adeguamenti necessari.

### Gestione del periodo di inattività RT - Auto end of day
Dalla versione **2.0.0.15** è possibile abilitare l'end of day (fine giornata) automatico di posware. Per farlo abilitare i seguenti parametri nella tabella `tabparametriextra`:

| Modulo     | Parametro | Valore | Environment
| ----------- | :----------------: |------|-------|
`posware.posdevice` | `isSelfCheckOutEnvironment` | `true` | `production`
`posware.fiscalprinter` | `enableRTEODAfterInactivePeriod` | `true` | `production`

Posware effettuerà o meno l'end of day previsto da normativa RT se superato il tempo massimo consentito.
> Per poter effettuare con successo l'operazione, all'avvio dell'adattatore la stampante deve essere rilevata ed inizializzata con successo. Qualora in fase di avvio l'inizializzazione delle perifiche dovesse presentare errori, lo stato di "end of day richiesto" non potrà essere rilevato. Di conseguenza non la chiusura automatica non verrà lanciata.

## Procedura di installazione
> Prima di procedere con l'installazione, è necessario allineare il file di configurazione dell'adattatore `Posware_Sco.exe.config` con il contenuto del file di riferimento `Posware_Sco.exe.config.reference`. <br>
> E' necessario effettuare questa operazione solo la prima volta che si aggiorna dalla versione 1.x.x.x dell'adattatore.

1. Aggiornare Posware all'ultima versione tramite il setup `PosUpdate` (Runtime **.net 4.8**).
2. Sovrascrivere l'eseguibile dell'adattatore `Posware_Sco.exe` con la versione aggiornata fornita.
3. Sovrascrivere il file di configurazione `Posware_Sco.exe.config` con l'ultimo aggiornato se la versione precedentemente installata era inferiore alla **2.0.0.0** (Posware 4.0). 

**Per il supporto alla lotteria degli scontrini, è necessario aggiornare i workflow della SCO. Contattare Diebold per le istruzioni sull'adeguamento.** 
> Senza adeguare il software sarà comunque possibile utilizzare la versione **2.0.0.0**. Saranno disponibili tutte le funzionalità al netto della **lotteria degli scontrini**.