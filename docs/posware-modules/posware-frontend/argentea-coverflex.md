---
tags:
    - Posware frontend
    - Argentea
    - Coverflex
    - Welfare
---

# Posware Frontend - Coverflex via Argentea

**Prima revisione documento: 8 settembre 2026** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---
## Cenni preliminari

Il presente documento descrive come configurare **Posware** per accettare pagamenti tramite **card Coverflex**, utilizzando il **pagamento ibrido Argentea** (*Hybrid BPE*).

Il processo include la configurazione del database e l'aggiornamento dell'interfaccia utente della cassa.

!!! info "Disponibilità"
    L'integrazione Coverflex è disponibile su **Posware `4.2`** e **Posware `4.3`**, con il medesimo comportamento di cassa e la medesima configurazione.

    L'unica differenza fra le due versioni riguarda il contenuto del Record 03 nel log di vendita, segnalata [in questo paragrafo](#pagamento-con-card-coverflex-registrazioni-nel-log-di-vendita).

**Coverflex** è una piattaforma di welfare aziendale che emette una card fisica/virtuale sulla quale possono coesistere più borsellini:

- Il credito **Buoni pasto elettronici** (BPE).
- Il credito **Welfare** (buoni acquisto / fringe benefit).
- Il saldo della **carta bancaria** collegata al conto.

L'integrazione utilizza il device di pagamento EFT Argentea già presente in cassa, lo stesso impiegato per i pagamenti con carta di credito, bancomat e strumenti analoghi. **Non è previsto alcun collegamento diretto fra Posware e i sistemi Coverflex:** l'intera transazione è mediata dal terminale Argentea.

La soluzione di pagamento Coverflex / Argentea prevede le seguenti modalità di pagamento:

- Pagamento con **card Coverflex**, che in un'unica transazione può utilizzare Buoni pasto elettronici, credito Welfare e carta bancaria collegata.
- Pagamento con **carta bancaria ordinaria (non Coverflex)**, gestito dallo stesso tasto di cassa e trattato come un normale pagamento EFT.

!!! tip "Workflow Coverflex"
    Tutte le modalità di pagamento elencate utilizzano il workflow di **pagamento ibrido Argentea**: Posware invia al terminale l'importo totale da pagare e l'importo massimo pagabile con i buoni, il cliente presenta la propria card al POS-EFT ed è il terminale a decidere la ripartizione fra i borsellini disponibili.

Posware garantisce inoltre i seguenti requisiti funzionali:

- Presenza di informazioni nel log di vendita per conoscere l'esatto ammontare dei pagamenti effettuati e permettere le corrette operazioni di riconciliazione e quadratura finanziaria di cassa nel punto vendita.
- Differenziazione dei metodi di pagamento riscosso / buoni pasto numerati verso il Registratore Telematico (Stampante RT) per consentire una precisa comunicazione dei dati sui corrispettivi all'Agenzia delle Entrate.

## Requisiti

L'integrazione richiede due abilitazioni esterne a Posware, da concordare con il fornitore del servizio EFT:

- La funzione di **pagamento ibrido** deve essere esplicitamente abilitata sul terminale Argentea.
- **Il terminale deve essere censito e abilitato lato Coverflex per l'esercente.**

Sulla cassa devono inoltre essere presenti i driver Argentea nella versione minima indicata di seguito.

### Versioni software e dipendenze di terze parti

L'integrazione fa riferimento alla documentazione Argentea dell'**08/07/2026** `rev. 04.04`.<br>
L'integrazione fa riferimento ai driver Argentea `v4.2.1.0`.<br>
**Quest'ultima è la versione minima necessaria al funzionamento dell'integrazione. Eventuali versioni precedenti non sono supportate da questo modulo.**

!!! danger "Breaking change versione minima driver Argentea"
    All'avvio Posware verifica la versione di `C:\EPP2\pagamento.dll`. Se la versione rilevata è inferiore a `4.2.1.0`, viene mostrato un avviso all'operatore e **il funzionamento dell'integrazione non è garantito.**

