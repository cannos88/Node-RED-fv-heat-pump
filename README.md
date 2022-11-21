# Progetto WIP ma funzionante

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
E' inoltre presente un input boolean che consente di disattivare o attivare l'intera automazione.
![alt text](https://github.com/cannos88/Node-REd-fv-heat-pump/blob/master/img/HA_Dashboard_component.PNG?raw=true)\
La configurazione è presente nei file:\
[input_boolean.yaml](https://github.com/cannos88/Node-REd-fv-heat-pump/blob/master/input_boolean.yaml)\
[input_number.yaml](https://github.com/cannos88/Node-REd-fv-heat-pump/blob/master/input_number.yaml)

Questi valori verranno salvati in quattro variabili di flusso cosi da poter essere richiamate in ogni punto (nodo giallo registra il dato).
