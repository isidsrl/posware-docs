# Fast4Cash Plug-In per Posware
#### Manuale di attivazione ed utilizzo

 Il presente documento descrive i requisiti minimi, le configurazioni e le procedure necessarie all'attivazione ed utilizzo del plug-in dedicato a Fast4Cash (F4C), app di gestione device di Cash Management.

##### Data di creazione del documento: 21/05/2021
##### Ultima revisione documento: 13/12/2021

---

## Cenni preliminari
### Versioni software
La versione minima di **Posware** richiesta è la v. `4.1.500` per Windows 7/8/10. <br>
La versione certificata di **Fast4Cash** è la v. `4.0.0.407`
> Posware per Windows XP (deprecato) non è supportato ed il supporto non è previsto.


### Device supportati
A far data **19/05/2021** sono supportati i seguenti device:

* NCR CTM - Release HW R6
* GLORY - Serie CI10


## Installazione e configurazione
### Procedura di installazione
Il plug-in è incluso out-of-the-box in Posware, nel pacchetto di installazione `PosUpdate`. <br>
Aggiornando Posware alla versione minima supportata, il plug-in è subito configurabile e non sono necessarie ulteriori installazioni.

### Configurazione
#### Impostazioni
Di seguito le impostazioni configurabili del plug-in nella tabella `tabparametriextra`:

| Modulo      | Parametro   | Valore    | Valore di default    | Note |
| ----------- | ----------- |-----------|----------------|------|
| `posware.plugins.f4c`    | `communicationPort`|Porta di comunicazione verso il driver F4C.| `6363` |
| `posware.plugins.f4c`    | `ipAddress`|Indirizzo IP da usare per la comunicazione verso il driver F4C| `127.0.0.1` |
| `posware.plugins.f4c`    | `waitTimeBeforePaymentStateRequestInMilliseconds`|Tempo di attesa *in ms* da osservare tra l'avvenuto incasso e la richiesta stato pagamento | `2000` | Generalmente non è necessario intervenire su questo parametro a meno di indicazioni specifiche da parte di IT-Avantec. Valore minimo **200ms**, massimo **3000ms**
| `posware.plugins.f4c`    | `overallPaymentProcessTimeoutInMinutes`|Tempo massimo permesso per l'intero processo di pagamento, *in minuti*| `10` |In caso di blocco permanente del device o del driver F4C, dopo questo lasso di tempo Posware annulla il pagamento e ritorna alla schermata principale. In assenza del doppio monitor permette di chiudure la transazione senza dover riavviare l'intero PoS che rimarrebbe bloccato sulla schermata del driver F4C. E' da considerare come uscita di emergenza. Valore minimo **2**, massimo **15**
| `posware.plugins.f4c`    | `enableF4CMessageLogging`|true/false| `true` |Se abilitato, viene generato il log dettagliato dei messaggi inviati e ricevuti tra Posware e F4C.
| `posware.plugins.f4c`    | `enableUserAbort`|true/false| `true` |Se abilitato, viene concesso all'operatore di poter Annullare l'operazione verso F4C. Leggere attenamente i paragrafi Workflow di utilizzo e F.A.Q.
| `posware.plugins.f4c`    | `showGUIOnRefund`|true/false| `false` |Se abilitato, durante l'operazione di restituzione denaro (in caso di annullo pagamenti o annullo scontrino), viene visualizzata l'interfaccia di scelta di F4C. **Supportato a partire da Posware 4.1.560**
| `posware.plugins.f4c`    | `floatReportFilePath`|Percorso del file dei versamenti effettuati tramite il management di F4C| `C:\IT-Avantec\F4C\Management\xml\CTMFloatReport.xml` |Richiamando la funzione di versamento o prelievo da Posware, dopo aver effettuato le operazioni nel management di F4C, verrà letto questo file ottenere i dati del versato. **Supportato a partire da Posware 4.2**
| `posware.plugins.f4c`    | `pickupReportFilePath`|Percorso del file dei prelievi effettuati tramite il management di F4C| `C:\IT-Avantec\F4C\Management\xml\CTMPickupReport.xml` |Richiamando la funzione di versamento o prelievo da Posware, dopo aver effettuato le operazioni nel management di F4C, verrà letto questo file ottenere i dati del prelevato. **Supportato a partire da Posware 4.2**

Tutte le configurazioni, *se non presenti in `tabparametriextra`*, al primo utilizzo del plug-in vengono **inizializzate automaticamente al valore di default riportato in tabella**. <br>
In caso di valore imputato fuori dal range permesso, vengono utilizzati i valori di default senza causare blocchi o notificare errori.

