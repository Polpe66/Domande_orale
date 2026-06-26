

> **AVVISO PER PAGANELLI:** La prof.ssa chiede *tutto*, anche argomenti o dettagli non direttamente raffigurati nelle immagini dell'esame (es. il protocollo P4). **Tutti** vengono interrogati su Fourier. Per la Discrete Fourier Transform (DFT) non le interessano le formule a memoria, ma vuole che i concetti siano chiarissimi.

---

## Parte 2: Networking - Prof. ssa Paganelli

### 1. MAC Wireless e Collisioni
* **Comunicazione Cablata vs Wireless:** Spiega la differenza. Perché i protocolli di collision detection usati nelle reti cablate (come CSMA/CD) non bastano per il wireless?

	Nelle reti cablate con mezzo fisico condiviso è possibile usare protocolli MAC come il CSMA/CD (Carrier Sense Multiple Access with Collision Detection), che permette di ascoltare se la rete è libera e di rilevare le collisioni mentre avvengono. Affinché il rilevamento funzioni, il tempo di trasmissione del pacchetto deve essere maggiore o uguale a 2T (dove T è il tempo di propagazione massimo verso il terminale più lontano, e 2T rappresenta il Round Trip Time o RTT). Questo vincolo impone una dimensione minima del frame, garantendo che il trasmettitore stia ancora trasmettendo quando l'eventuale segnale di collisione torna indietro, permettendogli di accorgersene. Poichè a fine comunicazione completa del frame, il buffer si svuota.
	
	Nel contesto wireless, la comunicazione avviene tramite onde elettromagnetiche e il rilevamento delle collisioni in fase di trasmissione (CD) risulta impossibile. A causa del forte path loss (l'attenuazione del segnale nello spazio libero è proporzionale alla distanza, tipicamente modellizzata come 1/d^n, con n che varia tra 2 e 4 a seconda dell'ambiente), la potenza del segnale in arrivo è ordini di grandezza inferiore rispetto alla potenza del segnale emesso. Di conseguenza, un'antenna in fase di trasmissione viene completamente accecata dal proprio segnale e non è in grado di ascoltare il canale per rilevare eventuali trasmissioni sovrapposte. Per questo si rilevano collisioni solo al ricevitore.
	
	
	Inoltre, a causa della distribuzione spaziale dei nodi una collizione al tramettitore potrebbe non apparire tale al ricevitore.
	A seguito di tutte le questioni esplicate derivano due problematiche fondamentali: il problema del terminale nascosto e il problema del terminale esposto.
	
	Per queste ragioni, non potendo rilevare attivamente le collisioni, è necessario prevenirle alla radice utilizzando protocolli di Collision Avoidance come il CSMA/CA. Questo protocollo, per mitigare i problemi spaziali citati, si avvale (spesso in via opzionale) del "virtual carrier sensing" basato sull'handshake RTS/CTS (Request To Send / Clear To Send), un meccanismo di prenotazione del canale derivato direttamente dai protocolli fondativi MACA e MACAW.

--- 

* **Nodi Nascosti ed Esposti:** Spiega i tre schemi (Hidden terminal, Exposed terminal e la soluzione RTS/CTS).
	Nel caso del **terminale nascosto** ci troviamo nella situazione in cui due nodi trasmettitori non si percepiscono fisicamente, ovvero sono reciprocamente al di fuori dei rispettivi range comunicativi, ma tentano la comunicazione simultanea verso lo stesso nodo ricevitore centrale, generando una collisione distruttiva. 
	
	Nel caso del **terminale esposto** un nodo percepisce l'etere occupato a causa di una trasmissione vicina e si astiene dall'inviare dati a un ricevitore lontano, sebbene questa sua potenziale comunicazione non andrebbe a interferire in alcun modo con la ricezione in corso, bloccando di fatto trasmissioni compatibili e sprecando banda.
	
	Il protocollo MACA risolve queste inefficienze tramite due pacchetti di controllo che permettono di prevenire la collisione sui dati. Il mittente invia un pacchetto RTS (Request To Send) indicando l'indirizzo sorgente, la destinazione e la durata stimata della comunicazione. Se il destinatario è pronto, risponde con un pacchetto CTS (Clear To Send), replicando l'informazione sulla durata. Tutti i nodi che intercettano queste informazioni aggiornano il proprio Network Allocation Vector (NAV), un timer interno che forza il silenzio radio per il tempo pattuito.
	Per la risoluzione del terminale esposto, un nodo che si trova nel range del mittente ma non in quello del destinatario riceverà l'RTS ma non ascolterà il successivo CTS. Da questo deduce che la ricezione avverrà lontano da lui e che una sua trasmissione verso un altro nodo non creerebbe collisioni a quel ricevitore, sentendosi libero di avviare una propria comunicazione. 
	Nel caso del terminale nascosto, il nodo si trova fuori dal range del mittente ma all'interno di quello del destinatario. Esso non sentirà l'RTS, ma riceverà il CTS. Questo lo informa che un dispositivo vicino sta per ricevere dati, costringendolo a rimanere disciplinatamente in silenzio per il tempo indicato.
	Con MACA le uniche collisioni possibili avvengono tra pacchetti RTS simultanei, i quali essendo molto piccoli non comportano perdita di payload utile. Per la risoluzione delle contese sugli RTS in caso di ingorgo si utilizza l'algoritmo Binary Exponential Backoff. 
	
	Da questa architettura deriva MACAW, che migliora l'affidabilità introducendo un pacchetto di riscontro ACK, oltre a tecniche per il controllo di congestione volte a rendere l'allocazione della banda più equa (fairness). 
	Infine, combinando MACAW con carrier sensing per evitare l'invio inutile di RTS quando il canale locale è palesemente occupato, si ottiene il protocollo ibrido definitivo CSMA/CA.
