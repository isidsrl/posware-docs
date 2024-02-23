# Posware - Gestione prodotto check-out
#### Programmazione prodotti per check-out
---
> ## Versioni software di riferimento
>
>
> - Posware **4.x** o superiore
> - Posware per Windows XP **4.1.x/4.2.x** o superiore
>


## Last Update Date
> Data ultimo aggiornamento 05/01/2021



### Configurazione
La procedura di attivazione del controllo di pesatura con bilancia check-out si effettua configurando il front-end nelle seguenti tabelle

### Config_Cassa
| Campo | Valore |
| ------ | ------ |
|bilancia| Nome opos|
|ForzaPesoCheckout| 0/1|
|PercTolleranzaCheck| Percentuale di scostamento dal rilevato sul peso|
|ValTolleranzaCheck| Valore max scostamento sul valore calcolato|
|StampaRigaPeso| 0/1 *(Attiva / Disattiva)*|
|StatoInizBilancia| 0/1 *(Disattiva / Attiva)*|
|MaxCapBilancia| Massimo peso per controllo (espresso in Kg)|
|CompEtichPesoSupBil| 0/1/2 |

**ForzaPesoCheckout**
> - 0 = Il prodotto sarà sempre pesato per controllo
> - 1 = Se viene imposto una quantità manuale, questa ha priorità sul controllo che sarà bypassato

**CompEtichPesoSupBil** 
> - 0 = Considera il valore dell'etichetta 
> - 1 = Considera il valore pesato dalla bilancia 
> - 2 = Fai scegliere all'operatore

**MaxCapBilancia** = Serve per impostare qual'è la portata massima della bilancia. Se il peso calcolato dall'etichetta risulti superiore alla portata il controllo viene saltato. Se è 0 il campo non viene considerato. 
### Articoli
In caso di articolo il cui peso deve essere controllato dal front-end essendo stato pesato su altra bilancia, lo stesso deve essere configurato secondo questo schema

| Campo | Valore |
| ------ | ------ |
|CodiceEan|2xxxxxxxyyyyK|
|Um|KG|
|bpeso|true / peso|
|Prezzo| Prezzo al Kg |
|CheckOut| 0/1 *(Disattivo/Attivo)* |

In caso di articolo deve essere pesato dal front-end lo stesso deve essere configurato secondo questo schema

| Campo | Valore |
| ------ | ------ |
|CodiceEan|xxxxx|
|Um|KG|
|bpeso|true / peso|
|Prezzo| Prezzo al Kg |
|CheckOut| 0/1 *(Disattivo/Attivo)* |

Il valore assunto da CodiceEan  (xxxxx) è quello del codice PLU da digitare in cassa

### Config_Tast
| Funzione | Operazione |
| ------ | ------ |
|ATTIVA_BIL|Attiva il controllo tramite bilancia di checkout|
|DISATTIVA_BIL|Disattiva il controllo tramite bilancia di checkout|
|TARA|Usata prima dell'articolo da pesare imposta la tara di quest'ultimo leggendo il peso dalla bilancia|
|TARA_MANUALE|Usata prima dell'articolo da pesare imposta manualmente la tara di  quest'ultimo.|

Il valore della tara deve essere espresso in grammi.






### FAQ
> - Il prodotto non viene valorizzato <br>
> *Verificare il prezzo di vendita che deve essere diverso da zero* 
> - Posware mostra un messaggio di peso non congruente anche se il peso è corretto <br>
> *Verificare che il campo relativo alla percentuale di tolleranza sia valorizzaato*
> - Se imposto il peso a mano ( 0,345 * 1200 ), Posware mi chiede di pesare <br>
> *Verificare lo stato del campo ForzaPesoCheckout, deve essere a 1*