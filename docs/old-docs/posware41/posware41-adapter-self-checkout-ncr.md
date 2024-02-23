# Posware 4.1 - Adapter Self-CheckOut NCR
#### Note di attivazione ed utilizzo
Data aggiornamento documento: **31/05/2021** <br>
Ultima versione rilasciata: `2.0.0.17`

---
> ## Versioni software di riferimento
>
>
> - Posware **4.1.384.0** richiesto dall'adattatore **2.0.0.14**
> - Posware **4.1.432.0** richiesto dall'adattatore **2.0.0.16**
> - Posware **4.1.516.0** richiesto dall'adattatore **2.0.0.17** o superiore
>

## Cenni preliminari

### Posware
La versione minima di posware richiesta è la **4.1.384.0**. E' richiesta l'installazione del **.net Framework 4.0** <br>
Consultare le note di installazione della versione 4.1.x per info approfondite.

A far data **05/01/2021**, l'ambiente SCO NCR **non è compatibile con il .net Framework 4.8**. Pertanto sarà necessario installare Posware per Windows XP tramite il setup `PosUpdateNET40`. 

> ATTENZIONE: Non installare il .net Framework 4.8 sull'ambiente SCO NCR, il software SCO smette di funzionare.

Il setup permetterà l'installazione della versione .net 4.0 rilevando l'ambiente NCR SCO.

### Stampanti
A far data **05/01/2021** le stampanti RT certificate con la SCO NCR sono:

* Epson - tutti i modelli
* Stampante NCR fornita con la SCO

### Breaking Changes
A partire dalla release **2.0.0.12** con Posware **4.1**, le connection string di Posware vengono elaborate solo dal file `Posware.exe.config` e non più dal file config dell'adattatore `Posware_NCRSco.exe.config` . <br>
Per evitare errori di invio dei dati al server centrale, verificare l'allineamento dei parametri `ConnessioneServer` e `Connessione` del file `Posware_NCRSco.exe.config` e riportarli eventualmente nel file `Posware.exe.config`

## Nuove funzionalità
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
> Prima di procedere con l'installazione, è necessario allineare il file di configurazione dell'adattatore `Posware_NCRSco.exe.config` con il contenuto del file di riferimento `Posware_NCRSco.exe.config.reference`. <br>
> E' necessario effettuare questa operazione solo la prima volta che si aggiorna dalla versione 1.x.x.x oppure 2.0.0.11 dell'adattatore.

1. Aggiornare Posware in versione **.net 4.0** , tramite il setup `PosUpdateNET40`. **Non installare** la versione .net **4.8** ne tantomeno il runtime .net 4.8
2. Sovrascrivere l'eseguibile dell'adattatore `Posware_NCRSco.exe` con la versione aggiornata fornita.
3. Sovrascrivere il file di configurazione `Posware_NCRSco.exe.config` con l'ultimo aggiornato se la versione precedentemente installato era inferiore alla **2.0.0.12** (Posware 4.0). 

**Per il supporto alla lotteria degli scontrini, se l'ultimo aggiornamento del software a bordo NCR SCO è antecedente al 10/01/2021, sarà necessario contattare NCR per le istruzioni sull'adeguamento.** 
> Senza adeguare il software sarà comunque possibile utilizzare la versione **2.0.0.13**. Saranno disponibili tutte le funzionalità tranne che il supporto **alla lotteria degli scontrini**.

