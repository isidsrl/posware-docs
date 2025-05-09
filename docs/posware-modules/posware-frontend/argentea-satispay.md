---
tags:
    - Frontend
    - Argentea
    - Satispay
---

# Posware Frontend - Argentea Satispay

**Prima revisione documento: 8 maggio 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---
## Cenni preliminari
Il presente documento descrive le configurazioni necessarie a livello di database per l'utilizzo di **Satispay** come metodo di pagamento attraverso l'integrazione con **Argentea**. Inoltre verrà descritto brevemente il workflow per poter usare tale tipologia di pagamento.

## Configurazione
Per poter effettuare i pagamenti con **Satispay**, attraverso l'integrazione con **Argentea**, sarà necessario configurare adeguatamente alcuni parametri a livello di database.

Con riferimento alle tabelle `tipi_pagamenti` sia del database `cassa` che di quello `posware` è necessario impostare i seguenti metodi di pagamento:

1. **Argentea Satispay:**
    - Il campo `codice` va impostato a un **valore intero non ancora utilizzato per gli altri tipi di pagamento già presenti nella tabella** e non deve corrispondere a uno dei valori speciali usati per alcuni metodi di pagamento specifici.
    - Il campo `descrizione` contiene la **descrizione del tipo di pagamento** e deve quindi avere un valore significativo che faccia capire che si tratti di **Argentea Satispay**.
    - Il campo `PagTipo` deve essere impostato a **11**.
    - Il campo `tipoPagRT` va configurato con valore pari a **1**.

2. **Ticket numerati:**
    - Il campo `codice` va impostato a un **valore intero non ancora utilizzato per gli altri tipi di pagamento già presenti nella tabella** e non deve corrispondere a uno dei valori speciali usati per alcuni metodi di pagamento specifici.
    - Il campo `descrizione` contiene la **descrizione del tipo di pagamento** e deve quindi avere un valore significativo che faccia capire che si tratti dei **ticket numerati**. Un valore utilizzato solitamente è *TicketCounter*, ma non è obbligatorio usare necessariamente quest'ultimo.
    - Il campo `ticket` deve essere impostato a **SI**.
    - Il campo `obbligatorio` va configurato con valore pari a **true**.
    - Il campo `tipoPagRT` deve avere valore pari a **4**.


Per quanto riguarda la tabella `tabparametriextra` del database `cassa`, i parametri necessari sono i seguenti:

|Modulo|Parametro|Valore|
|-------|------|--------|
|EPPLIB|PROTOCOLLO|AR|
|EPPLIB|satispayBpeEnabled|1|
|EPPLIB|satispayBpeMaxNumber|8|
|EPPLIB|satispayBpePaymentCode|*Leggere nota sottostante*|
|EPPLIB|satispayDummyType|0|

A proposito del parametro *satispayBpePaymentCode*, quest'ultimo deve corrispondere al valore del tipo di pagamento corrispondente ai ticket numerati, contenuto nel campo `codice` nella tabella `tipi_pagamenti` del database `cassa`.

Infine è necessario aggiungere un pulsante alla grafica di Posware ed agganciarci il pagamento **Argentea Satispay**.

## Workflow di utilizzo
Una volta completata la configurazione del metodo di pagamento **Argentea Satispay**, il workflow per poterlo utilizzare è il seguente:

1. Dopo aver premuto il tasto **SUBTOTALE**, inserire la quantità che si desidera pagare con il metodo di pagamento **Argentea Satispay**
2. Una volta inserita la quantità, premere il pulsante assegnato al pagamento **Argentea Satispay**
3. A questo punto comparirà sullo schermo del POS un QR Code che il cliente dovrà scansionare con l'app di Satispay per completare con successo il pagamento