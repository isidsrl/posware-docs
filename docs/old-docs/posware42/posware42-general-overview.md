# Posware 4.2
#### Manuale di attivazione ed utilizzo

 Il presente documento descrive i nuovi requisiti minimi e le procedure necessarie all'installazione di **Posware v4.2** e superiori.

##### Data ultimo aggiornamento: 19/10/2021

---
> ## Versioni software di riferimento
>
>
> - Posware **4.2.0.0** o superiore
> - .NET Framework 4 - Windows XP/Embedded/2009
> - .NET Framework 4.8 - Windows 10/8.1/8/7
>
 
## Cenni preliminari

### Posware
La versione minima di Posware richiesta è la v. **`4.1.404`** . <br>
Se la versione installata è minore, per effettuare l'upgrade alla versione v. **`4.2.x`** è necessario prima aggiornare Posware ad una versione pari o superiore alla v. **`4.1.404`**.





### Sistema operativo
Sono supportate tutte le versioni di Windows a partire da Windows XP \*. <br>
<mark>**Preferire sempre sistemi operativi più aggiornati ove possibile al fine di minimizzare migrazioni forzate in futuro.**</mark>

Per le ultime feature disponibili e le massime prestazioni possibili, utilizzare **Windows 10 version 1909** (*November 2019 Update*) o superiore. <br>
Accelerazione via GPU disponibile a partire da **Windows 7**.

> Con sistemi operativi basati su Windows XP, sono presenti limitazioni nella fruizione di servizi moderni quali WebService e servizi Cloud. <br>
> **Tutti i servizi che utilizzano il protocollo di sicurezza TLS 1.2 non sono supportati.** <br>
> Tutte le ottimizzazioni basate sull'accelerazione grafica hardware (GPU) non saranno attive in quanto non supportate. <br>
> Queste limitazioni sono ereditate dall'obolescenza intrinseca di Windows XP, non più supportato ed aggiornato da Microsoft. Prendere visione della sezione Breaking Changes contenente le informazioni end-of-support per Posware.

### Tabella di riferimento .NET Framework
Seguire con scrupolo la tabella di riferimento per installare la corretta versione di .NET framework per ogni sistema operativo.
Si raccomanda di leggere le note allegate.

