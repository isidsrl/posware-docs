---
tags:
    - Frontend
    - Log
---

# Posware Frontend - Log Transazione Posware: Specifiche Tecniche e formato dati

**Prima revisione documento: 13 ottobre 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---
## Introduzione
Specifiche tecniche del tracciato log delle transazioni POS Posware con campi a larghezza fissa, strutturati in record tipologici concatenati che descrivono l'intero ciclo di vita di uno scontrino, inclusi articoli, sconti, pagamenti, promozioni e chiusura.

Ogni record indica posizione iniziale, lunghezza, tipo e descrizione del campo, con convenzioni uniformi per numerici, stringhe e numerici con virgola virtuale per importi e quantità.

## Accesso ai Dati da Database Posware
I dati del tracciato log sono memorizzati nel database **Posware** all'interno della tabella `log`. Questa tabella MySQL utilizza una struttura normalizzata che mantiene ogni record di log come singola riga, dove il campo `Buffer` contiene l'intero record formattato secondo le specifiche descritte in questo documento.

### Struttura della Tabella Log
La tabella è definita con i seguenti campi principali:

- **Id**: Chiave primaria auto-incrementale per identificazione univoca
- **Pdv**: Codice punto vendita (6 caratteri)
- **Data**: Data dell'operazione (8 caratteri, formato YYYYMMDD)
- **Cassa**: Numero terminale cassa (3 caratteri)
- **Progressivo**: Numero progressivo del record (6 caratteri)
- **Tipo**: Tipo di record (2 caratteri, es: "00", "01", "02", ecc.)
- **Buffer**: Contenuto completo del record formattato (255 caratteri max)
- **TransactionId**: Numero della transazione Posware

### Ordinamento Corretto per la Lettura
**IMPORTANTE**: Per leggere correttamente il log e mantenere la sequenza logica di dati ed operazioni, è necessario interrogare la tabella rispettando un ordinamento preciso. Sono disponibili due sequenze di ordinamento equivalenti:

#### Opzione 1: Ordinamento per ID
```sql
SELECT * FROM log [Where CONDITION]
ORDER BY Id ASC;
```

#### Opzione 2: Ordinamento per Chiavi Logiche
```sql
SELECT * FROM log [Where CONDITION]
ORDER BY Pdv ASC, Data ASC, Cassa ASC, Progressivo ASC;
```

Entrambi gli ordinamenti garantiscono che i record vengano letti nella sequenza corretta in cui sono stati generati dal sistema POS, preservando l'integrità del flusso transazionale e permettendo una corretta interpretazione dei dati secondo le specifiche del presente documento.

La mancata applicazione del corretto ordinamento può portare a interpretazioni errate del flusso delle transazioni e a errori nell'elaborazione dei dati di log.

## Indice Record
### Raggruppamento per Categorie

- **Transazione**: 00 Inizio, 01 Dettaglio, 02 Sconto, 03 Pagamento, 04 Fine
- **Metadati e controllo**: 05 Info transazione precedente, 06 Informazioni operative, 09 Montanti, 10 Info testata, 13 Info pagamenti, 14 Info aggiuntive
- **Promozioni e coupon**: 15 Testata promo, 25 Dettaglio promo, 26 Coupon emesso, 27 Coupon redento, 35 Dettaglio premio
- **Servizi e rettifiche**: 17 Ricariche EPAY, 18 Pagamento utenze EPAY, 20 Rettifica

## Legenda Tipi di Dato

- **numeric** = Tipo numerico intero con riempimento a sinistra con zeri (pad-left 0-filled)
- **string** = Tipo stringa con riempimento a destra con spazi vuoti (pad-right blank-filled)
- **numeric(x,y)** = Numerico con virgola virtuale, _X_ cifre totali di cui _Y_ decimali; es. 123456 → 1234,56 per numeric(6,2)
- **boolean** = Campo binario 0/1 o flag (0 = falso, 1 = vero)
- **numeric(date)** = Data nel formato yyyymmdd
- **numeric(time)** = Orario nel formato hhmmss

## Note sulla Formattazione dei Dati

- I tipi **numeric**, inclusi quelli con virgola virtuale sono riempiti con zeri ed allineati a sinistra (pad-left 0-filled)
- I tipi **string** sono riempiti con spazi vuoti a destra (pad-right blank-filled)
- Il formato **numeric(x,y)** indica _X_ cifre totali di cui _Y_ decimali, con virgola virtuale:
  - **numeric(6,2)** = 6 cifre totali di cui 2 decimali (es: 123456 = 1234,56)
  - **numeric(7,2)** = 7 cifre totali di cui 2 decimali (es: 1234567 = 12345,67)
  - **numeric(3,3)** = 3 cifre totali di cui 3 decimali (es: 123 = 0,123)

## Linee Guida di Parsing

- Estrarre i campi per posizione e lunghezza; per numeric(x,y) convertire dividendo per 10^y in fase di interpretazione applicativa
- Mappare i campi boolean 0/1 su false/true o No/Sì, rigettando valori non ammessi
- Per numeric(date) validare il formato yyyymmdd e i range calendario; per numeric(time) validare hhmmss con zeri significativi