#### Attivazione sul front-end
Per utilizzare il plug-in sono richiesti 3 passaggi:

1. Configurarlo come gestore di Cash Management per l'inizializzazione.
2. Inserire un pulsante per richiamare il pagamento
3. Inserire un pulsante per richiamare il gestore del device

##### 1. Attivare il plug-in di F4C come gestore di Cash Management
Per inizializzare il plug-in, è necessario configurarlo come gestore di Cash Management. <br>
A tal fine, configurare le voci nella tabella `tabparametriextra` con i valori di seguito riportati:

| Modulo      | Parametro   | Valore    | 
| ----------- | ----------- |-----------|
`CASHMANAGEMENT`|`LIBRERIA`|`Posware.F4C.dll`
`CASHMANAGEMENT`|`NOMECLASSE`|`Posware.F4C.Fast4CashCashManagementHandler`

Dopo aver configurato i parametri, avviare Posware. Verrà inizializzato il plug-in con tutte le sue impostazioni di default, all'interno di `tabparametriextra`, per essere immediatamente utilizzabile.

##### 2. Pulsante di pagamento
1. Aggiungere un nuovo record alla tabella `tipi_pagamenti`. I campi obbligatori per definire il pagamento come Cash Management sono:

    | Resto      | EFT   | PagTipo    | TipoPagRT    |
    | ----------- | ----------- |-----------|----------------|
    | `1`    | `true`|`1`| `0`

    > I campi restanti possono essere configurati in base alle esigenze.

2. Aggiungere il pulsante alla grafica ed agganciare il nuovo pagamento

##### 3. Pulsante per richiamare il gestore
1. Aggiungere un nuovo record nella tabella `config_tast` inserendo nel campo `funzione` il valore `GESTF4C`
> I campi restanti possono essere configurati in base alle esigenze.

2. Aggiungere un nuovo record nella tabella `grafica` inserendo nel campo `oper` il valore `GESTF4C`
> I campi restanti possono essere configurati in base alle esigenze.

## Funzionalità
### Scenari supportati
A far data **19/05/2021** è supportato unicamente lo scenario **One-To-One**: *1 device di Cash Management collegato direttamente ad 1 terminale Point-Of-Sales (cassa)*. <br>
Sono supportate tutte le funzionalità di gestione tramite F4C. <br>
Lo scenario d'uso a cui si fa riferimento, in F4C è supportato dal modulo `F4C-GDI+`.

Non sono supportati altri scenari quali multi terminale PoS verso il device di Cash Management.

### Workflow di utilizzo
1. Dopo aver premuto `Subtotale`, premere il tasto assegnato al pagamento cash management
2. In presenza di singolo monitor, verrà mostrata la schermata del driver F4C con lo stato del pagamento in attesa
> In presenza di due monitor, la schermata del driver verrà mostrata sul monitor che è stato configurato in F4C <br>
> In caso di errori, verranno riportate a video le informazioni guidate per la risoluzione direttamente da F4C

3. Una volta completato il pagamento, la schermata di F4C si chiuderà e Posware riceverà il dato sull'incassato.
4. Se il pagamento salda l'intera transazione, lo scontrino viene stampato e l'eventuale resto erogato dal device viene riportato nella transazione

##### Casi particolari nella erogazione del resto
Qualora il device non riuscisse ad **erogare completamente il resto**, Posware riceverà l'ammontare pendente e **mostrerà un messaggio all'operatore** per far recare il cliente al box informazioni al fine di ritirare il resto non erogato. **Lo stesso messaggio viene stampato sullo scontrino**.

> Posware > 4.1.560

A partire da Posware `v.4.1.560`, è possibile eseguire l'operazione di **Annullo pagamenti**: il cash management restituirà automaticamente l'importo versato dal cliente fino a quel momento. <br>
Qualora il device non dovesse restituire il denaro - o restituirlo solo parzialmente - Posware riceverà l'ammontare non dispensato e mostrerà un messaggio all'operatore per far recare il cliente al box informazioni al fine di ritirare il resto non erogato. **Lo stesso messaggio viene stampato sullo scontrino.**

La stessa procedura viene eseguita anche in caso di **annullo scontrino** se è stato introdotto del denaro. 

##### IMPORTANTE
In caso di errore del device, che non viene risolto dall'operatore, al termine della procedura F4C, Posware riceverà l'errore e non accetterà il pagamento. <br>
In presenza di doppio monitor, è anche possibile annullare il pagamento senza attendere l'operazione del driver F4C, premendo il pulsante `Annulla operazione`. 

