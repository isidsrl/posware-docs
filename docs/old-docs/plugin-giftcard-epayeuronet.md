# Plug-In Gift Card EPay-Euronet
#### Manuale di attivazione ed utilizzo

> La procedura descrive come attivare la redenzione delle gift card, gestite tramite il servizio EPay di Euronet, ed i passi operativi per la redenzione da parte dell'utente finale <br>
> A far data del 02-04-2020, è supportata in via del tutto preliminare, la redenzione online delle Gift Card con successivo **pagamento manuale** della transazione.
---
> ## Versioni software di riferimento
>
>
> - Posware **4.0.0.154** o superiore
>
>
>

## Installazione ed attivazione

### Posware
La versione minima di posware richiesta è la v. **4.0.0.154**
### Sistema operativo
Windows, tutte le versioni supportate. <br>
Per i sistemi operativi basati su **kernel XP** v5.1 (WXP Embedded, WES09 e POSReady 2009) è necessario installare l'aggiornamento [`KB4467770`](https://www.catalog.update.microsoft.com/Search.aspx?q=KB4467770) per il supporto **TLS 1.2**. Windows 7 e successivi **non** hanno bisogno dell'aggiornamento.


### Parametri necessari
Per poter utilizzare la funzione di redention delle Gift, i seguenti parametri andranno compilati nella tabella `tabparametriextra`:

| Modulo      | Parametro   | Valore    | Environment    |
| ----------- | ----------- |-----------|----------------|
| `EPPLIB`    | `EPAYGIFTUSER`|`username` fornito da Epay/cliente| `production` |
| `EPPLIB`    | `EPAYGIFTPASSWORD`|`password` fornita da Epay/cliente|`production`|
| `EPPLIB`    | `EPAYGIFTTERMINALID`|`Terminal Id` fornito da Epay/cliente|`production`|
| `EPPLIB`    | `EPAYGIFTSERVERADDRESS`|<https://precision.epayworldwide.com/gxml> oppure <https://backup.precision.epayworldwide.com/gxml>|`production`|

### Operazioni per l'attivazione
##### Collegamento alla rete
E' necessario il **collegamento del POS ad internet**. <br>
Se il POS non è direttamente connesso, le chiamate necessarie possono essere indirizzate ad un gateway, tramite **reverse proxy**. <br>
Il server di barriera o un computer **della stessa LAN del pos** che accede ad internet, può essere utilizzato.

Per effettuare su windows, tramite **netsh**, queste configurazioni, usare il comando:
> Il comando deve essere eseguito **sul server**, non sui POS

`netsh interface portproxy add v4tov4 listenport=`**`$porta`**` listenaddress=0.0.0.0 connectport=443 connectaddress=precision.epayworldwide.com`



Al posto della variabile **$porta** è necessario specificare la porta che verrà utilizza per la comunicazione **dalla cassa**. Questa porta dovrà essere poi riportata nel parametro `EPAYGIFTSERVERADDRESS`. Esempio:
> Se a `listenport` viene passato 5744 , ed il server in `EPAYGIFTSERVERADDRESS` è **<https://precision.epayworldwide.com/gxml>** , il parametro `EPAYGIFTSERVERADDRESS` dovrà diventare **<https://precision.epayworldwide.com:5744/gxml>**

Disattivare il firewall o aggiungere una eccezione sulla porta scelta e verificare la corretta comunicazione dalla cassa tramite una connessione **telnet**, **nmap**, **Test-NetConnection** di PowerShell o qualunque tool simile.
##### Creazione del pulsante in Posware
Creare un nuovo pulsante o associare ad uno esistente l'operazione **`GIFTCOVID19`** assegnandovi come descrizione `GIFT EPAY` (o la descrizione desiderata.) 

>In questa guida si farà riferimento alla descrizione `GIFT EPAY` per la spiegazione all'utente finale.

Una volta assegnata l'operazione al tasto, la funzione, in fase di pagamento (a seguito di `SUBTOTALE`), è immediatamente utilizzabile. <br>
Vedi la procedura di utilizzo per l'operatività della cassiera.

### Note per il tecnico
All'interno del file predefinito di log di posware formato **\*.log**  (es.: `posware-log_2020-04-02.6.log`)  è possibile trovare  tutte le operazioni di Check Status e Redeem del servizio, con il dettaglio raw delle chiamate e le risposte da parte del server. <br>
Queste informazioni sono disponibili solo nel **nuovo** formato **.log**. Nel vecchio formato **xml** non sono disponibili. <br>
I record sono registrati come livello log `INFO`.

## Utilizzo da parte dell'utente finale
### Cenni preliminari / requisiti
Prima di poter redimere il buono / gift card, è necessario entrare in fase di **pagamento** premendo il tasto `SUBTOTALE`.
### Procedura di utilizzo
1. Entrare in fase di **pagamento** premendo il tasto `SUBTOTALE`
2. Premere il tasto `GIFT EPAY`
3. Digitare il numero della Gift Card o passare a scanner il suo barcode
4. Attendere qualche secondo
5. Se l'operazione è andata a buon fine, verrà mostrata una *finestra popup* con la dicitura contenente l'ammontare redento dalla Gift:

    > Incasso gift online avvenuto. Effettuare un pagamento manuale di **€ 2,5**

    * In caso di POS *sprovvisto di Touch Screen*, verrà mostrata sul **display operatore** la dicitura

        > Incasso avvenuto. Pagare **€ 2,5**

6. Effettuare ***manualmente*** un pagamento pari alla somma redenta per chiudere l'operazione (nell'esempio cui sopra, **€ 2,5** ).
7. Assicurarsi di aver effettuato il pagamento manuale.

---
##### Qualora l'operazione non dovesse andare a buon fine, riprovare la procedura partendo dal punto 1)
---
## F.A.Q.
* #### Come faccio a verificare il credito residuo di una gift?
    > Se il POS è dotato di **touch screen**, basta eseguire l'operazione prima di entrare in fase di pagamento. Al di fuori del subtotale, verrà mostrata una popup con il credito residuo e quello utilizzabile per la spesa. Se non sono stati battuti prodotti sarà mostrato solo il credito residuo. <br><br>
    > Se il POS è dotato di **tastiera fisica** o **tastiera e monitor non touch**, la funzione non è supportata.
* #### Posso effettuare pagamenti parziali con la gift?
    > Sono supportati crediti residui per le gift ma non incassi parziali. In altre parole **non è possibile scegliere** quanto pagare con la gift.Verrà sempre detratto il valore massimo disponibile sulla gift card oppure, se il totale della spesa è minore, il valore totale della spesa. <br>
    > **Esempi:** <br>
    > Con una gift da **25€** ed una spesa da **15€**, verranno incassati solo **15€** e gli altri **10€** rimanenti saranno utilizzabili in spese future. <br>
    > Con una gift da **25€** ed una spesa da **35€**, verrà incassato tutto il credito della gift (che non sarà quindi più utilizzabile), ed i rimanenti **10€** dovranno essere pagati con un'altro metodo di pagamento. 
* #### E' possibile pagare con più gift lo stesso scontrino?
    > Si, basta ripetere l'operazione **ricordandosi prima di effettuare il pagamento manuale**. <br><br>
    >**Senza il pagamento manuale, il totale a pagare non andrebbe a scalare e verrebbe richiesto di nuovo l'incasso dell'intero importo della spesa a danno del consumatore.**