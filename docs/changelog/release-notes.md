# Posware - Release Notes
#### Raccolta di release notes

---
> ## Versioni software di riferimento
>
>
> - Posware **4.2** o superiore
> - Posware per Windows XP **4.2.x** o inferiore. Supporto terminato il **31/12/2021**.
>

     

## Current Release notes
##### Release `4.2.694.0` del 14/09/2023
### What's New
- Aggiunto il supporto nativo per il CashManagement Cashmatic













### Bug fixes and issued adressed
- Corretta la vendita kit in promozione


















### Breaking Changes
 








### Known issues









---

---
# Release Notes History
## 4.2.638.0 del 24/07/2023
### What's New
- Cambiato algoritmo di calcolo del massimo pagabile per forma di pagamento che ora tiene conto dell'imponibile e non degli articoli
### Bug fixes and issued adressed
### Breaking Changes
- In caso di pagamento eft=true e pagtipo=0 posware non permette più l'imputazione dell'importo manuale, passerà al terminale sempre il massimo pagabile per la forma di pagamento scelta
## 4.2.565.0 del 08/05/2023
### What's New
- Aggiunto il nome dell'operatore in fase di visualizzazione schermata richiesta password operatore l'operatore 
- Implementato un nuovo comportamento per verifica check-out, in caso di parametro 3 nei campi `CompEtichPesoSupBil` e/o `CompEtichPesoInfBil` di `config_cassa`, viene sempre bloccata la vendita.
- Implementata la stampa messaggio di fine scontrino e messaggi di trailer in stampa scontrino di cortesia
- Implementata la stampa delle informazioni aggiuntive su articolo se abilitato il flag di stampa scontrino.
### Bug fixes and issued adressed
- In caso di passaggio multiplo di tessera viene corretta la stampa su scontrino delle informazioni tessera.
- In caso di fidelity card bloccata il messaggio viene enfatizzato con un pop-up bloccante. <br>
Viene inserito lo stato tessera anche nel file LOG nel record 06 in posizione 52.1 che rispecchia lo stato della carta.
- Corretto il timeout della funzione prelievo/versamento su Fast4Cash
- In caso di gift card privativa posware è stato introdotto il controllo sul seriale per evitare vendite multiple della stessa codeline.
- Corretta in fase di ripresa sospesi la valorizzazione dei campi prezzoUnitario, di riga e quantità su prodotti a prezzo variabile
### Breaking Changes
- La versione 4.2 non supporta il driver PosForNet ISID per il Registratore Telematico RT-ONE. È supportato solo l'OPOS ufficiale Diebold/Nixdorf
## 4.2.401 del 25/11/2022
### What's New
- Implementato un controllo sull'imputazione del prezzo di vendita di un articolo con varia prezzo arrotondando comunque a due cifre decimali
- Implementato il beep in caso di tessera fidelity sconosciuta

### Bug fixes and issued adressed
- Corretto lo storno ultimo articolo in caso di prodotto a peso variabile.
- Corretto un bug che impediva l'applicazione di qualsiasi promozione in caso di tessera bloccata.
- Corretta l'impaginazione della stampa scontrino di cortesia

