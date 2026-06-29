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
	Per la risoluzione del terminale esposto, un nodo che si trova nel range del mittente ma non  in quello del destinatario riceverà l'RTS ma non ascolterà il successivo CTS. Da questo deduce che la ricezione avverrà lontano da lui e che una sua trasmissione verso un altro nodo non creerebbe collisioni a quel ricevitore, sentendosi libero di avviare una propria comunicazione. 
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
	Nel protocollo CSMA/CA, vengono prioritizzati i pacchetti grazie a questi IFS, Interframe Space, che sono tempi di attesa di default da rispettare. Nel caso del DIFS, abbiamo un tempo di attesa obbligatorio lungo che permette di evitare di interrompere messaggi a priorità maggiore. Invece il tempo di attesa più corto, ovvero il SIFS, viene rispettato dai pacchetti come quelli di handshake (RTS, CTS, ACK) perché hanno massima priorità. Il sistema funziona poiché DIFS > SIFS, quindi sicuramente l'apertura di una nuova comunicazione oppure l'invio di un frame dati, non interrompe i messaggi con priorità maggiore che partiranno sempre prima. 
	Inoltre, i tempi come il DIFS scorrono atomicamente in modo contiguo quando il canale è libero: se il canale diventa occupato prima del termine, il timer si azzera e si riparte da capo, a differenza del contatore dell'exponential backoff che invece si può congelare e riprendere da dove si era fermato.
---

### 2. Reti Cellulari (4G vs 5G)
* **Riconoscimento Architettura:** (Immagine con source/target BS) Che tipo di rete è? 4G o 5G? Perché? 
	È possibile capire che ci troviamo in una rete 4G nell'immagine grazie alle sue componenti specifiche: infatti possiamo notare che sono presenti un S-Gateway e un P-Gateway. Brevemente, la struttura di una rete cellulare 4G o 5G è composta da 3 blocchi funzionali: la RAN (Radio Access Network), che è la collezione delle base station che permettono l'accesso radio alla rete; la backhaul network, che rappresenta la rete di trasporto cablata; e la Core Network, dove avviene la gestione della mobilità, l'autenticazione, il pricing e la connettività verso Internet.
	
	Nel dettaglio, la rete 4G (chiamata EPC - Evolved Packet Core) è composta dall'**UE** (User Equipment), ovvero l'host wireless che dispone nella SIM di un identificativo univoco denominato IMSI (International Mobile Subscriber Identity) e implementa uno stack protocollare di rete completo. 
	L'**eNodeB** rappresenta il punto di accesso radio alla rete da parte dell'UE e ha il compito di comunicare con le altre base station per mitigare le interferenze, gestire la mobilità, eseguire l'handover e gestire il tunneling dei dati verso la core. 
	**L'MME** (Mobility Management Entity) rappresenta il cervello del piano di controllo della rete: si occupa dell'autenticazione, del tracciamento della posizione e della segnalazione per creare i tunnel.
	**L'HSS** rappresenta un database che contiene i profili degli abbonati, linterpellato per l'autenticazione e per salvarel'ultima posizione nota dell'UE. 
	**L'S-GW**  permette la comunicazione da e verso RAN e permette l'àncora di mobilità locale per il traffico dati (data plane) quando l'utente si sposta tra RAN diverse. 
	**Il P-GW** è l'ultimo punto di contatto tra la core e l'Internet pubblico: alloca gli indirizzi IP all'UE e applica funzionalità di NAT, QoS e policy.
	
	La rete 5G si differenzia nettamente dalla 4G poiché l'infrastruttura della core è stata stravolta per essere completamente cloud-native. 
	
	Sfrutta a pieno i concetti di SDN, NFV e Network Slicing: le funzioni di rete sono virtualizzate in modo da essere flessibili ed elastiche al variare del traffico. La core 5G Standalone non è retrocompatibile, e una delle poche cose rimaste invariate dal 4G è l'utilizzo del protocollo GTP-U (incapsulato su UDP) per fare tunneling a livello di data plane. 
	Già nel 4G c'era una parziale separazione dettata dai principi SDN: il control plane (MME, HSS) comunica tramite protocolli sicuri come SCTP, mentre i dati passano da S-GW e P-GW in GTP-U, con l'eNodeB che fa da ponte tra i due mondi. 
	
	Il 5G porta all'estremo questo concetto disaccoppiando totalmente il piano dati da quello di controllo. Nella core 5G troviamo nuove funzioni equivalenti a quelle 4G: al posto degli eNodeB ci sono i **gNodeB;** nel data plane, al posto di S-GW e P-GW abbiamo un unico modulo ad altissime prestazioni chiamato **UPF** (User Plane Function); le funzioni dell'MME vengono divise tra **AMF** (gestione accessi, mobilità e autenticazione) e **SMF** (gestione delle sessioni dati); al posto dell'HSS abbiamo **l'AUSF** (server per le logiche di autenticazione) e **l'UDM** (database per il mapping degli utenti). Infine, abbiamo il PCF, che sostituisce il vecchio PCRF per la gestione centralizzata delle policy di rete.

