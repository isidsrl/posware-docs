---
tags:
    - Frontend
    - Argentea
    - Satispay
---

# Posware Frontend - Argentea Satispay

**Prima revisione documento: 9 maggio 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---
## Cenni preliminari
Il presente documento descrive come configurare **Posware** per accettare pagamenti tramite **Satispay**, utilizzando il modulo **Argentea Satispay**. Il processo include la configurazione del database e l'aggiornamento dell'interfaccia utente della cassa.

## Configurazione del Database
Per abilitare **Argentea Satispay** come metodo di pagamento, è necessario aggiornare le seguenti tabelle:

#### Tabella `tipi_pagamenti` (sia database`cassa` che `posware`)

- **Argentea Satispay:**
    - `codice`: valore intero univoco non utilizzato da altri metodi di pagamento
    - `descrizione`: *"Argentea Satispay"* o descrizione equivalente
    - `PagTipo`: 11
    - `tipoPagRT`: 1

- **Ticket numerati:**
    - `codice`: valore intero univoco non utilizzato da altri metodi di pagamento
    - `descrizione`: *"TicketCounter"* o descrizione equivalente
    - `ticket`: "SI"
    - `obbligatorio`: **true**
    - `tipoPagRT`: 4


#### Tabella `tabparametriextra` (database`cassa`)

|Modulo|Parametro|Valore|
|-------|------|--------|
|EPPLIB|PROTOCOLLO|AR|
|EPPLIB|satispayBpeEnabled|1|
|EPPLIB|satispayBpeMaxNumber|8|
|EPPLIB|satispayBpePaymentCode|*Codice del tipo di pagamento "Ticket numerati"*|
|EPPLIB|satispayDummyType|0|


#### Interfaccia Utente della cassa
Aggiungere un pulsante alla grafica di Posware ed agganciarci il pagamento **Argentea Satispay**.


## Workflow di Utilizzo

1. Premere il tasto **SUBTOTALE** sulla cassa
2. Inserire l'importo da pagare con **Argentea Satispay**
3. Selezionare il pulsante dedicato ad **Argentea Satispay**
4. Sul display del POS apparirà un QR Code
5. Il cliente scansionerà il QR Code utilizzando l'app di Satispay per completare il pagamento