---

* **Interferenza al Ricevitore:** Come mai si considera l'interferenza al *receiver* e non al *transmitter*?
	La collisione è un fenomeno locale, a causa della spazialità delle stazioni risulta che la collisione nel trasmittente non è tale nel ricevitore. Inoltre, a causa della fisica delle onde elettromagnetiche, il segnale in arrivo da un nodo lontano è infinitamente meno potente di quello che l'antenna genera in uscita. Di conseguenza, quando un nodo trasmette, il suo stesso segnale acceca completamente il segnale in arrivo. Questo limite fisico rende impossibile ascoltare il canale mentre si inviano dati, per cui si valuta l'interferenza e si gestisce la collisione solo ed esclusivamente dal punto di vista del ricevitore.

---

* **Protocolli:** Descrivi MACA. Cosa contengono RTS e CTS? (La durata della trasmissione per impostare il NAV).
	Il protocollo MACA nasce per rimuovere il problema del terminale esposto e nascosto. Funziona grazie a due pacchetti di controllo, RTS e CTS, che permettono di prevenire la collisione (collision avoidance) anziché tentare di rilevarla a posteriori. L'RTS viene inviato da chi vuole iniziare la comunicazione e contiene l'ID del mittente, del destinatario e, informazione fondamentale, la durata stimata della trasmissione. Il destinatario, se pronto a ricevere, risponde con un CTS che duplica l'informazione principale sulla durata. In questo modo si ottiene una gestione completa delle situazioni problematiche. Il terminale esposto viene risolto poiché riceve l'RTS ma non il CTS, deduce di non essere nel range comunicativo del ricevente e sa che una sua eventuale trasmissione verso altri nodi non creerebbe collisioni. Il terminale nascosto, invece, non percepisce il mittente (non riceve l'RTS) ma, trovandosi nel range del destinatario, riceve il CTS. Appena lo intercetta, imposta il NAV (Network Allocation Vector), un timer interno basato sulla durata letta nel pacchetto di controllo, che lo obbliga a restare disciplinatamente in silenzio per il tempo necessario alla trasmissione del frame. Con questo protocollo le collisioni avvengono solo tra pacchetti RTS simultanei; essendo piccoli non ci sono perdite di payload utile e l'ingorgo viene smaltito usando l'algoritmo Binary Exponential Backoff. Dopo questo protocollo nasce MACAW, che migliora MACA aggiungendo l'ACK a seguito della ricezione dei dati e tecniche per rendere il throughput maggiormente equo (fairness). Infine, aggiungendo a MACAW l'ascolto fisico preliminare dell'etere (carrier sensing) per non inviare RTS quando il canale è già palesemente occupato, si è ottenuto lo standard CSMA/CA (e non il CSMA/CD, che è per le reti cablate).

---



* **Ritardi:** Cosa sono i ritardi inter-frame (SIFS, DIFS)? A cosa servono?
	Nel protocollo CSMA/CA, vengono prioritizzati i pacchetti grazie a questi IFS, Interframe Space, che sono tempi di attesa di default da rispettare. Nel caso del DIFS, abbiamo un tempo di attesa obbligatorio lungo che permette di evitare di interrompere messaggi a priorità maggiore. Invece il tempo di attesa più corto, ovvero il SIFS, viene rispettato dai pacchetti come quelli di handshake (RTS, CTS, ACK) perché hanno massima priorità. Il sistema funziona poiché DIFS > SIFS, quindi sicuramente l'apertura di una nuova comunicazione, ad esempio l'invio di un frame dati, non interrompe i messaggi con priorità maggiore che partiranno sempre prima. Inoltre, i tempi come il DIFS scorrono atomicamente in modo contiguo quando il canale è libero: se il canale diventa occupato prima del termine, il timer si azzera e si riparte da capo, a differenza del contatore dell'exponential backoff che invece si può congelare e riprendere da dove si era fermato.
	

### 2. Reti Cellulari (4G vs 5G)
* **Riconoscimento Architettura:** (Immagine con source/target BS) Che tipo di rete è? 4G o 5G? Perché? (S-GW/P-GW = 4G; UPF = 5G).
* **Handover:** Come funziona l'handover?

---

* **Tunneling:** Come viene realizzato il tunnel tra gateway e base station? 
	Il tunneling nel piano dati delle reti mobili (LTE/4G) viene realizzato utilizzando il protocollo **GTP** (*GPRS Tunneling Protocol*), che è incapsulato all'interno di pacchetti UDP per garantire efficienza e bassa latenza.
	Nell'architettura standard 4G LTE, che implementa il **routing indiretto**, la connessione viene divisa in due tunnel GTP in cascata:
	1. Un primo tunnel tra il **P-GW** (Home Network) e l'**S-GW** (rete visitata).
	2. Un secondo tunnel tra l'**S-GW** e la **Base Station** (BS) a cui è collegato il dispositivo.
	
	Questa architettura permette la gestione della mobilità: durante un **handover**, l'S-GW deve semplicemente cambiare l'endpoint del secondo tunnel (spostandolo dalla BS sorgente alla BS target). Il primo tunnel verso il P-GW rimane inalterato, rendendo lo spostamento trasparente per la rete esterna (Internet) e impedendo alle connessioni in corso di cadere.
	
	Alla base di questo funzionamento c'è la rigida separazione tra i piani della rete:
	* **Data Plane (Piano Dati):** Nodi come BS, S-GW e P-GW usano **GTP-U** su UDP per trasportare il traffico utente in modo veloce.
	* **Control Plane (Piano di Controllo):** Entità "intelligenti" come MME e HSS, che gestiscono l'autenticazione, la mobilità e la creazione dei tunnel, comunicano usando protocolli estremamente affidabili come **SCTP/IP** (*Stream Control Transmission Protocol*), preferito al TCP per la sua maggiore robustezza nel gestire segnali di controllo critici.
	* BS sta a metà fra data plane e control plane, poichè comunica con S-GW e MME.
---





### 3. Software Defined Networking (SDN) & OpenFlow
* **Generalized Forwarding:** Spiega lo schema OpenFlow (con l'header del pacchetto) partendo dal concetto di *generalized forwarding*.
	Nel routing tradizionale si inoltra in base all'IP di destinazione. Nel generalized forwarding di  SDN, gli switch usano tabelle "Match-Action". Il "match" può essere fatto su molteplici campi dell'header del pacchetto (MAC address, IP sorgente/destinazione, porte TCP/UDP, ecc.), permettendo azioni complesse (inoltra, droppa, modifica, invia al controller)
	
---
* **Architettura SDN:** Descrivi Control Plane, Data Plane, Application Plane e API (Southbound/Northbound).
	L'architettura SDN si struttura su tre livelli fondamentali e indipendenti:
	
	**Data Plane (Piano Dati)**: È composto da switch semplici e veloci, il cui unico compito è  inoltrare meccanicamente i pacchetti. Non possiedono alcuna logica di routing interna, ma si limitano ad applicare le regole di *forwarding* (match-action) dettate dalle *flow table* fornitegli dal livello superiore.
	
	**Control Plane (Piano di Controllo)**: Rappresenta il coordinatore logicamente centralizzato della rete. Comunica verso il basso (con gli switch del Data Plane) tramite le **Southbound API**, utilizzando protocolli standard come OpenFlow. Comunica invece verso l'alto (con il livello applicativo) tramite le **Northbound API**. Ovviamente il controllo risulta distribuito in modo da avere fault tollerance e resilienza.
	
	**Application Plane (Piano Applicativo)**: È il vero "cervello" della rete SDN, dove operano le applicazioni che gestiscono le policy di rete (es. routing, load balancing, firewall e controllo degli accessi). Queste applicazioni comunicano le loro "intenzioni" al Control Plane tramite le Northbound API e, grazie al disaccoppiamento (unbundling), possono essere fornite da sviluppatori software di terze parti in modo del tutto indipendente dal produttore dell'hardware fisico.
	
  * *Topologia (6 host, 3 switch):* Spiega come funziona l'inoltro dei pacchetti in questa topologia.

---

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