---

## Interfaccia con il terminale Argentea

Per ogni richiesta di pagamento Posware invia al terminale i seguenti valori:

|Parametro|Valore inviato da Posware|
|---------|-------------------------|
|`TermId`|Contenuto del parametro `argenteaCoverflexTerminalId`|
|`Amount`|Importo totale da pagare, in centesimi. Coincide sempre con il massimo pagabile della forma di pagamento Coverflex|
|`AmountVoucher`|Importo massimo pagabile con i buoni, in centesimi. Calcolato come il minore fra il pagabile del pagamento *BPE Coverflex* (determinato dalle eventuali formule configurate su quel pagamento) e `Amount`|
|`PayType`|Fisso a **1** (`COVERFLEX`)|

!!! info "Provider gestiti"
    Il pagamento ibrido Argentea supporta anche il provider `EDENRED` (`PayType` = 3). **Questa integrazione richiede esclusivamente il provider Coverflex** e non gestisce Edenred UAM.

---

## Ripartizione degli importi

La risposta del terminale determina come Posware ripartisce l'incasso su più codici di pagamento distinti:

| Componente restituita da Argentea | Codice di pagamento Posware utilizzato |
|-----------------------------------|----------------------------------------|
| Quota pagata con Buoni pasto elettronici | Pagamento *BPE Coverflex* |
| Quota pagata con credito Welfare (buoni acquisto) | Pagamento *Welfare Coverflex* |
| Quota residua a carico della carta bancaria collegata al conto Coverflex | Pagamento *Coverflex* (quello associato al tasto in grafica) |
| Intero importo, in caso di carta non Coverflex | Pagamento carta ordinario, individuato dal codice acquirer |

!!! info "Buoni pasto e credito Welfare sono alternativi"
    Come da specifiche Argentea, nella stessa transazione il terminale utilizza **o** i Buoni pasto elettronici **o** il credito Welfare, mai entrambi. Il dettaglio dei buoni restituito dal terminale riguarda **esclusivamente i buoni pasto**: i Buoni Acquisto Welfare non vengono mai conteggiati.

    - Se la risposta contiene il dettaglio dei buoni pasto (quantità e tagli), la quota buoni viene registrata come **BPE**, per l'ammontare effettivamente risultante dal dettaglio.
    - Se la risposta non contiene alcun dettaglio buoni pasto, la quota buoni viene registrata come **Welfare**, per un ammontare pari all'importo massimo pagabile con i buoni inviato nella richiesta.

    La quota residua, non coperta dai buoni, viene sempre attribuita alla carta bancaria collegata al conto Coverflex e registrata sul pagamento *Coverflex*.

!!! warning "Non è possibile distinguere il Welfare dall'esaurimento dei buoni pasto"
    Quando il pagamento è autorizzato con credito Welfare, **il terminale non fornisce alcuna informazione aggiuntiva** che permetta di distinguere fra "credito Welfare utilizzato" e "buoni pasto esauriti con addebito integrale sulla carta collegata".

    La sola discriminante disponibile è l'assenza del dettaglio dei buoni pasto, ed è quella adottata da Posware.

La somma degli importi registrati sui diversi codici di pagamento coincide sempre con l'importo autorizzato dal terminale e può essere utilizzata per eventuali riconciliazioni con i sistemi esterni Argentea e Coverflex.

---

## Vincoli sull'importo del pagamento

I vincoli descritti in questo paragrafo valgono per entrambi gli scenari di pagamento descritti nel seguito, indipendentemente dal tipo di carta presentata dal cliente.

!!! warning "L'importo è sempre il massimo pagabile"
    Analogamente al pagamento EFT Bancomat / Carte di credito, **l'operatore non può digitare un importo parziale.** Qualunque importo venga imputato prima di premere il tasto, Posware forza sempre il massimo pagabile per quella forma di pagamento.

    Se il massimo pagabile risulta pari a zero, il pagamento viene inibito e all'operatore viene mostrato il messaggio di pagamento non possibile.

