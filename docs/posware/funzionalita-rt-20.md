---
tags:
    - Posware
    - XML7
    - RT
---

# Posware - Funzionalità RT 2.0 (XML 7)

**Prima revisione documento: 03/02/2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Manuale di attivazione ed utilizzo

Il presente documento descrive i requisiti minimi e le procedure necessarie all'attivazione delle funzionalità previste dalle specifiche del Registratore Telematico 2.0, inclusa la compatibilità con la versione 7.0 dei tracciati XML.

---

## Versioni software di riferimento
Posware **4.3.0** o superiore

## Requisiti

### Stampanti

A far data **23/06/2024** le stampanti RT 2.0 supportate sono:

* Epson - tutti i modelli
* RT-ONE

!!! danger "Copertura funzionale parziale"
    **Non tutte le funzionalità RT 2.0 riportate in questo documento sono supportate dai registratori telematici o da Posware.**
    
    Consultare la [tabella di compatibilità delle funzionalità](#tabella-di-compatibilita-delle-funzionalita) per verificare se il vostro modello di stampante RT supporta una specifica funzionalità in Posware.

### Server RT

Il supporto RT 2.0 ed XML7 al Server RT Epson richiede la versione minima del firmware `4.01.0` del **01/04/2021**, consultare la [tabella di compatibilità delle funzionalità](#tabella-di-compatibilita-delle-funzionalita) per verificare le funzioni supportate. <br>
Per configurare le singole funzioni, non vi sono differenze tra le normali stampanti RT ed il server, ad eccezione della nuova funzione di trasmissione Iva/Reparti (seguire le indicazioni alla sezione [Configurazione Server RT](#configurazione-server-rt) ). <br>
Continuare la lettura dei successivi paragrafi di questo documento.

### Firmware stampanti

Di seguito l'elenco dei firmware/stampanti certificati:

* Epson - tutti i modelli: `11.02 build 255` oppure `7.02 build 255`
* RT-ONE: `8.0.7`

**Qualora venga utilizzata una versione di firmware diversa da quelle riportate in elenco, il funzionamento ed il supporto non è garantito. Possono verificarsi problemi quali errori imprevisti ed annulli della transazione.**

### Driver stampanti

L'aggiornamento del firmware della stampante deve avvenire contestualmente all'aggiornamento dei driver OPOS qualora fossero obsoleti. Di seguito le versioni minime supportate:

!!! danger "Errori con OPOS non aggiornati"
    Qualora si ometta di aggiornare il driver, la stampante restituirà in maniera random degli errori **OPOS 207** durante varie fasi di emissione dello scontrino ed annulli imprevisti.

=== "Diebold RT-One"
    Driver **OPOS** Diebold, versione minima `1.4.3`

=== "EPSON"
    Driver **OPOS** Epson, versione minima `V22`.

!!! info "Download dei driver"
    Rivolgersi ai produttori per il download dei driver aggiornati.

## Procedura di installazione

La versione 4.3 di Posware supporta nativamente RT 2.0 e non necessita di procedure particolari di installazione.
Se stai aggiornando da Posware 4.1, fai riferimento alla documentazione di Posware 4.2 per i dettagli sull'installazione e configurazione.

## Funzionalità

### Nuove forme di pagamento
Il Registratore Telematico 2.0 supporta due nuove forme di pagamento: `Non riscosso fattura` e `Sconto a pagare`. Entrambe sono supportante in Posware.

#### Sconto a pagare
La forma di pagamento `Sconto a pagare` può essere utilizzata in Posware solo tramite il meccanismo dei **Coupon pagamento**. <br>
Non è possibile far imputare pagamenti di questo tipo in modo arbitrario all'operatore. La tracciatura avviene tramite gli assodati meccanismi dei coupon pagamento.

#### Non riscosso

Non ci sono particolari vincoli su questo tipo di pagamento. Possono essere creati molteplici pagamenti in base alla necessità ( non riscosso carta, non riscosso contanti, ecc... ) <br>
I campi riportati in tabella devono rispettare questi valori dovrà essere configurato sulla base del valore in tabella:

#### Non riscosso segue fattura

Usando questa forma di pagamento, Posware al termine dell'emissione del documento commerciale di vendita, farà partire la procedura di Fattura immediata per rilasciare una fattura elettronica.<br>

!!! warning "Licenza richiesta"
    Questa funzionalità richiede un modulo aggiuntivo *Fatturazione elettronica* sottoposto a licenza.

#### Configurazione
Per abilitare l’uso dei pagamenti `Non riscosso`, `Non riscosso segue fattura` e `Sconto a pagare`, configurare un nuovo pagamento tramite la tabella `tipi_pagamenti`. <br>
Nella tabella a seguire sono riportati solo i campi il cui valore deve essere impostato come da tabella per il corretto funzionamento.

| Pagamento RT      | Valore campo `TipoPagRT`         | Valore campo `TipoPag` | Valore campo `resto` 
| -----------       | ---------------- |-----|-----|
|Sconto a pagare    |               `6`  | `2` | `0` se il coupon pagamento viene bruciato totalmente, `1` se deve essere emesso un coupon pagamento della differenza
|Non riscosso       |  `2`           | Non sono presenti vincoli particolari | `0`: il pagamento RT non permette di avere resto
|Non riscosso segue fattura   |  `7`           | Non sono presenti vincoli particolari | `0`: il pagamento RT non permette di avere resto

Assegnando un pulsante al pagamento, sarà possibile utilizzarlo nel comune workflow di Posware. <br>
Nel pagamento RT `Sconto a pagare` dovranno essere eseguite anche tutte le configurazioni relative all'emissione e bruciatura dei coupon pagamento.

##### Addendum per Registratore Telematico RT-One

Abilitando in `tabparametriextra` la funzione `sendPayment` a `true` (vedi tabella), Posware invierà alla stampante la programmazione dei pagamenti secondo la propria tabella.

| Modulo      | Parametro   | Valore    | Valore di default    | Note |
| ----------- | ----------- |-----------|----------------|------|
| `posware.fiscalprinter.rtone`    | `sendPayment`|true/false| `false` | Impostandolo a `true`, Posware invierà al registratore telamtico, oltre alla configurazione IVA, anche la configurazione dei pagamenti in base alla sua tabella interna. Impostare a `false` per programmare manualmente la tabella sul R.T.

**Se il parametro `sendPayment` sarà impostato a `false`, i pagamenti in stampante andranno manualmente configurati tenendo sempre conto delle indicazioni dei valori precedentemente riportati come valori accettati del campo `TipoPagRt`**.

#### Ticket numerati manuali

##### Configurazione

1. Aggiungere un nuovo record alla tabella `tipi_pagamenti`
2. Impostare il campo `tipoPagRT` al valore **4**
3. Impostare il campo `ticket` al valore **SI**
4. Impostare il campo `obbligatorio` al valore **true**
5. Aggiungere il pulsante alla grafica ed agganciare il pagamento

##### Workflow di utilizzo

1. Dopo aver premuto subtotale, premere il tasto assegnato al pagamento Ticket numerati
2. Alla richiesta **Inserire taglio** imputare il taglio dei Ticket e premere il tasto INVIO
3. Verrà visualizzato a Display Operatore/Schermo, il numero massimo dei ticket utilizzabili per la transazione in corso in base al taglio imputato
4. Il numero massimo dei ticket utilizzabili viene inserito nella casella di input. E' possibile confermare con **INVIO** oppure imputare un numero di ticket diverso. Premere **INVIO** per confermare.

>Se viene erroneamente imputato un numero di Ticket da usare maggiore del massimo utilizzabile, viene mostrato il messaggio MAX PAG. *"NOME PAGAMENTO"* XX €, riportando il valore massimo pagabile. A questo punto è possibile ripetere la procedura dal punto 1.

#### Ticket numerati su pagamento elettronico e dematerializzato

##### Configurazione

Alla configurazione già prevista per il pagamento, è unicamente necessario impostare il campo `tipoPagRT` al valore **4**

##### Workflow di utilizzo
Il riconoscimento del numero dei ticket avviene automaticamente senza alcuna variazione al workflow esistente.

##### Ulteriori note sul workflow
Si ribadisce che in caso di pagamento buono pasto **elettronico**, è possibile pagare soltanto in una unica operazione il *massimo importo pagabile*. <br>
Questo importo viene calcolato automaticamente dal front-end in base alle configurazioni impostate (esclusione reparti/promo ecc...). <br>
**Non è possibile specificare manualmente quanto si desidera pagare.**

Se durante l'operazione di pagamento il provider dei buoni pasto restituirà un importo minore al massimo pagabile (per insufficienza di credito o numero massimo dei buoni utilizzati), questo verrà usato come importo di riferimento ed avverrà un pagamento parziale della transazione (Esempio 3).

###### Esempio 1: transazione di 25€ con massimo pagabile calcolato di 20€ e cifra da pagare digitata 15€
1. Viene digitato il pagamento buono pasto elettronico con i 15€ desiderati
2. Al provider viene inoltrata una richiesta di pagamento di 20€ - non 15.
3. Il provider conferma l'avvenuto pagamento ed i 20€ conteggiati.
4. Per chiudere la transazione è necessario saldare il rimanente con altro metodo di pagamento o contanti

###### Esempio 2: transazione di 25€ con massimo pagabile calcolato di 20€ e nessuna cifra manuale da pagare digitata
1. Viene digitato il pagamento buono pasto elettronico
2. Al provider viene inoltrata una richiesta di pagamento di 20€
3. Il provider conferma l'avvenuto pagamento ed i 20€ conteggiati
4. Per chiudere la transazione è necessario saldare il rimanente con altro metodo di pagamento o contanti

###### Esempio 3: transazione di 25€ con massimo pagabile calcolato di 20€ e credito massimo disponibile sui buoni di 16€
1. Viene digitato il pagamento buono pasto elettronico (anche digitando deliberatamente una cifra da incassare verrebbe ignorata)
2. Al provider viene inoltrata una richiesta di pagamento di 20€
3. Il provider conferma l'avvenuto pagamento di soli 16€
4. Il pagamento parziale di 16€ viene conteggiato e per chiudere la transazione sarà necessario saldare i restanti 9€ con altro metodo di pagamento o contanti

##### Provider supportati
Al momento, tutti i provider di buoni pasto dematerializzati ed elettronici previsti in posware, supportano anche la funzione di ticket numerato. <br>
Si riporta l'elenco per completezza:

| Provider    | Funzionalità   | Supporto Ticket numerati    | Note |
| ----------- | ---------------- |---------|------|
|Argentea| Buoni Pasto Elettronici|SI||
|Argentea| Buoni Pasto Dematerializzati|SI||
|Diebold Buono Chiaro| Buoni Pasto Dematerializzati|SI||
|Satispay| Buoni Pasto Elettronici|SI||

### Nota di credito

#### Configurazione

Non sono necessarie configurazioni

#### Workflow di utilizzo

!!! info "Valido solo per la procedura manuale"
    I workflow della procedura di nota credito **Totale e Parziale rimangono invariati**. I cambiamenti impattano solo la procedura **Manuale**.

1. Premere il tasto assegnato alla funzione Nota di Credito
2. Alla richiesta Tipo Nota Credito, selezionare **Manuale** e premere il tasto OK
3. Alla richiesta Causale, selezionare la causale desiderata e continuare. Se configurate, vengono visualizzate le richieste di Info aggiuntive, imputarle e continuare.
4. Verrà richiesta la **Tipologia Documento** come previsto da normativa su XML7. <br>Le scelte possibili sono:
    1. **Matricola**: da utilizzare nel caso in cui il documento originale sia stato emesso da un registratore telematico diverso da quello che sta emettendo attualmente la nota di credito
    2. **POS**: da utilizzare quando è necessario emettere un documento di reso utilizzando altri elementi probanti l’acquisto Come previsto dalla Circolare 3/E del 21-feb-20. In questo caso l'elemento è una ricevuta POS
    3. **ND**: da utilizzare quando è necessario emettere un documento di reso utilizzando altri elementi probanti l’acquisto Come previsto dalla Circolare 3/E del 21-feb-20. In questo caso l'elemento è una ricevuta da misuratore fiscale (MF), ricevuta WEB o di altra origine.

    In base alla **tipologia documento** scelta, verranno formulate altre richieste. <br>
    Per la tipologia documento **Matricola** dovranno essere forniti:

    - la data del documento commerciale di origine cui si fa riferimento;
    - l’identificativo del documento commerciale a cui si fa riferimento. Vengono fatte due richieste, una per il numero documento ed una per il numero della chiusura fiscale. Queste informazioni sono riportate nello scontrino nel formato "Documento N. XXXX-YYYY" dove XXXX è il numero della chiusura fiscale ed YYYY è il numero del documento;
    - la matricola del dispositivo originario che ha generato il documento di riferimento;

    Per la tipologia documento **POS** o **ND** dovranno essere forniti:

    - la data del documento commerciale di origine a cui si fa riferimento;


5. E' ora possibile aggiungere gli articoli alla transazione e completare come previsto da normale procedura
6. Se è necessario digitare il codice lotteria, è possibile digitarlo **solo prima del Subtotale** (a seguito di tutti gli articoli e prima di entrare nella fase di scelta del pagamento). Si entra nella fase di scelta del pagamento alla pressione del tasto **Subtotale**.

### Arrotondamento tramite Registratore Telematico

#### Configurazione

Per abilitare l'arrotondamento automatico previsto dal DL N.50/2017, effettuato direttamente dal Registratore Telematico, è necessario:

1. Abilitare sulla stampante la funzionalità di arrotondamento tramite programmazione/tool ufficiali (*Posware non invia la programmazione alla stampante*). È supportata solo la configurazione di **arrotondamento completo (sia in eccesso che in difetto)**.
2. Impostare nella tabella di configurazione `tabparametriextra` il parametro `isRTCashPaymentRoundingSupportEnabled` del modulo `posware.fiscalprinter` al valore `True`
3. Impostare nella tabella `tipi_pagamenti`, per il pagamento di tipo **contanti** che si vuole arrotondare, il campo `tipoPagRT` al valore `0` (*contante*) ed il campo `Arrotondamento` al valore `0,05`
4. Se sono state impostate delle formule di arrotondamento personalizzate, **è necessario rimuoverle** per non creare conflitti. <mark>Le due feature non sono compatibili tra loro.</mark>

#### Sistema di verifica della configurazione

All'avvio di Posware, verrà verificata la configurazione dell'arrotondamento del Frontend rispetto a quella programmata nella stampante. In caso venisse rilevata una configurazione errata, verrà mostrato un messaggio popup con l'errore:

```
La configurazione sulla stampante dell'arrotondamento DL N.50/2017 non è corretta. Per funzionare correttamente è necessario abilitare l'arrotondamento standard (codice 1). Se non si vuole attivare l'arrotondamento sulla stampante, disabilitare anche in Posware l'opzione - IsRTCashPaymentRoundingSupportEnabled - in tabparametriextra. Verificare la programmazione.
```
oppure:
```
Nella stampante è abilitato l'arrotondamento DL N.50/2017. Abilitare la funzionalità di Posware - IsRTCashPaymentRoundingSupportEnabled - in tabparametriextra oppure disabilitare l'opzione nella stampante. Verificare la programmazione.
```
**Anche in caso di Server Rt Epson, Posware controlla lo stato di attivazione della funzione.** <br>
**È mandatorio allineare le impostazioni di arrotondamento tra Posware e Registratore Telematico. In caso contrario Posware si chiuderà automaticamente alla chiusura del messaggio di avviso.**

#### Workflow di utilizzo
Non sono previsti variazioni rispetto al workflow di arrotondamento esistente in Posware.

#### Limitazioni
Il Registratore Telematico effettua l'arrotondamento solo su transazioni **integralmente pagate in contanti**.  In caso di pagamenti misti di qualsiasi tipo, l'arrotondamento non viene effettuato.

#### Informazioni registrate nel log di transazione
Una transazione con arrotondamento da registratore telematico, prevede che il totale della transazione - corrispettivo - sia pari al totale effettivo dello scontrino al lordo degli arrotondamenti: una transazione avente ammontare scontrino **1.02 €**, aumenterà il **totale gran fiscale** del registratore telematico di **1.02 €**, nonostante il consumatore, in caso di pagamento integralmente in contante, paghi effettivamente soltanto **1.00 €**

È possibile riscontrare tale condizione anche nei tipi record 00/04 di inizio/fine transazione, nei campi `Totale transazione` e `Corrispettivo Fiscale a Fine transazione`.

Al fine di riconoscere il valore degli arrotondamenti da R.T. e poter eseguire eventuali quadrature, nel log cassa viene inserita una informazione aggiuntiva (record 14) con codice informazione `RTRA` (RT Rounding Amount). <br>
*Questo record 14 si aggiunge ad altri eventuali record di informazione aggiuntiva.*

Campo| Inizio|	Lunghezza|	Tipo|	Valore esempio|Note
-----|-------|-----------|------|-----------------|-----
Codice del punto vendita, Fisso per PdV|	1|	4|	N|	`0001`|Codice punto vendita di esempio
Numero terminale cassa (001-999)	|5	|3	|N	|`001`|Numero terminale di esempio
Progressivo record	|8	|5	|N	|`12345`| Progressivo di esempio
Tipo Record (fisso "14")	|13	|2	|N	|`14`|Tipo record *Informazione aggiuntiva*
Tipo Informazione	|15	|1	|C	|`S`| Informazione aggiuntiva di livello *Scontrino*
Codice Informazione	|16	|5	|C	|`RTRA`|RT Rounding Amount
Codice a barre (in caso Tipo Informazione=A)|	21|	20|	C	| |Non applicabile
Informazione|	41|	100|	C|	`-2`|Ammontare **positivo o negativo** del valore di arrotondamento RT applicato alla transazione. Se positivo viene memorizzato **senza segno**.

In caso di transazione pagata integralmente in *Contanti*, **intesa come tipo pagamento *Contanti* a livello di registratore telematico**, potranno essere presenti uno o più record **03 - Pagamento**. 

Il campo `Importo pagamento` (posizione 17 lungo 9, tipo numerico 7,2) di tale record, rappresenta l'effettivo valore pagato dal consumatore **inclusi gli arrotondamenti RT siano essi in eccesso o in difetto**. In presenza di record multipli, l'ammontare effettivamente pagato dal consumatore sarà la somma dei valori del campo `Importo pagamento`.

Per ottenere la quadratura con il totale della transazione, è necessario sommare **l'opposto** del valore degli arrotondamenti, al totale effettivamente pagato dal consumatore, sottraendo infine l'eventuale resto. <br>
Il calcolo può essere riassunto nella formula: <br>
`Totale valore dei pagamenti` + **[**`Valore Info Aggiuntiva` ***(-1)]** - `Resto` = `Totale corrispettivo scontrino`

A seguire alcuni esempi riassunti in tabella per facilitare la consultazione:

##### Transazione avente ammontare scontrino **1.02 €**, il consumatore paga effettivamente **1.00 €**
---

Corrispettivo fiscale transazione	| Valore record 03| Valore Info Aggiuntiva RTRA| Resto |Note
-----|-------|-----------|----|------
1.02 | 1.00 | -2| 0 | Quadratura: 1.00 + (+2) - 0 = 1.02

##### Transazione avente ammontare scontrino **1.08 €**, il consumatore paga effettivamente **1.10 €**
---

Corrispettivo fiscale transazione	| Valore record 03| Valore Info Aggiuntiva RTRA| Resto |Note
-----|-------|-----------|----|------
1.08 | 1.10 | 2| 0 | Quadratura: 1.10 + (-2) - 0 = 1.08

##### Transazione avente ammontare scontrino **1.07 €**, il consumatore paga effettivamente **1.05 €**
---

Corrispettivo fiscale transazione	| Valore record 03| Valore Info Aggiuntiva RTRA| Resto |Note
-----|-------|-----------|----|------
1.07 | 1.05 | -2| 0 | Quadratura: 1.05 + (+2) - 0 = 1.07

##### Transazione avente ammontare scontrino **1.08 €**, il consumatore paga **2 €** <mark>con resto di **0.90 €**</mark>. Il valore effettivamente pagato sarà comunque di **1.10€**
---

Corrispettivo fiscale transazione	| Valore record 03| Valore Info Aggiuntiva RTRA| Resto |Note
-----|-------|-----------|----|------
1.08 | 2 | 2| 0.90 | Quadratura: 2 + (-2) - 0.9 = 1.08

##### Eccezioni alla regola e funzionalità di arrotondamento ibrido
In caso di transazione pagata con buoni pasto (manuali o elettronici), il workflow tipico prevede prima il pagamento con i buoni pasto e solo successivamente il pagamento della differenza in contanti (o altro metodo di pagamento).<br>
Con la combinazione di pagamenti buono pasto + contanti, trattandosi di *pagamento misto*, il Registratore Telematico **<mark>non</mark> effettua l'arrotondamento del contante**. Alla data del 24-09-2021 l'AdE non ha previsto questa casistica.

Per ovviare alla problematica, è stata prevista in Posware la funzionalità di **arrotondamento ibrido** approfondita nel prossimo paragrafo.


#### Arrotondamento ibrido
Solo nel caso in cui una transazione venisse pagata con la combinazione di pagamenti **buoni pasto + contanti** (di conseguenza verrebbe meno l'arrotondamento del RT), Posware effettuerà l'arrotondamento come previsto dalla procedura esistente previa introduzione del RT 2.0. <br>
La feature è di default **disabilitata**. Per abilitarla, impostare nella tabella di configurazione `tabparametriextra` il parametro `allowHybridRoundingWithRTCashRoundingEnabled` del modulo `posware.cart.payments` al valore `True`. <br>
Le configurazioni normalmente previste per l'arrotondamento da parte di Posware devono rimanere configurate (valore arrotondamento, codice articolo per la maggiorazione e codice pagamento per la detrazione). <br>
**La feature è supportata unicamente da Posware Frontend. Gli adattatori SCO non prevedono tale funzionalità.**


##### Considerazioni e disclaimer riguardo l'arrotondamento ibrido
Utilizzando l'arrotondamento ibrido, le minusvalenze e le plusvalenze generate dagli arrotondamenti **vengono conteggiate nei corrispettivi.** <br>
Utilizzando l'arrotondamento nativo del Registratore Telematico no. <br>
La feature è stata introdotta <mark>unicamente al fine di non creare disservizi rispetto ai workflow di pagamento buoni pasto consolidati da tempo.</mark> <br>
La feature è di default **disabilitata**. <br>
**Prima di abilitare questa feature è necessario verificare insieme al consulente la compabilità con gli adempimenti fiscali del cliente.**


### Vendita di servizi

La vendita di un _servizio_ con il Registratore Telematico, consiste nel vendere un comune prodotto **contrassegnato come servizio**. <br>
Lo scenario è supportato da Posware sia con il Registratore Telematico in configurazione mono-ateco che multi-ateco. <br>
In caso di Server Rt **è necessario** inviare la configurazione Iva/Reparti come da indicazioni alla sezione [Configurazione Server RT](#configurazione-server-rt)

#### Configurazione

Per vendere un prodotto come servizio, impostare nella tabella `Articoli` il campo `TipoCodice`, del prodotto che si vuole vendere come servizio, al valore `S` . <br>
Non sono richieste ulteriori configurazioni particolari.

> Se il firmware del Registratore Telematico è appena stato aggiornato oppure la versione di Posware è appena stata installata, la vendita come servizio sarà possibile solo dopo la prima chiusura fiscale (necessaria al fine di inviare la tabella IVA aggiornata con i reparti dedicati ai servizi)


### Gestione multi-ateco

Posware supporta la gestione multi-ateco del Registratore Telematico. In base al Registratore Telematico in uso, sono presenti limitazioni dettate dall'hardware. <br>
In caso di Server Rt **è necessario** inviare la configurazione Iva/Reparti come da indicazioni alla sezione [Configurazione Server RT](#configurazione-server-rt)

#### Configurazione

1. Se per variare le impostazioni ateco del Registratore Telematico è richiesta la chiusura fiscale, effettuarla da Posware oppure tramite il tool del Registratore Telematico.
3. Se Posware è aperto, chiudere Posware.
4. Abilitare la funzionalità sul Registratore Telematico e programmare i codici Ateco. 
5. Inserire nella tabella `reparti`, il codice Ateco (oppure l'indice assegnato dal Registratore Telematico) nel campo `RTAtecoIndex` del reparto che si vuole assegnare ad un determinato codice ateco.
6. Abilitare la feature impostando nella tabella di configurazione `tabparametriextra` il parametro `isRTMultiATECOCodeSupportEnabled` del modulo `posware.fiscalprinter` al valore `True`.  
7. Avviare Posware. Qualora si dovesse ricevere un errore di configurazione, effettuare una chiusura fiscale e riavviare Posware.
8. Qualora si dovesse continuare a ricevere un errore di configurazione, è necessario verificare la programmazione.

#### Sistema di verifica della configurazione
All'avvio di Posware, verrà verificata la configurazione dei codici ateco del Frontend rispetto a quella programmata nella stampante. In caso venisse rilevata una configurazione errata, verrà mostrato un messaggio popup con uno di questi possibili messaggi di errore:

```
Sono presenti XX reparti con indice ateco 0. Se il supporto a codici ateco multipli è attivo, per ogni reparto cassa è obbligatorio specificare un indice codice ateco che sia diverso da 0.
```
oppure:

```
Il numero degli indici ateco configurati nei reparti di Posware (XX) supera quelli programmati nella stampante (YY). Verificare la programmazione.
```

oppure:

```
Nella stampante sono stati configurati dei codici ateco. Abilitare la funzionalità multi ateco di Posware in tabparametriextra oppure rimuovere gli indici programmati nella stampante. Verificare la programmazione.
```
Tutti gli errori sono auto-esplicativi. <br>
**Anche in caso di Server Rt Epson, Posware controlla lo stato di attivazione della funzione.** <br>
**È mandatorio allineare le impostazioni multi-ateco tra Posware e Registratore Telematico. In caso contrario Posware si chiuderà automaticamente alla chiusura del messaggio di avviso.**

#### Limitazioni
* Il Registratore Telematico non permette di mischiare in una unica transazione, prodotti appartenenti a diversi codici ateco. <br>
In tal caso la transazione verrà annullata causando errore opos `207` oppure errore opos generico.
* Per effettuare cambiamenti ai codici ateco o abilitare/disabilitare il supporto multi ateco, è quasi sempre necessario effettuare una chiusura fiscale.
* La programmazione dei codici ateco non viene effettuata dal Posware. È necessario utilizzare i tool specifici del Registratore Telematico
* Non è possibile in nessun caso assegnare un prodotto ad uno specifico codice ateco. Solo i reparti possono essere assegnati, di conseguenza tutti i prodotti di quel determinato reparto verranno venduti su quello specifico codice ateco.


## Tabella di compatibilità delle funzionalità
Il supporto è da considerarsi a partire dalla versione `4.3.0` se non specificato diversamente nelle note.

| Funzionalità | EPSON | RT-ONE | Server RT EPSON   |  Note
| ----------- | -------|--------|-------------------|-------
Nuove forme di pagamento| Supportato | Supportato|Supportato|
Ticket numerati manuali|Supportato|Supportato|Supportato|
Ticket numerati su pagamento elettronico e dematerializzato|Supportato|Supportato|Supportato|
Nota di credito|Supportato|Supportato|Supportato|
Arrotondamento tramite Registratore Telematico|Supportato|Supportato|Supportato|
Vendita di servizi|Supportato|**Non supportato da Posware**|Supportato|
Gestione multi-ateco|Supportato|**Non supportato da Posware**|Supportato|

### Addendum
* Nel caso di nota di credito manuale e tipologia documento **Matricola**, è possibile inserire il codice lotteria durante la transazione qualora presente nel documento commerciale di origine. Come da normativa, non è possibile imputare il codice lotteria in caso di tipologia documento **ND** o **POS**. In tal caso verrà segnalata operazione non valida ed il codice verrà ignorato.
* Il codice lotteria in fase di nota credito può essere imputato solo dopo aver inserito tutti gli articoli nella transazione e prima della scelta del pagamento. In tutte le altre fasi guidate, sarà segnalata una operazione non consentita.
* In caso di presenza nella transazione di un **codice fiscale**, il codice lotteria *rimuove* il codice fiscale. Porre attenzione a quale dei due codici erano presenti sul documento di origine. I due codici sono sempre mutuamente esclusivi.

## Configurazione Server RT

### Cenni preliminari

Per poter utilizzare le funzioni di vendita a servizi, vendita con codici in esenzione e/o multiateco è **necessario** effettuare la trasmissione Iva/Reparti su ServerR.

### Configurazioni

E' stata introdotta una funzione di trasmissione Iva/Reparti su ServerRt che permette la corretta gestione della programmazione in Posware. Questa funzione è richiamabile manualmente, predisponendo un terminale specializzato, ed incrociando il codice `TX_PROG_SERVER_RT` presente nella tabella `Config_Tast` (inserirlo se mancante) in grafica ad un opportuno pulsante. <br>
Al fine di configurare opportunamente la tabella delle Ive occorre tener presente che i codici di esenzione dovranno seguire quanto specificato nella seguente tabella:

| IndiceVatTable | Simb.Stampa | Descrizione | Natura |
| ---- | ----| ----| ---- |
| 0 | ES | Esente | N4 |
| 10 | EE | Esclusa | N1 |
| 11 | NS | Non Soggetta | N2 |
| 12 | NI | Non Imponibile | N3 |
| 13 | RM | Regime del margine | N5 |
| 14 | AL | Operazione non IVA | N6 |

!!! tip "Consulare la documentazione EPSON"
    La tabella sopra riportata segue quanto indicato dal manuale Epson per il Server Rt, per variazioni o tabelle aggiornate fare sempre riferimenti alla documentazione ufficiale.

### Limitazioni e vincoli

- Questa programmazione sarà consentita solo a **giornata chiusa**
- Lo stato di **giornata chiusa** nei server RT si ottiene dopo aver eseguito la chiusura fiscale di tutti i tills e la chiusura del Server stesso
- Se la **giornata è aperta** sarà mostrato un messaggio che informa che lo stato del Server non è congruente.