## Sequenza Tipica dei Record
Il flusso tipico di una transazione inizia con 00, segue 01 per le righe, eventuali 02/15/25/26/27 per promozioni, sconti e coupon, uno o più 03 per i pagamenti, e si chiude con 04.
Eventuali 10/14/13/16/17/18 possono essere presenti in transazioni che coinvolgono la tessera loyalty,  provider di pagamento esterni o funzionalità opzionali (lotteria scontrini, metadati fornite dall'utente come info aggiuntive), mentre 05 e 20 compaiono rispettivamente per annulli/resi e rettifiche fuori flusso.

**Delimitazione delle Transazioni**: Uno scontrino è sempre delimitato da un record di **inizio "00"** e un record di **chiusura "04"**. La presenza di un record "00" senza il corrispondente record "04" indicherebbe uno scontrino ancora in corso, tuttavia questo scenario non si verifica nella pratica operativa poiché **la cassa invia al server di barriera solo scontrini completi **.

**Sequenze di Record**: All'interno della sequenza 00-04, tutti i tipi di record intermedi (01, 02, 03, 05, 09, 10, 13, 14, 15, 16, 17, 18, 20, 25, 26, 27, 35) appartengono necessariamente allo scontrino corrente. Leggendo il log in sequenza progressiva, una volta incontrato un record tipo "00", tutti i record successivi fino al record "04" corrispondente fanno parte della stessa transazione.

**Eccezione: Record Tipo "06"**: L'unico tipo di record che può esistere anche al di fuori delle sequenze 00-04 è il **record tipo "06" (Informazioni)**. Questi record contengono metadati operativi e informazioni di sistema, identificabili tramite il campo "Tipo Operazione" presente nel buffer del record. I record "06" possono comparire:

- Immediatamente dopo l'apertura del programma e prima dell'inizio di uno scontrino
- Tra diverse transazioni (eventi operativi come apertura/chiusura operatore, pause, operazioni di cassa) 

**Flusso di Lettura**: Dopo la chiusura di uno scontrino con record "04", nel flusso si possono incontrare solamente:

- Un nuovo record "00" (inizio immediato di un nuovo scontrino)
- Un record "06" (eventi operativi come cambio operatore, pause, aperture cassetto, ecc.) 

Questa struttura garantisce la corretta interpretazione del flusso transazionale e permette di raggruppare logicamente i record appartenenti a ciascuno scontrino durante l'elaborazione dei dati di log.

---

## RECORD 00 - Inizio Transazione

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "00") |  | <= 4.2.x |
| 15 | 2 | numeric | **Tipo Transazione:** |  | <= 4.2.x |
|   |   |   | 00 = Transazione di vendita fiscale |  | <= 4.2.x |
|   |   |   | 03 = Fondo cassa |  | <= 4.2.x |
|   |   |   | 04 = Prelievo |  | <= 4.2.x |
|   |   |   | 05/06/07 = Commissione vendita |  | <= 4.2.x |
|   |   |   | 08 = Annullo transazione precedente |  | <= 4.2.x |
|   |   |   | 11 = Nota credito |  | <= 4.2.x |
|   |   |   | 12 = Movimento non fiscale |  | <= 4.2.x |
|   |   |   | 20 = Rettifica |  | <= 4.2.x |
|   |   |   | 21 = Registro Mancato funzionamento |  | <= 4.2.x |
|   |   |   | 22 = Annullo Ricarica |  | <= 4.2.x |
|   |   |   | 23 = Pagamento Utenze (Prot. Euronet) |  | <= 4.2.x |
|   |   |   | 24 = Attivazione Gift Card |  | <= 4.2.x |
| 17 | 5 | numeric | Numero Transazione (ID univoco Posware, progressivo) |  | <= 4.2.x |
| 22 | 5 | numeric | Numero Scontrino Fiscale; se TipoTransazione=12 contiene il Numero documento |  | <= 4.2.x |
| 27 | 4 | numeric | Codice numerico operatore |  | <= 4.2.x |
| 31 | 8 | numeric(date) | Data Operazione (yyyymmdd) |  | <= 4.2.x |
| 39 | 6 | numeric(time) | Ora operazione (hhmmss) |  | <= 4.2.x |
| 45 | 1 | boolean | Flag1: Transazione completata |  | <= 4.2.x |
| 46 | 1 | boolean | Flag2: Transazione annullata (annullata/storno) |  | <= 4.2.x |
| 47 | 1 | boolean | Flag3: Con sconto al totale |  | <= 4.2.x |
| 48 | 1 | boolean | Flag4: Transazione in Training |  | <= 4.2.x |
| 49 | 1 | boolean | Flag5: Sconto su Articoli |  | <= 4.2.x |
| 50 | 1 | boolean | Flag6: Posizione chiave attiva |  | <= 4.2.x |
| 51 | 1 | boolean | Flag7: Transazione sospesa |  | <= 4.2.x |
| 52 | 1 | boolean | Flag8: Ristampa |  | <= 4.2.x |
| 53 | 1 | boolean | Flag9: IVA esente |  | <= 4.2.x |
| 54 | 1 | boolean | Flag10: Scontrino fatturato |  | <= 4.2.x |
| 55 | 1 | boolean | Flag11: Resi merci presenti |  | <= 4.2.x |
| 56 | 13 | string | Codice Cliente Fidelity (13 caratteri) |  | <= 4.2.x |
| 69 | 10 | string | Matricola fiscale cassa (10 caratteri) |  | <= 4.2.x |
| 79 | 8 | numeric(6,2) | Totale transazione (totale scontrino) |  | <= 4.2.x |
| 87 | 8 | numeric(6,2) | Sconti articoli incondizionati (promo incondizionate) |  | <= 4.2.x |
| 95 | 8 | numeric(6,2) | Sconti condizionati (promo a fine scontrino) |  | <= 4.2.x |
| 103 | 8 | numeric(6,2) | Sconti manuali da tastiera |  | <= 4.2.x |
| 111 | 8 | numeric(6,2) | Punti assegnati |  | <= 4.2.x |
| 119 | 8 | numeric(6,2) | Punti utilizzati |  | <= 4.2.x |
| 127 | 1 | boolean | Transazione con carta fidelity |  | <= 4.2.x |
| 128 | 1 | boolean | Transazione con scontrini sospesi |  | <= 4.2.x |
| 129 | 13 | string | Codice Commissione di Vendita |  | <= 4.2.x |
| 142 | 1 | boolean | Nuova Commissione (1=Nuovo, 0=Modifica) |  | <= 4.2.x |
| 143 | 2 | numeric | Causale Movimento Finanziario |  | <= 4.2.x |
| 145 | 1 | boolean | Scontrino Automatico |  | <= 4.2.x |
| 146 | 13 | string | Barcode dello scontrino |  | <= 4.2.x |
| 159 | 9 | numeric(7,2) | Corrispettivo Fiscale a inizio transazione |  | <= 4.2.x |
| 168 | 2 | string | Causale Movimento non fiscale |  | <= 4.2.x |
| 170 | 4 | numeric | Numero ZReport attuale |  | <= 4.2.x |
| 174 | 11 | string | Matricola RT |  | <= 4.2.x |

