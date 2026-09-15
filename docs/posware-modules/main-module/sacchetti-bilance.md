---
tags:
    - Posware
    - StoreServer
---

# Posware Module - Sacchetti bilance

**Prima revisione documento: 15 settembre 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Glossario

- **WebApp:** applicazione web che permette di gestire in modo centralizzato le funzionalità disponibili nello *StoreServer*;
- **Sospeso:** documento "parcheggiato" da una cassa in attesa di essere ripreso;
- **Sacchetto bilancia** (o *sacchettone*): documento sospeso di tipo sacchettone, emesso dalle bilance del reparto;
- **Stato:** condizione del sacchetto — *Pendente* (in attesa di essere elaborato in cassa) oppure *Elaborato*;
- **Riga articolo:** singolo prodotto pesato che compone il sacchetto.

## Cenni preliminari

La funzione **Sacchetti bilance** permette all'utente di:

- **consultare i sacchetti bilancia** emessi in una giornata, con filtri per stato e ricerca per codice a barre;
- **cambiare lo stato** di un singolo sacchetto, contrassegnandolo come *pendente* o come *elaborato*;
- **consultare le righe articolo** di un sacchetto;
- **esportare in CSV le righe articolo** di un sacchetto, per importarle manualmente quando la cassa non lo ha elaborato;
- **esportare** l'elenco visualizzato in Excel, PDF, stampa o appunti.

La funzione si trova nella sezione ***Gestione archivi*** della **WebApp** dello *StoreServer*.

!!! info "Sostituzione di un programma di back-office"
    Questa funzione sostituisce l'utility desktop *Visualizzazione Sospeso* (`Pos_Sospesi.exe`) limitatamente ai documenti di tipo sacchettone. Le differenze rispetto al vecchio programma sono elencate nel capitolo [Differenze rispetto a Pos_Sospesi](#differenze-rispetto-a-pos_sospesi).

### Versioni software

La funzione è inclusa nel **PoswareModule** dello *StoreServer*. È necessario che il modulo sia licenziato e attivo.

## Installazione

La funzione è inclusa **out-of-the-box** nel *PoswareModule*: non richiede installazioni aggiuntive né interventi sulla base dati, perché utilizza le tabelle dei sospesi già scritte dalle casse e dalle bilance.

## Permessi richiesti

| Permesso | Cosa consente |
|---|---|
| `store.data.management.view` | Accesso alla voce di menu, consultazione dell'elenco e delle righe articolo, esportazioni (elenco e CSV del dettaglio) |
| `store.data.management.manage` | Cambio di stato dei sacchetti (*Contrassegna come pendente*, *Contrassegna come elaborato*) |

Entrambi i permessi appartengono ai ruoli **SystemAdmin**, **Technician** e **StoreManager**. Non sono assegnabili a ruoli personalizzati: si veda il documento [Gestione ruoli e permessi](../ruoli-permessi.md).

!!! note "Utenti in sola consultazione"
    Se l'utente dispone solo di `store.data.management.view`, in cima alla pagina compare l'avviso *"Sola consultazione"* e le due voci di cambio stato del menu azioni risultano disabilitate. **Dettaglio righe** ed **Esporta dettaglio (CSV)** restano sempre disponibili.

## Funzionalità

### Accesso alla funzione

Nella barra laterale della WebApp selezionare ***Gestione archivi*** e poi la voce ***Sacchetti bilance***.

All'apertura la pagina mostra i sacchetti della **giornata odierna**, in **tutti gli stati**, ordinati per **data e ora** crescenti.

### Barra dei filtri

La barra in alto a destra della scheda contiene i tre filtri e il menu di esportazione.

| Controllo | Comportamento | Valore iniziale |
|---|---|---|
| **Data** | Seleziona il giorno da consultare. La ricerca riguarda sempre un solo giorno | Data odierna |
| **Stato** | *Tutti*, *Pendente*, *Elaborato* | *Tutti* |
| **Ricerca** | Filtra per codice a barre del sacchetto | vuoto |
| **Esporta** | Menu con i formati di esportazione dell'elenco | — |

Ogni modifica ai filtri ricarica immediatamente l'elenco: non è previsto alcun pulsante di conferma.

#### Ricerca per codice a barre