| Windows     | .NET Framework   | Note    | Link |
| ----------- | ---------------- |---------|------|
| Windows XP based (Embedded/Embedded 2009)    | 4.0 |Si consiglia di installare anche gli aggiornamenti da Windows Update. In particolare l'update [4.0.3](https://support.microsoft.com/en-us/help/2600211/update-4-0-3-for-microsoft-net-framework-4-runtime-update) -  * Alcune limitazioni presenti  **Installazione `Client Profile` non supportata |[Offline Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net40-offline-installer) |
| Windows 7 / Embedded 7 / POSReady 7    | 4.8 ||[Offline Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-offline-installer) - [Web Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer) |
| Windows 8 / Embedded 8 Industry / 8.1 / Embedded 8.1 Industry   | 4.8 ||[Offline Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-offline-installer) - [Web Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer) |
| Windows 10    | 4.8 |Seguire le [indicazioni ufficiali](https://docs.microsoft.com/en-us/dotnet/framework/install/on-windows-10) Microsoft per l'installazione, non sempre è necessaria.|[Offline Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-offline-installer) - [Web Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer) |

La tabella completa fornita da Microsoft può essere consultata [qui](https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#net-framework-48)

### Supporto PosForNet
<mark>Posware `4.2` non supporta alcun driver PosForNet</mark>. Sono gestiti solo i driver OPOS dei registratori telematici e delle stampanti termiche supportate.

### Adeguamenti per il driver OPOS Diebold-Nixdorf con RT-One
Se la stampante collegata a Posware è la Diebold RT-One, è necessario impostare nel file `Posware.exe.config` l'attributo `useLegacyV2RuntimeActivationPolicy` dell'elemento `startup` a **true**. *L'attributo è lo stesso utilizzato per abilitare la compatibilità con Crystal Report.*

Esempio:
```
  <startup useLegacyV2RuntimeActivationPolicy="true">
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.8" />
  </startup>
```

Qualora si stesse effettuando una migrazione dal driver PosForNet ISID al driver OPOS Diebold-Nixdorf, è possibile rimuovere dal file `Posware.exe.config` l'elemento

```
<NetFx40_LegacySecurityPolicy enabled="true" />
```
In caso di errore di tipo `System.AccessViolationException`, ripristinare l'elemento.

> A partire dalla versione 4.1.370, rilasciata con gli adeguamenti alla normativa RT XML7, il driver PosForNet è stato sostituito con il driver OPOS ufficiale Diebold. Consultare il documento **Adeguamenti RT XML 7** per ulteriori approfondimenti

### Adeguamenti per Crystal Report
Per poter continuare ad utilizzare le stampe generate da Crystal Report, è necessario:

- Copiare all'interno della cartella di posware il Plug-In *Posware.Legacy.CrystalReport.dll* . 
- Impostare nel file `Posware.exe.config` l'attributo `useLegacyV2RuntimeActivationPolicy` a true. L'attributo fa parte dell'elemento `<startup>`

Il file dll non viene distribuito assieme al setup di Posware in quanto entrato in fase di obsolescenza. Verrà sostituito nelle prossime versioni. Al momento viene garantità la retro-compatibilità. Il file è disponibile su richiesta a chi ne fa uso.

> IMPORTANTE: **Non attivare** l'attributo `useLegacyV2RuntimeActivationPolicy` se non si fa uso di Crystal Report o se il registratore telematico collegato non è un Diebold-Nixdorf RT-One

### Adattatori Self-Checkout (SCO)
A partire da Posware **`4.2`**, la **major e minor version** degli adattatori è stata allineata con Posware Frontend <br>
La versione minima degli adattatori SCO supportata è la v.**`4.2.x`**.








### Breaking Changes
#### Aggiornamento automatico della tabella IVA / Reparti dei registratori telematici
A partire dalla versione `4.2`, Posware sincronizzerà **sempre ed in maniera automatica le tabelle IVA** dei registratori telematici a partire dalla sua configurazione IVA (tabella `codiva`). <br>
Al primo upgrade dalla versione `4.1.x` alla `4.2.x`, se il registratore telematico ha effettuato transazioni (la giornata fiscale è aperta), verrà eseguita una chisura fiscale per poter riallineare la tabella IVA/Reparti del registratore telematico. <br>
Se il registratore telematico dall'ultima chiusura fiscale non ha effettuato nuove transazioni, le tabelle IVA verranno automaticamente sincronizzate senza la necessità di una ulteriore chiusura fiscale.

Nel caso in cui l'impostazione `CaricaTabIVA` all'interno della tabella `config_cassa` fosse stata disattivata, il setup di aggiornamento riabiliterà automaticamente questa impostazione.




#### Termine del supporto delle tastiere fisiche
A partire dalla versione 4.1, non verrà più sviluppata e manutenuta la compatibilità con le tastiere fisiche. I sistemi non ancora dotati di touchscreen, dovranno effettuare l'upgrade per continuare ad utilizzare tutte le ultime funzionalità. <br>
***Il supporto attuale, ove presente, non verrà rimosso.*** Non verrà comunque garantita in futuro, in caso di modifiche/aggiornamenti alle funzionalità esistenti, la compatibilità tastiera fisica. <br>
**Le nuove funzionalità relative alla lotteria degli scontrini, saranno compatibili solo con i sistemi touchscreen.**

#### Termine del supporto ai sistemi Windows XP based.
A partire dalla versione 4.1, l'installazione su sistema operativo Windows XP è stata deprecata ed è entrata in fase maintenance: saranno forniti solo bugfix ed aggiornamenti di legge fino al 31/12/2021. <br>
**La versione `4.2.x` per Windows XP rientra in questa scadenza e sarà rilasciata fino al 31/12/2021.** <br>
Dal 01/01/2022 non verranno effettuati ulteriori rilasci. Eventuali nuove release del ramo 4.2.x saranno disponibili solo per o.s. pari o successivi a Windows 7.

Sono coinvolti tutti i sistemi operativi basati su Windows XP - major version 5  (XP/Embedded/Foundation/PosReady 2009).

I nuovi sviluppi contempleranno solo gli o.s. da Windows 11 a Windows 7 ed i runtime di riferimento saranno il .NET Framework 4.8 e  NET 5.x/6.x. 

#### PosUpdate per o.s. Windows XP based
A partire dalla versione `4.1`, il file di aggiornamento di versione per Windows XP è ora nominato **PosUpdateNET40**_4.1.x.x . <br>
Posware `4.2` per Windows XP mantiene la nomenclatura **PosUpdateNET40**_4.2.x.x . <br>
Per tutti gli altri sistemi operativi con runtime .net Framework 4.8, il nome del file rimane invariato all'attuale **PosUpdate**_4.2.x.x

### Procedura di installazione
> Se l'aggiornamento viene effettuato da Posware versione **`4.0.x`**, effettuare prima l'aggiornamento alla versione **`4.1.404`** (o altra versione superiore alla `4.1.404`). <br>
Consultare il documento `Posware 4.1 - .NET 4.x Runtime Notes` per le istruzioni di upgrade dalla versione 4.0.x.
---
> Se l'aggiornamento viene effettuato da una versione di Posware minore della **`4.1.404`**, aggiornare almeno a quella versione prima di effettuare l'upgrade alla versione **`4.2.x`**

Di seguito i passi da seguire per l'installazione da una versione pari o superiore alla **`4.1.404`**

1. Aggiornare firmware e i driver OPOS dei registratori telematici.
2. Aggiornare Posware tramite la comune procedura di `PosUpdate`.
    -  Se l'aggiornamento avviene per la prima volta dalla versione `4.1`, verrà effettuata automaticamente una chiusura fiscale ed aggiornata la tabella IVA / Reparti del registratore telematico. Vedi Breaking Changes.

**IMPORTANTE**: Se l'aggiornamento avviene per la prima volta dalla versione `4.1`, eseguire sul server di barriera gli script di aggiornamento database `carscar_mysql_4.2.0.0.sql`, su database MySql, oppure `carscar_sqlServer_4.2.0.0`, su database SQLServer

##### Considerazioni sull'hardware ed il sistema operativo
Se l'hardware del POS è recente, ma è stata applicata una immagine di sistema operativo *obsoleta*, è altamente consigliato migrare ad un sistema operativo più recente, nell'ordine di priorità Windows 10 -> Windows 8 -> Windows 7. <br>
<mark>**Preferire sempre sistemi operativi più aggiornati ove possibile per minimizzare migrazioni forzate in futuro.**</mark>

##### Installazione del .NET Framework 4.x

> Installare sempre **l'ultima versione** disponibile del .NET Framework. <br>
> Come da tabella di riferimento, per gli S.O. Windows XP Based utilizzare la `v4.0`. <br>
> Per tutti gli altri Windows più recenti, installare la `v4.8`.

In caso di nuova installazione in laboratorio, è possibile **utilizzare l'installer Web del .net Framework 4.8** con la cassa collegata direttamente ad Internet. L'installer Web scaricherà ed installerà i soli componenti necessari aggiornati per il sistema operativo in uso permettendo di risparmiare tempo. 

Qualora l'installazione venga fatta **in loco / in serie** sulle postazioni di produzione, **utilizzare gli installer Offline**.

> Per Windows XP e .NET Framework 4, **è possibile utilizzare solo gli installer Offline**. 



### Rollback
Sono possibili due scenari di rollback:

- Dalla versione `4.2.x` alla `4.1.x` (fino alla versione `4.1.404`, **non ulteriormente precedente**).
- Da una minor version ad un'altra appartenente sempre al ramo `4.2.x`

##### Rollback dalla versione `4.2.x` alla versione `4.1.x`.
Per effettuare il rollback, seguire i seguenti passaggi:

1. Disinstallare l'aggiornamento da pannello di controllo
2. Eliminare tutti i file .dll e .exe dalla cartella C:\Programmi\Posware ( C:\Program Files\Posware )
3. Accedere al database MySql `cassa` ed eliminare dalla tabella `dettagli_carscar` la colonna `tipoArticolo` . Salvare la tabella.
4. Accedere al database `posware` del server di barriera ed eliminare dalla tabella `dettagli_carscar` la colonna `tipoArticolo` . Salvare la tabella.
6. Reinstallare tramite `PosUpdate` una versione `4.1.x` che sia maggiore o pari alla `4.1.404`
7. Reinstallare nuovamente eventuali Plug-In presenti prima dell'aggiornamento.

> Non è necessario disinstallare alcun aggiornamento eventualmente eseguito per il .NET Framework <br>
> Non è necessario disinstallare alcun aggiornamento eseguito per i driver OPOS <br>
> Non è necessario effettuare il downgrade di firmware dei registratori telematici. <br>
> Ripristinare **solo Posware**.

##### Rollback da una versione `4.2.x` ad una precedente sempre del ramo `4.2.x`.
Si consideri come esempio il rollback da una ipotetica versione `4.2.150` alla versione `4.2.100`

1. Disinstallare l'aggiornamento `4.2.150` da Pannello di controllo di Windows
2. Disinstallare l'aggiornamento `4.2.100` da Pannello di controllo di Windows
3. Eliminare tutti i file .dll e .exe dalla cartella C:\Programmi\Posware ( C:\Program Files\Posware )
4. Reinstallare tramite `PosUpdate` la versione `4.2.100`
5. Reinstallare nuovamente eventuali Plug-In presenti prima dell'aggiornamento.




### Known issues
#### Posware termina in modo inaspettato dopo aver effettuato la chiusura fiscale
Se dopo aver lanciato la chiusura fiscale il programma termina inaspettatamente, verificare che il parametro `PROTOCOLLOBPE` del modulo `EPPLIB` in `tabparametriextra` **non** sia impostato erroneamente ad `AR`. Questa impostazione è valida unicamente se sono presenti i buoni pasto elettronici tramite **Argentea**, in tutti gli altri casi va impostata a stringa vuota. Effettuare la stessa verifica per il parametro `PROTOCOLLOBP`. Dopo aver impostato a blank questi parametri la chiusura verrà effettuata correttamente senza causare l'uscita del programma.

#### Posware restituisce un errore di tipo System.AccessViolationException
Qualora si utilizzino stampanti o moduli esterni con librerie non native .NET, il nuovo runtime 4.8 può mostrare questo errore. <br>
Per risolvere il problema è necessario modificare nel file **Posware.exe.config**, da *false* a *true*, la stringa:
>Prima: <br>
>`<NetFx40_LegacySecurityPolicy enabled="false"/>` <br>
>Dopo: <br>
>`<NetFx40_LegacySecurityPolicy enabled="true"/>`