---
tags:
    - Posware
    - Celiachia
---

!!! warning "DOCUMENTO PRELIMINARE"
    Le informazioni e i dettagli contenuti in questo documento possono essere soggetti a modifiche e non costituiscono un protocollo definitivo.

# Specifiche integrazione bonus celiachia in Posware

**Prima revisione documento: 18 settembre 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**

## Introduzione
L’integrazione del bonus celiachia in **Posware** consente ai punti vendita di accettare come metodo di pagamento i buoni digitali messi a disposizione dal Sistema Sanitario Nazionale tramite la tessera sanitaria del cliente.

Grazie a questa funzionalità, i clienti celiaci possono acquistare i prodotti riconosciuti come idonei dalla propria ASL di riferimento, utilizzando il budget mensile previsto dalla normativa.

Lo scopo di questa documentazione è fornire tutte le informazioni necessarie per:

- **Configurare correttamente il database** in modo da abilitare il bonus celiachia come metodo di pagamento.
- **Comprendere la logica di riconoscimento degli articoli idonei**, che può avvenire tramite flag specifico nell’anagrafica o mediante formule dinamiche.
- **Gestire i flussi operativi in cassa**, inclusi i casi particolari di pagamento parziale, pagamento con importo specifico e note di credito.
- **Affrontare le principali anomalie ed errori** che possono verificarsi durante i pagamenti e gli storni.

Questa integrazione è stata sviluppata inizialmente per il provider **Argentea**, con **ARIA** come gestore del bonus celiachia, ma la logica documentata consente di estendere l’utilizzo anche ad altri provider compatibili.

È importante ricordare che, prima di poter utilizzare il bonus celiachia nel proprio punto vendita, il rivenditore deve:

1. **Contattare la propria ASL di riferimento**, che fornirà l’abilitazione e la lista aggiornata dei prodotti acquistabili.
2. **Assicurarsi che gli articoli siano correttamente configurati nel gestionale Posware**, in modo da essere riconosciuti al momento del pagamento.

## Configurazione del database
Per abilitare il **bonus celiachia** come metodo di pagamento e per fare funzionare tutti i vari processi legati a questa integrazione, è necessario aggiornare le seguenti tabelle:

#### Tabella `tipi_pagamenti` (sia database`cassa` che `posware`)

- **Bonus celiachia:**
    - `codice`: valore intero univoco non utilizzato da altri metodi di pagamento
    - `descrizione`: *"Bonus Celiachia"* o descrizione equivalente
    - `PagTipo`: 12
    - `tipoPagRT`: 1

#### Interfaccia Utente della cassa
Aggiungere un pulsante alla grafica di Posware ed agganciarci il pagamento **Bonus celiachia**.

#### Tabella `tabparametriextra` (database`cassa`)
Solo per l'integrazione specifica con il provider **Argentea**, con **ARIA** come gestore del bonus celiachia, sarà necessario verificare la presenza di questo record:

|Modulo|Parametro|Valore|
|-------|--------|------|
|*posware.plugins.celiacevoucher*|*protocol*|**AR/ARIA**|

Inoltre, bisogna configurare alcuni parametri necessari a regolare l'invio dei dati per la rendicontazione celiachia dalla cassa al server di barriera. 

|Modulo|Parametro|
|-------|--------|
|*posware.plugins.celiacevoucher*|*sendBatchSize*|
|*posware.plugins.celiacevoucher*|*sendTimeout*|
|*posware.plugins.celiacevoucher*|*lastSequenceIdSent*|

Tali parametri però hanno impostazioni differenti tra Posware :material-tag:`4.2` e Posware :material-tag:`4.3`.

##### Posware :material-tag:`4.2`
Di seguito le configurazioni necessarie per ognuno dei parametri in Posware :material-tag:`4.2`:

- Il parametro *sendBatchSize* indica il numero massimo di transazioni che potranno essere scritte con un'unica query.<br>
**Ha come valore di default 10 e come valore massimo accettato 50.**
- Il parametro *sendTimeout* indica il **timeout in millisecondi** per la scrittura dei dati della rendicontazione celiachia sul database del server di barriera. **È necessario che sia basso per non rallentare eccessivamente la cassa.**<br>
**Ha come valore di default 500 e come valore massimo accettato 2000.**
- Il parametro *lastSequenceIdSent* indica a quale progressivo della tabella locale si è arrivati a copiare i dati nella tabella sul server di barriera.

##### Posware :material-tag:`4.3`
Di seguito le configurazioni necessarie per ognuno dei parametri in Posware :material-tag:`4.3`:

- Il parametro *sendBatchSize* indica il numero massimo di transazioni che si potranno inviare allo StoreServer sul server di barriera in un unico invio.<br>
**Ha come valore di default 10 e come valore massimo accettato 50.**
- Il parametro *sendTimeout* indica il **timeout in millisecondi** per l'invio dei dati della rendicontazione celiachia allo StoreServer.<br>
**Ha come valore di default 500 e come valore massimo accettato 1500.**
- Il parametro *lastSequenceIdSent* indica a quale progressivo di invio al server di barriera si è arrivati con successo.

#### Tabella `operazioni_auto` (database`cassa`)
Se non è già presente o è configurato in modo diverso, è necessario inserire un record con le seguenti caratteristiche:

- `Operazione`: **"OB"**
- `Stato`: 1
- `Intervallo`: 30

#### Tabella `passioperazioni_auto` (database`cassa`)
Se non è già presente o è configurato in modo diverso, è necessario inserire un record con le seguenti caratteristiche:

- `CodOperazione`: "OB"
- `CodSubOperazione`: **"CD"**
- `Sequenza`: valore intero univoco e maggiore di 0 non utilizzato da altre suboperazioni di OB

!!! warning "Suboperazione CD"
    La presenza o meno di tale record determina l'attivazione o meno dell'invio dei dati al server di barriera.


## Invio articoli bonus celiachia
### Riconoscimento degli articoli idonei al bonus celiachia
Gli articoli acquistabili tramite bonus celiachia vengono individuati attraverso due criteri mutuamente esclusivi:

1. **Flag dedicato nella tabella `art_agg`:** il campo `Celiachia` valorizzato a **1** identifica in modo diretto e univoco i prodotti idonei.
2. **Formula di inclusione nei pagamenti:** nella tabella `tipi_pagamenti`, per i pagamenti con `PagTipo = 12`, il campo `formulainclusione` consente di definire condizioni di selezione più articolate per l’inclusione degli articoli.

Il comportamento del sistema è quindi il seguente: se il flag `Celiachia` è impostato a **1**, non bisogna inserire la formula nella tabella `tipi_pagamenti`. Se invece la formula è presente, la selezione degli articoli passa integralmente attraverso la logica prevista nella formula, sia che il flag `Celiachia` sia valorizzato sia che non lo sia.

!!! warning "Indicazione operativa"
    Per chi invia gli articoli al database della cassa, la regola generale è la seguente:

    - se non vi sono esigenze particolari, utilizzare il flag `Celiachia = 1` e non configurare la formula nei pagamenti;
    - se non è possibile gestire l’invio del flag o si richiedono criteri di selezione più complessi, configurare direttamente la formula nei pagamenti, senza valorizzare il flag per evitare confusione.


## Guida all'utilizzo
### Limiti e vincoli
!!! info "Utilizzo del bonus celiachia"
    Il pagamento tramite tessera sanitaria per clienti affetti da celiachia sarà richiedibile solo e soltanto se nello scontrino sono presenti dei prodotti per celiachi, riconosciuti dalla ASL di riferimento.

    Per ogni pagamento in fase di richiesta, verrà comunicata al provider utilizzato la lista dei prodotti per cui si richiede il pagamento tramite bonus celiachia. Tale lista sarà comprensiva dei dati riguardanti EAN, quantità (obbligatoriamente intere) e importo totale per cui si richiede il pagamento.

    A tal proposito, Posware prima della chiusura dello scontrino fa una verifica finale in cui confronta i prezzi unitari degli articoli salvati per la rendicontazione con quelli effettivi sullo scontrino aggiornato a fine transazione. **Se anche un solo prezzo non coincide, Posware avvisa l’operatore della discrepanza e lo scontrino viene annullato.**

