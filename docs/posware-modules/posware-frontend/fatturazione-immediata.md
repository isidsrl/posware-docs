---
tags:
    - Frontend
---

# Posware Frontend - Fatturazione immediata

**Prima revisione documento: 26 aprile 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Glossario

- **WebApp:** applicazione web che offre la possibilità di gestire in modo centralizzato le funzionalità disponibili nel StoreServer;
- **PoswareModule:** plug-in del StoreServer dedicato alla gestione del PPC;
- **SdI:** si tratta del Sistema di Interscambio. Gestito dall'Agenzia delle Entrate, è un sistema informatico in grado di ricevere e inoltrare le fatture ed effettuare controlli sui file ricevuti;
- **Cedente:** l'azienda che genera ed emette la fattura, anche chiamato a volte **Fornitore**;
- **Fornitore:** si intende sempre il **Cedente**, i termini sono intercambiabili ed usati entrambi nel documento per riferirsi alla stessa cosa;
- **Cessionario:** l'entità che riceve la fattura e che quindi di fatto effettua l'acquisto;
- **PA:** Pubblica Amministrazione;
- **AdE:** Agenzia delle Entrate;

## Cenni preliminari

La funzione di **fatturazione elettronica immediata** (o *Direct Invoice*) permette di emettere una fattura elettronica direttamente dal **Posware Frontend** subito dopo la chiusura di una transazione, senza dover accedere alla WebApp dello StoreServer.

A differenza della fatturazione differita disponibile dalla WebApp, questa modalità consente all'operatore di cassa di:

- Attivare la richiesta di fattura durante la transazione
- Selezionare il cliente destinatario subito dopo l'emissione del documento commerciale
- Ottenere conferma immediata dell'emissione della fattura elettronica

!!! info "Differenza con la fatturazione differita"
    La fatturazione differita dalla WebApp permette di emettere fatture in un secondo momento, partendo dagli estremi della transazione. La fatturazione immediata da cassa invece avviene contestualmente alla chiusura della vendita.

### Vincoli ed esclusioni

