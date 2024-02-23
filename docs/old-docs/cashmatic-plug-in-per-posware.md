# Cashmatic Plug-In per Posware
#### Manuale di attivazione ed utilizzo

 Il presente documento descrive i requisiti minimi, le configurazioni e le procedure necessarie all'attivazione ed utilizzo del plug-in dedicato a Cashmatic, device di Cash Management.

##### Data di creazione del documento: 05/09/2023
##### Ultima revisione documento: 18/09/2023

---

## Cenni preliminari
### Versioni software
La versione minima di **Posware** richiesta è la v. `4.2.694` per Windows 7/8/10. <br>
La versione supportata di firmware **Cashmatic** è la v. `0067`
 


### Device supportati
A far data **18/09/2023** sono supportati i seguenti device:

* Cashmatic - Serie SELFpay 460
* Cashmatic - Serie SELFpay 660


## Installazione e configurazione
### Pre requisiti
**IMPORTANTE**: Verificare sempre prima di una installazione che la versione del firmware installata sul dispositivo sia supportata tra quelle riportate nella sezione versioni software, in caso contrario non procedere all'installazione in produzione <br>
`Ogni segnalazione derivante da un malfunzionamento su una versione di firmware non supportata non potrà essere gestita in priorità`







### Procedura di installazione
Il plug-in è incluso out-of-the-box in Posware, nel pacchetto di installazione `PosUpdate`. <br>
Aggiornando Posware alla versione minima supportata, il plug-in è subito configurabile e non sono necessarie ulteriori installazioni.







### Configurazione
#### Impostazioni
Di seguito le impostazioni del plug-in configurabili nella tabella `tabparametriextra`:

| Modulo      | Parametro   | Valore    | Valore di default    | Note |
| ----------- | ----------- |-----------|----------------|------|
| `posware.plugins.cashmatic` | `port` | Porta di comunicazione verso il device | `50301` |
| `posware.plugins.cashmatic` | `endPoint`| Indirizzo IP del device |  | Specificare sempre il prefisso https:// e l'indirizzo ip interno del device |
| `posware.plugins.cashmatic` | `communicationTimeoutInMilliseconds`| Tempo di attesa *in millisecondi* per la comunicazione | `10000` | Rappresente il tempo massimo permesso per la comunicazione verso il device  |
| `posware.plugins.cashmatic`    | `paymentTimeoutInMilliseconds`|Tempo massimo permesso per l'intero processo di pagamento, *in millisecondi*| `30000` |In caso di blocco permanente del device, dopo questo lasso di tempo Posware annulla il pagamento e ritorna alla schermata principale. E' da considerare come uscita di emergenza. Valore minimo **30000**, massimo **120000** |
| `posware.plugins.cashmatic`    | `depositRefillTimeoutInMilliseconds`|Tempo massimo permesso per l'intero processo di fondocassa/versamento, *in millisecondi*| `30000` |In caso di blocco permanente del device, dopo questo lasso di tempo Posware annulla l'operazione di fondocassa/versamento e ritorna alla schermata principale. E' da considerare come uscita di emergenza. Valore minimo **30000**, massimo **120000**|
| `posware.plugins.cashmatic`    | `username`| Username | | Username di accesso al web server. |
| `posware.plugins.cashmatic`    | `password`| Password | | Password di accesso al web server. |
| `posware.plugins.cashmatic`    | `minimumDenominations`| Pezzi minimi per taglio | `5:0|10:0|20:0|50:2|100:0|200:0|500:0|1000:0|2000:0` | In questa sezione vengono indicati i pezzi minimi per ogni taglio. Sotto questa soglia il sistema avvisa il cassiere di effettuare un versamento. |
| `posware.plugins.cashmatic`    | `maximumGapDenominations` | Pezzi prima del massimo contenuto per tipologia | `coins:10|notes:10` | In questa sezione si indicano quanti pezzi devono mancare al raggiungimento del massimo contenuto per tipologia di contenitore |