!!! info "Pagamenti parziali e pagamenti con specifica di importo"
    Con questa integrazione, vengono permessi pagamenti con bonus celiachia parziali e multipli, oltre che i pagamenti con specifica di importo.

    In caso di pagamento con specifica di importo, l'operatore dovrà selezionare la cifra desiderata dal cliente e premere il pulsante a cui è agganciato il pagamento con bonus celiachia. In questo particolare scenario, Posware calcola la lista di prodotti massima che si può pagare con la cifra selezionata e alla fine si aprirà un pop-up che recita "È possibile pagare XX,XX euro, continuare?" (SI/NO). Questo perché l’importo selezionato deve corrispondere al prezzo complessivo di un numero intero di prodotti, infatti **non è assolutamente ammesso il pagamento di importi che corrispondano a frazioni di prodotto**.

    **Esempio:**<br>
    Totale carrello = 9€ (3 prodotti da 3€)

    - **Pagamenti ammessi**: 3€ (1 prodotto), 6€ (2 prodotti), 9€ (3 prodotti)
    - **Pagamenti non ammessi**: 8€, 7€, 5€, ecc..., perché non corrispondono al valore complessivo di prodotti interi

!!! info "Salvataggio di ogni richiesta inviata nel log trace di Posware"
    Per ogni richiesta inviata (sia di pagamento che di storno) verrà salvata la richiesta raw in un trace log di Posware. Solo se si riesce a ricevere una risposta dal provider si potranno aggiornare i dati nel database. **Nel caso che la risposta non sia stata ricevuta si potrà recuperare la richiesta inviata dal log trace di Posware.**

!!! info "Dettagli utili sull'integrazione con Argentea e provider celiachia ARIA"
    Quando si effettua un pagamento con bonus celiachia con il provider **Argentea**, con **ARIA** come gestore del bonus celiachia, nella risposta che restituisce Argentea tra i vari dati sono presenti un **Authorization Code** e un **Transaction Id**.<br>
    L'Authorization Code è un codice attribuito da ARIA al momento della conferma della transazione, mentre il Transaction Id, a differenza del suo omonimo in Posware, è un codice fornito da Argentea che identifica il singolo pagamento, così che lo si possa usare in caso sia necessario effettuare lo storno di quel singolo pagamento.

    Questi due dati verranno salvati anche nel buffer di Posware (posizione 131, lunghezza 50), all'interno del record di tipo **03** della tabella `log`, relativo al pagamento celiachia.

!!! warning "Privacy dei dati sensibili dei clienti"
    Con questa integrazione, non viene assolutamente memorizzata nessuna informazione riguardante la tessera sanitaria e il PIN necessari per l'utilizzo del bonus celiachia.

!!! warning "Note di credito nell'ambito del bonus celiachia"
    In caso di nota di credito di uno scontrino in cui sono stati effettuati pagamenti con il bonus celiachia, è necessario effettuare il rimborso tramite buono reso.

!!! danger "Promozioni legate ai pagamenti"
    **Le promozioni su pagamento che effettuano uno sconto, non sono compatibili con il pagamento e-voucher celiachia.**
 
    Assicurarsi di non avere attive promozioni sui pagamenti quando si usa il **pagamento e-voucher celiachia**.<br>
    Eventuali sconti apportati dopo aver incassato un e-voucher, causerebbero **disallineamenti in fase di rendicontazione**, con rifiuto da parte dell'ente del rimborso dovuto.
    

### Gestione degli errori e delle anomalie operative
Durante l’utilizzo del bonus celiachia in cassa, Posware gestisce in maniera guidata una serie di errori e condizioni particolari che possono verificarsi durante il pagamento o lo storno delle transazioni.<br>
Di seguito vengono descritti i principali casi d’errore, i messaggi visualizzati all’operatore e le azioni da intraprendere.