> Una volta annullata l'operazione, Posware ignorerà qualsiasi dato proveniente da F4C. Prima di procedere, è di estrema importanza assicurarsi che il cliente sia stato rimborsato dell'eventuale versato nel device.




## Troubleshooting per i tecnici
### File log
La seguente tabella riepiloga come le informazioni di logging vengono tracciate all'interno dei file di log disponibili. Per abilitare il log dedicato, vedere il paragrafo `Configurazione`

| Operazione      | Livello minimo di Logging   | File log |
| ----------- | ----------- |-----------|
| Inizializzazione    | `Info`|Log principale di posware
| Connessione    | `Trace`|Entrambi 
| Errori di connessione    | `Error`|Entrambi
| Pagamento e conferma incasso   | `Trace`|Log principale di posware
| Errori di annullo, errori gravi | `Error`|Log principale di posware
| Traccia di annullo operazione dell'operatore | `Trace`|Log principale di posware
| Errori di tempo di attesa esaurito (timeout) | `Trace`|Log principale di posware
| Messaggi ingresso ed uscita da F4C | `Info`|Log dedicato plug-in F4C
| Info messaggi gestiti in ingresso ed uscita da F4C | `Info`|Log dedicato plug-in F4C

> La regola generale è: le info necessarie ad un **troubleshooting rapido** sono inserite anche nel log principale, quelle di **comunicazione dettagliata verso F4C** sono inserite nel log dedicato.

I nomi di **default** dei log sono:

| File log      | Nome di default | Percorso | Info
| ----------- | ----------- |--------|-----|
| Log principale di posware    | `posware-log_${shortdate}.log`| Cartella principale di Posware | `${shortdate}` = Data del log in formato AAAA-MM-DD . Esempio: `posware-log_2021-05-19.log`
| Log dedicato plug-in F4C    | `f4cMessagelog_${shortdate}.log`| Cartella principale di Posware | `${shortdate}` = Data del log in formato AAAA-MM-DD . Esempio: `f4cMessagelog_2021-05-19.log`

### F.A.Q
##### Il device si è bloccato e non riesco a recuperarlo a causa di un guasto. Come ritorno a Posware?
In caso di errore non recuperabile, Posware ritornerà automaticamente alla transazione dopo il tempo specificato nell'impostazione `overallPaymentProcessTimeoutInMinutes`. Se presente un doppio monitor, è inoltre possibile tornare immediatamente premendo il tasto `Annulla operazione`: Posware interromperà l'ascolto dei messaggi da parte di F4C e tornerà immediatamente alla transazione. **In questo ultimo caso, qualunque operazione fatta sul device (incasso/resto) viene ignorata.**

##### Mentre attendevo un pagamento, Posware ha annullato l'operazione ignorando l'incasso del device
Posware ha un meccanismo intrinseco di sicurezza da eventuali blocchi del driver F4C. <br>
Se il tempo di attesa, in minuti, settato  nella impostazione `overallPaymentProcessTimeoutInMinutes` viene superato, Posware considera il pagamento come annullato e ritorna alla transazione. <br>
Eventuali rimborsi di denaro versato nel device dovranno essere effettuati manualmente (attraverso il management o fisicamente). Qualora un pagamento durasse molto tempo, a causa di errori del device da correggere o altro, alzare il valore di questa variabile.

##### Quando richiedo un pagamento al device, Posware restituisce un errore
In caso di device occupato o driver non pronto (per errore software o hardware del device), Posware restituisce l'errore riportato da F4C all'operatore. Per entrare nel merito dell'errore da parte di F4C, contattare IT-Avantec

##### Sono presenti squadrature tra l'incassato del device ed il log di Posware
Verificare se nei log sono presenti **Annulli manuali** da parte dell'operatore che interrompono bruscamente l'operazione. <br>
Nel log principale di Posware sono riportate le seguenti diciture:

- **Sempre riportata:**
    - `Annullo operazione pagamento F4C richiesto manualmente dall'operatore.`
- **Riportata se l'annullo è avvenuto:**
    - `Pagamento annullato, restituire il denaro già incassato`
- **Riportata se l'annullo è stato bloccato:**
    - `Annullo operazione del pagamento F4C da parte dell'operatore non abilitato.`

E' possibile **impedire gli Annulli manuali** sia agendo sulle autorizzazioni del pulsante (chiave operatore) che tramite l'impostazione del plug-in `enableUserAbort`. Consultare la tabella delle impostazioni di questo documento.

##### Cosa si intende per Annullo manuale?
Si intende la pressione del tasto `Annulla operazione` mentre è in corso la transazione F4C. In presenza di doppio monitor è possibile usare la GUI di Posware e premere il tasto.