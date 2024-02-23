# Posware 4.2 - Funzionalità Adapter NCR SelfServ™ Checkout
#### Specifiche funzionali e di integrazione
Data aggiornamento documento: **04/10/2021**

---
> ## Versioni software di riferimento
>
>
> - Posware **4.2.x**
> - Posware NCR Adapter **4.2.x**
> - NCR SelfServ™ Checkout Release **5.1.3 build 56**
> - ADK **4.5.1 Build 28**
> - TB5 (Transaction Broker) **Runtime 2.8.0.0161**



## Cenni preliminari
Il seguento documento descrive in dettaglio i requisiti tecnici, funzionali e le funzionalità supportate dell'adattatore Posware per NCR SelfServ Checkout.

I requisiti riportati garantiscono il corretto funzionamento rispetto alle funzionalità previte ed il supporto. <br>
Viene automaticamente escluso quanto eventualmente non riportato. L'esclusione è totale sia ai fini di garanzia di funzionamento che di supporto tecnico.

Per il dettaglio delle funzionalità supportate, continuare con la lettura di questo documento.








## Requisiti hardware e software
### Posware
La versione minima di Posware richiesta è la **4.2.0**. E' richiesta l'installazione del **.net Framework 4.0**

A far data **05/01/2021**, l'ambiente SCO NCR **non è compatibile con il .net Framework 4.8**. Pertanto sarà necessario installare Posware per Windows XP tramite il setup `PosUpdateNET40`. 




### Registratori telematici
A far data **04/10/2021** i registratori telematici ( stampanti RT ) certificati con la SCO NCR sono:

* Epson - tutti i modelli con le seguenti versioni firmware/driver 
    * Firmware `11.02 build 255` oppure `7.02 build 255`
    * Driver **OPOS** Epson, versione `V22`. 

> Il funzionamento ed il supporto con stampanti non certificate non è previsto.




### Hardware SCO
La versione HW SCO certificata è la **`R5.0 with EUR recycler`**
> Il funzionamento della soluzione ed il supporto a versioni non certificate non è previsto.







## Funzionalità supportate

> Eventuali funzionalità non riportate in elenco sono da considerarsi automaticamente non supportate / fuori scenario.

#### Funzionalità di Attendente/Supervisore

|Funzione|Supportato|Note|
| :- | :-: |----|
|Validazione supervisore|SI|
|Avvio autonomo alla partenza della SCO|SI|
|Shutdown automatico allo spegnimento della SCO|SI|
|Apertura e chisura della cassa|SI|
|Login da RAP|SI|
|Logout da RAP|SI|
|Modalità supervisore|SI|
|Modalità funzionamento ridotto (degraded)|SI|Previsto funzionamento solo contanti o solo pagamenti EFT
|Modalità Training|**No / Non applicabile**|Impostando Posware in modalità *senza stampante* (`dummy`) si possono comunque effettuare tutte le prove di funzionamento senza causare l'emissione di scontrini/documenti commerciali.
|Cash Management|SI|
|Avvio Funzioni di utilità e LaunchPad NCR|SI|
|Invio segnale di sincronizzazione alla SCO (Heartbeat)|SI|
|Stampa Report Fiscali|SI|Solo tramite file generato dalla SCO
|Stampa Report supervisore|SI|Solo tramite file generato dalla SCO
|Stampa report CashManagement|SI|Solo tramite file generato dalla SCO

#### Funzionalità in fase di vendita

|Funzione|Supportato|Note|
| :- | :-: |---|
|Avvio di una transazione|SI|
|Vendita ordinaria di un prodotto|SI|
|Visualizzazione degli sconti su prodotto ( % e valore) nella ricevuta digitale|SI|Limitati a sconto % su prodotto e sconto a valore/taglio prezzo su prodotto
|Scelta prodotto da list di lookup (PickList)|SI|
|Vendita di un prodotto digitando il codice articolo|SI|
|Aggiunta Loyalty Card alla transazione (via scanner o codice digitato)|SI|
|Loyalty Card bloccata / non riconosciuta|SI|
|Staff Discount Acknowledgement|SI|
|Gestione prodotto con restrizioni|SI|Restrizioni per età, prodotto leggero/pesante, prodotto da non pesare, prodotto da non inserire nello shopper
|Gestione prodotto non riconosciuto|SI|
|Vendità prodotto con limite temporale (Time Restricted Item)|**NO**|
|Vendita prodotto con prezzo su richiesta|SI|
|Vendita prodotto con quantità su richiesta|SI|
|Vendita prodotto con limite su quantità|**NO**|
|Vendita prodotto previa verifica visiva del supervisore|SI|
|Vendita prodotto con pesatura preventiva (Weight Required Item)|**NO**|
|Vendita prodotti collegati (Linked Item)|SI|Supportati solo tramite funzionalità Kit di Posware
|Segnalazione articolo non vendibile a causa di errori di codifica|SI|
|Supporto Coupon Sconto|SI|Solo durante la fase di vendita (itemization)
|Supporto Coupon Pagamento|SI|Solo durante la fase di pagamento
|Verifica Coupon non valido|SI|Presente nella fase di vendita (itemization) e di pagamento
|Aggiunta di un prodotto al passaggio di un coupon|**NO**| *Non esiste una funzionalità corrispondente in Posware*
|Richiesta addebito busta|SI|Singolo codice con singolo prezzo supportato.
|Vendita prodotto a lettura di peso/prezzo|SI|

#### Funzioni di assistenza al cliente come attendente/supervisore
|Funzione|Supportato|Note|
| :- | :-: |----|
|Storno singolo articolo|SI|
|Annulla transazione/Storna tutti gli articoli|SI|
|Gestione errori periferiche di stampa|SI|Sia in fase di avvio della SCO che in fase di stampa della ricevuta (scontrino)
|Sospensione della transazione|SI|
|Cambio prezzo articolo dalla SCO|**NO**|
|Vendita a reparto dalla SCO|**NO**|

#### Funzioni di sicurezza
|Funzione|Supportato|Note|
| :- | :-: |----|
|Notifica peso non coincidente|SI|Dedotto automaticamente dall'HW della SCO oppure fornendo i dati nell'anagrafica dei prodotti nell'archivio della SCO
|Prodotto/Peso aggiunto nella busta/bilancia|SI|
|Uso busta del consumatore|SI|
|Prodotto/Peso rimosso dalla busta/bilancia|SI|
|Non imbustare il prodotto|SI|

#### Funzioni di pagamento
|Funzione|Supportato|Note|
| :- | :-: |----|
|Paga in contanti|SI|
|Paga con POS EFT|SI|Supportati unicamente i provider protocollo Argentea e Scambio Importo 17
|Paga con Gift Card|SI|Supportata solo la Gift Card nativa di Posware
|Supporto ai pagamenti parziali|SI|
|Supporto all'arrotondamento contante|SI|Supportati sia tramite registratore telematico che tramite Posware
|Supporto inserimento codice Lotteria Scontrini|SI|* Necessario aggiornamento di NCR rispetto alla versione certificata

##### Note su *Lotteria degli scontrini*
Dalla versione **2.0.0.14** dell'adattatore Posware, è possibile inserire il codice lotteria tramite SCO. E' abilitato l'inserimento sia manuale che tramite scanner. <br>
Il pulsante `Codice Lotteria` appare nella schermata dei pagamenti (in base alla configurazione del cliente potrebbe essere altrove).

> E' necessario un adeguamento del software SCO per poter utilizzare la feature. Contattare NCR per gli adeguamenti necessari.
