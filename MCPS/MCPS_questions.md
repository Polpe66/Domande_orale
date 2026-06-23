# Esame MCPS - Guida allo Studio Completa

> **AVVISO PER PAGANELLI:** La prof.ssa chiede *tutto*, anche argomenti o dettagli non direttamente raffigurati nelle immagini dell'esame (es. il protocollo P4). **Tutti** vengono interrogati su Fourier. Per la Discrete Fourier Transform (DFT) non le interessano le formule a memoria, ma vuole che i concetti siano chiarissimi.
> 
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

## Parte 2: Networking - Prof. ssa Paganelli

### 1. MAC Wireless e Collisioni
* **Comunicazione Cablata vs Wireless:** Spiega la differenza. Perché i protocolli di collision avoidance usati nelle reti cablate (come CSMA/CD) non bastano per il wireless?
* **Nodi Nascosti ed Esposti:** Spiega i tre schemi (Hidden terminal, Exposed terminal e la soluzione RTS/CTS).
* **Interferenza al Ricevitore:** Come mai si considera l'interferenza al *receiver* e non al *transmitter*?
  * *Risposta:* Perché la collisione è un fenomeno **locale** che avviene all'antenna di chi riceve. Il trasmettitore sa solo se l'etere vicino a lui è libero, ma non può sapere se, nel punto esatto in cui si trova il ricevitore, sta arrivando contemporaneamente un altro segnale che corromperà i dati.
* **Protocolli:** Descrivi MACA. Cosa contengono RTS e CTS? (La durata della trasmissione per impostare il NAV).
* **Ritardi:** Cosa sono i ritardi inter-frame (SIFS, DIFS)? A cosa servono?

### 2. Reti Cellulari (4G vs 5G)
* **Riconoscimento Architettura:** (Immagine con source/target BS) Che tipo di rete è? 4G o 5G? Perché? (S-GW/P-GW = 4G; UPF = 5G).
* **Handover:** Come funziona l'handover?
* **Tunneling:** Come viene realizzato il tunnel tra gateway e base station? (GTP over UDP/IP).

### 3. Software Defined Networking (SDN) & OpenFlow
* **Generalized Forwarding:** Spiega lo schema OpenFlow (con l'header del pacchetto) partendo dal concetto di *generalized forwarding*.
  * *Risposta:* Nel routing tradizionale si inoltra in base all'IP di destinazione. Nel *generalized forwarding* di SDN, gli switch usano tabelle "Match-Action". Il "match" può essere fatto su molteplici campi dell'header del pacchetto (MAC address, IP sorgente/destinazione, porte TCP/UDP, ecc.), permettendo azioni complesse (inoltra, droppa, modifica).
* **Architettura SDN:** Descrivi Control Plane, Data Plane, Application Plane e API (Southbound/Northbound).
  * *Topologia (6 host, 3 switch):* Spiega come funziona l'inoltro dei pacchetti in questa topologia.
* **Limitazioni di OpenFlow e superamento con P4:** Quali sono i limiti di OpenFlow e perché è stato superato da P4?
  * *Risposta:* OpenFlow è vincolato ai protocolli di rete standard; le sue tabelle di match funzionano solo su header predefiniti (es. IPv4, TCP). **P4** (Programming Protocol-independent Packet Processors) supera questo limite rendendo il data plane completamente programmabile: permette di definire il parsing di pacchetti custom e protocolli proprietari non ancora inventati, dicendo allo switch *come* estrarre e processare i campi.
* **Applicazione Reale:** Esempio SDN (Google B4).
*  **In che modo il forwarding generalizzato differisce dal forwarding basato sulla destinazione?**                                                                                                                                             
     Risposta: Il destination-based forwarding tradizionale decide l'instradamento basandosi solo sull'indirizzo IP di destinazione. Il                                                 
     forwarding generalizzato (SDN/OpenFlow) generalizza radicalmente questo concetto con l'astrazione match+action: può usare qualsiasi                                                
     campo dell'header (L2: MAC, L3: IP sorgente/destinazione, protocollo, ToS; L4: porte TCP/UDP) come chiave di match, e le azioni non si                                             
     limitano a forward ma includono anche drop, modify (es. NAT, VLAN rewriting) e send to controller. Permette inoltre il routing per-                                                
     flusso (regole diverse per flussi diversi), impossibile nel routing destination-based classico. 
-  **Cosa si intende per operazione match-action di un router o switch? Nel caso del destination-based forwarding, cosa viene confrontato e quale azione viene eseguita?**                                                                                                                                                  
     Match-action è l'astrazione fondamentale di SDN/OpenFlow: un pacchetto arriva allo switch → viene confrontato con le voci della flow table (match) su uno o più campi dell'header (MAC, IP, ToS, protocollo, porte TCP/UDP) → se c'è corrispondenza, viene eseguita                                                     
     l'azione associata:       
     - forward: inoltra su una porta                                 
     - drop: scarta            
     - modify: modifica header (NAT, VLAN)                                   
     - send to controller: invia al controller                                                                                                                                                                                                               
     Destination-based forwarding è un caso particolare:                      
     - Match: solo l'indirizzo IP di destinazione                                       
     - Azione: forward sulla porta della route più specifica 
- **Nel caso del forwarding generalizzato in SDN, cita tre campi che possono essere confrontati (match) e tre azioni che possono essere eseguite.**                                                                                                                                                                                                                                    
     Tre campi matchable:                              
     1. Indirizzo MAC (sorgente/destinazione) — L2                                               
     2. Indirizzo IP (sorgente/destinazione) — L3                                            
     3. Porta TCP/UDP (sorgente/destinazione) — L4         
     
     Altri possibili: protocollo IP, ToS/DSCP, VLAN ID.                                                                                                                             
     Tre azioni:                          
     1. forward: invia il pacchetto su una porta specifica                                                     
     2. drop: scarta il pacchetto                            
     3. modify: modifica campi dell'header (es. NAT, VLAN rewriting)                                                           
     Altre: send to controller (invia al controller per decisione centralizzata).

### 4. Teoria dei Segnali (Domanda Fissa)
* **Serie di Fourier:** Scrivi la formula di s(t) e dei coefficienti, spiegando a cosa serve.
  * *Risposta/Formula s(t):* Serve a scomporre qualsiasi segnale periodico in una somma infinita di sinusoidi e cosinusoidi (armoniche).
    `s(t) = a_0/2 + Σ [a_n cos(2πn f_0 t) + b_n sin(2πn f_0 t)]`
  * *Coefficienti:* `a_0` rappresenta la componente continua (il valore medio del segnale). I coefficienti `a_n` e `b_n` si calcolano tramite l'integrale del segnale moltiplicato rispettivamente per il coseno e il seno sul periodo T. Indicano l'"ampiezza" (o il peso) di ciascuna frequenza all'interno del segnale originale.
* **DFT, Campionamento e Quantizzazione:**
  * *Campionamento (Teorema di Shannon-Nyquist):* Discretizzazione nel tempo. Per ricostruire il segnale, la frequenza di campionamento f_s deve essere > 2 f_max.
  * *Quantizzazione:* Discretizzazione dell'ampiezza. I valori continui campionati vengono approssimati a livelli discreti predefiniti (es. binari).
  * *Discrete Fourier Transform (DFT):* Serve per analizzare lo spettro delle frequenze di un segnale campionato nel tempo discreto (input: N campioni temporali; output: N numeri complessi delle frequenze).