---

* **Handover:** Come funziona l'handover?
	L'handover permette di cambiare base station senza interrompere il segnale, ma parte dal presupposto che l'UE sia già connesso alla rete (ad esempio una visited network, se siamo in roaming). La procedura iniziale di attacco alla rete prevede che l'UE ascolti il primary e secondary sync della base station e decida di associarsi a quella con segnale migliore inviando il suo IMSI. La rete risponde avviando l'autenticazione: l'MME visitato comunica con l'HSS domestico, il quale fornisce le chiavi di autenticazione tramite un protocollo challenge-response (le chiavi vengono calcolate in parallelo anche dall'UE). Nel frattempo l'HSS domestico salva la nuova posizione. Con l'autenticazione che ha successo, dato che vige l'indirect routing, viene creato un tunnel fra il P-GW domestico e l'S-GW visitato. Nella rete visitata ue riceve un IP temporaneo, che S-GW tramite traduzione NAT può identificare.

	Quindi, a seguito dell'associazione e dell'autenticazione, vengono creati i tunnel fra P-GW domestico e S-GW visitato, e un tunnel fra S-GW visitato e l'eNodeB per poter comunicare con l'UE, utilizzando sempre il protocollo di tunneling GTP (e non GPT).
	
	Detto questo, il vero e proprio handover è possibile e avviene in questo modo: 1) BS decide tutto e invia una Handover Request alla target BS; 2) la target BS alloca le risorse radio necessarie e risponde inviando un Handover Ack; 3) la source BS comunica all'UE che l'handover può procedere e gli ordina di staccarsi e collegarsi alla nuova antenna; 4) durante questo istante di transizione, tutto il traffico in arrivo alla source BS non viene perso, ma viene inoltrato direttamente alla target BS; 5) la target BS dice all'MME che ora è lei la nuova BS di riferimento, e l'MME istruisce l'S-GW che semplicemente cambia l'endpoint del tunnel GTP facendolo puntare alla nuova BS; 6) la target BS dice alla source BS che l'handover ha avuto successo e la source BS rimuove e libera le risorse radio allocate precedentemente; 7) abbiamo ora il tunnel aggiornato alla target BS con connessione diretta all'UE.
	
	N.B. Tutta la logica di negoziazione è spostata all'edge, ovvero gestita direttamente tra le due base station senza coinvolgere la Core Network. Questo avviene proprio per garantire bassissima latenza nel cambio di cella e un'enorme scalabilità del sistema.
	
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
	OpenFlow rappresenta lo standard comunicativo fra controller e switch. Per generalized forwarding si intende un paradigma di inoltro non basato esclusivamente sull'indirizzo IP di destinazione, come nel caso del routing IP tradizionale, ma basato sulla possibilità di fare match su svariati campi degli header del pacchetto a diversi livelli. Nello specifico, la struttura permette di fare match a livello link , a livello di rete e a livello di trasporto. Questo permette di discriminare molte più casistiche e, di conseguenza, avere azioni più ricche. Le azioni possibili della flow table sono forward, drop, modify e send to controller.

	La cosa interessante è che con questo disaccoppiamento, dispositivi logici che prima risultavano necessariamente fisici e proprietari possono essere emulati programmando nella maniera corretta le entry della flow table. Lo stesso hardware generico può agire da switch, router, firewall o NAT, togliendo spesso la necessità di acquistare middlebox specifiche.
	
	Sappiamo che nelle SDN c'è il paradigma di disaccoppiamento fra la logica di controllo e il forwarding: nel control plane si definisce il comportamento della rete, e nel data plane si istruisce l'hardware su come applicare quel comportamento tramite le flow table. I messaggi "Controller-to-Switch" (e non simmetrici) sono inviati dal controller e includono Features (per ottenere le capacità dello switch), Configure (per impostare parametri), Packet-Out (per ordinare allo switch di far uscire un pacchetto da una specifica porta) e Flow-Mod (per aggiungere, modificare o eliminare regole dalla flow table). I messaggi "Asincroni" (e non asimmetrici) partono spontaneamente dallo switch verso il controller e includono Port-Status (indica un cambiamento fisico nella porta), Flow-Removed (indica che una entry della tabella è scaduta o è stata rimossa) e Packet-In (usato per inviare un pacchetto sconosciuto al controller affinché decida cosa farne). Infine ci sono i veri messaggi "Simmetrici", come Hello ed Echo, che possono essere generati da entrambe le parti per instaurare e mantenere viva la connessione.
	