---

## RECORD 01 - Dettaglio Articolo/Reparto

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "01") |  | <= 4.2.x |
| 15 | 20 | numeric | Codice Articolo/Reparto (Barcode/PLU/Reparto) |  | <= 4.2.x |
| 35 | 1 | string | Tipo Articolo o Reparto (A/R/P premio) |  | <= 4.2.x |
| 36 | 2 | numeric | Codice IVA |  | <= 4.2.x |
| 38 | 3 | numeric | Codice Reparto di appartenenza |  | <= 4.2.x |
| 41 | 6 | numeric(4,2) | Quantità prodotto |  | <= 4.2.x |
| 47 | 8 | numeric(6,2) | Importo unitario |  | <= 4.2.x |
| 55 | 8 | numeric(6,2) | Importo Totale riga (lordo sconti) |  | <= 4.2.x |
| 63 | 1 | boolean | Prodotto in promozione |  | <= 4.2.x |
| 64 | 10 | string | Codice Riferimento |  | <= 4.2.x |
| 74 | 10 | string | Codice Paniere |  | <= 4.2.x |
| 84 | 2 | string | Tipo promozione (se riferibile a prodotto) |  | <= 4.2.x |
| 86 | 6 | numeric(4,2) | Valore sconto su prodotto |  | <= 4.2.x |
| 92 | 6 | numeric(4,2) | Totale punti su articolo/reparto |  | <= 4.2.x |
| 98 | 1 | boolean | Punti segno (0=+;1=−) |  | <= 4.2.x |
| 99 | 1 | boolean | Flag1: Stornato |  | <= 4.2.x |
| 100 | 1 | boolean | Flag2: Reso |  | <= 4.2.x |
| 101 | 1 | boolean | Flag3: Vuoto |  | <= 4.2.x |
| 102 | 1 | boolean | Flag4: Battuta manuale a reparto |  | <= 4.2.x |
| 103 | 1 | boolean | Flag5: Battuta manuale da tastiera |  | <= 4.2.x |
| 104 | 1 | boolean | Flag6: Articolo non trovato (forzato) |  | <= 4.2.x |
| 105 | 1 | boolean | Flag7: Da bilancia |  | <= 4.2.x |
| 106 | 1 | boolean | Flag8: Prezzo richiesto |  | <= 4.2.x |
| 107 | 1 | boolean | Flag9: Articolo coupon con barcode |  | <= 4.2.x |
| 108 | 1 | boolean | Flag10: Sacchettone da bilancia |  | <= 4.2.x |
| 109 | 1 | boolean | Flag11: Articolo del sacchettone |  | <= 4.2.x |
| 110 | 1 | boolean | Flag12: Coupon di Catalina |  | <= 4.2.x |
| 111 | 1 | boolean | Prodotto premio |  | <= 4.2.x |
| 112 | 6 | string | Indicazione del Riferimento |  | <= 4.2.x |
| 118 | 1 | boolean | Ricarica online (1) / offline (0) |  | <= 4.2.x |
| 119 | 6 | numeric(3,3) | Quantità decimale |  | <= 4.2.x |
| 125 | 1 | boolean | Coupon validato online |  | <= 4.2.x |
| 126 | 1 | boolean | Date del coupon valide |  | <= 4.2.x |
| 127 | 1 | boolean | Articolo appartenente a Kit |  | <= 4.2.x |
| 128 | 20 | string | Barcode Kit/Sacchettone (se flag=1) |  | <= 4.2.x |
| 148 | 20 | string | Gruppo Promo (se presente NP con gruppo promo) |  | <= 4.2.x |
| 168 | 40 | string | Descrizione articolo in cassa |  | <= 4.2.x |
| 208 | 6 | string | Codice Listino utilizzato (se gestione listini manuale) |  | <= 4.2.x |

