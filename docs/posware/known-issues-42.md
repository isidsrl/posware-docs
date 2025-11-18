# Posware 4.2 - Known issues

## Posware termina in modo inaspettato dopo aver effettuato la chiusura fiscale

Se dopo aver lanciato la chiusura fiscale il programma termina inaspettatamente, verificare che il parametro `PROTOCOLLOBPE` del modulo `EPPLIB` in `tabparametriextra` **non** sia impostato erroneamente ad `AR`. Questa impostazione è valida unicamente se sono presenti i buoni pasto elettronici tramite **Argentea**, in tutti gli altri casi va impostata a stringa vuota. Effettuare la stessa verifica per il parametro `PROTOCOLLOBP`. Dopo aver impostato a blank questi parametri la chiusura verrà effettuata correttamente senza causare l'uscita del programma.

## Posware restituisce un errore di tipo System.AccessViolationException

Qualora si utilizzino stampanti o moduli esterni con librerie non native .NET, il nuovo runtime 4.8 può mostrare questo errore. <br>
Per risolvere il problema è necessario modificare nel file **Posware.exe.config**, da *false* a *true*, la stringa:

Prima: <br>

```xml
<NetFx40_LegacySecurityPolicy enabled="false"/>`
```

Dopo:
``` xml
<NetFx40_LegacySecurityPolicy enabled="true"/>
```