## 4.2.238 del 15/06/2022
### What's New
- Aggiunta la possibilità di impedire l'inserimento di un prodotto nella transazione tramite la digitazione del codice interno. È possibile impostare a True/False l'impostazione `enableUserToInputProductCode` del modulo `posware.instance` in tabella `tabparametriextra` - Valore di default **True** . Impostando il valore a **False** l'operatore potrà usare solamente barcode.
- Introdotti alcuni miglioramenti minori al workflow di emissione fatture come documenti non gestionali.
### Bug fixes and issued adressed
- Aggiornato il supporto al firmware build 258 per le stampanti Epson ripristinando la possibilità di emettere coupon residui con la forma di pagamento *Sconto a pagare*
- Corretto il funzionamento della nota di credito in presenza di prodotti kit all'interno della transazione
- Corretta la scrittura della tabella ristampa_scontrini che in alcune condizioni proponeva aliquote iva errate
### Breaking Changes
- La versione 4.2 non supporta il driver PosForNet ISID per il Registratore Telematico RT-ONE. È supportato solo l'OPOS ufficiale Diebold/Nixdorf
## 4.2.128.0 del 25/02/2022
### What's New
- Nuovo PlugIn disponibile. Aggiunto il supporto all'attività promozionale [NoiAmiamoLaScuola](https://noiamiamolascuola.it/) del GruppoVeGe.
- Aggiunto nel log delle transazioni, nel tipo record 10 - vendita  KIT, l'informazione del codice a barre dell'articolo KIT (padre)
- Aggiunto al Server RT 2.0 Epson il controllo di congruità sui codici Ateco configurati.
- Aggiornato il supporto alle funzionalità PinDispatching di Argentea.
### Bug fixes and issued adressed
- Nella richiesta di nota di credito viene ora differenziato il messaggio di transazione non trovata e di transazione già sottoposta a nota di credito precedente.
- In caso di storno di un prodotto collegato veniva stornata una quantità errata.
- A seguito dell'emissione di un buono reso nella procedura di nota credito, veniva stampato un ulteriore documento gestionale poi annullato.
### Breaking Changes
- La versione 4.2 non supporta il driver PosForNet ISID per il Registratore Telematico RT-ONE. È supportato solo l'OPOS ufficiale Diebold/Nixdorf
## 4.2.23.0 del 12/11/2021
### What's New
- Aggiunto il supporto alle seguenti feature del Registratore Telematico 2.0: Arrotondamento, Vendita di servizi, Gestione Multi-Ateco, Pagamento non riscosso segue fattura, Sconto a pagare. <br>
Le feature non sono disponibili su tutti i registratori telematici. <br>
A seguito dell'upgrade dalla 4.1 alla 4.2 verrà automaticamente effettuata una chiusura fiscale.
> Leggere attentamente il documento <mark>Posware 4.2 - Funzionalità RT 2.0</mark> per tutti gli approfondimenti
### Breaking Changes
- La versione 4.2 non supporta il driver PosForNet ISID per il Registratore Telematico RT-ONE. È supportato solo l'OPOS ufficiale Diebold/Nixdorf
## 4.1.544 del 28/06/2021
### What's New
- Aggiunta nel report dei coupon pagamenti a fine transazione, la stampa dei codici coupon che verranno generati come conseguenza di un utilizzo parziale di un precedente coupon pagamento.
### Bug fixes and issued adressed
- Ripristinato il corretto funzionamento del touch scroll nella lista di selezione dei prodotti usata nella nota di credito e nella finestra di prenotazione dei regali loyalty 
## 4.1.525 del 09/06/2021
### What's New
- Aggiunto nel **log cassa** le informazioni relative alla *lotteria degli scontrini* tramite record 14 - Info Aggiuntive
- La **richiesta remota** verso la cassa con di tipo *VE - Versione*, restituisce ora la **File version** anzichè la Product version, in linea con l'adozione del semantic versioning.
### Bug fixes and issued adressed
- [*Posware per Windows XP*] Ripristinato il funzionamento del comando lotteria degli scontrini. Versioni di Windows >= 7 non erano affette dal bug

## 4.1.516 del 31/05/2021
### What's New
- Aggiunta la possibilità di stampare a fine transazione il report dei coupon pagamenti generati in una nota di credito o usati durante una normale transazione di vendita. E' possibile abilitare i report su `tabparametriextra` tramite le impostazioni:
    - `posware.printsettings` - `printCouponCodesGeneratedReportInCreditNote` - true/false
    - `posware.printsettings` - `printPaymentCouponCodesUsedReportInTransaction` - true/false
- Aggiunto al plug-in di Cash Management Azkoyen il supporto al doppio monitor: configurando su `tabparametriextra` le impostazioni:
     - `enableDoubleMonitor` - true/false
     - `startXPosition`
     - `startYPosition`
     - `topMostScreen` - true/false <br>
    è possibile configurarne il comportamento
- Aggiunto il plug-in di supporto al driver di Cash Management Fast4Cash. Sono supportati i device NCR CTM e Glory CI10. Per approfondimenti consultare il documento dedicato **PlugIn Fast4Cash** (cartella docs)
- Aggiornato il plug-in di gestione **Argentea** con supporto alla libreria `v3.9.7.7`. **Aggiornamento necessario, vedi Breaking Changes.**
- Aggiunte nuove configurazioni per **la validazione del codice lotteria**. Consultare il documento dedicato nella cartella docs.

### Bug fixes and issued adressed
- [*Posware per Windows XP*] Ripristinato il comportamento dei pulsanti da tastiera fisica Su/Giù
- Corretta la lettura del valore decimale dei ticket numerati su alcuni ambienti Windows con internazionale non italiana
- In caso di errore di rete durante la sospensione dello scontrino sul server di punto vendita, viene ora mostrato un messaggio di errore Popup come avviso bloccante
- Aggiornata la funzione di lettura del corrispettivo fiscale su stampante RT-One
- Corretto lo storno di un prodotto a peso variabile presente in multiple battute nello scontrino.