Il campo **Ricerca** lavora sul codice del sacchetto e si comporta in due modi:

- digitando **esattamente 13 caratteri** (la lunghezza di un EAN13) viene cercato il codice **esatto**;
- digitando **un numero diverso di caratteri** vengono cercati tutti i codici che **iniziano** con quanto digitato.

!!! tip "Lettura con pistola"
    Il campo accetta al massimo 20 caratteri. Leggendo l'etichetta del sacchetto con il lettore di codici a barre si ottiene direttamente il singolo documento.

### Elenco dei sacchetti

L'elenco riporta, per ciascun sacchetto:

| Colonna | Contenuto |
|---|---|
| **Codice** | Codice a barre del sacchetto. Cliccandolo si aprono le righe articolo |
| **Data e ora** | Momento di emissione, nel formato `GG/MM/AAAA HH:MM` |
| **Stato** | *Pendente* oppure *Elaborato* |

Le intestazioni di colonna sono cliccabili per cambiare l'ordinamento; ordinando per **Data e ora** i sacchetti vengono disposti in ordine cronologico (data e, a parità di data, ora). In fondo all'elenco si trovano il conteggio dei risultati a sinistra, la paginazione al centro e il selettore del numero di righe per pagina (10, 25 o 50) a destra.

!!! info "Caricamento a pagine"
    I dati vengono richiesti al server una pagina alla volta: anche giornate con migliaia di sacchetti si aprono rapidamente. Ordinamento, ricerca e cambio pagina comportano sempre una nuova interrogazione.

### Azioni su un sacchetto

Su ogni riga il pulsante **Azioni** apre un menu con quattro voci.