---

## RECORD 02 - Sconto (fine scontrino, non riferibile a prodotto)

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "02") |  | <= 4.2.x |
| 15 | 10 | string | Codice riferimento origine sconto |  | <= 4.2.x |
| 25 | 10 | string | Paniere del riferimento (se presente) |  | <= 4.2.x |
| 35 | 2 | string | Messaggio del riferimento |  | <= 4.2.x |
| 37 | 13 | numeric | Codice articolo coupon rilasciato |  | <= 4.2.x |
| 50 | 13 | numeric | Codice coupon rientrato (utilizzato) |  | <= 4.2.x |
| 63 | 2 | numeric | Tipologia sconto |  | <= 4.2.x |
| 65 | 6 | numeric(4,2) | Valore dello sconto |  | <= 4.2.x |
| 71 | 6 | numeric(4,2) | Punti accumulati |  | <= 4.2.x |
| 77 | 10 | string | Codice Iniziativa |  | <= 4.2.x |
| 87 | 1 | boolean | Promo Automatica/Manuale |  | <= 4.2.x |
| 88 | 13 | string | Codice autorizzante |  | <= 4.2.x |

---

## RECORD 03 - Pagamento

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "03") |  | <= 4.2.x |
| 15 | 2 | numeric | Tipo di Pagamento |  | <= 4.2.x |
| 17 | 9 | numeric(7,2) | Importo pagamento |  | <= 4.2.x |
| 26 | 9 | numeric(7,2) | Importo in valuta estera |  | <= 4.2.x |
| 35 | 5 | numeric | Quantità (solo fondo cassa e prelievi) |  | <= 4.2.x |
| 40 | 1 | boolean | Annullo pagamento (solo fondo cassa/prelievi) |  | <= 4.2.x |
| 41 | 1 | boolean | Flag Valuta |  | <= 4.2.x |
| 42 | 1 | boolean | Pagamento coupon |  | <= 4.2.x |
| 43 | 1 | boolean | Sub pagamento |  | <= 4.2.x |
| 44 | 1 | boolean | Corrispettivo non pagato |  | <= 4.2.x |
| 45 | 1 | boolean | Sconto fidelity |  | <= 4.2.x |
| 46 | 1 | boolean | EFT manuale |  | <= 4.2.x |
| 47 | 1 | boolean | Chiave MGR attiva |  | <= 4.2.x |
| 48 | 8 | string | Indicatore personalizzazioni |  | <= 4.2.x |
| 56 | 9 | numeric(7,2) | Resto |  | <= 4.2.x |
| 65 | 8 | numeric(date) | Data Operazione Pagamento (yyyymmdd) |  | <= 4.2.x |
| 73 | 6 | numeric(time) | Ora Operazione Pagamento (hhmmss) |  | <= 4.2.x |
| 79 | 15 | string | Numero assegno/codice coupon |  | <= 4.2.x |
| 94 | 8 | string | Codice CAB banca |  | <= 4.2.x |
| 102 | 8 | string | Codice ABI banca |  | <= 4.2.x |
| 110 | 3 | string | Codice internazionale valuta |  | <= 4.2.x |
| 113 | 18 | numeric(12,6) | Valore del cambio valuta |  | <= 4.2.x |
| 131 | 50 | string | Codice GIFT CARD o Id Terminale (Argentea) |  | <= 4.2.x |

---

