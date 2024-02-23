# Posware 4.1 - .NET 4.x Runtime Notes
#### Manuale di attivazione ed utilizzo

 Il presente documento descrive i nuovi requisiti minimi e le procedure necessarie all'installazione di **Posware v4.1** e superiori.


##### Data ultimo aggiornamento: 08/01/2021

---

> ## Versioni software di riferimento
>
>
> - Posware **4.1.0.0** o superiore
> - .NET Framework 4 - Windows XP/Embedded/2009
> - .NET Framework 4.8 - Windows 10/8.1/8/7
>
 

## Cenni preliminari

### Posware
La versione minima di posware richiesta è la v. **4.0.0.150**. Se la versione installata è minore, per effettuare l'upgrade alla versione 4.1.x è necessario prima upgradare al minimo alla versione **4.0.0.150**.

### Sistema operativo
Sono supportate tutte le versioni di Windows a partire da Windows XP \*. Per le ultime feature disponibili e le massime prestazioni possibili, utilizzare Windows 7 o superiore. **Windows 10 version 1909** (*November 2019 Update*) consigliato ove possibile.

> Con sistemi operativi basati su Windows XP, sono presenti limitazioni nella fruizione di servizi moderni quali WebService e servizi Cloud. 
> Tutti i servizi che utilizzano il protocollo di sicurezza TLS 1.2 non sono supportati.
> Inoltre tutte le ottimizzazioni basate sull'accelerazione grafica hardware (GPU) non saranno attive in quanto non supportate.
> Queste limitazioni sono ereditate dall'obolescenza intrinseca di Windows XP, non più supportato ed aggiornato da Microsoft. Prendere visione della sezione Breaking Changes contenente le informazioni end-of-support per Posware.

### Tabella di riferimento .NET Framework
Seguire con scrupolo la tabella di riferimento per installare la corretta versione di .NET framework per ogni sistema operativo. <br>
Si raccomanda di leggere le note allegate.