---
* **Architettura SDN:** Descrivi Control Plane, Data Plane, Application Plane e API (Southbound/Northbound).
	L'architettura SDN si struttura su tre livelli fondamentali e indipendenti:
	
	**Data Plane (Piano Dati)**: È composto da switch semplici e veloci, il cui unico compito è  inoltrare meccanicamente i pacchetti. Non possiedono alcuna logica di routing interna, ma si limitano ad applicare le regole di *forwarding* (match-action) dettate dalle *flow table* fornitegli dal livello superiore.
	
	**Control Plane (Piano di Controllo)**: Rappresenta il coordinatore logicamente centralizzato della rete. Comunica verso il basso (con gli switch del Data Plane) tramite le **Southbound API**, utilizzando protocolli standard come OpenFlow. Comunica invece verso l'alto (con il livello applicativo) tramite le **Northbound API**. Ovviamente il controllo risulta distribuito in modo da avere fault tollerance e resilienza.
	
	**Application Plane (Piano Applicativo)**: È il vero "cervello" della rete SDN, dove operano le applicazioni che gestiscono le policy di rete (es. routing, load balancing, firewall e controllo degli accessi). Queste applicazioni comunicano le loro "intenzioni" al Control Plane tramite le Northbound API e, grazie al disaccoppiamento (unbundling), possono essere fornite da sviluppatori software di terze parti in modo del tutto indipendente dal produttore dell'hardware fisico.
	
  * *Topologia (6 host, 3 switch):* Spiega come funziona l'inoltro dei pacchetti in questa topologia.


---

* **Limitazioni di OpenFlow e superamento con P4:** Quali sono i limiti di OpenFlow e perché è stato superato da P4?
	OpenFlow nasce come protocollo semplice con circa 12 campi dell'header con cui fare match, ma ad oggi siamo arrivati a oltre 40, rendendone complesso e pesante l'uso. Inoltre, il moderno traffico dei datacenter introduce nuovi protocolli di incapsulamento (come nel caso delle VXLAN) che un hardware OpenFlow pre-impostato non è in grado di riconoscere e discriminare. La più grande problematica, infatti, risiede nel fatto che nelle SDN classiche il control plane è programmabile, ma il data plane non lo è. Quest'ultimo è progettato con funzionalità fisse dai vendor hardware (chip ASIC rigidi), e di conseguenza risulta impossibile gestire in modo efficiente e rapido protocolli non previsti in fase di fabbricazione.

	P4 permette di superare questo ostacolo introducendo un linguaggio di programmazione direttamente per il data plane. Oltre a stabilire _cosa_ deve succedere, P4 permette di specificare al chip _come_ processare i dati, definendo la composizione delle flow table e la struttura logica vera e propria dello switch. Il codice P4 viene compilato appoggiandosi a un _architecture model_ (fornito dal produttore del chip) ed è composto da tre stadi logici: un **Parser**, che scompone i pacchetti in ingresso estraendo esattamente i campi dell'header definiti dal programmatore; la pipeline **Match-Action**, dove i dati attraversano le flow table custom; e infine il **Deparser**, che ricompone l'header del pacchetto (applicando le eventuali modifiche) e lo inoltra. 
	Abbiamo la possibilità di specificare controlli personalizzati.