## RECORD 04 - Fine Transazione

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "04") |  | <= 4.2.x |
| 15 | 1 | boolean | Transazione annullata |  | <= 4.2.x |
| 16 | 1 | boolean | Sconto sul totale |  | <= 4.2.x |
| 17 | 1 | boolean | Transazione sospesa |  | <= 4.2.x |
| 18 | 1 | boolean | Bollini rifiutati |  | <= 4.2.x |
| 19 | 1 | boolean | Tasto esclusione sconti |  | <= 4.2.x |
| 20 | 1 | boolean | Resto con pagamento 20 |  | <= 4.2.x |
| 21 | 1 | boolean | Clienti in circolarità |  | <= 4.2.x |
| 22 | 1 | boolean | Chiave MGR attiva |  | <= 4.2.x |
| 23 | 9 | numeric(7,2) | Totale transazione |  | <= 4.2.x |
| 32 | 5 | numeric | Numero articoli venduti |  | <= 4.2.x |
| 37 | 6 | numeric(time) | Ora fine transazione (hhmmss) |  | <= 4.2.x |
| 43 | 8 | numeric(date) | Data fine transazione (yyyymmdd) |  | <= 4.2.x |
| 51 | 8 | string | Indicatore personalizzazioni |  | <= 4.2.x |
| 59 | 5 | numeric | Numero bollini sul totale |  | <= 4.2.x |
| 64 | 5 | numeric | Numero bollini sui reparti |  | <= 4.2.x |
| 69 | 5 | numeric | Numero bollini in somma |  | <= 4.2.x |
| 74 | 5 | numeric | Numero bollini su singolo articolo |  | <= 4.2.x |
| 79 | 5 | numeric | Numero bollini su offerta |  | <= 4.2.x |
| 84 | 4 | string | 4 byte vuoti non usati |  | <= 4.2.x |
| 88 | 4 | string | (vuoto) |  | <= 4.2.x |
| 92 | 4 | string | (vuoto) |  | <= 4.2.x |
| 96 | 4 | string | (vuoto) |  | <= 4.2.x |
| 100 | 9 | numeric(7,2) | Importo sconti accumulati (articolo/totale) |  | <= 4.2.x |
| 109 | 9 | numeric(7,2) | Importo del resto |  | <= 4.2.x |
| 118 | 5 | numeric | Numero punti maturati |  | <= 4.2.x |
| 123 | 5 | numeric | Cumulo punti |  | <= 4.2.x |
| 128 | 9 | numeric(7,2) | Corrispettivo Fiscale a fine transazione |  | <= 4.2.x |

---

## RECORD 05 - Informazioni Transazione Precedente (Annullata/Reso)

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "05") |  | <= 4.2.x |
| 15 | 5 | numeric | Numero Transazione di riferimento |  | <= 4.2.x |
| 20 | 8 | numeric(date) | Data Transazione (yyyymmdd) |  | <= 4.2.x |
| 28 | 4 | numeric | Z Report Number della transazione di riferimento |  | <= 4.2.x |
| 32 | 4 | numeric | Numero Scontrino della transazione di riferimento |  | <= 4.2.x |
| 36 | 11 | string | Matricola fiscale stampante del documento di riferimento |  | <= 4.2.x |

---

## RECORD 06 - Informazioni

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "06") |  | <= 4.2.x |
| 15 | 2 | numeric | **Tipo Operazione (0..15):** |  | <= 4.2.x |
|   |   |   | 00 = Apertura Operatore |  | <= 4.2.x |
|   |   |   | 01 = Chiusura Operatore |  | <= 4.2.x |
|   |   |   | 02 = Apertura Cassetto |  | <= 4.2.x |
|   |   |   | 03 = TOTALE |  | <= 4.2.x |
|   |   |   | 04 = Pausa Operatore |  | <= 4.2.x |
|   |   |   | 05 = Fine Pausa Operatore |  | <= 4.2.x |
|   |   |   | 06 = Chiusura Fiscale Manuale |  | <= 4.2.x |
|   |   |   | 07 = Stampa Totali |  | <= 4.2.x |
|   |   |   | 08 = Chiusura Fiscale da PC |  | <= 4.2.x |
|   |   |   | 09 = Codice Generico |  | <= 4.2.x |
|   |   |   | 10 = Versione Posware |  | <= 4.2.x |
|   |   |   | 11 = Chip Card KO |  | <= 4.2.x |
|   |   |   | 12 = Messaggio |  | <= 4.2.x |
|   |   |   | 13 = Cambio Chiave Amministratore |  | <= 4.2.x |
|   |   |   | 14 = Start Programma |  | <= 4.2.x |
|   |   |   | 15 = Passaggio Tessera Fidelity |  | <= 4.2.x |
| 17 | 4 | numeric | Codice Operatore |  | <= 4.2.x |
| 21 | 8 | numeric(date) | Data Operazione (yyyymmdd) |  | <= 4.2.x |
| 29 | 6 | numeric(time) | Ora Operazione (hhmmss) |  | <= 4.2.x |
| 35 | 13 | numeric | Codice inserito / risposta operatore (se tipo=12) |  | <= 4.2.x |
| 48 | 4 | numeric | Versione del programma (codice) |  | <= 4.2.x |
| 52 | 1 | numeric | Flag 1-7 non usato |  | <= 4.2.x |
| 59 | 1 | boolean | Flag8: Chip Card diversa da inizio transazione |  | <= 4.2.x |
| 60 | 9 | numeric(7,2) | Importo della Chiusura Fiscale |  | <= 4.2.x |
| 69 | 6 | numeric | Codice Messaggio (solo se tipo=12) |  | <= 4.2.x |
| 75 | 3 | numeric | Numero stampe/visualizzazioni (solo se tipo=12) |  | <= 4.2.x |
| 78 | 1 | string | Sistema Operativo (W/D/L) |  | <= 4.2.x |
| 79 | 11 | numeric(9,2) | Gran Totale Fiscale |  | <= 4.2.x |
| 90 | 11 | numeric(9,2) | Totale Note Credito Giornaliero |  | <= 4.2.x |
| 101 | 11 | numeric(9,2) | Gran Totale Note Credito |  | <= 4.2.x |
| 112 | 5 | numeric | Numero della chiusura fiscale |  | <= 4.2.x |
| 117 | 10 | string | Matricola fiscale cassa |  | <= 4.2.x |
| 127 | 15 | string | Versione programma X.X.X.X |  | <= 4.2.x |
| 142 | 13 | string | Codice Tessera Fidelity (se tipo=15) |  | <= 4.2.x |
| 155 | 11 | string | Matricola RT |  | <= 4.2.x |
| 166 | 2 | numeric | Numero file pendenti da inviare ad AdE |  | <= 4.2.x |

