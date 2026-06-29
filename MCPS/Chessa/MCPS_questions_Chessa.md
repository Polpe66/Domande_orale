
> **AVVISO PER CHESSA:** È estremamente pignolo sulle unità di misura e sulle formule (in particolare quella della Lifetime).

---

## Parte 1: Cyber-Physical Systems (CPS) - Prof. Chessa

### 1. Duty Cycle, Energia e Lifetime (Tempo di vita)
* **Concetto di Duty Cycle:** Cos'è il duty cycle?
* **Analisi del codice (Immagine: duty cycle - I):** Prendendo in considerazione questo blocco di codice, parla del duty cycle in Arduino.
  * *Cosa dire:* Spiega come funziona l'accensione e lo spegnimento dei componenti e cosa implica questo per il duty cycle.
  * *Relazione con l'energia:* C'è una relazione tra duty cycle ed energia? (Impara la formula a memoria!)
  * *Power-up/down:* In questo ciclo ci sono molti accensioni e spegnimenti. Perché lo facciamo?
* **Stati del Processore:** Qual è la differenza tra *idle* e *delay* nel processore?
* **Librerie & Hardware:** Cosa fanno le funzioni nella libreria `lowbattery.h`? Come si riaccende il microprocessore, qual è il meccanismo? (Risposta: tramite un timer o un interrupt esterno).
* **Calcolo della Lifetime:** Spiega il tempo di vita (lifetime) e come si ottiene.
  * *Attenzione:* È estremamente pignolo sulle unità di misura. Puoi usare i *mAh* o i *Joule* per l'energia, ma devi motivare il perché. Devi farlo nella formula della lifetime, poiché è data da: `Capacità della Batteria / Energia per ciclo` (ricordati di parlare anche dei "leaks", le dispersioni).

### 2. Arduino & Interrupts
* **Analisi del codice (Immagine: Arduino - interrupt):** Commenta il codice dicendo cosa fanno i vari comandi.
* **Esecuzione e Vincoli:** Quando viene eseguito l'interrupt handler (ISR)? Cosa è permesso fare agli interrupt e perché? (Risposta: non devono interferire con l'esecuzione normale e devono essere estremamente brevi).
* **La keyword volatile:** A cosa serve `volatile`? Perché è necessario e perché il valore non sarebbe consistente altrimenti?
  * *Risposta:* Per via dell'ottimizzazione del compilatore, il valore di una variabile potrebbe essere scritto in un registro locale invece che nella memoria RAM. Alla fine dell'interrupt, i valori dei registri tornano allo stato precedente, perdendo la modifica. `volatile` forza il compilatore a leggere/scrivere sempre direttamente in memoria.

### 3. Protocollo MQTT & IoT
* **Paradigma Publish/Subscribe:** Spiega il paradigma in generale e poi la sua implementazione in MQTT.
  * *Differenze con TCP/IP:* In che modo MQTT estende le capacità di TCP/IP?
  * *Disaccoppiamento:* Spiega il disaccoppiamento spaziale, temporale e di sincronizzazione in MQTT (Publishers e Subscribers non hanno bisogno di conoscersi, di essere attivi contemporaneamente o di bloccarsi in attesa di messaggi).
* **Filtraggio per tipo:** Fai un esempio di filtraggio dei messaggi. (Risposta: In MQTT il filtraggio avviene tramite i *Topic*, organizzati gerarchicamente, es. `casa/cucina/temperatura`).
* **Notify:** Cos'è un Notify? (Concetto di notifica asincrona di un evento verso i subscriber).
* **Sessioni e Fallimenti:** Cosa sono le sessioni persistenti in MQTT e a cosa servono?
  * Una connessione può fallire in MQTT? Cosa facciamo quando fallisce? (Risposta: Last Will and Testament - LWT).
  * TCP è sufficiente per garantire una buona Quality of Service (QoS)? Quale meccanismo usiamo per rimediare alle disconnessioni? (Risposta: il Keep-Alive timer).
