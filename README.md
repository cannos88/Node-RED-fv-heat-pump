# Progetto WIP ma funzionante

# Node-RED-fv-heat-pump

Il seguente flusso NodeRed permette di massimizzare l'autoconsumo fotovoltaico per scaldare casa attraverso tramite la pompa dii calore.
N.B. il flusso si basa su entity esposte da Home Assistant

La lettura dei prelievi ed immissioni in rete è stata realizzzata attraverso la creazione di 2 sensori diversi in Home Assistant, i quali prelevano il medesimo valore da uno shelly EM con TA fissato sul cavo che dal contatore del distributore giunge al quadro domestico.
La pompa di calore utilizzata è una Ariston Nimbus 50M con integrazione Ariston Thermo via HACS.

## Flusso Node Red
