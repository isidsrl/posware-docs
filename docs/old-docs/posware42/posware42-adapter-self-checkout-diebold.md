# Posware 4.2 - Adapter Self-CheckOut Diebold
#### Note di attivazione ed utilizzo
Data aggiornamento documento: **11/11/2021** <br>
Ultima versione rilasciata: `4.2.0.0`

---
## Versioni software di riferimento


- Posware **4.2.0.x** con Adapter Diebold SCO **4.2.0.0**
 


## Cenni preliminari




### Posware
La versione minima di posware richiesta è la **4.2**. E' richiesta l'installazione del **.net Framework 4.8**



### Registratori telematici
A far data **05/01/2021** le stampanti RT certificate con la SCO Diebold sono:

* Epson - tutti i modelli

### Scenari di upgrade supportati
È possibile aggiornare alla versione `4.2.x` dell'adattatore a partire dalla versione `2.0.0.14`. <br>
Qualora l'adattatore non fosse già aggiornato almeno alla versione `2.0.0.14`, fare riferimento al documento `Posware 4.1 - Adapter Self-CheckOut Diebold` per dettagli su come aggiornare.

### Breaking Changes
Non sono presenti breaking changes.

## Nuove funzionalità

### Compatibilità con la versione 4.2.x di Posware
L'adattatore è compatibile con la versione `4.2.x` di Posware

### Arrotondamento dei pagamenti contanti dal registratore telematico
L'adattatore supporta la feature di arrotondamento contanti da parte del registratore telematico. <br>
Fare riferimento al documento `Posware 4.2 - Funzionalità RT 2.0` per la configurazione della feature.

### Vendita prodotti come servizi e gestione multi-ateco
L'adattatore supporta le feature di vendita prodotti come servizi e la gestione multi-ateco. <br>
Fare riferimento al documento `Posware 4.2 - Funzionalità RT 2.0` per la configurazione della feature.

## Funzionalità riportate dalla versione `2.x`
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
> Prima di procedere con l'upgrade, assicurarsi di avere installata almeno la versione `2.0.0.14`.

1. Aggiornare Posware all'ultima versione tramite il setup `PosUpdate` (Runtime **.net 4.8**).
2. Sovrascrivere l'eseguibile dell'adattatore `Posware_Sco.exe` con la versione aggiornata fornita.

**Per il supporto alla lotteria degli scontrini, è necessario aggiornare i workflow della SCO. Contattare Diebold per le istruzioni sull'adeguamento.** 
> Senza adeguare il software sarà comunque possibile utilizzare la versione **2.0.0.0**. Saranno disponibili tutte le funzionalità al netto della **lotteria degli scontrini**.