---

## RECORD 09 - Montanti

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "09") |  | <= 4.2.x |
| 15 | 5 | numeric | Numero Transazione |  | <= 4.2.x |
| 20 | 5 | numeric | Numero Scontrino Fiscale |  | <= 4.2.x |
| 25 | 10 | string | Codice Montante |  | <= 4.2.x |
| 35 | 8 | numeric(6,2) | Accumulato |  | <= 4.2.x |
| 43 | 8 | numeric(6,2) | Goduto |  | <= 4.2.x |
| 51 | 8 | numeric(6,2) | VariazionePos |  | <= 4.2.x |
| 59 | 8 | numeric(6,2) | VariazioneNeg |  | <= 4.2.x |
| 67 | 100 | string | Codice carta o seriale gift card |  | <= 4.2.x |

---

## RECORD 10 - Informazioni Aggiuntive Testata

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "10") |  | <= 4.2.x |
| 15 | 3 | string | Tipo info aggiuntiva (FID/SOS/POS/ORD/KIT; con Tipo Transazione=12: CCF/RSC/IND/LOC/CAP/PRV/PIV/CFI/TEL) |  | <= 4.2.x |
| 18 | 100 | string | Informazione; per POS/ORD contiene numero file; per KIT contiene barcode KIT e quantità (numeric(6,2)) |  | <= 4.2.x |

---

## RECORD 13 - Informazioni Aggiuntive Pagamenti

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "13") |  | <= 4.2.x |
| 15 | 2 | string | Tipo di pagamento |  | <= 4.2.x |
| 17 | 100 | string | Informazione |  | <= 4.2.x |

---

## RECORD 14 - Informazioni Aggiuntive

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "14") |  | <= 4.2.x |
| 15 | 1 | string | Tipo Informazione (A=Articolo, S=Scontrino) |  | <= 4.2.x |
| 16 | 5 | string | Codice Informazione |  | <= 4.2.x |
| 21 | 20 | string | Codice a barre (se Tipo=A) |  | <= 4.2.x |
| 41 | 100 | string | Informazione |  | <= 4.2.x |

---

## RECORD 15 - Testata Promozione

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "15") |  | <= 4.2.x |
| 15 | 5 | numeric | Numero Transazione |  | <= 4.2.x |
| 20 | 5 | numeric | Numero Scontrino Fiscale |  | <= 4.2.x |
| 25 | 10 | string | Codice Promo |  | <= 4.2.x |
| 35 | 6 | string | Riferimento |  | <= 4.2.x |
| 41 | 6 | string | Paniere |  | <= 4.2.x |
| 47 | 2 | string | Tipo Effetto (S,B,P,C,M) |  | <= 4.2.x |
| 49 | 8 | numeric(6,2) | Sconto |  | <= 4.2.x |
| 57 | 8 | numeric(6,2) | Variazione Montante |  | <= 4.2.x |
| 65 | 10 | string | Montante |  | <= 4.2.x |
| 75 | 6 | string | Codice Messaggio |  | <= 4.2.x |
| 81 | 13 | string | Codice Coupon |  | <= 4.2.x |
| 94 | 3 | numeric | Numero applicazioni |  | <= 4.2.x |
| 97 | 3 | numeric | Pezzi in promozione |  | <= 4.2.x |
| 100 | 1 | boolean | Promo Automatica/Manuale |  | <= 4.2.x |
| 101 | 13 | string | Codice Autorizzazione |  | <= 4.2.x |
| 114 | 1 | string | Tipo Promozione (A/R/S/P) |  | <= 4.2.x |
| 115 | 1 | boolean | Richiede Coupon |  | <= 4.2.x |
| 116 | 20 | string | Raggruppamento Promo |  | <= 4.2.x |
| 136 | 5 | numeric(3,2) | Sconto Percentuale digitato |  | <= 4.2.x |
| 141 | 20 | string | Codice promozione (20 bytes) |  | <= 4.2.x |

