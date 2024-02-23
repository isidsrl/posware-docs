# Posware 4.2 - Adapter Self-CheckOut NCR
#### Note di attivazione ed utilizzo
Data aggiornamento documento: **11/11/2021** <br>
Ultima versione rilasciata: `4.2.0.0`

---
> ## Versioni software di riferimento
>
>
> - Posware **4.2.0.0** richiesto dall'adattatore **4.2.0.0**



## Cenni preliminari
È disponibile l'elenco esaustivo delle funzionalità supportate dall'adattatore. Fare riferimento al documento `Posware 4.2 - Funzionalità Adapter NCR SelfServ™ Checkout`. 
### Posware
La versione minima di posware richiesta è la **4.2.0.0**. E' richiesta l'installazione del **.net Framework 4.0**


A far data **05/01/2021**, l'ambiente SCO NCR **non è compatibile con il .net Framework 4.8**. Pertanto sarà necessario installare Posware per Windows XP tramite il setup `PosUpdateNET40`. 

> ATTENZIONE: Non installare il .net Framework 4.8 sull'ambiente SCO NCR, il software SCO smette di funzionare.

Il setup permetterà l'installazione della versione .net 4.0 rilevando l'ambiente NCR SCO.

### Registratori telematici
A far data **05/01/2021** i registratori telematici certificati con la SCO NCR sono:

* Epson - tutti i modelli

A far data **11/11/2021** i registratori telematici compatibili, ma non certificati / supportati, sono:

* NCR modello 130A

### Scenari di upgrade supportati
È possibile aggiornare alla versione `4.2.x` dell'adattatore a partire dalla versione `2.0.0.14`. <br>
Qualora l'adattatore non fosse già aggiornato almeno alla versione `2.0.0.14`, fare riferimento al documento [`Posware 4.1 - Adapter Self-CheckOut NCR`](../posware41/posware41-adapter-self-checkout-ncr.md) per dettagli su come aggiornare.

### Breaking Changes
Non sono presenti breaking changes.

## Nuove funzionalità

### Compatibilità con la versione 4.2.x di Posware
L'adattatore è compatibile con la versione `4.2.x` di Posware

### Arrotondamento dei pagamenti contanti dal registratore telematico
L'adattatore supporta la feature di arrotondamento contanti da parte del registratore telematico. <br>
Fare riferimento al documento  [`Posware 4.2 - Funzionalità RT 2.0`](posware42-funzionalita-rt-20.md) per la configurazione della feature.

### Vendita prodotti come servizi e gestione multi-ateco
L'adattatore supporta le feature di vendita prodotti come servizi e la gestione multi-ateco. <br>
Fare riferimento al documento [`Posware 4.2 - Funzionalità RT 2.0`](posware42-funzionalita-rt-20.md) per la configurazione della feature.

## Funzionalità riportate dalla versione `2.x`
### Lotteria degli scontrini
Dalla versione **2.0.0.14** è possibile inserire il codice lotteria tramite SCO. E' abilitato l'inserimento sia manuale che tramite scanner. <br>
Il pulsante `Codice Lotteria` appare nella schermata dei pagamenti (in base alla configurazione del cliente potrebbe essere altrove).

* Una volta accettato il codice lotteria, viene confermata l'operazione tramite messaggio popup.
* E' possibile sovrascrivere il codice lotteria con un'altro ripetendo l'operazione. L'ultimo codice inserito sarà quello inviato all'Ade.
* Una volta inserito il codice lotteria non è previsto poterlo rimuovere
* Se il cliente digita un codice lotteria **vuoto**, viene rifiutato e segnalato con un messaggio
* Se il cliente digita un codice lotteria inesistente, viene comunque accettato e seguirà le regole previste dall'AdE (non parteciperà alla lotteria)

> E' necessario un adeguamento del software SCO per poter utilizzare la feature. Contattare NCR per gli adeguamenti necessari.

