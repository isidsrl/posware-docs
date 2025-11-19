---
tags:
    - Posware
    - Posware Frontend
---

# Posware - Overview generale

**Prima revisione documento: 27 agosto 2024** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Cenni preliminari
Il presente documento descrive i nuovi requisiti minimi e le procedure necessarie all'installazione o all'aggiornamento di *Posware Frontend* alla versione :material-tag:`4.3` dell'applicativo, così da poter associare le casse con lo *StoreServer*, il nuovo sistema centralizzato per la gestione e l'analisi del punto vendita.

!!! warning "Ordine di installazione obbligatorio"
    Negli scenari standard di installazione ex novo e migrazione (si esclude quindi lo scenario ibrido), è necessario installare Posware :material-tag:`4.3` prima lato **cassa** e poi sullo **StoreServer**.

## Requisiti minimi

### Hardware
- 8 Gb di RAM
    - 4 Gb di RAM sono utilizzabili solo se il Sistema Operativo è Windows 7
- Disco rigido SSD (tipo SATA o NVME) da almeno 128 Gb
- Connessione ad Internet per l'installazione delle dipendenze

### Software
!!! warning "Obbligo di tutte le dipendenze in Posware :material-tag:`4.3`"
    Le dipendenze elencate di seguito sono da considerarsi **tutte obbligatorie** per poter utilizzare Posware :material-tag:`4.3`.

=== "Windows"
    - Windows 7 **SP1** con update **Servicing Stack 2018** o superiore, oppure
    - Windows 10 :material-tag:`1809` **Redstone 5 Build 17763** o superiore, oppure
    - Windows 11

=== ".NET"
    - .NET Framework :material-tag:`4.8` . [Vedi tabella di riferimento](../../posware/dot-net-table-reference.md).

=== "MySql"
    - MySQL :material-tag:`4.1` o :material-tag:`5.6` in caso di migrazione da *Posware Frontend* :material-tag:`4.2`, oppure
    - MySQL :material-tag:`5.6` in caso di nuova installazione da zero

=== "Dipendenze autoinstallanti tramite *SetupPosware*"
    **Le seguenti dipendenze vengono installate automaticamente dal pacchetto di installazione.** <br>
    Vengono qui riportate per completezza e conoscenza del prodotto.

    - WebView2 Edge Runtime :material-tag:`109.0.1518.140`
    - .NET :material-tag:`6` Hosting Bundle
    - Monroes OPOS Common Control Objects :material-tag:`1.14.001`
    - Pos_Sbv Client :material-tag:`1.1.0` (per la ricezione variazioni)

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

## Breaking changes, feature rimosse e deprecate

L'elenco può essere consultato a [questo documento](../../posware/breaking-changes.md).

## Installazione e aggiornamenti

Sono previsti questi due applicativi per le procedure di installazione e di aggiornamento di *Posware Frontend*:

- [SetupPosware](./setup-posware.md)
- [PosUpdate](./pos-update.md)

In particolare, *SetupPosware* è il nuovo setup riprogettato per l'uscita di Posware :material-tag:`4.3` .

Esso può essere usato esclusivamente per le casistiche di:

- Nuova installazione di *Posware Frontend* sui terminali casse
- Migrazione da Posware :material-tag:`4.2.x`

*PosUpdate* è invece l'applicativo da usare esclusivamente per aggiornare *Posware Frontend* sui terminali casse alla loro ultima **Build** version, in linea con la **Major** e **Minor** version installata (Es.: da :material-tag:`4.3.680` a :material-tag:`4.3.700`).

I dettagli specifici sul funzionamento dei due applicativi sono consultabili nelle rispettive sezioni.