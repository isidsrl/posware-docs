---
tags:
    - Posware
    - StoreServer
---

# Posware Module - Scheda cassa e scontrino in tempo reale

**Prima revisione documento: 30 settembre 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Glossario

- **WebApp:** applicazione web che permette di gestire in modo centralizzato le funzionalità disponibili nello *StoreServer*;
- **Cassa** (o *terminale*): postazione Posware del punto vendita, raggiungibile in rete dallo *StoreServer*;
- **Scheda cassa:** pagina della WebApp dedicata a una singola cassa;
- **Scontrino in corso:** scontrino aperto in cassa e non ancora chiuso, con gli articoli battuti fino a quel momento.

## Cenni preliminari

La **Scheda cassa** raccoglie in una sola pagina tutto quello che riguarda una cassa:

- stato (online, operatore collegato, offline, trasmissioni disattivate) e incasso corrente del giorno;
- operatore collegato, transazione in corso e ora dell'ultimo aggiornamento;
- avvisi, informazioni di sistema (stampante fiscale, versioni, periferiche) e stato RT;
- operazioni eseguibili sulla cassa

Dalla scheda si apre la **vista in tempo reale dello scontrino in corso**: una seconda card, a destra della scheda (sotto, su schermi piccoli), mostra gli articoli che il cassiere sta battendo e si aggiorna ogni 2 secondi.

!!! info "Sostituzione di un programma di back-office"
    La vista in tempo reale sostituisce la modalità *Online* dell'utility desktop `Pos_View.exe`. **Non è richiesto alcun aggiornamento del software di cassa.** La consultazione dello storico scontrini della giornata sarà disponibile in una pagina dedicata.

### Versioni software

La funzione è inclusa nel **PoswareModule** dello *StoreServer*. È necessario che il modulo sia licenziato e attivo.

## Installazione

Nessuna installazione aggiuntiva. Per la vista in tempo reale lo *StoreServer* legge il database della cassa: deve poter raggiungere le casse sulla **porta TCP 3306** (MySQL). Verificare firewall e VLAN di barriera.

### Configurazione

I parametri si trovano in `appsettings.json`, sezione `Posware:PosLiveReceipt`. I valori predefiniti sono quelli standard delle casse Posware e normalmente non vanno modificati.

| Parametro | Default | Descrizione |
|---|---|---|
| `ConnectTimeout` | `00:00:02` | attesa massima della connessione |
| `QueryTimeout` | `00:00:03` | attesa massima della lettura |
| `MaxRows` | `500` | righe massime lette per scontrino |

Le credenziali di accesso al database della cassa sono quelle standard di Posware e non sono configurabili.

!!! warning
    Un valore non valido (per esempio un timeout a zero) impedisce l'avvio dello *StoreServer*: il messaggio di errore indica la sezione `Posware:PosLiveReceipt`.

### Permessi

| Permesso | Effetto |
|---|---|
| `store.devices.pos.view` | apre la lista casse e la scheda cassa |
| `store.devices.pos.livereceipt.view` | mostra il pulsante *Apri vista live* e permette di leggere lo scontrino in corso |
| `store.devices.pos.remotekeychange.view` | mostra il pulsante *Cambio posizione chiave* |
| `store.devices.pos.remoteshutdown.execute` | mostra il pulsante *Spegni cassa* |

Il permesso `store.devices.pos.livereceipt.view` è incluso nel ruolo **ShiftManager** (responsabile di turno) e nei ruoli con tutti i permessi.

## Utilizzo

### Aprire la scheda

Da **Dispositivi › Casse** cliccare sul **numero** della cassa (primo cerchio colorato) oppure sull'etichetta **Cassa N**.

### Operazioni

| Pulsante | Cosa fa |
|---|---|
| **Aggiorna status** | rilegge dalla cassa stato e informazioni di sistema |
| **Cambio posizione chiave** | vedi [Cambio livello chiave da remoto](cambio-chiave-remoto.md) |
| **Spegni cassa** | spegne la cassa, solo se è nella schermata di login |
| **Riassocia** | ritrasmette alla cassa i parametri di collegamento allo *StoreServer* |
| **Disattiva / Riattiva invio variazioni** | sospende o riprende l'invio di articoli e prezzi alla cassa |
| **Elimina cassa** | rimuove la cassa dall'anagrafica e torna alla lista casse |

Quando un'operazione non è disponibile il pulsante è disattivato e sotto il titolo ne è indicato il motivo (per esempio *Operatore collegato: la cassa non è nella schermata di login*).

### Scontrino in tempo reale

1. Premere **Apri vista live**: la card *Scontrino in corso* entra da destra;
2. le righe compaiono nell'ordine di battitura con quantità e importo; le righe appena aggiunte sono evidenziate e il **totale** si aggiorna;
3. il badge indica lo stato della vista:

| Badge | Significato |
|---|---|
| **LIVE** | la vista si aggiorna ogni 2 secondi |
| **IN PAUSA** | aggiornamento sospeso con il pulsante pausa; premere di nuovo per riprendere |
| **OFFLINE** | la cassa non risponde; la vista riprende da sola quando torna raggiungibile |

Le righe particolari sono indicate prima della descrizione:

| Indicazione | Significato | Effetto sul totale |
|---|---|---|
| **STORNO** | articolo stornato | sottratto |
| **RESO** | articolo reso | sottratto |
| **STORNO REPARTO** / **RESO REPARTO** | storno o reso di una vendita a reparto | sottratto |
| **CARTA FEDELTÀ** | carta fedeltà letta in cassa (numero carta) | nessuno |

Messaggi possibili:

- *Nessuno scontrino in corso*: la cassa non ha uno scontrino aperto;
- *Cassa non raggiungibile*: la cassa è spenta, scollegata dalla rete o la porta 3306 non è raggiungibile.

Il pulsante **Apri vista live** è disattivato se la cassa è *Offline* o ha le *Trasmissioni disattivate*. Chiudendo la card la WebApp smette di interrogare la cassa.

## Differenze rispetto a Pos_View

| Pos_View | StoreServer |
|---|---|
| programma installato sul PC server di barriera | pagina della WebApp, da qualsiasi postazione |
| nessun controllo di accesso | permesso dedicato `store.devices.pos.livereceipt.view` |
| righe senza ordinamento garantito | righe nell'ordine di battitura |
| solo descrizione e valore | quantità, importo, resi/storni evidenziati, totale e numero scontrino |
| errore di connessione segnalato solo dal colore dell'etichetta | messaggio *Cassa non raggiungibile* e ripresa automatica |