### Gestione del periodo di inattività RT - Auto end of day
Dalla versione **2.0.0.12** è possibile abilitare l'end of day (fine giornata) automatico di posware. Per farlo abilitare i seguenti parametri nella tabella `tabparametriextra`:

| Modulo     | Parametro | Valore | Environment
| ----------- | :----------------: |------|-------|
`posware.posdevice` | `isSelfCheckOutEnvironment` | `true` | `production`
`posware.fiscalprinter` | `enableRTEODAfterInactivePeriod` | `true` | `production`

Posware effettuerà o meno l'end of day previsto da normativa RT se superato il tempo massimo consentito.
> Per poter effettuare con successo l'operazione, all'avvio dell'adattatore la stampante deve essere rilevata ed inizializzata con successo. Qualora in fase di avvio l'inizializzazione delle perifiche dovesse presentare errori, lo stato di "end of day richiesto" non potrà essere rilevato. Di conseguenza non la chiusura automatica non verrà lanciata.

### Gestione dei prodotti con restrizioni
Se non previsto dal software integrato della SCO, dalla versione **2.0.0.13** è possibile indicare nella tabella articoli di posware i flag per la vendita di articoli soggetti a restrizioni.
> Le due gestioni sono mutuamente esclusive. Se il software SCO gestisce autonomamente la vendita con restrizioni, i flag su posware non vanno impostati.

I campi previsti della tabella articoli sono:

| Field     | Valore   | Descrizione | Note |
| ----------- | :----------------: |------|---|
| Sco_Prod_Leggero | 0 non attivo / 1 attivo | Il prodotto è troppo leggero e non deve essere pesato dalla bilancia della SCO|
| Sco_Prod_NonPesare | 0 non attivo / 1 attivo | Il prodotto non deve essere pesato dalla bilancia della SCO|
| Sco_Prod_Ingombrante | 0 non attivo / 1 attivo | Il prodotto non deve essere pesato dalla bilancia della SCO e non deve essere rilevato dal contenuto sacchetto|
| Sco_Prod_AltoRischio | 0 non attivo / 1 attivo (verifica età) | Il prodotto per essere venduto richiede una verifica dell'età oppure l'approvazione da parte del personale| A partire dalla versione **2.0.0.15**
| Sco_Prod_AltoRischio | 0 non attivo / 2 attivo (approvazione visiva) | Il prodotto per essere venduto richiede una verifica visiva ed approvazione da parte del personale|

##### A partire dalla versione `2.0.0.15`
 
La gestione del prodotto ad alto rischio, quando impostata ad `1`, supporta la lettura della data di nascita dalla SCO. Se l'utente inserisce la data di nascita durante la richiesta di approvazione, l'adattatore mostra sulla SCO i prodotti che devono essere rimossi dalla spesa per il mancato raggiungimento dell'età. La successiva rimozione dovrà, eventualmente, essere eseguita **manualmente** dal cliente o dal personale del negozio. Non sono previste forzature in merito nel workflow della SCO.

## Procedura di installazione
> Prima di procedere con l'upgrade, assicurarsi di avere installata almeno la versione `2.0.0.14`.

1. Aggiornare Posware in versione **.net 4.0** , tramite il setup `PosUpdateNET40`. **Non installare** la versione .net **4.8** ne tantomeno il runtime .net 4.8
2. Sovrascrivere l'eseguibile dell'adattatore `Posware_NCRSco.exe` con la versione aggiornata fornita.

**Per il supporto alla lotteria degli scontrini, se l'ultimo aggiornamento del software a bordo NCR SCO è antecedente al 10/01/2021, sarà necessario contattare NCR per le istruzioni sull'adeguamento.** 
> Senza adeguare il software sarà comunque possibile utilizzare la versione **2.0.0.13**. Saranno disponibili tutte le funzionalità tranne che il supporto **alla lotteria degli scontrini**.