---
* **Applicazione Reale:** Esempio SDN (Google B4).
	Le reti WAN tradizionali sono basate sul principio di conservatività: sono composte da costosissimi collegamenti transatlantici e da router ad alte prestazioni. Per evitare perdite di dati in caso di guasti o picchi, i collegamenti vengono mantenuti a una saturazione del 30-40%, richiedendo un _overprovisioning_ (sovradimensionamento) del doppio o del triplo della capacità. Questo approccio tratta il traffico tutto allo stesso modo e rende impossibile fare Traffic Engineering in modo efficiente.
	
	Tutto questo porta a costi di scalabilità insostenibili, e per questo Google ha puntato sulla creazione di una rete inter-datacenter specifica chiamata **B4**. Google possiede due backbone separati: la **B2** per il traffico verso l'utente finale (Internet) e la **B4** per il traffico interno fra i datacenter. Questa separazione permette di discriminare il traffico nella B4 in base a latenza e priorità (ad esempio, le sincronizzazioni di backup in background hanno priorità bassa rispetto alle query live). Questo consente un Traffic Engineering molto aggressivo: la rete può accodare o bloccare pacchetti meno importanti se c'è un picco di traffico a maggiore priorità. Tutto questo è reso possibile dall'introduzione dell'SDN, che porta dinamicità e facilità di gestione centralizzata.
	
	B4, tuttavia, non è nata SDN da zero, ma derivava da una rete classica in cui i nodi effettuavano routing tramite protocolli standard (OSPF, IS-IS, BGP). È stata necessaria una trasformazione a fasi. Nella situazione intermedia sono stati aggiunti switch OpenFlow dotati di una sorta di modulo "glue", basato su Quagga per interagire con i moduli standard.
	A seguito è avvenuta la traslazione completa passando a tutti switch OF, mantenendo il protocollo BGP per i cluster. Questo permette di mantenere la comunicazioni con gli altri attori in gioco e favorisce il cambiamento.
	
	Nell'architettura SDN finale operano due coordinatori principali. Da un lato ci sono gli **OpenFlow Controller** di sito (cluster), che gestiscono localmente gli switch e astraggono l'hardware. Dall'altro, al di sopra di tutti, c'è il **Traffic Engineering (TE) Server** globale: avendo la mappa completa della rete, questo server gestisce i flussi di traffico e calcola i percorsi ottimali, permettendo a Google di far arrivare il consumo dei collegamenti fisici quasi al 100%, annullando gli sprechi.
---
*  **In che modo il forwarding generalizzato differisce dal forwarding basato sulla destinazione? Nel caso del destination-based forwarding, cosa viene confrontato e quale azione viene eseguita? Cosa si intende per operazione match-action di un router o switch?Nel caso del forwarding generalizzato in SDN, cita tre campi che possono essere confrontati (match) e tre azioni che possono essere eseguite.**

	Il forwarding destination-based si basa solo sul match del campo IP di destinazione (a livello di rete) e l'unica azione eseguita è l'inoltro (forwarding) sulla base del risultato della tabella di routing. Invece, con il forwarding generalizzato risulta possibile discriminare il traffico sulla base di molteplici campi dell'header del pacchetto a diversi livelli (link layer, network layer, transport layer). Nel caso delle reti SDN, tre esempi di campi che possono essere confrontati sono il MAC source, il MAC dest e l'IP source (o l'IP ToS). Questo si realizza attraverso l'operazione match-action, che consiste nell'effettuare specifiche azioni (come forward, drop e modify, ma anche send to controller) a seguito del riconoscimento di un pattern di flusso (ovvero i valori specifici assunti dai campi nell'header).