| Windows     | .NET Framework   | Note    | Link |
| ----------- | ---------------- |---------|------|
| Windows XP based (Embedded/Embedded 2009)    | 4.0 |Si consiglia di installare anche gli aggiornamenti da Windows Update. In particolare l'update [4.0.3](https://support.microsoft.com/en-us/help/2600211/update-4-0-3-for-microsoft-net-framework-4-runtime-update) -  * Alcune limitazioni presenti ** Aggiornamento driver `PosForNet` necessario *** Installazione `Client Profile` non supportata |[Offline Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net40-offline-installer) |
| Windows 7 / Embedded 7 / POSReady 7    | 4.8 ||[Offline Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-offline-installer) - [Web Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer) |
| Windows 8 / Embedded 8 Industry / 8.1 / Embedded 8.1 Industry   | 4.8 ||[Offline Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-offline-installer) - [Web Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer) |
| Windows 10    | 4.8 |Seguire le [indicazioni ufficiali](https://docs.microsoft.com/en-us/dotnet/framework/install/on-windows-10) Microsoft per l'installazione, non sempre è necessaria.|[Offline Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-offline-installer) - [Web Installer](https://dotnet.microsoft.com/download/dotnet-framework/thank-you/net48-web-installer) |

La tabella completa fornita da Microsoft può essere consultata [qui](https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/versions-and-dependencies#net-framework-48)

### Aggiornamenti per PosForNet
Nel caso in cui la stampante collegata utilizzi i driver PosForNet 1.12, si raccomanda di installare gli aggiornamenti come da tabella.

| Windows     | Update PosForNet   | Link |
| ----------- | :----------------: |------|
| Windows XP / Embedded    | `KB980087` |[Offline installer](https://web.archive.org/web/20200804104743/https://download.microsoft.com/download/2/0/8/2089FED1-A302-412E-B449-D4C5BA938059/KB980087.exe) |
| Windows XP PosReady 2009    | `KB2755802` |[Offline Installer](https://web.archive.org/web/20200804152106/https://download.microsoft.com/download/8/A/0/8A073598-D37E-4BAE-8FE4-26BD232A4F69/WindowsXP-KB2755802-x86-ENU.exe) |
| Windows 7 / Embedded 7 / PosReady 7   |`KB2755802`  |[Offline Installer](https://web.archive.org/web/20200804012041/https://download.microsoft.com/download/3/A/4/3A4FD69B-E3A5-4719-A1E2-9B028FBE40AB/Windows6.1-KB2755802.exe) 

> *Installare sempre la versione full del .net Framework. Le versioni denominate `Client Profile` non sono supportate*

##### Note sul driver PosForNet ISID per Diebold RT-One
Se la stampante collegata è la Diebold RT-One ed il driver PosForNet è quello prodotto da ISID, è necessario impostare nel file `Posware.exe.config` il flag `NetFx40_LegacySecurityPolicy` a true.

`<NetFx40_LegacySecurityPolicy enabled="true"/>`

> A partire dalla versione 4.1.370, rilasciata con gli adeguamenti alla normativa RT XML7, il driver PosForNet è stato sostituito con il driver OPOS ufficiale Diebold. Consultare il documento **Adeguamenti RT XML 7** per ulteriori approfondimenti

### Adeguamenti per Crystal Report
Per poter continuare ad utilizzare le stampe generate da Crystal Report, è necessario:

- Copiare all'interno della cartella di posware il Plug-In *Posware.Legacy.CrystalReport.dll* . 
- Impostare nel file `Posware.exe.config` l'attributo `useLegacyV2RuntimeActivationPolicy` a true. L'attributo fa parte dell'elemento `<startup>`

Il file dll non viene distribuito assieme al setup di Posware in quanto entrato in fase di obsolescenza. Verrà sostituito nelle prossime versioni. Al momento viene garantità la retro-compatibilità. Il file è disponibile su richiesta a chi ne fa uso.

> IMPORTANTE: **Non attivare** l'attributo `useLegacyV2RuntimeActivationPolicy` se non si fa uso di Crystal Report

### Breaking Changes
#### Termine del supporto delle tastiere fisiche
A partire dalla versione 4.1, non verrà più sviluppata e manutenuta la compatibilità con le tastiere fisiche. I sistemi non ancora dotati di touchscreen, dovranno effettuare l'upgrade per continuare ad utilizzare tutte le ultime funzionalità. <br>
***Il supporto attuale, ove presente, non verrà rimosso.*** Non verrà comunque garantita in futuro, in caso di modifiche/aggiornamenti alle funzionalità esistenti, la compatibilità tastiera fisica. <br>
**Le nuove funzionalità relative alla lotteria degli scontrini, saranno compatibili solo con i sistemi touchscreen.**

#### Termine del supporto ai sistemi Windows XP based.
A partire dalla versione 4.1, l'installazione su sistema operativo Windows XP è deprecata ed entra in fase maintenance: saranno forniti solo bugfix ed aggiornamenti di legge fino al 31/12/2021. I nuovi sviluppi coinvolgeranno gli o.s. da Windows 10 a Windows 7 ed il runtime di riferimento sarà il .net Framework 4.8/Core 3.x/5.x. <br>
Sono coinvolti tutti i sistemi operativi basati su Windows XP - major version 5  (XP/Embedded/Foundation/PosReady 2009).
#### PosUpdate per o.s. Windows XP based
A partire dalla versione 4.1, il file di aggiornamento di versione per Windows XP è ora nominato **PosUpdateNET40**_4.1.x.x <br>
Per tutti gli altri sistemi operativi con runtime .net Framework 4.8, il nome del file rimane invariato all'attuale **PosUpdate**_4.1.x.x

### Procedura di installazione
> Prima di procedere all'aggiornamento di versione, fare un backup integrale della cartella `C:\Programmi\Posware` ( `C:\Program Files\Posware` ) e del database `cassa`. Tenere da parte questi dati in caso di eventuale rollback. Vedi sezione Rollback.

1. Installare la versione di .Net Framework ed i suoi aggiornamenti come da tabella di riferimento.
2. In caso di utilizzo dei driver PosForNet, installare gli aggiornamenti come da tabella di riferimento.
3. Aggiornare Posware tramite la comune procedura di `PosUpdate`.

##### Considerazioni  sull'hardware
Se l'hardware del POS è recente, ma è stata applicata una immagine di sistema operativo *obsoleta*, valutare la migrazione ad un sistema operativo più recente, **al minimo Windows 7**. <br>
In ottica futura non si incorrerà in altri interventi qualora il Pos dovrà utilizzare servizi web moderni. <br>
Applicare una immagine già pronta basata su Windows 7 / 8 / 10, a volte può richiedere meno tempo che installare ed aggiornare un sistema esistente. <br>
**Si consiglia di effettuare tali valutazioni.**

##### Installazione del .NET Framework 4.x
> Installare sempre **l'ultima versione** disponibile del .NET Framework. <br>
> Come da tabella di riferimento, per gli S.O. Windows XP Based utilizzare la `v4.0`. <br>
> Per tutti gli altri Windows più recenti, installare la `v4.8`.

In caso di nuova installazione in laboratorio, è possibile **utilizzare l'installer Web del .net Framework 4.8** con la cassa collegata direttamente ad Internet. L'installer Web scaricherà ed installerà i soli componenti necessari aggiornati per il sistema operativo in uso permettendo di risparmiare tempo. 

Qualora l'installazione venga fatta **in loco / in serie** sulle postazioni di produzione, **utilizzare gli installer Offline**.

> Per Windows XP e .NET Framework 4, **è possibile utilizzare solo gli installer Offline**. 

### Rollback
Per effettuare il Rollback alla versione basata su .NET 3.5, seguire i seguenti passaggi:

1. Disinstallare l'aggiornamento da pannello di controllo
2. Eliminare la cartella C:\Programmi\Posware ( C:\Program Files\Posware )
3. Ripristinare la cartella dalla copia di backup precedentemente salvata 
4. Ripristinare il database dalla copia di backup precedentemente salvata 

> Non è necessario disinstallare alcun aggiornamento eseguito per il .NET Framework o PosForNet
> al fine di tornare alla versione 4.0.x basata su .NET 3.5. <br>
> Ripristinare **solo Posware**.

### Known issues
#### Posware termina in modo inaspettato dopo aver effettuato la chiusura fiscale
Se dopo aver lanciato la chiusura fiscale il programma termina inaspettatamente, verificare che il parametro `PROTOCOLLOBPE` del modulo `EPPLIB` in `tabparametriextra` **non** sia impostato erroneamente ad `AR`. Questa impostazione è valida unicamente se sono presenti i buoni pasto elettronici tramite **Argentea**, in tutti gli altri casi va impostata a stringa vuota. Effettuare la stessa verifica per il parametro `PROTOCOLLOBP`. Dopo aver impostato a blank questi parametri la chiusura verrà effettuata correttamente senza causare l'uscita del programma.

#### Posware restituisce un errore di tipo System.AccessViolationException
Qualora si utilizzino stampanti o moduli esterni con librerie non native .NET, il nuovo runtime 4.8 può mostrare questo errore.
Per risolvere il problema è necessario modificare nel file **Posware.exe.config**, da *false* a *true*, la stringa:
>Prima: <br>
>`<NetFx40_LegacySecurityPolicy enabled="false"/>` <br>
>Dopo: <br>
>`<NetFx40_LegacySecurityPolicy enabled="true"/>`

Si tratta delle stessa impostazione necessaria per rendere compatibili le stampanti che usano il driver PosForNet (vedi sezione *Note sul driver PosForNet ISID per Diebold RT-One*).