### Un solo pagamento Coverflex per scontrino

Dal vincolo sull'importo discende una conseguenza importante: **in uno scontrino può esistere un solo pagamento Coverflex andato a buon fine.**

L'importo richiesto al terminale coincide sempre con il massimo pagabile della forma di pagamento, e il terminale autorizza esattamente l'importo richiesto. Di conseguenza, al termine di un pagamento riuscito il massimo pagabile di quel codice risulta interamente consumato: **alla successiva pressione del tasto `Pagamento Coverflex` il massimo pagabile è pari a zero e il pagamento viene inibito.**

Questo vale in tutte le configurazioni, comprese quelle in cui il pagamento è limitato da formule di inclusione, da una percentuale massima pagabile o da un valore massimo pagabile: tali limitazioni riducono l'importo della singola richiesta, ma non consentono di ripetere l'operazione sul residuo.

Ne consegue che **i due scenari descritti nel seguito sono sempre alternativi fra loro:** nello stesso scontrino non possono coesistere un pagamento con card Coverflex e un pagamento con carta non Coverflex eseguiti dal medesimo tasto.

!!! info "Tentativi falliti e riprese"
    Il vincolo riguarda i soli pagamenti **andati a buon fine**.

    Un tentativo rifiutato dal terminale non consuma il massimo pagabile e non viene registrato: l'operatore può ripetere l'operazione, oppure chiudere la transazione con un'altra forma di pagamento.

---

## Pagamento con card Coverflex

Lo scopo di questa funzione è permettere al cliente finale di pagare presentando al POS-EFT la propria card Coverflex, utilizzando in un'unica transazione i Buoni pasto elettronici oppure il credito Welfare, con eventuale integrazione a carico della carta bancaria collegata al conto Coverflex.

### Pagamento con card Coverflex - Workflow di cassa

Al termine di una ordinaria transazione di vendita, a seguito della pressione del tasto subtotale, l'operatore preme un **tasto unico dedicato `Pagamento Coverflex`** che innesca la richiesta di pagamento tramite terminale Argentea.

Esempio di workflow standard di cassa:

1. Effettuare la normale vendita degli articoli.
2. Premere il tasto sub-totale.
3. Premere il tasto `Pagamento Coverflex` opportunamente configurato.
4. Il terminale di pagamento EFT richiede la presentazione della card.
5. Il cliente presenta la card Coverflex ed esegue l'eventuale verifica del titolare (PIN, contactless, ecc.).
6. A seguito dell'avvenuto pagamento viene chiusa la transazione ed emesso lo scontrino.
7. In caso di errori o rifiuto del pagamento viene mostrato un avviso che informa la cassiera, la quale può ritentare l'operazione o cambiare modalità di pagamento per chiudere la transazione.

Al termine dell'operazione vengono stampati **due scontrini non fiscali**: lo scontrino EFT prodotto dal terminale e lo scontrino di riepilogo Coverflex, che riporta l'importo totale, la quota pagata a coupon e la quota pagata a carta. Il numero di copie stampate è quello configurato sul pagamento *Coverflex* nella tabella `tipi_pagamenti`.

### Pagamento con card Coverflex - Registrazioni nel log di vendita

Le informazioni dettagliate sul pagamento Coverflex restituite da Argentea vengono memorizzate nel log di transazione di cassa, generando **un Record 03 per ciascuna componente incassata.**

Il Record 03 relativo alla quota a carico della carta collegata al conto Coverflex conterrà:

- L'esatto ammontare addebitato sulla carta collegata.
- L'identificativo univoco della transazione Argentea / Coverflex.

Al fine di garantire le quadrature del finanziario fiscale del punto vendita, viene generato un **ulteriore Record 03** dedicato alla componente buoni, che conterrà:

- L'esatto ammontare pagato tramite Buoni pasto elettronici oppure tramite credito Welfare.
- Il codice della tipologia di pagamento RT specifico configurato sul relativo pagamento. **Questo dettaglio non è presente in Posware `4.2`, dove sarà valorizzato sempre a 0**
- La quantità dei buoni pasto utilizzati, valorizzata solo nel caso dei Buoni pasto elettronici.
- Il medesimo identificativo univoco della transazione Argentea / Coverflex.

