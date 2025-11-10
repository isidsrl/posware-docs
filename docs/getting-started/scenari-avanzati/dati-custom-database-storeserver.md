---
tags:
    - Posware
    - StoreServer
---

# Aggiunta dati custom al database dello StoreServer

**Prima revisione documento: 6 novembre 2025** <br>
**Ultima revisione documento: {{ git_revision_date_localized }}**
---

## Cenni preliminari
Con l'introduzione dello *StoreServer*, non è più consentito modificare o estendere direttamente le tabelle del database `posware` aggiungendo campi custom.<br>
Qualsiasi intervento strutturale sulle tabelle standard del database può compromettere la stabilità dell’applicativo e la compatibilità con le future versioni del sistema.

Per questo motivo, eventuali necessità di estendere le informazioni contenute nelle tabelle esistenti devono essere gestite **tramite la creazione di nuove tabelle dedicate**, mantenendo una relazione logica con le tabelle originali del database dello *StoreServer*.

## Creazione di tabelle custom
Nel caso in cui sia necessario aggiungere dati custom, **non devono essere mai modificati o estesi i campi delle tabelle originali del database `posware`**.
Invece, è necessario procedere con la creazione di una nuova tabella che contenga i campi aggiuntivi desiderati.

Questa tabella dovrà rispettare i seguenti criteri:

1. **Struttura relazionale**<br>
La tabella custom deve contenere le **stesse chiavi primarie** (o gli stessi campi identificativi) della tabella originale, in modo da poter essere collegata tramite `JOIN` nel database dello *StoreServer*.
2. **Nomenclatura standardizzata**<br>
Il nome della nuova tabella deve seguire il formato:
``` sql
NomeRivenditore_NomeTabellaOriginale
```
Ad esempio, se il rivenditore *Seles* necessita di creare una tabella custom a partire dalla tabella `mantemp` del database `posware`, il nome corretto della nuova tabella sarà:
``` sql
seles_mantemp
```
3. **Gestione dei dati**<br>
Tutti i campi aggiuntivi dovranno essere gestiti e mantenuti esclusivamente all’interno della tabella custom, lasciando inalterata la tabella originale.<br>
Le operazioni di lettura e scrittura che richiedono dati combinati dovranno essere eseguite mediante `JOIN` tra la tabella originale e quella custom.

### Considerazioni operative
Nella gestione delle tabelle custom è importante tenere in considerazione alcuni aspetti operativi che riguardano la manutenzione e la compatibilità del sistema:

- Le tabelle custom non vengono gestite direttamente dallo *StoreServer*, pertanto la loro manutenzione e il loro aggiornamento restano a carico dell’integratore o del rivenditore.
- È consigliabile documentare internamente la struttura delle tabelle custom e le eventuali relazioni con le tabelle originali, per garantire la tracciabilità e la compatibilità futura con le versioni successive dello *StoreServer*.

### Esempio pratico
|Tabella originale|Tabella custom|Descrizione|Chiavi di collegamento|
|-----------------|--------------|-----------|----------------------|
|`mantemp`|`seles_mantemp`|Estensione della tabella `mantemp` con campi aggiuntivi personalizzati per il rivenditore *Seles*|Stesse chiavi primarie della tabella `mantemp`|

## Conclusione
In conclusione, per aggiungere dati custom al database dello *StoreServer*:

- **Non modificare** mai le tabelle del database `posware`.
- **Creare** una nuova tabella custom con le stesse chiavi della tabella di origine.
- **Rispettare** la convenzione di nomenclatura `NomeRivenditore_NomeTabellaOriginale`.
- **Gestire** i dati aggiuntivi tramite `JOIN` con le tabelle standard.