Tutte le configurazioni, *se non presenti in `tabparametriextra`*, al primo utilizzo del plug-in vengono **inizializzate automaticamente al valore di default riportato in tabella**.

#### Attivazione sul front-end
Per utilizzare il plug-in sono richiesti 2 passaggi:

1. Configurarlo come gestore di Cash Management per l'inizializzazione.
2. Inserire un pulsante per richiamare il pagamento


##### 1. Attivare il plug-in di Cashmatic come gestore di Cash Management
Per inizializzare il plug-in, è necessario configurarlo come gestore di Cash Management. <br>
A tal fine, configurare le voci nella tabella `tabparametriextra` con i valori di seguito riportati:

| Modulo      | Parametro   | Valore    | 
| ----------- | ----------- |-----------|
`CASHMANAGEMENT`|`LIBRERIA`|`Posware.Cashmatic.dll`
`CASHMANAGEMENT`|`NOMECLASSE`|`Posware.Cashmatic.CashmaticHandler`|

Dopo aver configurato i parametri, avviare Posware: verrà inizializzato il plug-in con tutte le sue impostazioni di default. <br>
Chiudere Posware e impostare all’interno di `tabparametriextra`, i valori per questi parametri mandatori con quelli in uso sul vostro device:

| Modulo      | Parametro   | Valore    | 
| ----------- | ----------- |-----------|
| `posware.plugins.cashmatic` | `endPoint`| `Inserire l'indirizzo Ip` |
| `posware.plugins.cashmatic` | `username`| `Inserire la username` |
| `posware.plugins.cashmatic` | `password`| `Inserire la password` |

> Importante, specificare sempre il prefisso https nell'endpoint

##### 2. Pulsante di pagamento
1. Aggiungere un nuovo record alla tabella `tipi_pagamenti`. I campi obbligatori per definire il pagamento come Cash Management sono:

    | Resto      | EFT   | PagTipo    | TipoPagRT    | Arrotondamento
    | ----------- | ----------- |-----------|----------------| ------ |
    | `1`    | `true`|`1`| `0` | `0,05` |

    > I campi restanti possono essere configurati in base alle esigenze.

2. Aggiungere il pulsante alla grafica ed agganciare il nuovo pagamento

 
 
 
 

## Funzionalità
### Scenari supportati
È supportato unicamente lo scenario **One-To-One**: *1 device di Cash Management collegato direttamente ad 1 terminale Point-Of-Sales (cassa)*.
 
 

Non sono supportati altri scenari quali multi terminale PoS verso il device di Cash Management.




### Workflow di pagamento
1. Dopo aver premuto `Subtotale`, premere il tasto assegnato al pagamento cash management
2. Verrà mostrato il messaggio di attesa pagamento e sul display del Cashmatic sarà mostrato l'importo da pagare
4. Una volta completato il pagamento, Posware riceverà il dato sull'incassato.
5. Se il pagamento salda l'intera transazione, lo scontrino viene stampato e l'eventuale resto erogato dal device viene riportato nella transazione

E' possibile richiedere un pagamento parziale contanti indicando l'importo da richiedere al cashmatic per poi premere il tasto assegnato al pagamento cash management



#### Caso particolare di pagamento ticket numerati e contanti con arrotondamento
È possibile, previa abilitazione dell'**arrotondamento ibrido per pagamenti misti (vedi FAQ e vedi docs Funzionalità RT2.0)**, pagare con un pagamento ticket numerato ed il rimanente della transazione con il device. Tutto ciò mantenendo l'arrotondamento contanti. <br>
Per farlo è sempre requisito pagare la transazione **prima** con il massimo pagabile in **ticket numerati** e la differenza con il device in contanti. 

Il comportamento è coerente con quanto già implementato per i pagamenti Contanti manuali. Per approfondimenti vedere il docs “Posware 4.2 - Funzionalità RT 2.0” sezione “Arrotondamento ibrido”



#### Resto non erogato
Possono verificarsi casi in cui il device non riesce ad erogare il resto (totalmente o parzialmente), tipicamente a causa di indisponibilità dei tagli necessari. <br>
In questo caso Posware notifica l’evento alla cassiera e stampa il messaggio su scontrino. 

 

