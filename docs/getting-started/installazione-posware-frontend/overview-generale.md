---
tags:
    - Posware
    - Posware Frontend
---

# Overview generale

**Prima revisione documento: 27 agosto 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Cenni preliminari
Il presente documento descrive i nuovi requisiti minimi e le procedure necessarie all'installazione o all'aggiornamento di *Posware Frontend* alla versione :material-tag:`4.3` dell'applicativo, così da poter associare le casse con lo *StoreServer*, il nuovo sistema centralizzato per la gestione e l'analisi del punto vendita.

## Requisiti minimi

### Hardware
- Un processore a **64bit** compatibile con il sistema operativo installato sulla cassa
- 8 Gb di RAM
    - 4 Gb di RAM sono utilizzabili solo se il Sistema Operativo è Windows 7
- Disco rigido SSD (tipo SATA o NVME) da almeno 128 Gb
- Connessione ad Internet per l'installazione delle dipendenze

### Software
!!! warning "Obbligo di tutte le dipendenze in Posware :material-tag:`4.3`"
    Le dipendenze soprastanti sono da considerarsi **TUTTE** obbligatorie per poter utilizzare Posware :material-tag:`4.3`.

- Windows 7 **SP1** con update **Servicing Stack 2018** o superiore, oppure
- Windows 10 :material-tag:`1809` **Redstone 5 Build 17763** o superiore, oppure
- Windows 11
- .NET Framework :material-tag:`4.8`
- MySQL :material-tag:`4.1` o :material-tag:`5.6` in caso di migrazione da *Posware Frontend* :material-tag:`4.2`, oppure
- MySQL :material-tag:`5.6` in caso di nuova installazione da zero
- Dipendenze autoinstallanti tramite *SetupPosware*:
    - WebView2 Edge Runtime :material-tag:`109.0.1518.140`
    - .NET :material-tag:`6` Hosting Bundle
    - Monroes OPOS Common Control Objects :material-tag:`1.14.001`
    - Pos_SBV Client :material-tag:`1.1.0` (per la ricezione variazioni)

Si prega di consultare attentamente le note varie sulle dipendenze di terze parti e gli aggiornamenti necessari nella relativa [sezione](./dipendenze-aggiornamenti-necessari.md) prima di procedere con l'installazione o l'aggiornamento di *Posware Frontend*. 

## Informazioni, credenziali e prassi comuni
#### Impostazioni comunemente usate per il database
- **Utente:** root
- **Password:** vbhg4132
- **Nome del database:** `cassa`

#### Impostazioni comunemente usate per Posware
- **Directory di default di Posware su sistemi 64bit:** *%PROGRAMFILES(x86)%\Posware*
- **Directory di default di Posware su sistemi 32bit:** *%PROGRAMFILES%\Posware*
- **Directory di default per gli aggiornamenti automatici di *PosUpdate*:** *C:\PosAgg*

#### Porte comunemente usate dall'ecosistema Posware
- **Pos_Fidelity:** 6850
- **Pos_Log:** 6951 (in precedenza era 6851)
- **StoreServer:** 6851
- **Pos_SBV:** 
    - **Porta comandi:** 6880 (in entrambe le versioni, sia cassa che server)
    - **Variazioni cassa:** 6852
    - **Variazioni server:** 6853
- **Posware Frontend:**
    - **Porte di comunicazione messenger:** 6855 e 6856

### Prassi comuni
!!! warning "Firewall di Windows sui terminali casse"
    **È assolutamente necessario disattivare il firewall di Windows sui terminali casse.**

## Breaking changes
Vengono elencati di seguito i diversi **breaking changes** introdotti in *Posware Frontend* a partire dalla versione :material-tag:`4.3` dell'applicativo:

- Nel record **01** , il barcode di un prodotto viene ora riportato unicamente con gli spazi vuoti (formato denominato **Barcode No Food** nella :material-tag:`4.2`). Non è più possibile usare il formato numerico. **Barcode No Food** è da considerarsi nuovo standard.
- Per il motivo appena elencato, l'impostazione **Barcode No Food** nella tabella `config_cassa` viene meno (come se fosse sempre impostata ad **1**).
- A partire dalla versione :material-tag:`4.3`, i file di configurazione di Posware, delle *Connection strings* e delle impostazioni utente sono stati riportati allo standard .NET rimuovendo l'approccio custom. Per maggiori informazioni consultare la relativa [sezione](./setup-posware.md#aggiornamento-dei-file-di-configurazione) del *SetupPosware*.

!!! warning "Nuova versione di MySQL richiesta nelle nuove installazioni"
    A partire dalla versione :material-tag:`4.3`, le nuove installazioni che **non migrano** dalla :material-tag:`4.2`, dovranno usare **MySQL** :material-tag:**`5.6`**. 

## Feature rimosse in Posware :material-tag:`4.3`
Nell'elenco sottostante vengono illustrate tutte le **feature di *Posware Frontend* che sono state rimosse** a partire dalla versione :material-tag:`4.3` dell'applicativo:

- Supporto alle stampanti IBM (RFImbopos e le sue dipendenze)
- Supporto alle stampanti NCR vecchie MF FASY, RealPrint e RealPOS
- Supporto alle stampanti Wincor TH230 e vecchie Wincor MF
- Supporto ai Controlli OCX e sue dipendenze che venivano usate per mostrare la telecamera. Tali tecnologie risalenti a Windows XP non sono più supportate a partire da Windows Vista.
- Il vecchio interfacciamento WinEPTS :material-tag:`2.x` è stato rimosso e sostituito con un nuovo interfacciamento.

## Feature deprecate che verranno rimosse in Posware :material-tag:`4.4`
Le seguenti **feature di *Posware Frontend* sono deprecate** a partire dalla versione :material-tag:`4.3` dell'applicativo e verranno quindi poi rimosse in Posware :material-tag:`4.4`:

- Gestione commissione di vendita
- Supporto alla reportistica legacy (Crystal Report, Microsoft Report)
- Nota di credito di Posware :material-tag:`4.2` e relative customizzazioni
- Integrazione Satispay Wally (:material-tag:`4.2`), sostituita dalla nuova integrazione nativa REST
- Integrazione di GetYourBill

## Installazione e aggiornamenti
Sono previsti questi due applicativi per le procedure di installazione e di aggiornamento di *Posware Frontend*:

- [SetupPosware](./setup-posware.md)
- [PosUpdate](./pos-update.md)

In particolare, *SetupPosware* è il nuovo setup riprogettato per l'uscita di Posware :material-tag:`4.3` .

Esso può essere usato esclusivamente per le casistiche di:

- Nuova installazione di *Posware Frontend* sui terminali casse
- Migrazione da Posware :material-tag:`4.2.x`

*PosUpdate* è invece l'applicativo da usare esclusivamente per aggiornare *Posware Frontend* sui terminali casse alla loro ultima **Build** version, in linea con la **Major** e **Minor** version installata (Es.: da :material-tag:`4.3.96` a :material-tag:`4.3.154`).

I dettagli specifici sul funzionamento dei due applicativi sono consultabili nelle rispettive sezioni.