---

## RECORD 16 - Informazioni Buono Chiaro (segue Rec. 03)

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "16") |  | <= 4.2.x |
| 15 | 30 | string | Codeline del buono pasto |  | <= 4.2.x |
| 45 | 3 | numeric | Codice emettitore buono pasto |  | <= 4.2.x |
| 48 | 8 | numeric(6,2) | Valore buono pasto |  | <= 4.2.x |
| 56 | 5 | numeric(3,2) | Percentuale sconto emettitore |  | <= 4.2.x |
| 61 | 5 | numeric(3,2) | Aliquota IVA su buono |  | <= 4.2.x |
| 66 | 5 | numeric(3,2) | Percentuale di scorporo |  | <= 4.2.x |
| 71 | 8 | numeric(6,2) | Valore rimborsato al retailer |  | <= 4.2.x |
| 79 | 8 | numeric(date) | Data scadenza buono (yyyymmdd) |  | <= 4.2.x |
| 87 | 8 | numeric(date) | Data scadenza rimborso (yyyymmdd) |  | <= 4.2.x |
| 95 | 1 | boolean | Buono da rispedire in cartaceo |  | <= 4.2.x |
| 96 | 10 | numeric | Codice Corporate Wincor Nixdorf |  | <= 4.2.x |
| 106 | 10 | numeric | Codice dispositivo |  | <= 4.2.x |
| 116 | 10 | numeric | Id transazione su Server |  | <= 4.2.x |
| 126 | 40 | string | Codeline del buono pasto |  | <= 4.2.x |

---

## RECORD 17 - Informazioni Ricariche EPAY EURONET

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "17") |  | <= 4.2.x |
| 15 | 30 | string | Codice EAN dell'articolo |  | <= 4.2.x |
| 45 | 30 | string | Codice PAN eventuale della Carta |  | <= 4.2.x |
| 75 | 30 | string | Codice Seriale del PIN Ricevuto |  | <= 4.2.x |
| 105 | 30 | string | Id Transazione HOST (Codice Transazione Epay) |  | <= 4.2.x |
| 135 | 30 | string | Id Transazione Client (Codice Transazione Posware) |  | <= 4.2.x |
| 165 | 10 | numeric(8,2) | Importo |  | <= 4.2.x |
| 175 | 1 | numeric | Tipo Operazione (0=Prenotazione, 1=Conferma, 2=Annullo) |  | <= 4.2.x |

---

## RECORD 18 - Informazioni Pagamento Utenze EPAY EURONET

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "18") |  | <= 4.2.x |
| 15 | 30 | string | Codice EAN dell'articolo (fittizio) |  | <= 4.2.x |
| 45 | 30 | string | Codeline letta a scanner |  | <= 4.2.x |
| 75 | 30 | string | Id Transazione HOST (Codice Transazione Epay) |  | <= 4.2.x |
| 105 | 30 | string | Id Transazione Client (Codice Transazione Posware) |  | <= 4.2.x |
| 135 | 10 | numeric(8,2) | Importo Bolletta |  | <= 4.2.x |
| 145 | 10 | numeric(8,2) | Importo Commissione |  | <= 4.2.x |
| 175 | 1 | numeric | Tipo Operazione (0=Prenotazione, 1=Conferma, 2=Annullo) |  | <= 4.2.x |

---

## RECORD 20 - Rettifica

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 2 | numeric | Tipo operazione (00=Annullo, 03=Rettifica pagamento, 04=Conciliazione sospesi) |  | <= 4.2.x |
| 3 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 7 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 10 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 15 | 5 | numeric | Numero Transazione |  | <= 4.2.x |
| 20 | 5 | numeric | Numero Scontrino Fiscale |  | <= 4.2.x |
| 25 | 4 | numeric | Codice numerico operatore |  | <= 4.2.x |
| 39 | 8 | numeric(date) | Data Operazione (yyyymmdd) |  | <= 4.2.x |
| 47 | 6 | numeric(time) | Ora operazione (hhmmss) |  | <= 4.2.x |

### Per tipo operazione = 00:

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 48 | 1 | boolean | Fisso a 1 |  | <= 4.2.x |

### Per tipo operazione = 03:

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 48 | 2 | numeric | Codice pagamento da rettificare/rettificante |  | <= 4.2.x |
| 50 | 9 | numeric(7,2) | Importo da rettificare/rettificato |  | <= 4.2.x |
| 59 | 1 | boolean | Record da rettificare/rettificante (0/1) |  | <= 4.2.x |

### Per tipo operazione = 04:

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 48 | 2 | numeric | Codice pagamento di conciliazione |  | <= 4.2.x |
| 50 | 9 | numeric(7,2) | Importo conciliato |  | <= 4.2.x |
| 59 | 100 | string | Note |  | <= 4.2.x |