E’ importante considerare che il resto non erogato dal device rientra generalmente in due scenari:

1. È un **di cui** del resto indicato da Posware: in base al pagato la transazione prevede resto che tutto o in parte non è stato erogato:

    ![][Clip01]

2. Non fa parte del resto della transazione ma è necessario restituire un valore per via di tagli troppo grandi inseriti nel device per pagare. Può accadere anche con pagamenti parziali multipli. Vedi paragrafo Eccezioni a seguire.

**Si sottolinea che qualsiasi sia la condizione del resto della transazione, il valore riportato nel messaggio "Ritirare Euro XX al box informazioni" è sempre corretto e rappresenta un ammontare ancora da dare che spetta al cliente.**

##### Eccezioni
Nel caso di pagamenti multipli in cui viene effettuato un pagamento parziale con cashmatic e il valore del versato supera quello del richiesto, il resto erogato dal device non verrà indicato quale resto della transazione. Nel caso in cui però, tale resto, totalmente o parzialmente, non fosse erogato, Posware **evidenzierà** il valore NON RESTITUITO ma indicherà comunque resto zero della transazione




##### Casi particolari nella erogazione del resto
Qualora il device non riuscisse ad **erogare completamente il resto**, Posware riceverà l'ammontare pendente e **mostrerà un messaggio all'operatore** per far recare il cliente al box informazioni al fine di ritirare il resto non erogato. **Lo stesso messaggio viene stampato sullo scontrino**. <br>
In caso di pagamento multiplo, ticket più contanti, Posware effettuerà correttamente l'arrotondamento integrando o scontando opportunamente lo scontrino, ma restituirà **sempre** l'indicazione di resto 0 (zero), anche se il cashmatic avrà erogato resto.

##### Annullo pagamenti 

E' nativamente prevista la possibilità di eseguire l'operazione di **Annullo pagamenti**: il cash management restituirà automaticamente l'importo versato dal cliente fino a quel momento. <br>
Qualora il device non dovesse restituire il denaro - o restituirlo solo parzialmente - Posware riceverà l'ammontare non dispensato e mostrerà un messaggio all'operatore per far recare il cliente al box informazioni al fine di ritirare il resto non erogato. **Lo stesso messaggio viene stampato sullo scontrino.**

La stessa procedura viene eseguita anche in caso di **annullo scontrino** se è stato introdotto del denaro. 

##### IMPORTANTE
In caso di errore del device, Posware riceverà l'errore e non accetterà il pagamento, inoltre è anche possibile annullare il pagamento senza attendere l'operazione di completamento, premendo il pulsante `Annulla operazione`, oppure attendendo il tempo di timeout impostato

> Una volta annullata l'operazione, è di estrema importanza assicurarsi che il cliente sia stato rimborsato dell'eventuale versato nel device.





### Workflow per prelievo
1. Premere il tasto assegnato al prelievo
2. Confermare l'utilizzo del cashmanagement
3. Impostare l'importo del prelievo o lasciare 0 (zero) per prelevare tutto il contenuto del cashmatic
4. Attendere la fine dell'operazione
5. Indicare la causale opportuna

##### Importante
Durante l'operazione di prelievo, il display operatore di Posware **NON mostrerà** lo stato di avanzamento. <br>
Tale funzione è prevista nel device Cashmatic dove viene mostrato l'avanzamento **nel display integrato**.

### Workflow per fondo cassa/versamento
1. Premere il tasto assegnato al fond cassa o prelievo
2. Confermare l'utilizzo del cashmanagement
3. Sul display del device comparirà lo stato avanzamento
4. Premere il tasto **`Annulla Operazione`** a completamento procedura
5. Indicare la causale opportuna
> In caso di timeout leggere la sezione FAQ "Mentre effettuavo un fondo cassa o versamento, Posware ha annullato l'operazione ignorando l'incasso del device"