#### 1. Budget mensile insufficiente
##### Messaggio visualizzato:
- *“Budget mensile insufficiente. Residuo XXX euro, proseguire con il pagamento?”* (con opzioni **Sì/No**)
- *“Budget mensile insufficiente”* (avviso senza possibilità di continuare)

##### Quando si verifica:
L’errore appare quando, al momento del pagamento con bonus celiachia, il saldo residuo della tessera sanitaria non è sufficiente a coprire l’importo richiesto.

- Se il sistema riesce a recuperare il saldo residuo, viene proposto all’operatore di utilizzarlo.
- Se invece il saldo residuo non è disponibile, viene mostrato solo l’avviso.

##### Cosa deve fare l’operatore:
Nel primo caso, dove c'è la possibilità di scegliere:

- Premendo **Sì**, Posware ripete la richiesta di pagamento utilizzando l’importo pari al saldo residuo disponibile.
- Premendo **No**, l’operatore può chiudere il popup e proseguire con altri metodi di pagamento.

#### 2. Errore durante lo storno dei pagamenti
##### Messaggio visualizzato:
- *“Non è stato possibile stornare il pagamento con codice di autorizzazione 'XXXXXXXXX' a causa di un errore. Riprovare?”* (con opzioni **Sì/No**)

##### Quando si verifica:
Durante l’annullo dei pagamenti o dell’intero scontrino, Posware deve inviare una richiesta al provider per annullare i pagamenti già effettuati con bonus celiachia.<br>
L’errore compare quando:

- il provider risponde con un esito negativo (KO);
- non si riceve alcuna risposta dal provider (assenza di connessione o problemi tecnici).

##### Cosa deve fare l’operatore:
- Premendo **Sì**, il sistema ritenta lo storno del pagamento.
- Premendo **No**, Posware interrompe il ciclo di richieste e segnala che non è stato possibile annullare correttamente tutti i pagamenti.<br>
In questo caso:
    - l’operatore può completare lo scontrino saldando l’importo residuo con altri metodi di pagamento;
    - oppure può annullare lo scontrino: in questo caso Posware tenterà nuovamente lo storno di tutti i pagamenti, ma se anche questo dovesse fallire verrà stampato un documento non fiscale con i riferimenti dei pagamenti non stornati.

#### 3. Mancanza di connessione, crash o interruzione di corrente
##### Quando si verifica:
Il problema può presentarsi se:

- durante l’annullo di uno scontrino non è disponibile la connessione internet e non è possibile contattare il provider;
- Posware viene interrotto improvvisamente (crash dell’applicazione o mancanza di corrente) durante un’operazione di pagamento o storno.

##### Comportamento del sistema:
In questi casi, i pagamenti restano “in sospeso”.<br>
Al successivo riavvio e login (*SignOn*), Posware avvia in automatico un **meccanismo di rollback**, tentando nuovamente lo storno dei pagamenti non annullati correttamente.

##### Cosa deve fare l’operatore:
- Nessuna azione immediata è richiesta: il sistema gestisce in automatico il tentativo di storno.
- Se dopo vari tentativi alcuni pagamenti non possono essere stornati, Posware:
    1. stampa un documento gestionale con il dettaglio dei pagamenti non stornati,
    2. mostra un avviso informativo all’operatore in cassa.

#### 4. Pagamento avviato ma non confermato
##### Quando si verifica:
Durante una transazione con bonus celiachia:

1. viene inviata la richiesta di pagamento al provider,
2. ma prima di ricevere la risposta il sistema subisce un crash o un’interruzione.

In questo caso non è stato possibile registrare correttamente l’esito del pagamento.

##### Cosa deve fare l’operatore:
- Lo storno non può essere eseguito in automatico.
- Sarà necessario recuperare manualmente i dati della richiesta dal **log trace di Posware**, dove vengono salvati tutti i dettagli delle comunicazioni inviate al provider.
- Con tali riferimenti, il pagamento potrà essere gestito manualmente.
