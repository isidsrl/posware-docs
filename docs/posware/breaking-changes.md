---
tags:
    - Posware
    - Posware Frontend
    - Breaking changes
---

# Posware - Breaking Changes

**Prima revisione documento: 27 agosto 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Versione di riferimento

- Posware :material-tag:`4.3`

## Breaking changes

Vengono elencati di seguito i diversi **breaking changes** introdotti in *Posware Frontend* a partire dalla versione :material-tag:`4.3` dell'applicativo:

=== "Tracciato Log"

    - Nel record **01** , il barcode di un prodotto viene ora riportato unicamente con gli spazi vuoti (formato denominato **Barcode No Food** nella :material-tag:`4.2`). Non è più possibile usare il formato numerico. **Barcode No Food** è da considerarsi nuovo standard.
    
=== "File di configurazione in cassa"
    - A partire dalla versione :material-tag:`4.3`, i file di configurazione di Posware, delle *Connection strings* e delle impostazioni utente sono stati riportati allo standard .NET rimuovendo l'approccio custom.<br> Per maggiori informazioni consultare [Aggiornamento dei file di configurazione](../getting-started/installazione-posware-frontend/setup-posware.md#aggiornamento-dei-file-di-configurazione) di *SetupPosware*.

=== "MySql"
    !!! warning "Nuova versione di MySQL richiesta nelle nuove installazioni"
        A partire dalla versione :material-tag:`4.3`, le nuove installazioni che **non migrano** dalla :material-tag:`4.2`, dovranno usare **MySQL** :material-tag:**`5.6`**.

=== "Campi tabelle"
    Il campo `indirizzo` della tabella `config_log` è ora un varchar 255 che può contenere l'endpoint dello StoreServer e non più unicamente indirizzi IP.

=== "Configurazioni"
    !!! warning "Nuova configurazione per le stampanti simulate"
        A partire dalla versione :material-tag:`4.3`, in caso sia necessario configurare una stampante simulata per eseguire delle prove, il campo `tipo_fiscalprinter` della tabella `config_cassa` deve essere impostato pari a "**DUMMY**" invece di "**WINCOR**", il valore che era utilizzato in precedenza per tale scopo.
    - Per il motivo elencato in tracciato log, l'impostazione **Barcode No Food** nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata ad **1**).
    - L'impostazione `keylock` nella tabella `config_cassa` non ha più alcun effetto (come se fosse sempre impostata a ***stringa vuota***).
    - L'impostazione `keyboard` nella tabella `config_cassa` non ha più alcun effetto (come se fosse sempre impostata a ***stringa vuota***).
    - L'impostazione `ToneIndicator` nella tabella `config_cassa` non ha più alcun effetto (come se fosse sempre impostata a ***stringa vuota***).
    - L'impostazione `PosNET` nella tabella `config_cassa` non ha più alcun effetto (come se fosse sempre impostata a ***0***).
    - L'impostazione `DataInizioModRT` nella tabella `config_cassa` non ha più alcun effetto (come se fosse sempre impostata a***0***).
    - L'impostazione `CaricaTabIVA` nella tabella `config_cassa` non ha più alcun effetto (come se fosse sempre impostata ad ***1***).

## Feature rimosse in Posware :material-tag:`4.3`

Nell'elenco sottostante vengono illustrate tutte le **feature di *Posware Frontend* che sono state rimosse** a partire dalla versione :material-tag:`4.3` dell'applicativo:

- Supporto alle stampanti IBM (RFImbopos e le sue dipendenze)
- Supporto alle stampanti NCR vecchie MF FASY, RealPrint e RealPOS
- Supporto alle stampanti Wincor TH230 e vecchie Wincor MF
- Supporto ai Controlli OCX e sue dipendenze che venivano usate per mostrare la telecamera. Tali tecnologie risalenti a Windows XP non sono più supportate a partire da Windows Vista.
- Il vecchio interfacciamento WinEPTS :material-tag:`2.x` è stato rimosso e sostituito con un nuovo interfacciamento.

## Feature ed altri elementi deprecati che verranno rimosse in Posware :material-tag:`4.4`

Le seguenti **feature di *Posware Frontend* sono deprecate** a partire dalla versione :material-tag:`4.3` dell'applicativo e verranno quindi poi rimosse in Posware :material-tag:`4.4`:

=== "Feature"
    - Gestione commissione di vendita
    - Supporto alla reportistica legacy (Crystal Report, Microsoft Report)
    - Nota di credito di Posware :material-tag:`4.2` e relative customizzazioni
    - Integrazione Satispay Wally (:material-tag:`4.2`) basata su SatispayBusinessNetSDK2, sostituita dalla nuova integrazione nativa REST che include anche il supporto ai Buoni Pasto e Fringe Benefit
    - Integrazione di GetYourBill
    - La funzionalità di nota di credito. Viene sostituita con una nuova feature che include il supporto a note di credito multiple
    - Le strutture carscar*

=== "Configurazioni"
    - L'impostazione `stampa_differita` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***true***).
    - L'impostazione `touch` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***true***).
    - L'impostazione `display`operatore* nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***stringa vuota***).
    - L'impostazione `esiste_disp_operatore` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***stringa vuota***).
    - L'impostazione `fatturasustampanteF` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***0***).
    - L'impostazione `anteprima` fattura nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***0***).
    - L'impostazione `visSceltaIvaReso` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***0***).
    - L'impostazione `numerocopiefattura` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***0***).
    - L'impostazione `EanPagamentoBollette` nella tabella `config_cassa`. Il campo sarà migrato in tabparametriextra.
    - L'impostazione `scontiManReso` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***0***).
    - L'impostazione `DoppioInvioAbilitato` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***0***).
    - L'impostazione `BloccoCartainEsaurimento` nella tabella `config_cassa` non avrà più alcun effetto (come se fosse sempre impostata a ***0***).
