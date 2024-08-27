---
tags:
    - Posware
    - Posware Frontend
---

# Overview generale

**Prima revisione documento: 27 agosto 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Cenni preliminari
Il presente documento descrive i nuovi requisiti minimi e le procedure necessarie all'installazione o all'aggiornamento di *Posware Frontend* alla versione 4.3 dell'applicativo, così da poter associare le casse con lo *StoreServer*, il nuovo sistema centralizzato per la gestione e l'analisi del punto vendita.

## Requisiti minimi
### Hardware
- Un processore compatibile con il sistema operativo installato sulla cassa
- 8 Gb di RAM in caso il sistema operativo sia Windows 10 o 11 [vedere con Alberto al suo ritorno]
- 4 Gb di RAM in tutti gli altri casi
- Disco rigido SSD (tipo SATA o NVE) da almeno 128 Gb
- Connessione ad Internet per l'installazione delle dipendenze

### Software
- Windows 7 **SP1** con update **ServiceStack 2018** o superiore
- Windows 10 v. `1809` **Redstone 5 Build 17763** o superiore
- Windows 11
- .NET Framework v. `4.8`
- WebView2 Edge Runtime v. `109.0.1518.140`
- .NET v. `6` Hosting Bundle
- Monroes OPOS Common Control Objects v. `1.14.001`
- Runtime di Argentea
- MySQL v. `4.1` o v. `5.6` in caso di migrazione da *Posware Frontend* v. `4.2`
- MySQL v. `5.6` in caso di installazione ex novo

Si prega di consultare attentamente le note varie sulle dipendenze di terze parti e gli aggiornamenti necessari nella relativa [sezione](./dipendenze-aggiornamenti-necessari.md) prima di procedere con l'installazione o l'aggiornamento di *Posware Frontend*. 

## Breaking changes
Vengono elencati di seguito i diversi **breaking changes** introdotti in *Posware Frontend* a partire dalla versione `4.3` dell'applicativo:

- Il formato del **barcode** nel record **01** è soltanto con gli spazi davanti, mentre fino a Posware v. `4.2` era denominato **Barcode No Food**.
- Per il motivo appena elencato, l'impostazione **Barcode No Food** nella tabella `config_cassa` non ha più senso, perché è come se fosse sempre impostata ad **1**.
- A partire dalla versione `4.3`, i file di configurazione di Posware, delle *Connection strings* e delle impostazioni utente sono stati riportati allo standard .NET rimuovendo l'approccio custom. Per maggiori informazioni consultare la relativa [sezione](./setup-posware.md#aggiornamento-dei-file-di-configurazione) del *SetupPosware*.

## Feature rimosse in Posware `4.3`
Nell'elenco sottostante vengono illustrate tutte le **feature di *Posware Frontend* che sono state rimosse** a partire dalla versione `4.3` dell'applicativo:

- RFImbopos e le sue dipendenze 
- Supporto alle stampanti IBM collegate a RFImbopos, assieme alle sue .dll e dipendenze varie (come, ad esempio, fiscal232.dll, ecc...)
- Supporto alle stampanti NCR vecchie MF FASY, RealPrint e RealPOS
- Supporto alle stampanti Wincor TH230 e vecchie Wincor
- Controllo OCX e sue dipendenze che venivano usate per mostrare la telecamera
- La .dll WENcrGTNPos.dll obsoleta, che aveva qualcosa a che fare con WinEPTS e GTN è stata non solo rimossa, ma anche sostituita

## Feature deprecate che verranno rimosse in Posware `4.4`
Le seguenti **feature di *Posware Frontend* sono deprecate** a partire dalla versione `4.3` dell'applicativo e verranno quindi poi rimosse in Posware v. `4.4`:

- Gestione commissione di vendita, sia la nostra che quella di Ibed
- Nota di credito di Posware v. `4.2`, sia quella interna di ISiD, che quella di GTN che supporta il Tax free, sia quella di Pellizzari. Le relative funzionalità di ognuna delle tre sono infatti ora coperte dalla nuova nota di credito di Posware v. `4.3`. In particolare, sono deprecate soltanto le procedure di nota di credito totale e parziale, mentre la nota di credito manuale di Posware v. `4.2` continua ad essere utilizzata anche nella nuova procedura di nota di credito di Posware v. `4.3`.
- Satispay v. `4.2`
- GetYourBill

## Installazione e aggiornamenti
Sono previsti questi due applicativi per le procedure di installazione e di aggiornamento di *Posware Frontend*:

- [SetupPosware](./setup-posware.md)
- [PosUpdate](./pos-update.md)

In particolare, il *SetupPosware* è il nuovo setup riprogettato per l'uscita di Posware v. `4.3` per poter installare *Posware Frontend* sui terminali casse, sia nel caso di migrazione da Posware v. `4.2.x`, sia nel caso di installazione ex novo dell'applicativo ed è usato quindi esclusivamente solo in queste due casistiche. Il *PosUpdate*, invece, è l'applicativo che viene usato esclusivamente per aggiornare *Posware Frontend* sui terminali casse alla loro ultima **Build** version, in linea con la **Major** e **Minor** version installata. 

I dettagli specifici sul funzionamento dei due applicativi sono consultabili nelle rispettive sezioni.