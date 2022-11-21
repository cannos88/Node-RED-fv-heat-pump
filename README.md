# Node-RED-fv-heat-pump

Il seguente flusso NodeRed permette di massimizzare l'autoconsumo fotovoltaico per scaldare casa attraverso tramite la pompa dii calore.
N.B. il flusso si basa su entity esposte da Home Assistant

La lettura dei prelievi ed immissioni in rete è stata realizzzata attraverso la creazione di 2 sensori diversi in Home Assistant, i quali prelevano il medesimo valore da uno shelly EM con TA fissato sul cavo che dal contatore del distributore giunge al quadro domestico.
La pompa di calore utilizzata è una Ariston Nimbus 50M con integrazione Ariston Thermo via HACS.

## Flusso Node Red
![alt text](https://github.com/cannos88/Node-REd-fv-heat-pump/blob/master/img/flow.PNG?raw=true)

# Indice

# Descrizione Flusso

Sono stati realizzati quattro input boolean su HA che forniscono i parametri su cui si baserà il flusso e sono liberamente editabili da Dashboard.
E' inoltre presente un input boolean che consente di disattivare o attivare l'intera automazione.\
![alt text](https://github.com/cannos88/Node-REd-fv-heat-pump/blob/master/img/HA_Dashboard_component.PNG?raw=true)\
La configurazione è presente nei file:\
[input_boolean.yaml](https://github.com/cannos88/Node-REd-fv-heat-pump/blob/master/input_boolean.yaml)\
[input_number.yaml](https://github.com/cannos88/Node-REd-fv-heat-pump/blob/master/input_number.yaml)

Questi valori verranno salvati in quattro variabili di flusso cosi da poter essere richiamate in ogni punto (nodo giallo registra il dato).

L'intero flusso è suddiviso in due sezione distinte:
Incremento di T ambientale richiesta
Reset T ambientale di Default

Incremento T ambientale:
Attraverso il nodo trigger state viene letto il valore della potenza immessa in rete espressa in W (il valore oscillerà tra 0 e 6000).
Il valore verrà salvato in una variabile di flusso chiamata potenzaEsportata.
Il nodo confronta dati si occuperà di confrontare il valore attuale del sensore con il valore di soglia impostato su HA e letto ad inizio flusso nella variabile sogliaImmissioni.
```js
var soglia = flow.get("sogliaImmissioni");
var potenzaEsportata = flow.get("potenzaEsportata");


msg.topic = "incrementoTemp";

if (potenzaEsportata >= soglia){
    msg.payload = "on";
}else{
    msg.payload = "STOP";
}
return msg;
```
se la potenza esportata è maggiore della soglia verrà inviato il comando on al nodo Trigger, altrimenti verrà inviato STOP in modo da stoppare il timer.
Questo è stato inserito al fine di evitare di invocare il servizio nel caso in cui le immissioni non siano stabili.

Al termine dell'attesa di 5 minuti viene verificata se l'automazione è attiva, successivamente si verifica che la temperatura esterna sia minore di 16.5 °C (valore letto tramite sensore NETATMO) e se la PDC sia in modalità invernale.

Se tutte le condizioni sono verificate si calcola la nuova T ambientale richiesta da settare:
```js
var tBase = flow.get("tBasePDC");
var dt = flow.get("tIncrementoPDC");
var newTemp = tBase+dt;
msg.payload=newTemp;


return msg;
```

Attraverso il nodo Change successivo si verifica che la temperatura da settare non sia stata già settata (in questo modo si evita di richiamare il servizio esterno di set infinite volte).
Si invoca quindi il servizio setTemperature delle API della PDC (in questo caso tramite l'integrazione HACS https://github.com/chomupashchuk/ariston-remotethermo-home-assistant-v2).
Dopo il set viene inviata la notifica Telegram con il set della nuova temperatura.
L'ultimo step è il salvataggio della t impostata in una variabile di flusso per la successiva verifica.

Reset T ambientale di Default:
Attraverso il nodo trigger state viene letto il valore della potenza prelevata dalla rete espressa in W (il valore oscillerà tra 0 e 6600).
Il valore verrà salvato in una variabile di flusso chiamata potenzaPrelevata.
Il nodo confronta dati si occuperà di confrontare il valore attuale del sensore con il valore di soglia impostato su HA e letto ad inizio flusso nella variabile sogliaPrelievi.
```js
var sogliaP = flow.get("sogliaPrelievi");
var potenzaImportata = flow.get("potenzaPrelevata");


msg.topic = "incrementoTemp";

if (potenzaImportata >= sogliaP){
    msg.payload = "on";
}else{
    msg.payload = "STOP";
}
return msg;
```
se la potenza prelevata è maggiore della soglia verrà inviato il comando on al nodo Trigger, altrimenti verrà inviato STOP in modo da stoppare il timer.

Al termine dell'attesa di 5 minuti viene verificata se l'automazione è attiva e se la PDC sia in modalità invernale.
Se entrambe le condizioni sono verificate si recupera la T di default da impostare.
```js
var obtainedData = flow.get("tBasePDC");
msg.payload = obtainedData;
return msg;
```
Attraverso il nodo Change successivo si verifica che la temperatura da settare non sia stata già settata.
Si invoca quindi il servizio di setTemperature delle API e si invia la notifica telegram.
L'ultimo step è il salvataggio della t impostata in una variabile di flusso per la successiva verifica.

La temperatura di default può essere impostata anche se la T esterna supera la soglia di 19.5 gradi.
Questo caso si utilizza sempre il flusso di reset T ambientale

### il flusso potrà essere oggetto di ulteriori modifiche future per migliorare l'efficienza