---

### 4. Teoria dei Segnali (Domanda Fissa)
* **Serie di Fourier:** Scrivi la formula di s(t) e dei coefficienti, spiegando a cosa serve.

	La teoria dei segnali è fondamentale: permette di estrarre informazioni, rimuovere il rumore, aumentare l'efficienza ed effettuare la conversione analogico-digitale.

	Un segnale è la variazione di un fenomeno fisico misurabile che porta informazione, dipendente da una o più variabili indipendenti. Noi ci interessiamo principalmente a segnali deterministici e monodimensionali, in cui l'unica variabile indipendente è il tempo.
	
	La Serie di Fourier si basa sul principio che un segnale periodico può essere scomposto come somma di infinite componenti oscillanti. Questo comporta un cambio di coordinate dal dominio del tempo al dominio della frequenza, usando funzioni base ortogonali dette $\phi_n(t)$.
	
	Infatti, un segnale continuo e periodico può essere rappresentato con il periodo base da $-\pi$ a $\pi$, la formula della Serie di Fourier è:
	
	$$s(t) = \frac{a_0}{2} + \sum_{n=1}^{\infty} \left[ a_n \cos(nt) + b_n \sin(nt) \right]$$
	
	La formula è composta da $a_0$, che (diviso per 2) rappresenta la componente continua, ovvero il valore medio del segnale nel periodo. A seguire troviamo la sommatoria delle armoniche, che rappresentano le funzioni oscillanti al variare della frequenza. I termini $a_n$ e $b_n$ sono i coefficienti di ampiezza. Nel dettaglio, $a_n$ è il coefficiente dei coseni, che indica il peso (la quantità) di funzioni cosinusoidali presenti nel segnale, mentre $b_n$ coefficiente dei seni, indica il peso delle componenti sinusoidali. 
	
	Questi coefficienti si calcolano tramite i seguenti integrali:
	
	$$a_0 = \frac{1}{\pi} \int_{-\pi}^{\pi} s(t) dt$$
	
	$$a_n = \frac{1}{\pi} \int_{-\pi}^{\pi} s(t) \cos(nt) dt$$
	
	$$b_n = \frac{1}{\pi} \int_{-\pi}^{\pi} s(t) \sin(nt) dt$$
	
	La serie di Fourier non è valida per qualunque funzione periodica. Non esiste una condizione necessaria, ma le condizioni sufficienti di Dirichlet affermano che, per avere una serie che converge nei reali, dobbiamo avere segnale continuo e continua a tratti, ovvero componenti finite nei sottointervali finiti e in caso di discontinuità limite che va ad un numero discreto.
	
	Sfruttando la formula di Eulero, i numeri complessi ci danno la possibilità di passare alla forma complessa della Serie di Fourier. I coefficienti $a_n$ e $b_n$ vengono racchiusi in un unico coefficiente complesso $S_n$, che rende la manipolazione matematica molto più agile.
	
	Nel caso di segnali non periodici, usiamo la Trasformata Continua di Fourier (CTFT). Questa prende un segnale aperiodico e lo considera "periodico" con un periodo $T$ che tende all'infinito. In questo modo, la frequenza fondamentale tende a $0$ e, di conseguenza, la distanza tra le armoniche si restringe infinitamente, facendo passare lo spettro da discreto a continuo ed estendendo i concetti di Fourier a tutti i segnali.
	
	Quindi, in sintesi: se la Serie di Fourier genera in output uno spettro di frequenze discreto (a valori definiti), la CTFT fornisce uno spettro continuo come output con frequenze che vanno da $-inf$ a $+inf$.
---