* **Scelta della QoS (Esempio QoS 2):** Qual è un caso tipico in cui serve la QoS 2 (Exactly once) e perché i duplicati non sarebbero tollerati?
  * *Risposta:* Un caso tipico riguarda gli **attuatori**. Se controlli un macchinario pesante, un braccio robotico o una valvola, ed erroneamente ripeti due volte l'azione a causa di un duplicato (tipico del QoS 1 "At least once"), rischi di causare danni fisici o disastri.
* **Messaggi Retained:** Cosa sono i messaggi trattenuti (retained messages)? Fai un esempio.

### 4. Architettura ZigBee & IEEE 802.15.4
* **Capacità della Rete:** Quante reti ci possono essere in una determinata area? (Pensa al 802.15.4).
  * *Risposta:* 1 rete per canale. Ricorda che un dispositivo ZigBee può supportare solo alcune frequenze dell'802.15.4; solitamente max 15 canali (quindi max 15 reti).
* **Dispositivi e Indirizzi:** Quanti dispositivi ci possono essere in una rete? (Risposta: max 2^16, lo short address è a 16-bit).
  * Come vengono calcolati/assegnati gli indirizzi in ZigBee? (Calcolo con il Cskip e distribuzione dei blocchi).
* **Join & Association:**
  * Spiega il processo di Join in ZigBee.
  * Qual è il risultato di uno scan? Come fa un dispositivo a scegliere a quale PAN ID associarsi?
  * *Domanda trabocchetto:* Perché c'è una "Data Request" dal client verso il router durante il join? (Risposta: Per comunicare serve un indirizzo di rete, il client ne è ancora sprovvisto, quindi deve essere lui a inizializzare lo scambio per farsi assegnare lo short address).
  * Disegna il diagramma dell'Associate Request.
* **Livelli e Superframe:**
  * Architettura generale e come si trova il canale giusto a cui connettersi.
  * Livello di Rete (Immagine: ZigBee IV): Cosa rappresenta il grafo? Dimensionamento dell'albero (max-depth, ecc.).
  * Superframe 802.15.4 (Immagine: IEEE 802.15.4 - I): Cosa contiene il beacon frame?
  * Come fa un end-device a ricevere messaggi dal coordinatore?
* **Binding:** Cosa è un binding? Perché si usano indirizzi a 64-bit? Cos'è l'Address Map? (Immagine: ZigBee V).

### 5. Energy Harvesting & Gestione della Batteria
* **Schema Harvest-Store-Use:** Spiega l'immagine dell'Energy Harvesting con il grafico della potenza immagazzinata e consumata.
* **Significato degli integrali:** L'integrale della potenza in ingresso (harvested) nel tempo rappresenta l'energia totale raccolta, mentre l'integrale della potenza in uscita (used/leaked) rappresenta l'energia consumata. Per mantenere il sistema vivo (neutralità energetica), l'integrale dell'energia raccolta deve essere maggiore o uguale a quello dell'energia consumata, tenendo conto delle perdite.
* **Caratteristiche dei Buffer:** Spiega e discuti le caratteristiche di un buffer (batteria/supercondensatore):
  1. *Capacità:* Quanta energia può immagazzinare.
  2. *Efficienza di carica/scarica:* Quanta energia si perde durante il trasferimento.
  3. *Leakage (Dispersione):* L'energia che si perde fisiologicamente nel tempo anche senza utilizzo.
  * *Qual è la più importante a livello progettuale?* Dipende dall'applicazione, ma nei sistemi WSN/IoT a lunghissimo termine, il **Leakage** e l'**Efficienza** sono spesso i fattori più critici (una batteria molto capiente è inutile se perde tutta la carica per leakage prima di poterla usare).
* **Modello di Kansal:** Come definisce Kansal la neutralità energetica? Spiega il suo algoritmo.

---

