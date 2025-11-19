---
tags:
    - Posware frontend
    - Argentea
    - Satispay
    - AMONEYSMART
---

# Posware Frontend - Satispay via Argentea AMONEYSMART

**Prima revisione documento: 9 maggio 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---
## Cenni preliminari

Il presente documento descrive come configurare **Posware** per accettare pagamenti tramite **Satispay**, utilizzando il modulo **Argentea Satispay**. 

Il processo include la configurazione del database e l'aggiornamento dell'interfaccia utente della cassa.

## Requisiti

La soluzione di pagamento Satispay / Argentea AMONEYSMART prevede esclusivamente le seguenti modalità di pagamento:

- Pagamento su Satispay Wallet – conto prepagato Satispay.  
- Pagamento Satispay via Buoni pasto elettronici, utilizzati **in modalità mista con il Wallet.**

!!! warning "Non è possibile pagare unicamente con i buoni pasto"
    Come da specifiche Satispay / Argentea alla data del 07/03/2024, è esplicitamente **esclusa la possibilità di pagare solo ed unicamente con i Buoni pasto elettronici.**
    È possibile tuttavia adottare alcuni approcci per mitigare il problema ed ottenere il pagamento con i soli BPE. Consultare [questo paragrafo](#possibili-strategie-di-mitigazione-per-pagare-unicamente-con-i-buoni-pasto-elettronici).

Posware garantisce inoltre i seguenti requisiti funzionali:

- Presenza di informazioni nel log di vendita per conoscere l’esatto ammontare dei pagamenti effettuati e permettere le corrette operazioni di riconciliazione e quadratura finanziaria di cassa nel punto vendita.  
- Differenziazione dei metodi di pagamento riscosso / buoni pasto numerati verso il Registratore Telematico (Stampante RT) per consentire una precisa comunicazione dei dati sui corrispettivi all’Agenzia delle Entrate.

!!! tip "Workflow Satispay"
    Tutte le modalità di pagamento elencate utilizzano il workflow ***MATCH_CODE*** di Satispay, che prevede la scansione di un QR Code da parte del cliente per innescare il pagamento.

L’uso di Satispay tramite Argentea AMONEYSMART prevede che il QR Code venga mostrato sul device di pagamento EFT già presente in cassa, utilizzato anche per i pagamenti con carta di credito, bancomat e strumenti analoghi.

### Versioni software e dipendenze di terze parti

L'integrazione fa riferimento alla documentazione Argentea del **07/03/2024** `rev. 3.89`.<br>
L'integrazione fa riferimento ai driver Argentea `v4.2.0.1`.<br>
**Eventuali successivi aggiornamenti non sono supportati da questo modulo.**

---

## Pagamento su Satispay Wallet

Lo scopo di questa funzione è permettere al cliente finale di pagare tramite l’app Satispay installata sullo smartphone, utilizzando unicamente il credito prepagato presente nel proprio Wallet Satispay.

In questo scenario non intervengono Buoni pasto elettronici, ma solo il conto prepagato Wallet.

### Pagamento su Satispay Wallet - Workflow di cassa

Al termine di una ordinaria transazione di vendita, a seguito della pressione del tasto subtotale, l’operatore preme un **tasto unico dedicato `Pagamento Satispay`** che innesca la richiesta di pagamento tramite terminale Argentea.

Il terminale POS-EFT mostrerà il QR Code che il cliente potrà scansionare usando l’app Satispay dal proprio smartphone, dal quale potrà confermare o rifiutare la richiesta di pagamento.

Esempio di workflow standard di cassa:

1. Effettuare la normale vendita degli articoli.  
2. Premere il tasto sub-totale.  
3. Premere il tasto `Pagamento Satispay` opportunamente configurato.  
4. Sul terminale di pagamento EFT viene mostrato il QR Code.  
5. Il cliente scansiona il QR Code con l’app Satispay.  
6. A seguito dell’avvenuto pagamento viene chiusa la transazione ed emesso lo scontrino.  
7. In caso di errori o rifiuto del pagamento viene mostrato un avviso che informa la cassiera, la quale può ritentare l’operazione o cambiare modalità di pagamento per chiudere la transazione.

Come previsto dalle linee guida di User Experience del servizio Satispay, in questo workflow viene utilizzato esclusivamente il credito Wallet.

### Pagamento su Satispay Wallet - Registrazioni nel log di vendita

Le informazioni dettagliate sul pagamento Satispay restituite da Argentea vengono memorizzate nel log di transazione di cassa.

In particolare, il Record 03 conterrà:

- L’esatto ammontare pagato tramite Wallet Satispay.  
- L’identificativo univoco della transazione Argentea / Satispay.

Queste informazioni sono utilizzabili per la riconciliazione e la quadratura finanziaria.

### Pagamento su Satispay Wallet - Corrispettivi verso Registratore Telematico

La tipologia di pagamento RT usata per notificare l’avvenuto pagamento al Registratore Telematico potrà essere configurata in base alle esigenze del cliente tramite le ordinarie configurazioni dei pagamenti previste in Posware.

In modalità predefinita, non vincolante, il pagamento Satispay Wallet viene registrato come `Pagamento Elettronico`.

### Pagamento su Satispay Wallet - Configurazione del pagamento in cassa

Per i dettagli sulla configurazione del pagamento in cassa relativi allo scenario Wallet, fare riferimento al paragrafo [Configurazione](#configurazione-dei-pagamenti-in-posware)

### Pagamento su Satispay Wallet - Ulteriori vincoli e casistiche

!!! warning "Vincoli Pagamento Wallet"
    1. È permesso l’uso delle formule per limitare l’ammontare del pagamento.  
    2. È permesso all’operatore imputare importi parziali da pagare per effettuare più pagamenti all’interno della stessa transazione.  
    3. È permesso lo storno della transazione di pagamento all’interno della transazione tramite le operazioni di `Annulla Pagamenti` e `Annulla scontrino`; in caso di pagamenti parziali multipli verranno stornati tutti.  

---

## Pagamento misto Buoni pasto elettronici e Saldo Wallet

Lo scopo di questa funzione è permettere al cliente finale di pagare tramite l’app Satispay usando sia i Buoni pasto elettronici sia il credito prepagato del proprio Wallet.

In questo caso il pagamento avviene in un’unica transazione che combina i due strumenti di pagamento, con priorità ai Buoni pasto elettronici.

### Pagamento misto - Workflow di cassa

Al termine di una ordinaria transazione di vendita, dopo la pressione del tasto subtotale, viene premuto **un tasto unico dedicato `Pagamento Satispay e BPE`** che innesca la richiesta di pagamento tramite terminale Argentea.

Il terminale POS-EFT mostrerà il QR Code che il cliente scansionerà con l’app Satispay dal proprio smartphone, dal quale potrà confermare o rifiutare la richiesta di pagamento.

Esempio di workflow standard di cassa:

1. Effettuare la normale vendita degli articoli.  
2. Premere il tasto sub-totale.  
3. Premere il tasto `Pagamento Satispay` opportunamente configurato per lo scenario misto.  
4. Sul terminale di pagamento EFT viene mostrato il QR Code.  
5. Il cliente scansiona il QR Code con l’app Satispay.  
6. A seguito dell’avvenuto pagamento viene chiusa la transazione ed emesso lo scontrino.  
7. In caso di errori o rifiuto del pagamento viene mostrato un avviso che informa la cassiera, che potrà ritentare l’operazione o cambiare modalità di pagamento.

In linea con le linee guida di User Experience del servizio Satispay, **è prevista una singola transazione di pagamento unificata, che permette al cliente di pagare usando sia i Buoni pasto elettronici Satispay sia il credito Wallet Satispay.**

!!! danger "Pagare unicamente con i buoni pasto"
    **Alla data del 07-03-2024 non è possibile forzare un pagamento usando solo ed esclusivamente i Buoni pasto elettronici, senza coinvolgere il Wallet.**
    È possibile tuttavia adottare alcuni approcci per mitigare il problema ed ottenere il pagamento con i soli BPE. Consultare [questo paragrafo](#possibili-strategie-di-mitigazione-per-pagare-unicamente-con-i-buoni-pasto-elettronici).

### Pagamento misto - Registrazioni nel log di vendita

Le informazioni dettagliate sul pagamento Satispay restituite da Argentea vengono memorizzate nel log di transazione di cassa, in modo analogo allo scenario Wallet puro.

Il Record 03 conterrà:

- L’esatto ammontare pagato tramite Wallet Satispay.  
- L’identificativo univoco della transazione Argentea / Satispay.

Al fine di garantire le quadrature del finanziario fiscale del punto vendita, viene generato un **secondo Record 03** dedicato esclusivamente ai Buoni pasto elettronici.

Questo secondo Record 03 conterrà:

- L'esatto ammontare pagato tramite Buoni pasto Satispay.
- Il codice della tipologia di pagamento RT specifico per i Buoni pasto. **Questo dettaglio non è presente in Posware `4.2`, dove sarà valorizzato sempre a 0**
- La quantità dei buoni pasto utilizzati. È possibile con questo valore ottenere il taglio del buono pasto.

La somma degli ammontari dei due Record 03 coincide con il totale pagato dal cliente finale e può essere utilizzata per eventuali riconciliazioni con sistemi esterni Argentea.

### Pagamento misto - Corrispettivi verso Registratore Telematico

La tipologia di pagamento RT utilizzata per notificare l’avvenuto pagamento al Registratore Telematico è configurabile tramite le normali tabelle di configurazione dei pagamenti di Posware, in base alle necessità del cliente.

La tipologia di pagamento RT relativa all’ammontare Wallet utilizza il valore configurato nel pagamento ad esso dedicato, indicato tramite il codice di pagamento incrociato nella tabella di configurazione Posware `tabparametriextra`.

In modalità predefinita, non vincolante, il pagamento misto Satispay risulta così registrato:

- `Ticket Numerati` per l’ammontare Buoni pasto elettronici.  
- `Pagamento Elettronico` per l’ammontare Wallet.

### Pagamento misto - Configurazione del pagamento in cassa

Per i dettagli sulla configurazione del pagamento in cassa relativi allo scenario Wallet, fare riferimento al paragrafo [Configurazione](#configurazione-dei-pagamenti-in-posware)

### Pagamento misto - Ulteriori vincoli e casistiche

!!! warning "Vincoli Pagamento misto BPE + Wallet"
    1. L’uso delle formule per limitare l’ammontare del pagamento viene applicato ai soli Buoni pasto elettronici.  
    2. In caso di utilizzo di formule, non è permesso all’operatore imputare importi parziali per effettuare più pagamenti all’interno della stessa transazione; il pagamento copre sempre il residuo completo della transazione, mentre la formula limita solo l’ammontare pagabile con Buoni pasto elettronici.  
    3. In presenza di pagamento misto con formule, viene richiesto di pagare l’intera transazione con Satispay usando sia Buoni pasto elettronici sia credito Wallet; se il credito disponibile non è sufficiente, la transazione viene rifiutata.  
    4. Per limitare l’importo del pagamento Satispay è possibile pagare prima con un altro metodo di pagamento che accetta importi parziali, come il contante, e poi ritentare il pagamento Satispay sul residuo.  
    5. In assenza di formule di limitazione è possibile effettuare più transazioni di pagamento; gli importi pagabili con Buoni pasto elettronici e Wallet coincidono e il credito Buoni pasto ha sempre priorità rispetto al credito Wallet.  
    6. È permesso lo storno della transazione di pagamento all’interno della transazione tramite le operazioni di `Annulla Pagamenti` e `Annulla scontrino`; in caso di pagamenti parziali multipli verranno stornate tutte le transazioni.  
    7. Non è possibile stornare separatamente l’ammontare pagato con Wallet o con soli Buoni pasto; in caso di annullo, le transazioni di pagamento vengono sempre stornate per intero.  

---

## Possibili strategie di mitigazione per pagare unicamente con i Buoni Pasto elettronici

I provider BPE tradizionali permettono di consumare un numero di BPE il cui valore può essere inferiore all'ammontare richiesto dal Frontend, restituendo a loro volta quando incassato.

Ad esempio:

- Si ha uno scontrino di 50€, con un pagabile BPE pari a 42€
- Posware invia la richiesta di pagamento al provider BPE per 42€
- Il provider può consumare al massimo 5 BPE da 8€ per un totale di 40€. Non può consumare 48€ in quanto supererebbe il valore da pagare richiesto che ha inviato Posware.
- Il provider restituisce a Posware 40€ (anzichè 42€) perchè è progettato per consumare il più possibile rispetto alla cifra richiesta
- Posware legge 40€ dal provider ed incassa
- L'operatore deve pagare la rimanenza con altro metodo di pagamento

<br>
**Satispay** prevede invece questo workflow:

- Si ha uno scontrino di 50€, con un pagabile BPE pari a 42€
- Posware invia la richiesta di pagamento a Satispay per 50€ e, con i BPE abilitati, pagabile BPE pari a 42€
- Satispay può consumare al massimo 5 BPE da 8€ per un totale di 40€. Non può consumare 48€ in quanto supererebbe il valore da pagare richiesto che ha inviato Posware.
- A questo punto possono avvenire due condizioni:

    1. Se il cliente ha i rimanenti 10€ nel Wallet (credito ordinario), Satispay restituisce a Posware 40€ di BPE e 10€ di Wallet. Posware registra i rispettivi pagamenti e l'operatore deve pagare la rimanenza con altro metodo di pagamento<br>
    2. Se il cliente **NON HA** i rimanenti 10€ nel Wallet (credito ordinario), **Satispay ==fallisce== il pagamento**

### Come pagare unicamente con i buoni pasto

È possibile avere pagamenti multipli di Satispay nella stessa transazione.<br>
L'operatore di cassa può sfruttare i pagamenti parziali per ottenere un pagamento con soli buoni pasto.<br>

Si procede come segue:

1. Si digita l'importo da pagare in Buoni Pasto. Questo può essere ottenuto:
    - Dietro richiesta al cliente stesso che può consultare il saldo sull'app
    - Vedere il massimo pagabile per il pagamento BPE Satispay che avete configurato, usando la funzione `Visualizza pagamenti`. La cifra si deve **arrotondare per difetto al taglio dei buoni pasto del cliente**, ad esempio:
        - Se il cliente ha 5 BPE da 8€ (48€ di BPE disponibili) e la cassa può pagarne 42€ (massimo pagabile), l'operatore digiterà 40€
2. Dopo aver imputato l'importo massimo, si preme Pagamento Satispay
3. L'importo viene passato a Satispay che a questo punto incasserà interamente usando solo i BPE
4. Il resto dell'ammontare potrà essere ancora pagato con Satispay Wallet o con un altro metodo di pagamento.

---

## Configurazione dei pagamenti in Posware

Per l’integrazione dei pagamenti Argentea / Satispay è necessario configurare sempre due pagamenti distinti in Posware:

- Un pagamento dedicato all’utilizzo dello scenario con solo Wallet Satispay.  
- Un pagamento dedicato allo scenario di pagamento misto Buoni pasto elettronici e Wallet.

Entrambi i pagamenti avranno configurazioni dedicate al fine di:

- Associare la corretta tipologia di pagamento RT.  
- Assegnare ciascun pagamento ad un pulsante dedicato nella grafica di cassa.

!!! warning "Il pulsante che richiama il pagamento è sempre unico"
    I pagamenti da configurare sono differenti al fine di distinguere a livello fiscale le forme di pagamento.<br>
    In grafica è sempre e solo necessario inserire un unico pulsante "Pagamento Satispay".<br>
    **Non inserire un pulsante dedicato unicamente ai Buoni Pasto Satispay**

Questa modalità di configurazione consente di specificare il tipo di pagamento RT da usare in base alle forme di pagamento ricevute da Satispay.

### Configurazione nelle tabelle

Per abilitare **Argentea Satispay** come metodo di pagamento, è necessario aggiornare le seguenti tabelle:

#### Tabella `tipi_pagamenti` (sia database`cassa` che `posware` sul server di barriera)

=== "Pulsante per richiamare il pagamento"

    - `codice`: valore intero univoco non utilizzato da altri metodi di pagamento
    - `descrizione`: *"Argentea Satispay"* o descrizione equivalente
    - `PagTipo`: 11
    - `tipoPagRT`: 1

=== "Pulsante per la configurazione dei ticket"

    - `codice`: valore intero univoco non utilizzato da altri metodi di pagamento
    - `descrizione`: *"BPE Satispay"* o descrizione equivalente
    - `ticket`: "SI"
    - `obbligatorio`: **true**
    - `tipoPagRT`: 4

#### Tabella `tabparametriextra` (database`cassa`)

|Modulo|Parametro|Valore|Note|
|-------|------|--------|----|
|EPPLIB|PROTOCOLLO|AR||
|EPPLIB|satispayBpeEnabled|0/1|Default ad `1`. Se impostato a zero, i buoni pasto non vengono abilitati|
|EPPLIB|satispayBpeMaxNumber|da 1 ad 8||
|EPPLIB|satispayBpePaymentCode|*Codice del tipo di pagamento "Ticket numerati"*||
|EPPLIB|satispayDummyType|0/1|Default a 0. È possibile impostarlo ad 1 per abilitare la modalità simulazione|

### Interfaccia utente della cassa

Aggiungere un unico pulsante alla grafica di Posware ed agganciarci il pagamento **Argentea Satispay**.