- È possibile fatturare solo a **clienti privati con Partita IVA** (liberi professionisti, aziende) e **clienti privati senza Partita IVA** (privati cittadini, ONLUS senza P.IVA)
- **Le Pubbliche Amministrazioni sono escluse** dalla fatturazione immediata diretta (vedere [sezione dedicata](#fatturazione-alla-pubblica-amministrazione))
- Il documento emesso è sempre di tipo **commerciale** (documento fiscale RT)
- È richiesta la connettività con lo StoreServer

### Versioni software

È richiesta la versione **v. 4.3.x** o superiore di **Posware Frontend**.

### Requisiti

- Licenza del modulo Fattura Elettronica attiva sullo StoreServer
- Dati del cedente (punto vendita) correttamente configurati sullo StoreServer
- Connettività di rete tra la cassa e lo StoreServer
- Stampante RT configurata e funzionante

## Configurazione

### Configurazione del pulsante

Per abilitare la fatturazione immediata, è necessario configurare un pulsante nella tastiera di Posware con l'operazione **`DIRECT_EINVOICE`**.

!!! warning "Migrazione dalla versione 4.2"
    Nella versione 4.2 la funzione era attivata dall'operazione `FATTURA`. Dalla versione 4.3 è necessario usare `DIRECT_EINVOICE`.

### Parametri di configurazione

I seguenti parametri sono configurabili nella tabella `tabparametriextra` con modulo `posware.einvoice`:

| Parametro | Tipo | Descrizione |
|-----------|------|-------------|
| `useThirdPartySoftwareForEInvoiceGeneration` | boolean | Se attivo, delega l'emissione della fattura a un software esterno (backoffice). Posware mostrerà solo gli estremi della transazione da fatturare |
| `printReceiptAsReminderForEInvoice` | boolean | Se attivo, stampa uno scontrino di reminder con gli estremi della transazione e il barcode per il recupero veloce |
| `treatUnpaidPaymentAsEInvoiceForPA` | boolean | Se attivo, il pagamento "non riscosso segue fattura" viene trattato automaticamente come fattura per PA, saltando la schermata di selezione cliente |

## Funzionalità

### Attivazione della fatturazione immediata

Durante una transazione di vendita, l'operatore può attivare la richiesta di fattura immediata premendo il pulsante configurato con l'operazione `DIRECT_EINVOICE`.

!!! warning "Momento di attivazione"
    L'attivazione (e la disattivazione) della fatturazione immediata è possibile **solo fino alla fase del subtotale** (inclusa). Una volta entrati nella fase dei pagamenti, non è più possibile modificare questa impostazione.

#### Disattivazione

Per disattivare la richiesta di fattura, premere nuovamente lo stesso pulsante. Verrà mostrata una finestra di conferma SI/NO.

!!! note "Migrazione dalla versione 4.2"
    Nella versione 4.2 la disattivazione era gestita dall'operazione `ANNULLA_RICHIESTA_FATTURA`. Dalla versione 4.3 si usa lo stesso pulsante `DIRECT_EINVOICE` per attivare e disattivare.

### Workflow di emissione

Dopo aver chiuso la transazione con la fatturazione immediata attivata:

1. **Sincronizzazione dati**: Il log della transazione viene inviato immediatamente allo StoreServer
2. **Verifica cedente**: Viene verificata la correttezza dei dati di fatturazione del punto vendita
3. **Schermata di selezione**: Appare una schermata per la scelta del cliente
4. **Emissione fattura**: Lo StoreServer genera la fattura elettronica
5. **Conferma**: L'operatore riceve conferma dell'avvenuta emissione

#### Schermata di selezione cliente

La schermata permette di:

- Ricercare un cliente nell'anagrafica
- Visualizzare i risultati della ricerca
- Selezionare il cliente destinatario della fattura

!!! info "Ricerca automatica tramite Fidelity Card"
    Se nella transazione è stata passata una tessera fidelity, il sistema tenta automaticamente di trovare il cliente associato. Se trovato, viene preselezionato ma l'operatore può comunque modificarlo cercandone un altro.

#### Tipologie di clienti disponibili

La selezione è limitata a:

- **Privati con Partita IVA**: liberi professionisti, aziende, ecc.
- **Privati senza Partita IVA**: privati cittadini, ONLUS senza P.IVA, ecc.

!!! danger "Pubbliche Amministrazioni escluse"
    Le Pubbliche Amministrazioni **non sono selezionabili** dalla fatturazione immediata in cassa. Vedere la [sezione dedicata](#fatturazione-alla-pubblica-amministrazione) per il workflow specifico.

### Registrazione di un nuovo cliente

Dalla cassa **non è possibile registrare un nuovo cliente** direttamente nell'interfaccia di fatturazione immediata (l'interfaccia è progettata per desktop/tablet/mobile con tastiera).

Sono disponibili le seguenti alternative:

1. **Pulsante nella Windows Form**: Apre il browser all'indirizzo dello StoreServer dove è possibile codificare il nuovo cliente con tastiera e mouse
2. **Secondo dispositivo**: Usare un tablet o smartphone per codificare il cliente e poi selezionarlo dalla cassa
3. **Box informazioni**: Registrare il cliente dal box informazioni e comunicare il riferimento alla cassa
4. **Fatturazione differita**: Chiudere la transazione senza fattura immediata e procedere successivamente dallo StoreServer

## Fatturazione alla Pubblica Amministrazione

La fatturazione alla Pubblica Amministrazione richiede campi aggiuntivi (CIG, CUP, ordine d'acquisto, contratto) non disponibili nell'interfaccia di cassa. Per questo motivo, le PA sono escluse dalla fatturazione immediata diretta.

### Workflow per la PA

Nella schermata di selezione cliente è presente:

- Un **avviso informativo** che indica l'impossibilità di fatturare direttamente alla PA
- Un **pulsante "Fattura alla Pubblica Amministrazione"**

Premendo questo pulsante:

1. Viene mostrato un avviso con gli estremi della transazione da fatturare
2. Se attivo il parametro `printReceiptAsReminderForEInvoice`, viene stampato uno scontrino di reminder
3. La finestra si chiude confermando l'operazione

L'operatore dovrà poi completare la fatturazione dallo StoreServer (cassa centrale o box informazioni).

### Fatturazione PA con pagamento "Non riscosso segue fattura"

Per le Pubbliche Amministrazioni che richiedono lo **split payment** (raramente _non_ è richiesto) è necessario utilizzare il pagamento "Non riscosso segue fattura" (`tipoPagRT = 7`).

#### Requisiti

- Il documento deve essere di tipo **commerciale** (fiscale)
- Il pagamento deve essere "Non riscosso segue fattura"
- Non è possibile combinare questo pagamento con ticket

#### Configurazione pagamenti

È possibile configurare più codici pagamento con `tipoPagRT = 7`, differenziati dal campo `PagXml` per indicare la modalità di pagamento nella fattura:

| Esempio | Descrizione | PagXml |
|---------|-------------|--------|
| Codice 1 | Non riscosso fattura contanti | MP01 |
| Codice 23 | Non riscosso fattura bonifico | MP05 |

#### Workflow

1. Effettuare la transazione fiscale
2. Pagare interamente con un pagamento "Non riscosso segue fattura"
3. Alla chiusura, si apre la schermata di fatturazione immediata
4. Premere il pulsante "Fattura alla Pubblica Amministrazione"
5. Completare la fatturazione dallo StoreServer

!!! tip "Parametro treatUnpaidPaymentAsEInvoiceForPA"
    Se attivo, il pagamento "Non riscosso segue fattura" salta automaticamente la schermata di selezione cliente e mostra direttamente l'avviso per la fatturazione differita. Utile se il cliente usa questa procedura **esclusivamente** per la PA.

#### Vincoli

Non è possibile usare il pagamento "Non riscosso segue fattura" quando:

- È stato selezionato un pagamento di tipo Ticket
- È stata attivata la fatturazione immediata usando il comando DIRECT_EINVOICE

In questi casi verranno mostrati i relativi messaggi di errore.

## Gestione errori e fallback

### Mancata raggiungibilità dello StoreServer

#### Mancata raggiungibilità permanente

Se lo StoreServer non è raggiungibile al momento della chiusura della transazione:

1. Viene mostrato un messaggio che indica l'impossibilità di procedere con la fatturazione immediata
2. È presente un pulsante **"Annulla operazione"**

Premendo "Annulla operazione":

1. Appare una conferma: *"Sei sicuro di voler annullare la fatturazione? Il documento commerciale di vendita è già stato emesso."*
2. Confermando con SI, viene mostrato un avviso con gli estremi della transazione:

```
## Operazione annullata 
#### Il documento commerciale di vendita cartaceo e la transazione sono stati comunque emessi. 
È ancora possibile emettere la fattura dallo StoreServer (cassa centrale o box informazioni).

Usare questi estremi della transazione per l'emissione:
Data: [data]
Cassa: [numero cassa]
Numero Transazione: [numero]
```

3. Se attivo il parametro `printReceiptAsReminderForEInvoice`, viene stampato uno scontrino di reminder

!!! warning "Documento già emesso"
    Il documento commerciale di vendita viene emesso **prima** della fatturazione elettronica. In caso di annullamento, il cliente esce con lo scontrino e l'operatore dovrà generare la fattura successivamente dallo StoreServer.

#### Mancata raggiungibilità temporanea

Se la schermata è stata caricata ma lo StoreServer diventa irraggiungibile:

1. Viene mostrato lo stesso messaggio di errore
2. L'operatore dovrà procedere con la fatturazione differita dallo StoreServer

### Transazione non trovata

Se il log della transazione non è stato ancora ricevuto dallo StoreServer:

1. È possibile **riprovare fino a 3 volte** premendo il pulsante "Riprova"
2. Ogni tentativo rinvia anche il log della transazione
3. Dopo 3 tentativi falliti, la procedura si chiude automaticamente notificando l'operatore

### Dati cedente non validi

Se i dati del cedente (punto vendita) non sono configurati correttamente sullo StoreServer:

1. Viene mostrata una schermata di errore
2. L'operatore viene informato che è necessario correggere i dati di fatturazione prima di poter emettere fatture immediate
3. L'unica opzione disponibile è "Annulla operazione" per ottenere gli estremi della transazione e fatturare successivamente

## Delega a software esterni

Se il parametro `useThirdPartySoftwareForEInvoiceGeneration` è attivo, Posware **non** avvia la procedura di emissione fattura ma mostra direttamente un messaggio con gli estremi della transazione:

```
#### Il pro-forma cartaceo e la transazione sono stati emessi.

***È necessario emettere la fattura dal proprio Back-Office.***

Usare questi estremi della transazione per l'emissione:

***Data***: [data]
***Cassa***: [numero cassa]
***Numero Transazione***: [numero]
```

Il documento viene marcato con un flag specifico nel log per indicare che la fatturazione è delegata a un sistema esterno.

## Dettagli tecnici

### Modifiche al tracciato Log Record 00

La fatturazione immediata utilizza i seguenti campi nel record 00 del log:

| Campo | Posizione | Lunghezza | Tipo | Descrizione |
|-------|-----------|-----------|------|-------------|
| Flag11 | 55 | 1 | Boolean | Indica che la transazione è flaggata per fatturazione elettronica |
| Flag fatturazione terze parti | 185 | 1 | Boolean | Indica che la fattura sarà emessa da sistema esterno |

#### Flag11 - Contrassegno fatturazione

Questo flag permette di:

- Tracciare lato StoreServer le transazioni che richiedono fatturazione
- Identificare documenti pendenti non ancora fatturati
- Incrociare con le fatture emesse per verificare completezza

#### Flag fatturazione terze parti

Se attivo (in base al parametro `useThirdPartySoftwareForEInvoiceGeneration`):

- Lo StoreServer **non** monitorerà l'emissione effettiva della fattura
- Il controllo è delegato al sistema esterno
- Il flag è sempre in aggiunta al Flag11

### Causale movimento

Quando la fatturazione immediata è attiva:

- La causale nel record 00 (posizione 168, 2 caratteri) viene impostata a **`DI`** (Direct EInvoice)
- Se disattivata, torna al default **`VF`** (Vendita Fiscale)
- Il tipo movimento rimane **00** (vendita fiscale)

### Stampa documento

Sul documento commerciale di vendita (quando attiva la fatturazione immediata) viene stampata una riga informativa che indica:

- Che il documento è un pro-forma fattura
- Che per esso è stata/sarà emessa fattura elettronica

## Riepilogo parametri

| Modulo | Parametro | Tipo | Default | Descrizione |
|--------|-----------|------|---------|-------------|
| posware.einvoice | useThirdPartySoftwareForEInvoiceGeneration | boolean | false | Delega emissione a software esterno |
| posware.einvoice | printReceiptAsReminderForEInvoice | boolean | false | Stampa scontrino reminder con estremi transazione |
| posware.einvoice | treatUnpaidPaymentAsEInvoiceForPA | boolean | false | Pagamento "non riscosso" trattato automaticamente come PA |

## FAQ

### Perché non riesco a selezionare un cliente PA?

Le Pubbliche Amministrazioni richiedono campi aggiuntivi (CIG, CUP, dati contratto/ordine) non disponibili nell'interfaccia di cassa. Usare il pulsante "Fattura alla Pubblica Amministrazione" e completare l'operazione dallo StoreServer.

### Perché non posso attivare la fatturazione dopo aver iniziato i pagamenti?

Per garantire la coerenza del documento fiscale, l'attivazione/disattivazione è consentita solo fino alla fase del subtotale.

### Cosa succede se lo StoreServer non è raggiungibile?

Il documento commerciale viene comunque emesso. Annotare gli estremi della transazione (o stampare il reminder se configurato) e procedere con la fatturazione differita dallo StoreServer.

### Posso usare la fatturazione immediata con i ticket?

No, non è possibile combinare ticket con il pagamento "Non riscosso segue fattura" richiesto per la fatturazione PA con split payment.

### Come faccio a registrare un nuovo cliente dalla cassa?

Non è possibile direttamente dall'interfaccia di cassa. Usare il pulsante per aprire il browser verso lo StoreServer, oppure registrare il cliente da un altro dispositivo (tablet, box informazioni) e poi selezionarlo dalla cassa.