L'identificativo di transazione, comune a tutti i Record 03 generati dalla stessa operazione, permette di ricondurre le singole componenti a un unico pagamento.

Queste informazioni sono utilizzabili per la riconciliazione e la quadratura finanziaria.

!!! warning "Nota sul taglio dei Buoni pasto elettronici"
    Coverflex può consumare nella stessa transazione buoni pasto di **tagli differenti**, ma il tracciato del log prevede un unico valore di taglio per Record 03.

    Posware registra pertanto il **taglio medio**, ottenuto dividendo l'ammontare complessivo dei buoni pasto per il numero di buoni utilizzati. Il dato attendibile per la quadratura resta quindi l'**ammontare complessivo** unitamente alla **quantità di buoni**, non il taglio.

!!! warning "Nota sul credito Welfare"
    A differenza dei Buoni pasto elettronici, il credito Welfare non ha un taglio predefinito: il valore utilizzato coincide esattamente con l'importo speso.

    Di conseguenza, nel Record 03 relativo al Welfare **non viene riportata alcuna quantità di buoni.**

### Pagamento con card Coverflex - Corrispettivi verso Registratore Telematico

La tipologia di pagamento RT usata per notificare l'avvenuto pagamento al Registratore Telematico potrà essere configurata in base alle esigenze del cliente tramite le ordinarie configurazioni dei pagamenti previste in Posware.

Ogni componente utilizza il valore `tipoPagRT` configurato sul proprio codice di pagamento.

In modalità predefinita, non vincolante, il pagamento Coverflex risulta così registrato:

- `Ticket Numerati` per l'ammontare Buoni pasto elettronici.
- `Pagamento Elettronico` per l'ammontare Welfare.
- `Pagamento Elettronico` per l'ammontare a carico della carta collegata al conto Coverflex.

!!! danger "Buono monouso non supportato"
    **Non configurare `tipoPagRT` = 9 (buono monouso) su nessuno dei pagamenti coinvolti nell'integrazione Coverflex.**

    L'integrazione non valorizza l'aliquota IVA del buono, informazione obbligatoria per i buoni monouso: la comunicazione dei corrispettivi verso il Registratore Telematico risulterebbe incompleta.

### Pagamento con card Coverflex - Configurazione del pagamento in cassa