---

## RECORD 25 - Dettaglio Promozione

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "25") |  | <= 4.2.x |
| 15 | 5 | numeric | Numero Transazione |  | <= 4.2.x |
| 20 | 5 | numeric | Numero Scontrino Fiscale |  | <= 4.2.x |
| 25 | 10 | string | Codice Promo |  | <= 4.2.x |
| 35 | 6 | string | Riferimento |  | <= 4.2.x |
| 41 | 6 | string | Paniere |  | <= 4.2.x |
| 47 | 13 | string | Codice Articolo |  | <= 4.2.x |
| 60 | 8 | numeric(6,2) | Sconto |  | <= 4.2.x |
| 68 | 6 | numeric(4,2) | Quantità su cui è applicato lo sconto |  | <= 4.2.x |
| 74 | 20 | string | Codice Articolo (20 bytes per non-EAN 13) |  | <= 4.2.x |
| 94 | 1 | string | Tipo Articolo o Reparto (A/R) |  | <= 4.2.x |

---

## RECORD 26 - Dettaglio Coupon Emesso

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "26") |  | <= 4.2.x |
| 15 | 13 | string | Barcode del Coupon Generato |  | <= 4.2.x |
| 28 | 1 | string | Tipo Coupon (S=Sconto, P=Pagamento, R=Reso) |  | <= 4.2.x |
| 29 | 8 | numeric(date) | Data Inizio Validità (yyyymmdd) |  | <= 4.2.x |
| 37 | 8 | numeric(date) | Data Fine Validità (yyyymmdd) |  | <= 4.2.x |
| 45 | 8 | numeric(6,2) | Valore del Coupon (può essere 0 per Attivatore) |  | <= 4.2.x |
| 53 | 20 | string | Codice della Promo di Redenzione |  | <= 4.2.x |

---

## RECORD 27 - Dettaglio Coupon Redento

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "27") |  | <= 4.2.x |
| 15 | 13 | string | Barcode del Coupon Redento |  | <= 4.2.x |
| 28 | 1 | string | Tipo Coupon (S=Sconto, P=Pagamento, R=Reso) |  | <= 4.2.x |
| 29 | 8 | numeric(6,2) | Valore Redento del Coupon |  | <= 4.2.x |
| 37 | 1 | boolean | Stato prima della transazione (0=Utilizzabile, 1=Redento) |  | <= 4.2.x |
| 38 | 1 | boolean | Usato effettivamente (0=No, 1=Sì) |  | <= 4.2.x |
| 39 | 8 | numeric(6,2) | Valore Iniziale del Coupon |  | <= 4.2.x |

---

## RECORD 35 - Dettaglio Premio

| Posizione | Lunghezza | Tipo | Descrizione | Note | Versione Posware |
|-----------|-----------|------|-------------|------|------------------|
| 1 | 4 | numeric | Codice del punto vendita (fisso per PdV) |  | <= 4.2.x |
| 5 | 3 | numeric | Numero terminale cassa (001-999) |  | <= 4.2.x |
| 8 | 5 | numeric | Progressivo record |  | <= 4.2.x |
| 13 | 2 | numeric | Tipo Record (fisso "35") |  | <= 4.2.x |
| 15 | 20 | string | Codice a barre |  | <= 4.2.x |
| 35 | 10 | string | Iniziativa |  | <= 4.2.x |
| 45 | 8 | numeric(6,2) | Punti scaricati |  | <= 4.2.x |
| 53 | 8 | numeric(6,2) | Contributo |  | <= 4.2.x |
| 61 | 1 | numeric | Flag tipo operazione (1=Ritiro immediato, 2=Prenotazione, 3=Ritiro da prenotazione, 4=Cancellazione) |  | <= 4.2.x |
| 62 | 13 | string | Codice Prenotazione |  | <= 4.2.x |
| 75 | 8 | numeric(6,2) | Quantità (Prenotata o Ritirata) |  | <= 4.2.x |

---

## Esempi Pratici
### Esempio Record 00 - Inizio Transazione
```
0002003000040000000010010400012025091717351910001000001             99017361  00002692000000000000006900000000000027000000000010000000000000000000225091730012000220270VF011199IEB0173610
```

### Esempio Record 01 - Dettaglio Articolo
```
000200300016018005529020159       A2200100010000000490000004901                      00000000000000000000000000      0001000000                                        AIELLO BAR MACI                               00PZ 000000
```

### Esempio Record 06 - Subtotale pre-pagamento
```
00020030001206030001202509171735450000000000000043000000000000000000000000000W0000000000000000000000000000000000000099017361  4.3.0.0                     99IEB01736100
```

### Esempio Record 03 - Pagamento
```
00020030001903010000001550000000000000100000000        00000004520250808162435                               EUR000000000000000000                                                  1001000100000000
```

### Esempio Record 04 - Fine Transazione
```
00020030002504000000000000001570000116244920250808        0000000000000000000000000                000000333000000045000000000000000000000004
```