### Breaking Changes
- A partire da questa versione, i pagamenti elettronici di tipo EFT (qualiasi protocollo), Satispay, BPE, BP Online **ignoreranno la configurazione con il parametro `resto` = true** (errata). Questi tipi di pagamenti non supportano il resto in nessun caso (by design).
- La versione minima certificata della libreria di interfacciamento **Argentea** (Monetica) è la `v3.9.7.7` . Prima di installare questa versione è assolutamente necessario aggiornare la libreria Argentea. In caso di versione installata inferiore, verrà mostrato un avviso non bloccante in fase di inizializzazione. **In tal caso il funzionamento non è garantito ne lo scenario supportato.** Per installare l'aggiornamento contattare Argentea.
- Per installare questa versione sulle **SCO (NCR/Diebold)** la versione minima richiesta dei rispettivi adattatori è la `v2.0.0.17`. Non funziona con le versioni precedenti degli adattatori, **aggiornare in concomitanza entrambi i software.**

## 4.1.463 del 08/04/2021
### What's New
- Separata la gestione dei pagamenti con PagRt=2 tra 'Ticket' e 'Non Riscosso' <br>
Indicando nella descrizione pagamento la parola *ticket*, su stampante fiscale sarà movimentata la parte **Ticket**, diversamente quella **Non riscosso**. <br>
La stampa della descrizione del pagamento nell'appendice non subisce variazioni.

### Bug fixes and issued adressed
- Corretta l'impaginazione della stampa prelievi/versamenti su stampante RTONE

### Breaking Changes
- Fare riferimento alla versione 4.1.300/344/373

### Known issues
- Fare riferimento alla versione 4.1.300/344/373

## 4.1.447 del 23/03/2021
### What's New
- Aggiunto il supporto iniziale ad XML7 per il Server RT Epson.
- Aggiunto il supporto alle stampe reportistiche della SCO NCR sulla stampante NCR 130A con firmware XML7

### Bug fixes and issued adressed
- Apportati alcuni cambiamenti per impedire alla stampante RT-One con il driver OPOS Diebold di annullare randomicamente la transazione.
- Allineati alcuni comportamenti grafici del nuovo PluContainer2 con il precedente

### Breaking Changes
- Fare riferimento alla versione 4.1.300/344/373

### Known issues
- Fare riferimento alla versione 4.1.300/344/373

## 4.1.432 del 08/03/2021
### What's New
- Aggiunto il supporto alla cancellazione nel meccanismo di retry stampa sulla SCO NCR. In caso di errore in emissione scontrino,  qualora l'utente dovesse premere il tasto `Annulla/Cancel`, la transazione verrà annullata. E' necessario aggiornare l'adattatore alla versione `2.0.0.16`.
- Aggiunto il supporto XML7 Nota credito sulle SCO Diebold ed NCR. E' ora possibile, da una cassa posware standard, emettere una nota di credito originariamente generata da una  SCO. E' necessario aggiornare gli adattatori alla versione `2.0.0.16`
- Aggiunto per le stampanti EPSON il supporto al firmware `7.01/11.01 build **249**`. E' garantita la retrocompatibilità con i precedenti.

### Bug fixes and issued adressed
- Corretto un problema che impediva l'aggiornamento automatico dell'intestazione dello scontrino su stampanti EPSON se l'opzione *Carica Logo* era attivata.
- Corretto l'errore OPOS 207 che poteva verificarsi in alcuni casi di emissione nota credito con le stampanti EPSON. La risoluzione è **legata al supporto firmware build 249.**

### Breaking Changes
- Fare riferimento alla versione 4.1.300/344/373

### Known issues
- Fare riferimento alla versione 4.1.300/344/373

## 4.1.418 del 22/02/2021
### What's New
- Aggiunto il supporto XML 7 per la stampante NCR. Consultare i cambiamenti introdotti nel documento apposito **Adeguamenti RT XML 7**
- Aggiunte ulteriori note sul workflow dei *ticket elettronici e dematerializzati numerati* nel documento **Adeguamenti RT XML 7**

### Bug fixes and issued adressed
- MOTORE PROMO: in alcune promo composte su scontrino, è stato riallineato il comportamento del motore rispetto alla versione 4.0.x con runtime v3.5

### Breaking Changes
- Fare riferimento alla versione 4.1.300/344/373

### Known issues
- Fare riferimento alla versione 4.1.300/344/373

## 4.1.404 del 08/02/2021

### What's New
- Aggiunto il supporto alla lotteria degli scontrini per le Self-Checkout Diebold. Consultare i cambiamenti introdotti nel documento apposito **Posware 4.1 Adapter Self-CheckOut Diebold** (cartella docs)
- Aggiunto il supporto ai ticket numerati XML 7 per i *ticket elettronici e dematerializzati*. Consultare i cambiamenti introdotti nel documento apposito **Adeguamenti RT XML 7**
- Quando un Codice Lotteria viene applicato alla transazione, di default viene ora visualizzato sul display cliente. Consultare il documento **Lotteria degli scontrini** per i dettagli su come disabilitare questo comportamento.
- Migliorata l'accuratezza dei messaggi sui prodotti con restrizioni all'interno dell'adattatore SCO NCR.
- Aggiunta sulla stampante RT-One la possibilità di far inviare al frontend la tabella dei pagamenti sovrascrivendo quella presente nella stampante. Per abilitare la feature impostare a `true` il parametro `sendPayment` del modulo `posware.fiscalprinter.rtone` in `tabparametriextra`. Di default è `false`