##### Importante
Durante le operazioni di fondo cassa o versamento, il display operatore di Posware **NON mostrerà** lo stato di avanzamento. 
Tale funzione è prevista nel device Cashmatic dove viene mostrato l'avanzamento **nel display integrato**.






## Avvisi gestiti
#### Minimo valore raggiunto
Quando il numero di pezzi raggiunge il livello minimo indicato in configurazione, il sistema notifica tale condizione indicando il taglio, l'attuale livello e l'atteso. <br>
La configurazione dei livelli minimo si trova nell'impostazione `minimumDenominations` ed è strutturata nel seguento modo:
> taglio:livello | taglio:livello | taglio:livello

*Esempio: 5:10 | 500: 5* <br>
Indica che per le monete da 5 cents minimo 10 pezzi mentre per le banconote da 5 euro minimo 5 pezzi

#### Massimo valore raggiunto
Quando il numero di pezzi sta per raggiungere il livello massimo indicato in configurazione, il sistema notifica tale condizione indicando il taglio, l'attuale livello ed il massimo consentito. <br>
La configurazione di riferimento si trova nell'impostazione `maximumGapDenominations` ed è strutturata nel seguente modo:
> tipologia:gap | tipologia: gap

*Esempio: coins:10|notes:5* <br>
Indica che per le monete si verrà avvisati 10 pezzi prima del limite massimo, mentre per le banconote si verrà avvisati 5 pezzi prima del limite massimo.










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
| Messaggi ingresso ed uscita da Cashmatic | `Info`|Log dedicato plug-in Cashmatic
| Info messaggi gestiti in ingresso ed uscita da Cashmatic | `Info`|Log dedicato plug-in Cashmatic

> La regola generale è: le info necessarie ad un **troubleshooting rapido** sono inserite anche nel log principale, quelle di **comunicazione dettagliata verso Cashmatic** sono inserite nel log dedicato.

I nomi di **default** dei log sono:

| File log      | Nome di default | Percorso | Info
| ----------- | ----------- |--------|-----|
| Log principale di posware    | `posware-log_${shortdate}.log`| Cartella principale di Posware | `${shortdate}` = Data del log in formato AAAA-MM-DD . Esempio: `posware-log_2023-09-05.log`
| Log dedicato plug-in Cashmatic    | `cashmaticMessagelog_${shortdate}.log`| Cartella principale di Posware | `${shortdate}` = Data del log in formato AAAA-MM-DD . Esempio: `cashmaticMessagelog_2023-09-05.log`

### F.A.Q
##### Il device si è bloccato e non riesco a recuperarlo a causa di un guasto. Come ritorno a Posware?
In caso di errore non recuperabile, Posware ritornerà automaticamente alla transazione dopo il tempo massimo pari a 3 minuti.

##### Mentre attendevo un pagamento, Posware ha annullato l'operazione ignorando l'incasso del device
Posware ha un meccanismo intrinseco di sicurezza da eventuali blocchi del device Cashmatic. <br>
Se il tempo di attesa, in millisecondi, settato  nella impostazione `paymentTimeoutInMilliseconds` viene superato, Posware considera il pagamento come annullato e ritorna alla transazione. <br>
Eventuali rimborsi di denaro versato nel device saranno effettuati automaticamente. Qualora un pagamento durasse molto tempo, a causa di errori del device da correggere o altro, alzare il valore di questa variabile.

##### Quando richiedo un pagamento al device, Posware restituisce un errore
In caso di device occupato o device scollegato, Posware restituisce l'errore di "device offline".

##### Cosa si intende per Annullo manuale?
Si intende la pressione del tasto `Annulla operazione` mentre è in corso la transazione di pagamento.

##### Mentre effettuavo un fondo cassa o versamento, Posware ha annullato l'operazione ignorando l'incasso del device
Posware ha un meccanismo intrinseco di sicurezza da eventuali blocchi del device Cashmatic. <br>
Se il tempo di attesa, in millisecondi, settato  nella impostazione `depositRefillTimeoutInMilliseconds` viene superato, Posware considera l'operazione come annullata e conclusa. <br>
Se è stato già effettuato un versamento parziale si consiglia di effettuare un'operazione di fondo cassa/versamento manuale in posware così da allineare il contenuto cassetto e avere la ricevuta stampata.

