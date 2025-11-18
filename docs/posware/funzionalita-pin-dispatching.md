---
tags:
    - Posware
    - Posware Frontend
    - Pin Dispatching
    - Argentea
    - Euronet
---

# Posware - Funzionalità Pin Dispatching

**Prima revisione documento: 27 agosto 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Manuale di attivazione ed utilizzo

Il presente documento descrive i requisiti minimi e le procedure necessarie all'attivazione della funzionalità Pin Dispatching presente in Posware.

---

## Versioni software di riferimento

Posware **4.3.0** o superiore

## Requisiti

### Posware

La versione minima di Posware richiesta è la `v4.3.0`. <br>
Alcune funzionalità presenti in questo documento potrebbero non essere supportate dal provider. Vedi [Tabella di compatibilità delle funzionalità](#tabella-di-compatibilita-delle-funzionalita).

### Provider supportati

A far data **11/09/2024** i provider supportati sono:

* Euronet tramite <mark>Argentea</mark>
* Epipoli/TSP tramite <mark>Argentea</mark>

Per conoscere le funzionalità supportate dal singolo provider consultare la [tabella di compatibilità delle funzionalità](#tabella-di-compatibilita-delle-funzionalita).

## Configurazione

La configurazione del plugin è suddivisa in tre operazioni:

1. Compilazione dell'anagrafica dei prodotti pin-dispatching
2. Configurazione di Posware
3. Configurazione del provider
4. Inserimento del pulsante _Ristampa ricevuta_ in grafica (opzionale)

### Compilazione anagrafica dei prodotti pin-dispatching

Per la compilazione dell'anagrafica dei prodotti, sono previsti due scenari:

1. Il prodotto da attivare è riconosciuto **esclusivamente tramite Barcode** (tipicamente EAN). I dati del prodotto vengono inseriti **solo nella tabella `articoli`**.
2. Il prodotto da attivare è riconosciuto tramite Barcode ed un _**Codice ricarica**_. Il Codice ricarica potrebbe essere legato ad un _codice operatore_. I dati del prodotto devono essere inseriti sia nella tabella `articoli` che nella tabella `codiciricarica`. Il _Codice ricarica_, assieme ad altri dati, viene collegato tramite Barcode alla tabella `articoli`.

#### Campi da compilare nella tabella articoli

Sono obbligatori i campi presenti nella tabella di esempio. Qualora si volesse, il campo `NonScontabile` può essere impostato anche a `0`.

| CodiceEAN      | Descrizione         |UM|bpeso| Prezzo | IVA | NonScontabile
| -----------       | ---------------- |-----|-----|---|---|----|
8032325292640| RICARICA WIND TRE 20 EURO|PZ|false|20,00|0|1

#### Campi da compilare nella tabella codiciricarica

Sono possibili 3 modalità di codifica differente:

1. Il provider utilizza Barcode che differiscono dai Codici ricarica.
2. Il provider utilizza Barcode che differiscono dai Codici ricarica ed è richiesto anche il Codice operatore.
3. Il provider utilizza Barcode e Codici ricarica identici ma è richiesto il Codice operatore.

Nella tabella a seguire sono stati codificati 4 prodotti (ricariche telefoniche) per dimostrare i 3 scenari indicati.

| CodOperatore      | CodTaglio          |CodRicarica|CodiceEAN| importoeurocent
| -----------       | ---------------- |-----|-----|---|
0| 0|`5060069483461`|`8032325292640`|2000|0
1| 0|`5060069483461`|`8032325292640`|3000|0
2| 0|`5060069488503`|`5060069488503`|5000|0
0| 1|`8032325056389`|`5060069484413`|1000|0

La **modalità 1** si realizza seguendo la codifica del primo record: si può notare l'aggancio del _Codice ricarica_ (campo **CodRicarica**) con il Barcode (campo **CodiceEAN**).

La **modalità 2** si realizza seguendo la codifica del secondo record: si può notare che il _Codice ricarica_ differisce dal Barcode (**CodiceEAN**) ed è stato indicato anche un _Codice operatore_ (**CodOperatore**).

La **modalità 3** si realizza seguendo il terzo record: si può notare che il _Codice ricarica_ corrisponde anche al Barcode (**CodiceEAN**). In questo caso particolare l'aggancio verrà utilizzato solo per legare un _Codice operatore_ (**CodOperatore**) al prodotto. 

Per evitare sovrapposizione nelle codifiche, è possibile avere per un determinato Codice operatore solo un particolare Codice Taglio (legame 1:1). <br>
**In tutti gli scenari cui sopra, il campo `CodTaglio` viene usato solo come progressivo per identificare univocamente un record rispetto ad un Codice operatore. Non è coinvolto in altre logiche.** <br>
È possibile notare ciò nel **quarto record di esempio**.

!!! tip "In breve"
    Qualora per il provider in uso non fosse un requisito necessario **specificare Codice operatore**, è possibile impostare il campo `CodOperatore` sempre a `0` ed usare un progressivo numerico libero per il campo `CodTaglio`. **In questo modo si utilizzerà la tabella solo per specificare un Codice ricarica differente dal Barcode**.

#### Campi da compilare nella tabella tipi_ean

È necessario inserire un record di tipo `R` nella tabella `tipi_ean` per ogni barcode (o parte di esso) che è necessario identificare come barcode di un servizio con attivazione tramite pin dispatching.

Per la codifica sono necessari solo 3 campi: **ean, descrizione, tipo**. I restanti possono essere lasciati come da valore di default.

| ean            | descrizione      |tipo |
| -------------- | ---------------- |-----|
8032325292640    | RICARICA TIM 20€ |`R`  |
803232528        | RICARICHE AMAZON |`R`  |

In tabella vengono riportati due esempi di codifica: il primo permette di assegnare per ogni singolo ean il riconoscimento come **servizio da attivare via pin dispatching**, il secondo contrassegna invece tutti gli EAN che iniziano con _**803232528**_ come **servizi da attivare via pin dispatching**.

#### Configurazione di Posware

**Per abilitare il plugin in Posware è per prima cosa necessario impostare il valore del parametro `PROTRICARICHE` del modulo `EPPLIB` nella tabella `tabparametriextra`.**

È inoltre possibile specificare se effettuare le operazioni di ricarica pin dispatching - richiesta ed attivazione - a chiusura scontrino. <br>
Per fare ciò impostare il campo `ricariche_alla_chiusura` della tabella `config_cassa` al valore `true`. <br>
Si consiglia di lasciare il valore impostato a `false` per avvantaggiarsi delle funzioni di validazione durante la vendita.

A seguire le impostazioni della tabella `tabparametriextra` per i singoli provider supportati.

=== "Impostazioni provider Argentea"
    È necessario specificare il solo protocollo ricariche. Tutte le altre configurazioni dovranno essere effettuate direttamente sui software Argentea.

    | Modulo      | Parametro   | Valore    | Valore di default    | Note |
    | ----------- | ----------- |-----------|----------------|------|
    | `EPPLIB`    | `PROTRICARICHE`|`AR`|  | AR = Argentea

=== "Impostazioni provider Euronet"

    | Modulo      | Parametro   | Valore    | Valore di default    | Note |
    | ----------- | ----------- |-----------|----------------|------|
    | `EPPLIB`    | `PROTRICARICHE`|`EN`|  | EN = Euronet
    | `EPPLIB`    | `EPAYMODE`|`RESERVE`|  | Lasciare inalterato il valore
    | `EPPLIB`    | `EPAYUSER`||  | Nome utente
    | `EPPLIB`    | `EPAYPASSWORD`||  | Password
    | `EPPLIB`    | `EPAYTERMINALID`||  | Id Terminale Euronet
    | `EPPLIB`    | `EPAYTIMEOUT`||`10000`  | Timeout di comunicazione in ms
    | `EPPLIB`    | `EPAYURL`||  | Indirizzo del gateway di comunicazione Euronet
    | `EPPLIB`    | `EPAYSTAMPAINT`|`0/1`| `0` | 1 = Stampa sulle ricevute l'intestazione del punto vendita comunicata ad Euronet.
    | `EPPLIB`    | `EPAYSCADFILTER`|`802109601,8056735`| `802109601,8056735` | Lasciare inalterato il valore

#### Inserimento del pulsante _Ristampa ricevuta_ in grafica

Per poter utilizzare la funzionalità di ristampa della ricevuta di attivazione, è necessario inserire un pulsante in grafica associandogli la funzione `RISTAMPA_RICARICA`.

## Procedura di installazione

Il plugin è direttamente integrato in Posware.

1. Aggiornare Posware tramite l'applicativo di aggiornamento `PosUpdate`.
2. Configurare Posware
3. Inviare l'anagrafica dei prodotti (_articoli_, _codici ricarica_, _tipi ean_)

## Funzionalità

### Attivazione di un servizio

Per attivare un servizio è necessario effettuare la vendita del prodotto tramite barcode. <br>
Il barcode viene riconosciuto come _**ricarica**_. Viene effettuata sia la vendita del prodotto in se che l'_**autorizzazione di attivazione al provider**_. <br>
Al termine della richiesta viene visualizzato il _**numero autorizzazione**_ sullo scontrino digitale (Auth. 123456).

!!! warning "Possibile fallimento dell'operazione"
    Qualora l'autorizzazione non dovesse andare a buon fine, verrà mostrato l'errore con il motivo ed il prodotto stornato.

È ora possibile pagare e chiudere la transazione. <br>
Al pagamento, il servizio viene definitivamente attivato e la transazione si conclude: oltre allo scontrino della transazione viene **stampata la ricevuta di attivazione con le istruzioni per usufruire del servizio acquistato**.

!!! warning "Possibile fallimento dell'operazione"
    Qualora la richiesta di attivazione finale non dovesse andare a buon fine, verrà mostrato l'errore con il motivo ed il prodotto stornato. <br>
    **La transazione non verrà chiusa e sarà possibile riprovare.**

### Ristampa Pin di un servizio attivato
Per ristampare la ricevuta con le istruzioni per usufruire del servizio acquistato, **è necessario avere lo scontrino oppure i riferimenti della transazione originale.** <br>
È richiesto inoltre l'inserimento di un pulsante con associata la funzione `RISTAMPA_RICARICA`.

Il workflow è il seguente:

1. Pressione del pulsante **Ristampa servizio PinDispatching**
2. Si inserisce il numero transazione originale, il numero della cassa che lo ha emesso e la data
3. La ricevuta viene ristampata. **Non viene ristampato lo scontrino originale, solo la ricevuta relativa al servizio di pin dispatching**

!!! warning "Funzione provider dipendente"
    Non tutti i provider supportano la ristampa della ricevuta.

## Tabella di compatibilità delle funzionalità

Il supporto è da considerarsi a partire dalla versione `4.3.0` se non specificato diversamente nelle note.

| Funzionalità | Provider Euronet | Provider Epipoli/Tsp via Argentea |  Note |
| ------------ | ---------------- |-----------------------------------|-------|
Attivazione del servizio| Supportato | Supportato|
Ristampa della ricevuta|Supportato|Supportato| Necessario il riferimento allo scontrino originale
Annullo (reso) di una ricarica avvenuta | Supportato|Non supportato| Annullo possibile solo se non si è usufruito del servizio in maniera completa o parziale