| Voce | Effetto |
|---|---|
| **Contrassegna come pendente** | Porta il sacchetto nello stato *Pendente*: la cassa potrà elaborarlo di nuovo |
| **Contrassegna come elaborato** | Porta il sacchetto nello stato *Elaborato* |
| **Dettaglio righe** | Apre la finestra con gli articoli del sacchetto |
| **Esporta dettaglio (CSV)** | Scarica il file CSV con le righe articolo del sacchetto (si veda [Esportazione del dettaglio in CSV](#esportazione-del-dettaglio-in-csv)) |

Dopo il cambio di stato l'elenco viene ricaricato mantenendo la pagina corrente. Se il filtro **Stato** non comprende più il nuovo stato, la riga sparisce dall'elenco: è il comportamento atteso, non una perdita di dati.

!!! note "Nessun vincolo tra gli stati"
    Un sacchetto elaborato può essere riportato a pendente e viceversa, senza conferme. Il sistema **non registra** l'autore né il momento del cambio di stato. L'annullamento non è previsto: non è un'operazione applicabile ai sacchetti bilancia.

### Dettaglio delle righe articolo

La finestra **Dettaglio sacchetto** mostra in sola lettura gli articoli che compongono il sacchetto:

- **Articolo** — codice dell'articolo;
- **Descrizione**;
- **Qtà** — quantità pesata;
- **Prezzo**;
- **Tipo**.

Se il sacchetto non ha righe viene mostrato il messaggio *"Nessuna riga presente per questo sacchetto"*.

Le righe si aprono in due modi: dalla voce **Dettaglio righe** del menu azioni oppure cliccando direttamente sul **Codice** nella prima colonna.

Nell'intestazione della finestra, a sinistra del pulsante di chiusura, il pulsante **Esporta dettaglio (CSV)** scarica le stesse righe in formato CSV; è disabilitato quando il sacchetto non ha righe.

### Esportazione del dettaglio in CSV

L'esportazione è pensata per il caso in cui la cassa **non riesca a elaborare automaticamente** un sacchetto: il file permette di importare i singoli articoli nel back-office di punto vendita o in un altro strumento.

Si ottiene dalla voce **Esporta dettaglio (CSV)** del menu azioni oppure dal pulsante omonimo nella finestra di dettaglio. Il file scaricato si chiama `sacchetto_<codice>.csv` e contiene una riga per articolo con le seguenti colonne, nell'ordine indicato:

| Colonna | Contenuto |
|---|---|
| `BarcodeSacchetto` | Codice a barre del sacchetto (uguale per tutte le righe) |
| `CodArt` | Codice dell'articolo |
| `Descrizione` | Descrizione dell'articolo |
| `Qta` | Quantità pesata |
| `Iva` | Aliquota IVA |
| `Prezzo` | Prezzo di riga |
| `IdRiga` | Numero progressivo della riga nel sacchetto |
| `Prezzounit` | Prezzo unitario (vuoto se non valorizzato dalla bilancia) |

Formato del file: separatore di campo **virgola** (`,`), decimali con il **punto** (`2.500`), codifica **UTF-8**, righe terminate da CRLF. Il file è generato con la libreria [CsvHelper](https://joshclose.github.io/CsvHelper/) ed è conforme allo standard **RFC 4180**: le descrizioni che contengono virgole, doppi apici o ritorni a capo vengono racchiuse tra doppi apici (con i doppi apici interni raddoppiati), quindi possono essere lette da qualunque importatore CSV standard senza spostamenti di colonna. Il file contiene tutti i campi della riga articolo tranne il tipo riga, non necessario per l'importazione.

!!! tip "Apertura in Excel"
    Con Excel in lingua italiana un doppio clic sul file può mostrare tutti i campi in un'unica colonna, perché Excel si aspetta il punto e virgola. Usare **Dati → Da testo/CSV** indicando la virgola come delimitatore, oppure importare il file direttamente nel software di cassa.

### Esportazione dell'elenco

Il menu **Esporta** nella barra dei filtri mette a disposizione quattro formati:

| Voce | Risultato |
|---|---|
| **Copia negli appunti** | Copia i dati come testo tabellare |
| **Excel** | Scarica un file `.xlsx` |
| **PDF** | Scarica un file `.pdf` in orientamento orizzontale |
| **Stampa** | Apre l'anteprima di stampa del browser |

!!! warning "L'esportazione riguarda la pagina visualizzata"
    Poiché i dati sono caricati a pagine, l'esportazione produce **solo le righe della pagina corrente**, non l'intero risultato del filtro. Per esportare più righe in un colpo solo, aumentare il numero di righe per pagina a 50 prima di esportare. Lo stesso avviso compare passando il mouse sul pulsante **Esporta**.

La colonna delle azioni non viene inclusa nei file esportati.

## Differenze rispetto a Pos_Sospesi

| Aspetto | `Pos_Sospesi.exe` | Sacchetti bilance |
|---|---|---|
| Documenti trattati | Tutti i sospesi (vendite, sospesi, note di credito, ordini, sacchettoni) | Solo sacchetti bilancia |
| Stati gestiti | Aperto, chiuso, annullato | Pendente, elaborato: l'annullamento non è applicabile ai sacchetti |
| Filtro per cassa | Presente | Non presente: non significativo per i sacchetti bilancia |
| Ricerca per codice | Assente: occorreva conoscere la data | Presente, per codice esatto o per prefisso |
| Ordinamento | Non definito | Per data e ora crescenti, modificabile dalle intestazioni |
| Stampa/esportazione dell'elenco | Anteprima di stampa dell'elenco completo | Excel, PDF, stampa e appunti della pagina visualizzata |
| Esportazione delle righe articolo | Assente | File CSV per sacchetto, importabile in cassa |

## Domande frequenti

**Perché non trovo un sacchetto che so essere stato emesso?**

: Verificare la **data** selezionata: la ricerca riguarda un solo giorno. Se la data è corretta, controllare che il filtro **Stato** non lo stia escludendo e provare a cercarlo per codice a barre.

**Posso creare o eliminare un sacchetto da questa pagina?**

: No. I sacchetti sono generati dalle bilance; dalla WebApp è possibile soltanto consultarli, cambiarne lo stato ed esportarne le righe.

**Ho contrassegnato un sacchetto come elaborato per errore.**

: È sufficiente selezionare di nuovo la riga e scegliere **Contrassegna come pendente**: non esistono vincoli di transizione tra i due stati.

**La cassa non ha elaborato il sacchetto: come recupero gli articoli?**

: Usare **Esporta dettaglio (CSV)** dal menu azioni o dalla finestra di dettaglio e importare il file nel software di cassa.

**L'esportazione dell'elenco dei sacchetti contiene meno righe di quelle che mi aspetto.**

: L'esportazione riguarda la pagina visualizzata. Portare il numero di righe per pagina a 50 e ripetere l'operazione.