##### Cashmatic Device Error: Stato offline
Se viene mostrato un errore di questo tipo potrebbe essere scollegato il dispositivo oppure, se si verifica durante la fase di installazione controllare il log di Posware, in presenza di un errore simile a quello di seguito riportato vuol dire che l'endpoint è errato e va corretto

>2023-09-12 16:42:50.6376|DEFAULT|ERROR|>>> Cashmatic: Cannot init Cashmatic Web Server <br>
Unity.ResolutionFailedException: Eccezione generata dalla destinazione di una chiamata. <br>
Exception occurred while: <br>
   on constructor:  CashmaticHttpRepository(ILogger logger, ISettingsRepository settingsRepository, ICashmaticSettingsRepository cashmaticSettingsRepository) <br>
        mapped to:  ICashmaticHttpRepository <br>
---> System.Reflection.TargetInvocationException: Eccezione generata dalla destinazione di una chiamata. ---> **System.UriFormatException: URI non valido: impossibile analizzare il nome host.**


##### Attivazione arrotondamento ibrido per pagamenti misti
Affinchè Posware possa gestire correttamente l'arrotondamento contanti nei casi di pagamenti misti, Ticket e Contanti è mandatorio che venga abilitata tale funzionalità secondo le modalità indicate nella guida "Posware 4.2 - Funzionalità RT 2.0" sezione "Arrotondamento ibrido"





















[Clip01]:data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAMgAyAAD/2wBDAFA3PEY8MlBGQUZaVVBfeMiCeG5uePWvuZHI////////////////////////////////////////////////////2wBDAVVaWnhpeOuCguv/////////////////////////////////////////////////////////////////////////wAARCAGWAVQDASIAAhEBAxEB/8QAGQABAAMBAQAAAAAAAAAAAAAAAAECAwQF/8QANBAAAgIABAMHAwQCAwADAAAAAAECEQMSEyExUVIUMkFhcZGhImLhIzOB0QSxQpLBcvDx/8QAFwEBAQEBAAAAAAAAAAAAAAAAAAECA//EABoRAQEBAQEBAQAAAAAAAAAAAAARARICIUH/2gAMAwEAAhEDEQA/AO0AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACKT4pB7hJLn7gMseS9hljyXsSAKNwi6dIFcTCc5WnWwA5O3YnTD2/I7didMPb8nMDUHT27F6Y+z/sduxemPs/7KN8d1lrYriTu1u9/Eswa9uxOmHt+R27F6Y+z/ALOfxVrY14upNNN7CC/bsTph7fkduxOmHt+TNRjmVpcOFioWlS3fMQaduxOmHt+R27E6Ye35KNRVpJVsQ1FeC4PaxBp27E6Ye35HbsTph7fkyko6aaW4UVV7VS8RBr27E6Ye35HbsTph7fkzqDfBLdrjxKTSUtuQg37didMPb8jt2J0w9vycwJB09uxOmHt+R27E6Ye35OYCDp7didMPb8jt2J0w9vycwEHT27E6Ye35HbsTph7fkzqF1lXGuIywqP8AZYNO3YnTD2/I7didMPb8mVRcW1Fe/AtUU1VLlvxEF+3YnTD2/I7di9MPb8meWFR/sTrJWy24J+Yg07di9MPb8jt2L0w9n/ZincEtk7JxHbjfHx3EGvbsTph7fkduxemHt+SmWGZbLhwsKqpJePiIi/bsTph7fkduxOmHt+SmWFR2+SFl2pJPZ8fMRWnbsTph7fkduxOmHt+TNqN20vHayJKOmmluINe3YvTH2f8AY7di9MfZ/wBlI2kraf8A4FdrfetxEX7di9MfZ/2O3YnTD2/JnbySuqfnwKp/ptUuKEVt27F6Ye35HbsXph7fkxm7d0uBObaDpCDX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