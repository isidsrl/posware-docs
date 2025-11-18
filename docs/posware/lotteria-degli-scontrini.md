---
tags:
    - Posware
    - Posware Frontend
    - Lotteria
    - Lotteria degli scontrini
---

# Posware - Lotteria degli scontrini

**Prima revisione documento: 27 agosto 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

Il presente documento descrive i nuovi requisiti minimi e le procedure necessarie all'attivazione della funzionalità Lotteria degli scontrini.

## Versioni software di riferimento

Posware **4.3** o superiore

## Requisiti

### Touchscreen

La funzionalità della lotteria degli scontrini è compatibile solo con sistemi touchscreen. Non è previsto il supporto tastiera.

### Stampanti

A far data **25/11/2020** le stampanti RT abilitate alla lotteria degli scontrini sono:

* Epson - tutti i modelli
* NCR
* RT-ONE

### Server RT
Il server RT Epson è supportato. **Non è previsto il supporto ad altri server RT.**

!!! danger "Firmware stampanti"
    E' necessario **aggiornare le stampanti all'ultima versione di firmware** disponibile con il supporto alla lotteria degli scontrini.<br>
    Qualora non viene aggiornato il firmware **e viene inserito un codice lotteria**, la stampante andrà in errore annullando la transazione.

## Configurazioni

Di seguito le impostazioni configurabili nella tabella `tabparametriextra`:

| Modulo      | Parametro   | Descrizione    | Valore di default    | Note |
| ----------- | ----------- |-----------|----------------|------|
| `posware.fiscalprinter`    | `lotteryCodeAlphanumericCheckEnabled`|Il codice deve contenere obbligatoriamente sia lettere che numeri| `true` | Quando abilitata, se il codice contiene solo lettere o solo numeri, non viene accettato. Sono accettati solo stringhe che contengono sia lettere che numeri **contemporaneamente**. *Spazi mai ammessi (anche con impostazione disabilitata)*
| `posware.fiscalprinter`    | `lotteryCodeMaximumLength`|Lunghezza massima possibile per il codice lotteria | `8` | Impostando un valore pari a zero `0` la validazione non viene eseguita. Valori errati non fanno scattare la validazione e vengono riportati nel log errori.
| `posware.fiscalprinter`    | `lotteryCodeMinimumLength`|Lunghezza minima richiesta per il codice lotteria | `8` | Impostando un valore pari a zero `0` la validazione non viene eseguita. Valori errati non fanno scattare la validazione e vengono riportati nel log errori.
| `posware.fiscalprinter`    | `showRTLotteryCodeAcceptedMessageToCustomer`|Visualizzazione del messaggi di accettato del codice lotteria | `true` | Se `false`, disabilita la visualizzazione del messaggio di accettazione codice lotteria sul display cliente. Questa impostazione è disponibile anche a partire dalla versione `4.1.300`

`lotteryCodeMaximumLength` e `lotteryCodeMinimumLength` possono essere abilitati/disabilitati anche singolarmente.

Tutte le configurazioni, *se non presenti in `tabparametriextra`*, al bootstrap di Posware in vengono **inizializzate automaticamente al valore di default riportato in tabella**. <br>
In caso di valore imputato fuori dal range permesso, vengono utilizzati i valori di default senza causare blocchi o notificare errori all'operatore.

## Registrazione nei log

Il codice lotteria viene riportato nel log cassa e nelle tabelle carico/scarico su `carscar_info`. L'informazione viene centralizzata a livello punto di vendita e sede.

### Log

Il record è di tipo `14`. I campi sono valorizzati secondo tabella:

| Tipo Informazione      | Codice Informazione   | Codice a barre    | Informazione |
| ----------- | ----------- |-----------|----------------|
| `S`   | `RTLC`| |  Codice lotteria (allineato a destra)|

### Carscar_info

I campi sono valorizzati secondo tabella:

|id_carscar | CodInfo      | Tipo   | Barcode    | Info |
|-----------| ----------- | ----------- |-----------|----------------|
| ParentId Carscar | `RTLC`| `S`   | |  Codice lotteria| 

## Procedura di installazione

1. Aggiornare Posware tramite la comune procedura di `PosUpdate`.
2. Configurare un nuovo pulsante nella tabella `grafica` con il campo `oper` impostato a `SET_RT_LOTTERYCODE`

## Workflow di utilizzo

### Vendita

1. Premere il tasto configurato come lotteria degli scontrini.
2. Alla richiesta, digitare il codice lotteria
3. Il front-end trasmetterà al registratore telematico il codice. Se accettato uscirà un messaggio di accettazione, altrimenti un errore. In caso di errore il codice non verrà applicato alla transazione.

!!! warning "La stampante non verifica la validità del codice lotteria"
    Diversi registratori telematici potrebbero accettare in modo silente anche un codice non esistente o formalmente errato.<br>
    In questo caso il codice errato verrà generalmente inviato all'AdE **ma non permetterà di partecipare alla lotteria degli scontrini.**

Il codice lotteria inserito verrà anche mostrato sul display cliente con un messaggio "Codice Lotteria accettato: XXXXXX". Per disabilitare questo messaggio vedere le note in [Addendum](#addendum) o consultare la tabella delle configurazioni [Configurazioni](#configurazioni)

### Nota di credito

Il codice lotteria viene automaticamente inserito da Posware, non è necessario aggiungerlo manualmente alla transazione. Se si tenterà di modificare il codice inserendone un altro, verrà mostrato un messaggio di *OPERAZIONE NON CONSENTITA*. Come previsto dalla normativa, in una nota credito non sono permesse operazioni manuali sul codice lotteria.

## Addendum

* E' possibile inserire il codice lotteria in qualsiasi momento durante una transazione di vendita **attiva**. Se la transazione non è iniziata, non è possibile inserire il codice lotteria.
* In caso di inserimento multiplo, l'ultimo codice inserito verrà applicato alla transazione ed il precedente sovrascritto
* In caso di presenza nella transazione di un **codice fiscale**, il codice lotteria *rimuove* il codice fiscale e non si potrà richiedere fattura/altri doc. fiscali a seguire. Questo comportamento è quanto previsto dalla normativa.
* E' possibile disabilitare la visualizzazione del messaggio di accettazione codice lotteria sul display cliente impostando, in `tabparametriextra`, a `false` il parametro `showRTLotteryCodeAcceptedMessageToCustomer` del modulo `posware.fiscalprinter`