Per i dettagli sulla configurazione del pagamento in cassa relativi allo scenario con card Coverflex, fare riferimento al paragrafo [Configurazione](#configurazione-dei-pagamenti-in-posware)

### Pagamento con card Coverflex - Ulteriori vincoli e casistiche

!!! warning "Vincoli Pagamento Coverflex"
    1. È permesso l'uso delle formule per limitare l'ammontare del pagamento. Le formule configurate sul pagamento *Coverflex* limitano l'importo complessivo della richiesta; le formule configurate sul pagamento *BPE Coverflex* limitano la sola quota pagabile con i buoni.
    2. **Non è permesso all'operatore imputare importi parziali:** il pagamento copre sempre l'intero massimo pagabile della forma di pagamento.
    3. Per limitare l'importo del pagamento Coverflex è possibile pagare prima con un altro metodo di pagamento che accetta importi parziali, come il contante, e poi eseguire il pagamento Coverflex sul residuo.
    4. **All'interno dello stesso scontrino è possibile un solo pagamento Coverflex andato a buon fine.** Consultare [questo paragrafo](#un-solo-pagamento-coverflex-per-scontrino).
    5. **Non è permesso lo storno del pagamento**, né tramite `Annulla Pagamenti` né tramite `Annulla scontrino`. Consultare [questo paragrafo](#storno-e-annullo-dei-pagamenti).
    6. Non è possibile stornare separatamente la quota buoni dalla quota card: le due componenti appartengono a un'unica transazione autorizzata dal terminale.

---

## Pagamento con carta non Coverflex

Il terminale Argentea, interrogato con la richiesta di pagamento ibrido, accetta anche carte che **non** appartengono al circuito Coverflex.

La funzione di pagamento ibrido **non può impedire a priori la lettura di una carta bancaria standard sul POS.** In questo scenario l'operazione non viene processata da Coverflex ma completata come ordinaria transazione bancaria (CB2): il terminale segnala alla cassa che la carta presentata non appartiene al circuito Coverflex e restituisce il codice acquirer della transazione.

Lo scopo di questa funzione è evitare che l'operatore debba individuare a priori il tipo di carta presentata dal cliente: **il tasto `Pagamento Coverflex` è in grado di gestire entrambi gli scenari.**

!!! info "L'importo è effettivamente addebitato"
    Come confermato da Argentea, in questo scenario la transazione bancaria viene regolarmente autorizzata e l'importo **è effettivamente addebitato sulla carta presentata**, come attestato dallo scontrino EFT.

    Registrando l'incasso su un codice di pagamento carta, Posware si mantiene quindi allineato al movimento contabile reale.

### Pagamento con carta non Coverflex - Workflow di cassa

Il workflow di cassa è identico a quello descritto per la card Coverflex. L'unica differenza percepita dall'operatore è la stampa del solo scontrino EFT, in assenza del riepilogo Coverflex.

### Pagamento con carta non Coverflex - Registrazioni nel log di vendita

Viene generato un **unico Record 03**, contenente:

- L'intero ammontare autorizzato.
- L'identificativo univoco della transazione Argentea.

**Nessun importo viene attribuito ai codici di pagamento Coverflex, BPE o Welfare.**

Il codice di pagamento utilizzato viene determinato con la seguente logica:

1. Posware ricerca il codice acquirer restituito dal terminale nella tabella `pagamenti_codritorno`. Se esiste una corrispondenza, viene utilizzato il codice di pagamento mappato.
2. Se non esiste alcuna corrispondenza, viene utilizzato il codice di pagamento indicato nel parametro `argenteaCoverflexBankPaymentCode` della tabella `tabparametriextra`.

!!! tip "L'assenza di mappatura non è un errore"
    La mancata mappatura del codice acquirer viene registrata nel log applicativo come avviso, ma **non costituisce un errore**: il pagamento viene comunque registrato sul codice di fallback configurato.

    La mappatura in `pagamenti_codritorno` è consigliata quando il punto vendita necessita di distinguere i circuiti nel finanziario di cassa.

### Pagamento con carta non Coverflex - Corrispettivi verso Registratore Telematico

La tipologia di pagamento RT utilizzata per notificare l'avvenuto pagamento al Registratore Telematico è configurabile tramite le normali tabelle di configurazione dei pagamenti di Posware, in base alle necessità del cliente.

La tipologia applicata è quella configurata sul codice di pagamento effettivamente utilizzato, secondo la logica di individuazione descritta sopra.

Poiché di norma tale codice coincide con il pagamento carte già in uso nel punto vendita, non è richiesta alcuna configurazione RT aggiuntiva.

### Pagamento con carta non Coverflex - Configurazione del pagamento in cassa

Per i dettagli sulla configurazione del pagamento in cassa relativi allo scenario con carta non Coverflex, fare riferimento al paragrafo [Configurazione](#configurazione-dei-pagamenti-in-posware)

### Pagamento con carta non Coverflex - Ulteriori vincoli e casistiche

!!! warning "Vincoli Pagamento carta non Coverflex"
    1. Valgono gli stessi vincoli sull'importo descritti per il pagamento con card Coverflex: l'importo è sempre pari al massimo pagabile.
    2. Le formule applicate sono quelle del pagamento *Coverflex*, ovvero del pagamento associato al tasto premuto, non quelle del codice di pagamento sul quale l'incasso viene successivamente registrato.
    3. **È permesso lo storno**, con i limiti descritti nel paragrafo seguente.

---

## Storno e annullo dei pagamenti

Le operazioni di `Annulla Pagamenti` e `Annulla scontrino` gestiscono i pagamenti Coverflex con due comportamenti radicalmente differenti.

=== "Pagamento con card Coverflex"

    **Lo storno non è possibile.**

    All'operatore viene mostrato un avviso che informa dell'impossibilità di annullare il pagamento e vengono ristampati gli scontrini delle transazioni interessate.

    L'operazione di rimborso deve essere gestita fuori linea, contattando Coverflex.

    !!! danger "Impatto operativo"
        Poiché il pagamento Coverflex non è stornabile, **è consigliabile posizionare il pagamento Coverflex come ultima operazione della transazione**, dopo aver verificato con il cliente l'importo e la disponibilità dei borsellini.

=== "Pagamento con carta non Coverflex"

    **Lo storno è possibile.**

    Trattandosi di una transazione puramente bancaria (CB2), l'annullamento segue le normali procedure di storno bancario previste dal terminale POS. Posware inoltra al terminale la richiesta di storno, stampa lo scontrino di storno e rimuove il pagamento dalla transazione.

    Lo storno viene tentato soltanto se il pagamento con carta non Coverflex risulta ancora presente fra i pagamenti dello scontrino.

    !!! danger "Non eseguire altri pagamenti EFT dopo il pagamento con carta non Coverflex"
        Lo storno inviato al terminale annulla **l'ultima operazione effettuata sul POS**, non una transazione individuata per identificativo.

        Se dopo il pagamento eseguito dal tasto `Pagamento Coverflex` viene effettuato un ulteriore pagamento EFT sullo stesso terminale, l'annullo agisce sull'ultima operazione del POS, ovvero **sul pagamento EFT successivo e non su quello che si intende stornare.**

        Lo scenario si presenta soltanto quando il massimo pagabile del pagamento *Coverflex* è limitato da una formula, da `PercMaxPagabile` o da `ValMaxPagabile`, lasciando un residuo che l'operatore chiude con un altro pagamento EFT.

        In presenza di questa configurazione, **chiudere il residuo con una forma di pagamento non EFT** (ad esempio il contante) oppure eseguire il pagamento Coverflex per ultimo.

    !!! danger "Il cliente deve essere ancora presente in cassa"
        Durante l'esecuzione dello storno **il terminale POS richiede l'inserimento o la lettura della medesima carta bancaria utilizzata per il pagamento originale.**

        Se il cliente si è già allontanato, o non dispone più della carta, l'operazione di storno non può essere portata a termine.

!!! info "I due casi sono sempre alternativi"
    Poiché in uno scontrino può esistere un solo pagamento Coverflex andato a buon fine, all'atto dell'annullo si presenta sempre e soltanto **uno** dei due casi descritti sopra. Consultare [questo paragrafo](#un-solo-pagamento-coverflex-per-scontrino).

---

## Configurazione dei pagamenti in Posware

Per l'integrazione dei pagamenti Argentea / Coverflex è necessario configurare **quattro pagamenti distinti** in Posware:

- Un pagamento dedicato all'incasso a carico della **carta bancaria collegata al conto Coverflex**, che è anche il pagamento associato al tasto in grafica.
- Un pagamento dedicato alla quota **Buoni pasto elettronici**.
- Un pagamento dedicato alla quota **Welfare**.
- Un pagamento dedicato alle **carte non Coverflex**, di norma il pagamento carte già presente nel punto vendita.

Tutti i pagamenti avranno configurazioni dedicate al fine di:

- Associare la corretta tipologia di pagamento RT.
- Assegnare il pagamento Coverflex al pulsante dedicato nella grafica di cassa.

!!! warning "Il pulsante che richiama il pagamento è sempre unico"
    I pagamenti da configurare sono differenti al fine di distinguere a livello fiscale le forme di pagamento.<br>
    In grafica è sempre e solo necessario inserire un unico pulsante "Pagamento Coverflex".<br>
    **Non inserire un pulsante dedicato unicamente ai Buoni Pasto Coverflex né uno specifico soltanto per il credito Welfare.**

!!! danger "Configurazione obbligatoria dei tre codici di pagamento accessori"
    I parametri `argenteaCoverflexBpePaymentCode`, `argenteaCoverflexFringeBenefitCode` e `argenteaCoverflexBankPaymentCode` devono essere **tutti** valorizzati con un codice di pagamento valido e diverso da zero.

    In assenza anche di uno solo di essi, alla pressione del tasto `Pagamento Coverflex` viene mostrato un messaggio di errore all'operatore e **il pagamento non viene eseguito.**

### Configurazione nelle tabelle

Per abilitare **Argentea Coverflex** come metodo di pagamento, è necessario aggiornare le seguenti tabelle:

#### Tabella `tipi_pagamenti` (sia database`cassa` che `posware` sul server di barriera)

=== "Configurazione Coverflex"

    - `codice`: valore intero univoco non utilizzato da altri metodi di pagamento
    - `descrizione`: *"Coverflex"* o descrizione equivalente
    - `eft`: **true**
    - `PagTipo`: 13
    - `tipoPagRT`: 1

=== "Configurazione BPE Coverflex"

    - `codice`: valore intero univoco non utilizzato da altri metodi di pagamento
    - `descrizione`: *"BPE Coverflex"* o descrizione equivalente
    - `eft`: **false**
    - `ticket`: "SI"
    - `tipoPagRT`: 4

=== "Configurazione Welfare Coverflex"

    - `codice`: valore intero univoco non utilizzato da altri metodi di pagamento
    - `descrizione`: *"Welfare Coverflex"* o descrizione equivalente
    - `eft`: **false**
    - `tipoPagRT`: 1

=== "Configurazione carta non Coverflex"

    - `codice`: di norma il codice del pagamento carte già configurato nel punto vendita
    - `descrizione`: *"Bancomat / Carte"* o descrizione equivalente
    - `eft`: **true**
    - `tipoPagRT`: 1

!!! tip "Formule sul pagamento BPE Coverflex"
    Il massimo pagabile con i Buoni pasto elettronici viene calcolato da Posware sul pagamento *BPE Coverflex*.

    Le formule di limitazione degli articoli acquistabili con buoni pasto vanno pertanto configurate **su quel codice di pagamento**, non su quello associato al tasto in grafica.

#### Tabella `tabparametriextra` (database`cassa`)

|Modulo|Parametro|Valore|Note|
|-------|------|--------|----|
|EPPLIB|PROTOCOLLO|AR|Protocollo EFT Argentea|
|EPPLIB|PROTOCOLLOBPE|AR|**Obbligatorio.** In assenza di questo valore l'integrazione Coverflex non viene inizializzata|
|EPPLIB|argenteaHybridPaymentPaymentProtocol|COVERFLEX|**Obbligatorio.** Abilita l'integrazione Coverflex sul pagamento ibrido Argentea|
|EPPLIB|argenteaCoverflexBpePaymentCode|*Codice del tipo di pagamento "BPE Coverflex"*|**Obbligatorio.** Diverso da 0|
|EPPLIB|argenteaCoverflexFringeBenefitCode|*Codice del tipo di pagamento "Welfare Coverflex"*|**Obbligatorio.** Diverso da 0|
|EPPLIB|argenteaCoverflexBankPaymentCode|*Codice del tipo di pagamento carte*|**Obbligatorio.** Diverso da 0. Utilizzato come fallback quando il codice acquirer non è mappato in `pagamenti_codritorno`|
|EPPLIB|argenteaCoverflexTerminalId|*Identificativo del terminale*|Facoltativo. Utilizzato negli scenari multi-banca per indirizzare la richiesta a uno specifico terminale. Lasciare vuoto per usare il terminale predefinito|
|EPPLIB|Dummy|True/False|Default a `False`. Se impostato a `True` abilita la modalità simulazione descritta [in questo paragrafo](#simulazione-posware)|

!!! warning "Il parametro *Dummy* è condiviso"
    Il parametro ***Dummy*** non è specifico dell'integrazione Coverflex: abilita la simulazione **su tutti i pagamenti elettronici** gestiti da EPPlib.

    **Non deve mai essere impostato a `True` su casse in produzione.**

#### Tabella `pagamenti_codritorno` (database`cassa`)

Tabella facoltativa, utilizzata per mappare il codice acquirer restituito dal terminale su uno specifico codice di pagamento Posware nello scenario di carta non Coverflex.

|Campo|Contenuto|
|-----|---------|
|`codritorno`|Codice acquirer restituito da Argentea|
|`pagamento`|Codice del tipo di pagamento Posware da utilizzare|

Si tratta della medesima tabella già impiegata dai pagamenti EFT tradizionali: se il punto vendita la utilizza già per distinguere i circuiti, **non è necessaria alcuna configurazione aggiuntiva** per l'integrazione Coverflex.

### Interfaccia utente della cassa

Aggiungere un unico pulsante alla grafica di Posware ed agganciarci il pagamento **Coverflex**.

---

## Modalità di collaudo

Sono disponibili due modalità di collaudo distinte e indipendenti fra loro: la **simulazione Posware**, che non coinvolge alcun terminale, e la **modalità demo Argentea**, che utilizza un POS reale collegato a un server Argentea configurato per il test.

### Simulazione Posware

Con il parametro `Dummy` impostato a `True`, alla pressione del tasto `Pagamento Coverflex` viene richiesto tramite tastiera virtuale un **codice test** che determina la risposta simulata del terminale.

|Codice test|Scenario simulato|
|-----------|-----------------|
|`0` *(o qualsiasi altro valore)*|Pagamento Coverflex eseguito, buoni pasto di taglio unico|
|`01`|Pagamento Coverflex eseguito, buoni pasto di tagli differenti|
|`02`|Pagamento eseguito con carta **non** Coverflex|
|`3`|Nessuna risposta ricevuta dal terminale|
|`7`|Transazione non eseguita per errore socket|
|`99`|Eccezione durante la chiamata al terminale|

!!! danger "Solo per collaudo"
    La modalità simulazione non dialoga con alcun terminale reale e non produce alcun incasso.
    **Deve essere utilizzata esclusivamente in ambienti di test.**

### Modalità demo Argentea

Argentea mette a disposizione per il provider Coverflex una modalità demo basata sulla mappatura di specifiche combinazioni di `Amount` e `AmountVoucher` verso esiti fissi.

La modalità demo è utilizzabile a condizione che:

- Il server Argentea abbia la modalità demo abilitata.
- Sul POS venga utilizzato un `termId` Argentea.

|Amount|AmountVoucher|Esito POS|Esito transazione|
|------|-------------|---------|-----------------|
|1000|800|OK|OK (200 carta, 800 voucher)|
|800|800|OK|OK (0 carta, 800 voucher)|
|40000|800|KO|KO - FALLITA|
|500|500|OK|KO - SCADUTA|
|1500|1500|KO|KO - CANCELLATA|
|Altro|Altro|OK|OK (0 carta, 800 voucher)|

Gli importi si intendono espressi in centesimi: la riga `1000` / `800` corrisponde quindi a uno scontrino di 10,00 EUR con 8,00 EUR di massimo pagabile in buoni.

!!! warning "Gli esiti demo dipendono dalla configurazione del terminale"
    Le combinazioni sopra riportate producono l'esito atteso **solo se l'ambiente di test è allineato alla mappatura documentata.**

    Nei collaudi condotti nell'agosto 2026 i casi previsti come `KO` hanno restituito esito positivo, con la dicitura *"UTILIZZATA CARTA NON COVERFLEX"* e addebito effettivo sulla carta, a causa di modifiche introdotte dal gestore del terminale nell'ambiente di test, che consentivano l'elaborazione di qualsiasi pagamento.

    In caso di esiti difformi dalla tabella, verificare con Argentea la configurazione dell'ambiente di test prima di ricondurre l'anomalia all'integrazione.