### Bug fixes and issued adressed
- Corretto l'errato inserimento in tracciato XML7 che poteva accadere in alcuni casi di rettifica della quantità dei ticket manuali calcolati tramite la funzionalità di calcolo automatico
- E' stata resa omogenea la visualizzazione, con / senza decimali, nel display operatore, di alcuni messaggi contenenti la quantità degli articoli in transazione
- Corretto nella funzionalità Nota Credito, un problema di mancato ripristino di un prodotto a cui vi era stato in origine applicato un *varia prezzo*

### Breaking Changes
- Fare riferimento alla versione 4.1.300/344/373

### Known issues
- Fare riferimento alla versione 4.1.300/344/373

## 4.1.387 del 25/01/2021
### What's New
- Aggiunto il supporto alla lotteria degli scontrini per le Self-Checkout NCR. Consultare i cambiamenti introdotti nel documento apposito **Posware 4.1 Adapter Self-CheckOut NCR** (cartella docs)
- Migliorata la gestione del PluContainer e qualità visita delle tab  (solo versione .net 4.8)

### Bug fixes and issued adressed
- Sostituita la generazione legacy dei barcode sui report Crystal Report. Sarà necessario adeguare i report esistenti al nuovo formato

### Breaking Changes
- Fare riferimento alla versione 4.1.300/344/373
- Consultare i cambiamenti introdotti da XML nel documento apposito **Adeguamenti RT XML 7**

### Known issues
- Fare riferimento alla versione 4.1.300/344/373

## 4.1.373 del 08/01/2020

### What's New
- Aggiunto il supporto XML 7 per stampanti EPSON ed RTONE.
- Aggiunto il supporto ai ticket numerati previsto da XML 7
- Aggiunto il supporto alle nuove specifiche XML 7 per i documenti di nota credito 
- Aggiunto nel log il doppio formato di versione prodotto. Vengono ora registrati sia la Product Version (4.1) che la File Version (4.1.373.0). Le nuove stringhe sostituiscono le precedenti nella stessa posizione.
- Aggiunto il supporto al periodo di inattività RT anche in ambiente SCO
- Aggiunto il supporto al runtime 4/4.8 anche in ambiente SCO
- Migliorata la gestione del sensore di rilevamento carta su stampante NCR
- Nuovo formato Change Log (questo file)
- Aggiunta la documentazione nella sotto cartella ***docs*** dove viene installato posware. In questa cartella verrà sempre inserita  la documentazione aggiornata.





### Bug fixes and issued adressed
- Corretto un bug nello svecchiamento listini dove alcuni record con data di inizio pari a quella di svecchiamento venivano eliminati anche non avendo altri record disponibili
- Migliorato il supporto al runtime Crystal Report su configurazioni x64 e sistema operativo Windows 10. Per attivarlo consultare il documento aggiornato *Posware 4.1 - .NET 4.x Runtime Notes* in merito al flag `useLegacyV2RuntimeActivationPolicy`
- Migliorato il sistema di notifica dei Plugin legacy. Se un plugin non viene trovato, viene riportato un DEBUG nel file di log.
- Corretto errato binding dei campi nella gestione di commissione di vendita 
- Ripristinata la dimensione delle colonne relative alle richieste in griglia. Es.: scelta causale della nota di credito ed altre
- In assenza del plugin di Crystal Report, il messaggio generico di operazione annullata è stato sostituito con uno di errore specifico













### Breaking Changes
- Fare riferimento alla versione 4.1.300/344
- Consultare i cambiamenti introdotti da XML nel documento apposito **Adeguamenti RT XML 7**







### Known issues
- Fare riferimento alla versione 4.1.300/344

## 4.1.300-4.1.344
### What's New
- Nuovo runtime basato su .NET Framework 4.8
- Aggiunto il supporto alla lotteria degli scontrini












### Bug fixes and issued adressed
### Breaking Changes
- Termine del supporto delle tastiere fisiche
- Termine del supporto ai sistemi Windows XP based

> Consultare il documento *Posware 4.1 - .NET 4.x Runtime Notes* (**Posware NET4.md.html**) per approfondimenti




### Known issues
- Posware termina in modo inaspettato dopo aver effettuato la chiusura fiscale
- Posware restituisce un errore di tipo System.AccessViolationException

> Consultare il documento *Posware 4.1 - .NET 4.x Runtime Notes* (**Posware NET4.md.html**) per approfondimenti sulla risoluzione