* **DFT, Campionamento e Quantizzazione:**
	### Digitalizzazione dei segnali: Campionamento e Quantizzazione

	L'elaborazione da segnale analogico a digitale avviene tramite due processi fondamentali: campionamento e quantizzazione. Sotto il rispetto delle condizioni di Nyquist-Shannon, il campionamento non introduce distorsione ed è teoricamente reversibile; al contrario, la quantizzazione è un'operazione irreversibile poiché introduce un errore di arrotondamento.
	
	#### 1. Campionamento (Teorema di Nyquist-Shannon)
	Nel campionamento operiamo nel dominio del tempo, estraendo il valore assunto dal segnale in istanti discreti, spaziati tra loro di un tempo $T_s$. La frequenza di campionamento è definita come $f_s = \frac{1}{T_s}$.
	
	Il teorema di Nyquist-Shannon afferma che, se un segnale ha banda limitata (ovvero il suo spettro diventa nullo per frequenze superiori a $f_{max}$), esso può essere ricostruito senza aliasing se e solo se la frequenza di campionamento soddisfa la condizione:
	
	$$f_s \geq 2 \cdot f_{max}$$
	
	
	
	Se questa condizione non viene rispettata ($f_s < 2 \cdot f_{max}$), si verifica l'**aliasing**: il campionamento crea delle repliche dello spettro che si sovrappongono. In tale scenario, non risulta possibile apprezare i cambiamenti del segnale.
	
	#### 2. Quantizzazione
	La quantizzazione opera nel dominio dell'ampiezza, trasformando ogni campione continuo in un valore intero rappresentabile con $R$ bit, in un intervallo compreso tra $0$ e $2^R - 1$. Poiché lo spazio dei valori ammissibili è finito, questa operazione comporta necessariamente un arrotondamento dei valori reali ai livelli di quantizzazione più vicini.
	
	* **Bit Rate**: Il numero di bit al secondo necessari per trasmettere il segnale è dato dal prodotto $R \cdot f_s$.
	* **Distorsione**: Per limitare l'errore di quantizzazione, occorre aumentare il numero di bit $R$. Tuttavia, all'aumentare di $R$ aumenta anche il bit rate, che deve essere compatibile con la capacità del canale di Shannon, definita come:
	
	$$C = B \cdot \log_2(1 + SNR)$$
	
	dove $B$ è la larghezza di banda del canale e $SNR$ il rapporto segnale-rumore.
	
	
	
	In sintesi, mentre il campionamento (se fatto correttamente) preserva l'informazione temporale, la quantizzazione sacrifica la precisione in ampiezza per rendere il segnale gestibile dai sistemi digitali.
	
  * *Discrete Fourier Transform (DFT):
	Nel contesto del Digital Signal Processing (DSP), la Trasformata Continua di Fourier (CTFT) non è direttamente implementabile poiché opera su segnali continui e su un dominio di frequenza infinito. Poiché i calcolatori digitali dispongono di memoria finita e possono gestire solo segnali campionati nel tempo, è necessario utilizzare la **Discrete Fourier Transform (DFT)**. La DFT prende in input una sequenza finita di $N$ campioni, denominati $s[k]$, e li trasforma in un array di $N$ coefficienti complessi nel dominio della frequenza. Essendo basata su una sommatoria finita, la DFT è facilmente calcolabile da un processore. La formula della DFT è: $$S[n] = \sum_{k=0}^{N-1} s[k] \cdot e^{-j \frac{2\pi}{N} nk}$$ Dove:  $s[k]$ è il segnale campionato nel tempo. $S[n]$ è il segnale nel dominio della frequenza (i coefficienti complessi).  $e^{-j \frac{2\pi}{N} nk}$ rappresenta il nucleo della trasformata, derivato dalla formula di Eulero. Grazie alla DFT, il computer può manipolare digitalmente i segnali, permettendo operazioni fondamentali come il filtraggio, la compressione (es. MP3, JPEG) e l'analisi spettrale. In pratica, per ottimizzare il calcolo di questa sommatoria, non si usa la formula diretta (che avrebbe una complessità computazionale di $O(N^2)$), ma l'algoritmo **FFT (Fast Fourier Transform)**, che riduce la complessità a $O(N \log N)$, rendendo possibile l'elaborazione dei segnali in tempo reale.
