# MQTT

**1. Paradigma Publish/Subscribe:

Il **paradigma Publish/Subscribe** (Pub/Sub) rappresenta uno schema di interazione debolmente accoppiato (_loosely coupled_), ideato come alternativa al classico modello client/server. In MQTT, i tre attori principali interagiscono nel modo seguente:

- **Publishers:** Client che producono eventi o dati (es. sensori di temperatura).
- **Subscribers:** Client che esprimono interesse per determinati argomenti (topic) per riceverne le notifiche.
- **Broker (Event Service):** Il server centrale, noto a tutti i client, che riceve i messaggi in ingresso dai publisher, applica un filtraggio (soggettivo o basato sul topic) e li distribuisce ai relativi subscriber.

Produttori (publishers) e i consumatori (subscribers) di dati sono **completamente disaccoppiati sotto tre dimensioni fondamentali**:

1. **Disaccoppiamento Spaziale:** I publisher e i subscriber **non hanno bisogno di conoscersi**. L'unico punto di contatto comune è l'indirizzo IP/hostname del broker.
2. **Disaccoppiamento Temporale:** I publisher e i subscriber **non devono essere necessariamente attivi e connessi alla rete nello stesso istante**.
3. **Disaccoppiamento di Sincronizzazione (Synchronization Decoupling):** Le operazioni di invio (_Publish_) o ricezione (_Notify_) **non bloccano il flusso di esecuzione** del codice dei client. 

MQTT è un protocollo di livello applicazione (OSI 5-6-7) che si appoggia direttamente sulla stabilità di TCP (livello 4, porta 1883 o 8883 con SSL)**.

MQTT estende e trasforma le capacità di trasporto native di TCP/IP attraverso questi pilastri ingegneristici:

- **Da Canale Punto-a-Punto a Distribuzione Multi-punto:** La connessione TCP è intrinsecamente orientata alla connessione 1-to-1, richiedendo un accoppiamento forte tra i due nodi.  MQTT, agendo da middleware di brokerage, converte questo stream TCP in flussi di comunicazione flessibili di tipo **1-to-N (multicast applicativo)**, **1-to-1** o persino **1-to-0** (se un dato viene pubblicato ma nessun subscriber vi è registrato).
- **Quality of Service (QoS) di Livello Applicativo:** Il protocollo TCP garantisce la consegna ordinata e affidabile dei pacchetti, ma questa garanzia **decade istantaneamente non appena uno dei due peer si disconnette** (poiché la sessione TCP muore). MQTT estende questa affidabilità introducendo **tre livelli di QoS (0, 1, 2)** gestiti a livello applicativo dal broker, consentendo di garantire la consegna dei dati (es. _Exactly Once_ con il QoS 2) anche attraverso disconnessioni fisiche multiple e riavvii dei nodi.
- **Filtraggio Semantico (Subject-based Filtering):** Mentre TCP si limita a instradare flussi di byte grezzi basandosi su IP e porte, MQTT introduce un'astrazione logica basata su topic strutturati in gerarchiche  (es. `casa/cucina/temperatura`), permettendo ai nodi di ricevere solo i dati applicativi di reale interesse.
- **Scalabilità Distribuita del Broker:** In un approccio client/server TCP tradizionale, la scalabilità risente dell'accoppiamento rigido. Con il broker di MQTT, le operazioni di instradamento ed elaborazione degli eventi possono essere parallelizzate su macchine distribuite orizzontalmente, consentendo di scalare la rete per accogliere decine di migliaia di dispositivi IoT simultaneamente


---

**2. Filtraggio per tipo:** Fai un esempio di filtraggio dei messaggi. (Risposta: In MQTT il filtraggio avviene tramite i _Topic_, organizzati gerarchicamente, es. `casa/cucina/sensoretemperatura`).

l filtraggio dei messaggi è l'operazione con cui il Broker seleziona quali messaggi inoltrare a quali subscriber. Esistono **tre diverse metodologie di filtraggio**:

1. **Filtraggio basato sul topic:** È il meccanismo nativo e fondamentale di **MQTT**. I messaggi contengono un tag testuale chiamato **Topic**. I client si iscrivono a determinati topic e il broker distribuisce i messaggi di conseguenza.
    - _Esempio:_ Un sensore pubblica sul topic `casa/cucina/temperatura`. Solo i client sottoscritti a quel topic esatto (o a pattern compatibili) riceveranno il dato.
2. **Filtraggio basato sul Contenuto:** I client si iscrivono definendo una vera e propria query logica sulle variabili del payload. Il broker deve aprire e analizzare il contenuto del pacchetto che deve essere in chiaro.
    - _Esempio:_ Il subscriber si registra con la query `temperatura > 30°`. Il broker riceve il dato, legge il payload e inoltra il messaggio solo se la condizione è vera.
    
3. **Filtraggio basato sul Tipo (Type-based):** Il filtraggio avviene in base alla struttura dati o alla **classe/tipo dell'oggetto** che rappresenta l'evento.

MQTT implementa il **TOPIC based filtering**, i topic sono stringhe strutturate gerarchicamente in livelli separati dal carattere `/`. 

I client devono accordarsi preventivamente sui topic.

Ci sono due **wildcard** utilizzabili esclusivamente in fase di sottoscrizione:

- **Wildcard a livello singolo (****+****):** Sostituisce **un singolo livello** della gerarchia.
    - _Esempio:_ Sottoscrivendosi a `home/firstfloor/+/presence`, il client riceverà i dati di presenza di qualsiasi stanza del primo piano (es. `home/firstfloor/bedroom/presence` o `home/firstfloor/kitchen/presence`), ma non da altri livelli o sotto-livelli.
- **Wildcard multilivello (****#****):** Sostituisce **qualsiasi numero di livelli successivi**. Deve essere posizionata tassativamente alla fine del topic.
    - _Esempio:_ Sottoscrivendosi a `home/firstfloor/#`, il client riceverà qualsiasi pubblicazione che inizi con quel prefisso (es. `home/firstfloor/bedroom/presence`, `home/firstfloor/kitchen/temperature/sensor1`, ecc.).
I topic che iniziano con il carattere dolalro (come `$SYS/broker/clients/total`) sono riservati dal sistema per le statistiche interne del broker.

---

- **Notify:** Cos'è un Notify? (Concetto di notifica asincrona di un evento verso i subscriber).
	Nel paradigma Publish/Subscribe, la **Notify** rappresenta la **primitiva di comunicazione asincrona** attraverso cui il Broker inoltra un evento o un messaggio a un _subscriber_.
	Quando un _publisher_, invia un messaggio al broker tramite l'operazione di `Publish`. Il broker riceve il messaggio, ne analizza il topic e si occupa di **notificarlo** (tramite la primitiva `Notify`) a tutti i client che avevano precedentemente espresso interesse per quel soggetto registrandosi con una `Subscribe`.

	La `Notify` è l'elemento che concretizza il disaccoppiamento del paradigma Pub/Sub:
	**Disaccoppiamento di Sincronizzazione:** Il subscriber non interroga continuamente il broker e non rimane bloccato in attesa. Esso continua a svolgere le sue attività locali; sarà il broker a interromperlo asincronamente inviandogli la notifica non appena il dato è disponibile. 
	
	**Disaccoppiamento Temporale:** Se il subscriber è offline, grazie alle _Persistent Sessions_ (e a QoS 1 o 2), il broker memorizza la notifica nel suo buffer interno e la recapiterà non appena il client si ricollegherà alla rete.


---

- **Scelta della QoS (Esempio QoS 2):** Qual è un caso tipico in cui serve la QoS 2 (Exactly once) e perché i duplicati non sarebbero tollerati?
	In MQTT, la **Quality of Service (QoS)** rappresenta un accordo sul livello di affidabilità della consegna dei messaggi tra il mittente e il destinatario. È fondamentale sottolineare che in MQTT **il flusso di QoS è asimmetrico**: esiste un accordo di QoS tra _Publisher_ e _Broker_, e un accordo indipendente tra _Broker_ e _Subscriber_. Il QoS effettivo con cui il messaggio viene recapitato al destinatario finale è pari al **minimo** tra i due livelli configurati (min(QoSpub​,QoSsub​))

	**QoS 0 (At most once - Al massimo una volta):** È un meccanismo _best-effort_ in cui i messaggi non vengono confermati dal destinatario né memorizzati dal broker. Nel pacchetto `PUBLISH` il `packetId` è impostato a 0. Garantisce la consegna solo finché la sessione TCP rimane attiva; se la connessione cade, il messaggio va perso.
    - _Caso d'uso:_ Letture frequenti di sensori ambientali (es. temperatura della stanza) in cui il dato corrente invecchia rapidamente e l'arrivo costante di campioni freschi rende irrilevante la perdita occasionale di un pacchetto.
    **QoS 1 (At least once - Almeno una volta):** Garantisce che il messaggio arrivi a destinazione, ma tollera la possibilità che si verifichino duplicati. Il mittente memorizza il messaggio e lo invia con un `packetId` valido. Il destinatario deve rispondere con un pacchetto **PUBACK**. Se il `PUBACK` va perso a causa di un'interruzione di rete, il mittente esegue una ritrasmissione impostando a `true` il flag **dupFlag** (duplicate).
    - _Caso d'uso:_ Notifica di allarme o stato (es. "Porta aperta"). L'importante è che l'evento sia notificato; se arriva due volte, l'applicazione ricevente può ignorare il duplicato in sicurezza.
	**QoS 2 (Exactly once - Esattamente una volta):** È il livello massimo di affidabilità (e il più lento, a causa dell'overhead). Garantisce che il messaggio venga consegnato all'applicazione destinataria **esattamente una volta**, eliminando qualsiasi duplicato.

	Per evitare la duplicazione del messaggio senza perdere dati, MQTT implementa un **doppio handshake a due vie** (:
1. **Invio del messaggio (****PUBLISH QoS 2****): Il mittente invia il messaggio.
2. **Ricezione e tracciamento dello stato (****PUBREC****):** Il destinatario riceve il `PUBLISH`, ne memorizza il `packetId` per identificare eventuali futuri duplicati e risponde con un pacchetto di _Publish Received_ (**PUBREC**). Il destinatario **non consegna ancora il messaggio all'applicazione** (lo tiene "congelato" nello stato del broker).
3. **Rilascio del messaggio (****PUBREL****):** Il mittente riceve il `PUBREC`, capisce che il destinatario ha preso in carico il pacchetto, **cancella il messaggio originale e risponde con un pacchetto di _Publish Release_ (**PUBREL**), dando il permesso di sbloccare il messaggio.
4. **Conferma di completamento (****PUBCOMP****):** Il destinatario riceve il `PUBREL`, consegna finalmente il messaggio all'applicazione destinataria, cancella lo stato associato a quel `packetId` (sapendo che non arriveranno più duplicati) e invia un pacchetto di _Publish Complete_ (**PUBCOMP**).

	Se ci fermassimo al primo scambio (`PUBLISH` → `PUBREC`), saremmo in una situazione analoga al QoS 1. Se il pacchetto `PUBREC` andasse perso sulla rete, il mittente, non ricevendo conferma, provvederebbe a **reinviare l'intero pacchetto** **PUBLISH**. Senza il secondo scambio (`PUBREL` → `PUBCOMP`).


	**Il sensore di insulina:** Se l'applicazione invia il comando non idempotente _"Somministra 1 unità di insulina"_, l'eventuale ricezione di un duplicato dovuto a una riconnessione TCP causerebbe la somministrazione di un'ulteriore unità, con conseguenze potenzialmente letali per il paziente.
	Transazioni bancarie:** Il comando _"Preleva 50€ dal conto"_ se duplicato dimezzerebbe il saldo in modo errato. In questi casi, garantire la consegna _Exactly Once_ è un requisito hardware e software non negoziabile

---

- **Sessioni e Fallimenti:** Cosa sono le sessioni persistenti in MQTT e a cosa servono? 
	Le sessioni persistenti che vengono attivante quando il cleanfòag in connessione viene  messo a falso, permetto che il broker salvi i emssaggi non consegnati con QOS 1 e 2 e le sottoscrizioni richeiste, inmodo che il subscribere a seguito della riconessione non debba riinserirre tutto
	Anche il client deve salvare tutti i messaggi ricevuti/inviati al broker non acked in modo da poterli conservare in seguito.
	Ovviamenre metter eil flag a flase non ha senso se si sua QoS 0 abbiamo overhead inutile.


---

- **Messaggi Retained:** Cosa sono i messaggi trattenuti (retained messages)? Fai un esempio.    
	Un retained messagge viene settatpo a seguito del publish nel flag reataine e indic ahc eil emssaggi deve essere salvato nel broker e appena qualcuno si iscrive al nuovo topic riceve il msg. Meccanismco importnae poichè che i dispositvi iot sono spesso in sleep e di cosnegeunz apoitrebbero non comunciare per lassi di tempo grandi. Tipico esempio quando mi collego con una lampadine il retained esasge è on così mia ccendo e va a baccetto col last wil messasggeà
	Si aggiorna ad ogni novo messagigio retained nviato dal topic e per eliianrlo nasata mandar un messaggio vuoto
---

- Una connessione può fallire in MQTT? Cosa facciamo quando fallisce? (Risposta: Last Will and Testament - LWT).
	Sebbene il protocollo di trasporto TCP fornisca un canale affidabile orientato alla connessione, esso soffre di una grave limitazione intrinseca nei contesti IoT: **non è in grado di rilevare immediatamente la disconnessione silenziosa di un host**. Se un dispositivo perde l'alimentazione la connessione TCP dal lato del Broker rimane aperta in uno stato inconsistente (_half-open_), poiché non è stato eseguito l'handshake di chiusura standard (FIN/ACK). Di conseguenza, il Broker (e l'intera rete IoT) continuerebbe a considerare attivo un nodo che in realtà è spento o danneggiato.

	Per risolvere questo problema di consistenza, MQTT introduce l'azione combinata di due meccanismi di livello applicativo: il **Keep Alive** e il **Last Will & Testament (LWT)**.

	Durante la fase di connessione, il client specifica nel pacchetto `CONNECT` un parametro a 16 bit chiamato **Keep Alive**, espresso in secondi. Questo rappresenta il tempo massimo che può trascorrere senza che il client invii alcun messaggio al Broker.
	
	Se il client non ha dati applicativi da trasmettere, invia periodicamente un pacchetto di controllo leggerissimo chiamato **PINGREQ** . Il Broker risponde con un pacchetto **PINGRESP** confermando che il canale è attivo.
	
	Se il Broker non riceve alcun pacchetto (né dati né `PINGREQ`) entro la scadenza dell'intervallo di Keep Alive  assume che la connessione sia fallita e attiva il meccanismo di **Last Will & Testament**.
	L'LWT consente a un client di definire preventivamente un "testamento" che il Broker dovrà pubblicare asincronamente per suo conto solo in caso di disconnessione anomala.
	
	**Configurazione (Will Flags):** Questo messaggio viene configurato all'inizio, inserendo nel payload del pacchetto **CONNECT** quattro campi dedicati:
	
	**willTopic****: il topic su cui pubblicare il testamento.
	**willMessage** **(Will Payload):** il contenuto informativo del testamento.
	**willQoS****: il livello di Quality of Service associato (0, 1 o 2).
	**willRetain****: il flag booleano per definire se il messaggio deve essere mantenuto memorizzato sul Broker.

	**Le cause di attivazione (Quando viene inviato):*
	- Rilevamento di un errore di I/O a livello di rete di trasporto.
	- Mancata ricezione dei messaggi di Keep Alive (`PINGREQ`) nei tempi previsti.
	- Chiusura improvvisa del socket TCP da parte del client senza aver prima inviato il pacchetto di chiusura applicativa **DISCONNECT**.
	- Chiusura della connessione da parte del Broker a causa di un errore di protocollo del client.

	Al contrario, se il client si scollega regolarmente inviando un pacchetto **DISCONNECT**, il Broker **cancella immediatamente l'LWT memorizzato** senza pubblicarlo

	L'utilizzo più potente e frequente dell'LWT avviene in accoppiata con i **Retained Messages** per mantenere aggiornato lo stato dei dispositivi nella rete:

	1. **Normali operazioni:** Una lampadina intelligente si connette al Broker inviando un pacchetto `CONNECT` nel quale imposta come **LWT** un messaggio con topic `home/devices/lamp1/status`, payload **"OFF"** e flag **willRetain = TRUE**. Una volta connessa, la lampadina pubblica sullo stesso topic un messaggio **Retained** con valore **"ON"**.
	2. **Consistenza per i Subscriber attuali:** Se un subscriber si iscrive a quel topic, riceve immediatamente `"ON"`.
	3. **Blackout improvviso:** La lampadina si rompe o salta la corrente. Il Broker non riceve il `PINGREQ` entro il Keep Alive, chiude la sessione e invia automaticamente l'LWT: un messaggio `"OFF"` sul topic `home/devices/lamp1/status`. Tutti i subscriber attivi vengono notificati immediatamente della disconnessione e sanno che la lampadina è ora offline.
	4. **Consistenza per i Subscriber futuri:** Poiché il testamento era stato configurato con `willRetain = TRUE`, il Broker sovrascrive il precedente stato memorizzato `"ON"` con il nuovo valore `"OFF"`. Quando un nuovo client si collegherà alla rete ore dopo e si sottoscriverà al topic, riceverà immediatamente `"OFF"`, evitando la grave inconsistenza di leggere un obsoleto stato `"ON"` per un dispositivo fisicamente spento o rotto
---
---
---

# ZIGBEE

- **Capacità della Rete:** Quante reti ci possono essere in una determinata area? (Pensa al 802.15.4).
	La capacità di rete nello standard IEEE 802.15.4 e in Zigbee si analizza distinguendo tra il **limite fisico di interferenza** e il **limite logico di indirizzamento**:

	Per far funzionare più reti nella stessa area **senza che si disturbino a vicenda**, ogni rete deve essere allocata su un canale radio fisicamente separato (ortogonale).

	**Banda 2.4 GHz (Worldwide):** Lo standard definisce **16 canali ortogonali** (canali 11-26). Teoricamente, possiamo avere **16 reti fisiche indipendenti**. Tuttavia, a livello pratico, i dispositivi commerciali Zigbee supportano solitamente solo **15 canali** (escludendo il canale 26 per rispettare le severe normative sulle emissioni spettrali o evitare interferenze), limitando il numero a **massimo 15 reti fisiche indipendenti**.
   
	**Banda 868 MHz (Europa):** Lo standard definisce **un solo canale fisico** (canale 0 a 868.3 MHz). In questa banda, in Europa, **non è possibile avere reti fisicamente isolate**: tutte le reti presenti nell'area devono condividere lo stesso canale radio.

	Se accettiamo che le reti condividano lo stesso canale fisico (contendendo il mezzo trasmissivo tramite il protocollo **CSMA-CA** dell'802.15.4), lo standard permette di separare logicamente le comunicazioni:

---

        
- **Dispositivi e Indirizzi:** * Quanti dispositivi ci possono essere in una rete? (Risposta: max 65536, lo short address è a 16-bit).
	Il numero massimo di dispositivi che possono teoricamente coesistere in una singola rete Zigbee (PAN) è pari a 216=65.536, poiché lo standard utilizza un **indirizzo logico a 16 bit** (_short network address_ o NWK_ADDR) per l'instradamento dei pacchetti.

	All'interno della rete, i dispositivi si dividono in tre ruoli logici distinti:

	- **Zigbee Coordinator (ZC):** Rappresenta la radice dell'albero, inizializza la rete e assume tassativamente l'indirizzo statico **0x0000**.
	- **Zigbee Router (ZR):** Nodi interni dell'albero che estendono la copertura della rete, abilitano il routing dei pacchetti e possono ospitare nodi figli.
	- **Zigbee End Device (ZED):** Rappresentano le foglie dell'albero (dispositivi di tipo RFD - _Reduced Function Device_ o FFD configurati come foglie). Non fanno routing e possono comunicare esclusivamente tramite il proprio nodo padre
---
- Come vengono calcolati/assegnati gli indirizzi in ZigBee? (Calcolo con il Cskip e distribuzione dei blocchi).
	Per evitare collisioni di indirizzi senza ricorrere a costosi protocolli di consenso distribuiti o a interrogazioni continue al coordinatore, Zigbee adotta uno **schema di indirizzamento statico e decentralizzato basato su una topologia logica ad albero simmetrico**.

	Il coordinatore definisce staticamente a tempo di compilazione tre parametri geometrici della rete:

	- Rm​ **(Max Routers):** Il numero massimo di router figli che un coordinatore o un router può ospitare.
	- Dm​ **(Max End Devices):** Il numero massimo di end device figli che un coordinatore o un router può ospitare.
	- Lm​ **(Max Depth):** La profondità massima dell'albero di rete.


	La funzione Cskip(d) rappresenta a porzione dello spazio di indirizzamento totale che un router posizionato a profondità d riceve dal proprio padre per poterlo distribuire ai propri discendenti (incluso se stesso).


	Il calcolo di Cskip(d) parte dal livello di profondità massima Lm​. A profondità massima, un router non può avere figli, pertanto il suo blocco contiene un solo indirizzo (quello del router stesso): Cskip(Lm​)=1

	Per un generico livello di profondità d<Lm​, la dimensione del blocco è definita dalla relazione di ricorrenza: Cskip(d)=1+Rm​⋅Cskip(d+1)+Dm​


	Un nodo padre (sia esso il Coordinatore o un Router) posizionato a profondità d con indirizzo di rete Aparent​, assegna gli indirizzi ai propri nodi figli applicando le seguenti regole matematiche deterministiche:

    ---
        
- **Analisi dell'albero (Immagine: Zigbee addresses - the tree):** Spiega l'assegnazione e il routing degli indirizzi guardando la struttura ad albero.
	In una rete Zigbee con topologia logica ad albero simmetrico (definita dai parametri geometrici Rm​, Dm​ e Lm​), lo spazio degli indirizzi a 16 bit viene distribuito gerarchicamente dall'alto verso il basso. Questo schema statico e deterministico consente di implementare il **Tree Routing**, un protocollo di instradamento estremamente efficiente e leggero che **non richiede tabelle di routing** memorizzate nei nodi sensori (ottimo per dispositivi constrained con RAM ridotta).

	Ogni router posizionato a profondità d con indirizzo di rete A conosce la dimensione del proprio blocco di indirizzi, definita dal parametro Cskip(d) (calcolato a tempo di compilazione).

	Quando il router A riceve un pacchetto destinato all'indirizzo D, applica un algoritmo decisionale basato su tre semplici controlli matematici:

	 **Verifica di Consegna Diretta (Figlio Diretto):** Il router controlla se la destinazione D corrisponde all'indirizzo di uno dei suoi figli diretti (router o end device). In caso positivo, invia il pacchetto direttamente al nodo di destinazione tramite collegamento fisico a 1 hop.
	
	 **Verifica di Discendenza (Downward Routing):** Il router verifica se la destinazione D fa parte dei suoi discendenti all'interno del proprio sotto-albero. Matematicamente, D si trova nel sotto-albero di A se e solo se è soddisfatta la seguente disuguaglianza: A<D<A+Cskip(d)
	 
	 **Se la condizione è VERA:** Il pacchetto deve essere inoltrato **verso il basso** (_downward_). Per identificare il corretto ramo discendente, il router individua il router figlio i-esimo (con i∈[1,Rm​]) che soddisfa la condizione:
    - Arouter,i​≤D<Arouter,i​+Cskip(d+1)
    - Una volta identificata la diramazione corretta, il pacchetto viene trasmesso a quel router figlio (che fungerà da _next-hop_).
    
     **Inoltro al Padre (Upward Routing):**
    - **Se la condizione di discendenza è FALSA** (ovvero se D≤A oppure D≥A+Cskip(d)), la destinazione si trova al di fuori della competenza del sotto-albero del router.
    - **Azione:** Il router inoltra immediatamente il pacchetto **verso l'alto**, trasmettendolo al proprio **nodo padre**. Il padre eseguirà a sua volta lo stesso algoritmo logico
---

-  **Il ruolo del Router:** Perché un router in ZigBee ha bisogno di avere un indirizzo di rete assegnato?
	In una rete Zigbee, un **Router** è un dispositivo di tipo **FFD (Full Function Device)** che svolge un ruolo infrastrutturale attivo. A differenza degli End Device, il Router deve **tassativamente possedere un proprio indirizzo di rete a 16 bit**
	
	Zigbee adotta uno schema di indirizzamento ad albero completamente **decentralizzato**: il Coordinatore non assegna gli indirizzi a tutta la rete, ma distribuisce dei blocchi (_range_) ai propri router figli, i quali faranno lo stesso con i loro discendenti. Quando un nuovo nodo richiede l'associazione a un Router, quest'ultimo agisce come padre e calcola matematicamente l'indirizzo da assegnare al figlio.


	Nel routing ad albero (_Tree Routing_), i pacchetti vengono inoltrati senza l'uso di tabelle, basandosi esclusivamente sulla struttura matematica degli indirizzi. Quando un Router con indirizzo A a profondità d riceve un pacchetto destinato a D, deve decidere se mandarlo verso il basso (ai suoi discendenti) o verso l'alto (al proprio padre).
	
	Se il Router non possedesse l'indirizzo A, non potrebbe definire i confini matematici del proprio sotto-albero, rendendo impossibile qualsiasi decisione di instradamento.


	Zigbee consente la coesistenza di Tree Routing e **Mesh Routing** (basato sul protocollo AODV). In modalità mesh, i Router mantengono una **Routing Table (RT)** per instradare i pacchetti lungo cammini ottimi.
	Ogni riga della Routing Table contiene il campo **Next-hop Address** (l'indirizzo a 16 bit del router successivo lungo il cammino).
	Durante la fase di scoperta dei percorsi (_Route Discovery_), i router propagano pacchetti di `RREQ` (Route Request) e `RREP` (Route Reply) registrando gli indirizzi dei nodi intermedi per stabilire le rotte.

	Un Router Zigbee non è un semplice ripetitore di segnale, ma è un nodo attivo che può ospitare a bordo fino a 240 applicazioni client/server (**Application Objects - APO**), ciascuna attestata su un Endpoint (da 1 a 240).
	Qualsiasi risorsa o servizio applicativo sulla rete è identificato univocamente dalla coppia **<network address, endpoint>**.
	
	Anche nel join è importante mica posso solo comunciare col coordinator.

---

- Spiega il processo di Join in ZigBee. 
	In Zigbee, un dispositivo può entrare a far parte di una rete esistente principalmente in due modi: tramite **Direct Join** (avviato dal padre) o tramite **Join through Association** (avviato dal figlio che desidera associarsi).
	
	Il processo di **Join per Associazione (Child-Side)** si articola in tre macro-fasi protocollari, attraversando l'intero stack (Application Layer, Network Layer e MAC Layer):

	1. **Inizio dell'Application Layer:** L'applicazione del dispositivo che vuole fare il join invoca la primitiva **NETWORK-DISCOVERY.request**, inviandola al livello inferiore (Network Layer - NWK).
	2. **Richiesta di Scansione:** Il Network Layer traduce questa chiamata ed emette una primitiva **SCAN.request** verso il livello MAC.
	3. **Active Scan a livello radio:** Il livello MAC esegue fisicamente un **Active Scan** (scansione attiva) inviando dei _Beacon Request_ nell'etere e ascoltando i _Beacon_ emessi dai Coordinatori o dai Router attivi nelle vicinanze.
	4. **Raccolta dei risultati:** Una volta completata la scansione, il MAC risponde al livello superiore con la primitiva **SCAN.confirm**, contenente la lista dei PAN ID rilevati, dei canali radio e della potenza del segnale (LQI).
	5. **Notifica all'Applicazione:** Il Network Layer propaga queste informazioni all'Application Layer tramite la primitiva **NETWORK-DISCOVERY.confirm**.
	6. **Selezione della Rete:** L'Application Layer analizza la lista dei nodi vicini, sceglie la PAN a cui collegarsi (cercando di associarsi al dispositivo più in alto possibile nell'albero per minimizzare gli hop) e invoca la primitiva **JOIN.request**. Questa primitiva contiene due parametri fondamentali:
	    - Il **PAN Identifier** della rete selezionata.
	    - Un flag che indica se il dispositivo si sta associando come **Router** o come **End Device**.
	7. **Inizio dell'Associazione:** Ricevuta la richiesta, il Network Layer individua il nodo padre target (un Router o il Coordinatore) e invoca la primitiva MAC **ASSOCIATE.request**.

	Questa è la parte più critica, regolata dallo standard IEEE 802.15.4, e si basa sulla **comunicazione indiretta** per consentire il risparmio energetico:

	1. **Invio della Richiesta:** Il MAC del dispositivo invia un frame fisico di **Association request** al nodo padre.
	2. **ACK di Ricezione:** Il MAC del padre risponde immediatamente con un **ACK** di livello fisico. _Attenzione:_ questo ACK conferma solo la corretta ricezione del pacchetto, ma **non** significa che l'associazione sia stata accettata.
	3. **Decisione del Padre:** Il MAC del padre inoltra la richiesta al proprio Network Layer tramite la primitiva **ASSOCIATE.indication**. Il Network Layer decide se accettare il nodo (verificando di non aver superato i limiti Rm​ o Dm​ di figli gestibili) e, in caso positivo, **calcola matematicamente e assegna al nuovo figlio uno short address a 16 bit**. Quindi, invoca la primitiva **ASSOCIATE.response**.
	4. **La fase di Polling:** Poiché l'associazione è una transazione indiretta, il dispositivo figlio non rimane in ascolto continuo per non consumare batteria. Esso attende un tempo prefissato (**Pre-defined waiting time**) e poi invia attivamente una **Data request** (operazione di _polling_) per chiedere se ci sono risposte pendenti per lui.
	5. **Consegna dell'Indirizzo:** Al ricevimento della _Data request_, il MAC del padre invia finalmente il frame di **Association response** contenente lo short address a 16 bit appena assegnato.
	6. **Conferma Finale:** Il figlio risponde con un **ACK**. Il suo livello MAC genera una primitiva **ASSOCIATE.confirm** per il livello NWK, il quale chiude la transazione verso l'applicazione con la primitiva **JOIN.confirm** con stato di _Success_. Da questo momento in poi, il dispositivo utilizzerà lo short address a 16 bit per ogni comunicazione nella rete
	
    ---
    
	
	
- Qual è il risultato di uno scan? Come fa un dispositivo a scegliere a quale PAN ID associarsi?
	    L'operazione di scansione viene avviata a livello applicativo tramite l'invocazione della primitiva **NETWORK-DISCOVERY.request**. Il livello Network (NWK) riceve la chiamata e la traduce per il livello inferiore inviando una primitiva **SCAN.request** al livello MAC.

	Il livello MAC esegue fisicamente un **Active Scan** (scansione attiva) nell'etere trasmettendo frame di _Beacon Request_ e ascoltando i _Beacon_ emessi dai Coordinatori o dai Router attivi nel vicinato. Al termine, il MAC risponde al livello Network con la primitiva **SCAN.confirm**, la quale risale infine all'applicazione sotto forma di primitiva **NETWORK-DISCOVERY.confirm**.

	Il risultato contenuto in questa conferma è una lista dettagliata di tutte le reti personali (PAN) rilevate nel raggio di copertura radio del dispositivo, la quale specifica per ciascuna:

	- Il **PAN ID** (Personal Area Network Identifier) della rete attiva rilevata.
	- L'**indirizzo a 64-bit** (o lo short address) del dispositivo coordinatore o del router che ha emesso il beacon, che si propone come potenziale nodo padre (P).
	- Il **canale radio** su cui la rete sta operando e l'indicazione di quanto tale canale sia disturbato o rumoroso (Energy Detection).

	Una volta ricevuta la lista delle reti disponibili il livello applicativo seleziona la PAN a cui desidera connettersi e avvia la primitiva **JOIN.request**.

	La selezione del nodo padre specifico (P) all'interno del vicinato radio e il ruolo con cui il dispositivo entrerà nella rete dipendono strettamente dalla topologia logica adottata:

- **In una topologia a stella (Star Topology):** La scelta è obbligata. Il nodo padre P deve essere tassativamente il **Coordinatore** della PAN, e il dispositivo richiedente può associarsi esclusivamente con il ruolo di **End Device**.
- **In una topologia ad albero o mesh (Tree/Mesh Topology):** Il nodo padre P può essere sia il **Coordinatore** della rete sia uno dei **Router** attivi. Il dispositivo può scegliere se effettuare il join come _Router_ (se ha capacità di Full Function Device per estendere la rete) o come _End Device_ (Reduced Function Device).

---

-  _Domanda trabocchetto:_ Perché c'è una "Data Request" dal client verso il router durante il join? (Risposta: Per comunicare serve un indirizzo di rete, il client ne è ancora sprovvisto, quindi deve essere lui a inizializzare lo scambio per farsi assegnare lo short address).
	In una rete wireless constrained, **il genitore non può fare** **push** **della risposta** perché non sa se in quel preciso millisecondo il client ha la radio accesa o se è in sleep per risparmiare batteria. Lo standard IEEE 802.15.4 risolve questo problema spostando l'iniziativa della transazione sul client (modello **PULL**): è sempre il dispositivo che deve ricevere il dato a dover dichiarare di essere sveglio e pronto all'ascolto tramite l'invio della **Data Request**.
	Inoltre non dispone ancora delll'indirizzo dir ete per comunciare e quindi il apdre non può comunciare direttametne col figlio
	
---

- Disegna il diagramma dell'Associate Request.
	
![[Pasted image 20260717172134.png]]

---
    
- Architettura generale e come si trova il canale giusto a cui connettersi.
	Il processo di creazione di una nuova rete personale wireless (PAN) è noto come **Network Formation** ed è ad esclusivo carico di un dispositivo di tipo FFD (_Full Function Device_) che assumerà il ruolo di **Zigbee Coordinator (ZC)**.

	A livello di stack protocollare, l'inizializzazione si sviluppa attraverso i seguenti passaggi gerarchici:

1. L'applicazione sul Coordinatore avvia la procedura invocando la primitiva **NETWORK-FORMATION.request** a livello Network (NWK).
2. Il Network Laye interroga il livello sottostante inviando una primitiva MAC **SCAN.request**. Questa primitiva richiede di effettuare una scansione dei canali radio disponibili nello spettro elettromagnetico.

	Per trovare il canale perfetto (quello con meno interferenze e senza conflitti logici), lo standard IEEE 802.15.4 prevede l'esecuzione sequenziale di **due diverse tipologie di scansione** gestite dal livello MAC:

	A. Scansione di Energia (Energy Detection - ED Scan)
	 _"Qual è il canale fisico meno rumoroso?"_.
	- **Funzionamento:** Il ricetrasmettitore radio si sintonizza uno dopo l'altro su tutti i canali supportati dalla propria banda di frequenza (ad esempio i 16 canali della banda a 2.4 GHz). Per ciascun canale, misura la potenza del segnale elettromagnetico presente nell'aria (rumore di fondo, interferenze da Wi-Fi, Bluetooth o forni a microonde) senza decodificare alcun pacchetto.
	- **Risultato:** Restituisce una lista di valori di energia per ogni canale. Il Network Layer scarterà i canali che superano una certa soglia di rumore, selezionando un sottoinsieme di **canali candidati a bassa energia** (i meno rumorosi).

	B. Scansione Attiva (Active Scan)
	_"Ci sono già altre reti attive nel mio vicinato su questi canali?"_.
	- **Funzionamento:** Nei canali candidati selezionati al passo precedente, il livello MAC trasmette un frame di **Beacon Request** e mantiene la radio accesa in ascolto. Se su quel canale opera già una rete in modalità _beacon-enabled_, il rispettivo Coordinatore risponderà inviando un **Beacon Frame**.
	- **Risultato:** Il livello MAC raccoglie tutte le risposte dei beacon ricevuti e compila una tabella che rimanda al Network Layer tramite la primitiva **SCAN.confirm**. Questa tabella contiene i **PAN ID** (Personal Area Network Identifier) e gli indirizzi delle reti già esistenti nell'area geografica.


	Una volta che il Network Layer del Coordinatore riceve la `SCAN.confirm` con i risultati dei due scan, esegue le scelte strategiche finali:

	1. Sceglie il canale che ha registrato il livello di rumore elettromagnetico minore.
	2. **Sceglie un identificatore di rete a 16 bit (PAN ID) che sia **completamente unico

    - Il Coordinatore assegna a se stesso l'indirizzo di rete statico di default **0x0000**.
    - Invia una primitiva **SET.request** al livello MAC per configurare nel chip radio il PAN ID scelto e l'indirizzo `0x0000`.
    - Infine, invoca la primitiva **START.request**. Da questo momento, il livello MAC del Coordinatore inizia a trasmettere periodicamente i propri **Beacon Frame**, rendendo la nuova rete visibile e pronta ad accettare richieste di join da parte di router e dispositivi figli

---

- Livello di Rete (Immagine: ZigBee IV): Cosa rappresenta il grafo? Dimensionamento dell'albero (max-depth, ecc.).
	Il grafo rappresenta la **topologia logica ad albero simmetrico (****Symmetrical Tree****)** della rete Zigbee, utilizzata per l'**assegnazione distribuita, deterministica e decentralizzata** degli indirizzi di rete a 16 bit (_short addresses_).
	
	**I Ruoli Logici nel Grafo:**
    - **La Radice (Coordinatore):** Rappresenta il punto di partenza della rete (indirizzo fisso `0x0000`).
    - **I Nodi Interni (Router):** Fungono da ripetitori e padri attivi che distribuiscono gli indirizzi ai propri figli.
    - **Le Foglie (End Devices):** Dispositivi terminali che non possono avere figli né fare routing.
    - **La Suddivisione in Blocchi (****Addressing Range****):** Il grafo mostra visivamente come ogni Router riceva dal proprio genitore un **range (o blocco) contiguo di indirizzi**. Il router utilizzerà questo blocco per assegnare gli indirizzi ai propri figli router e end-device.
    - **La distinzione semantica degli intervalli:** Accanto a ogni nodo router è indicato il suo indirizzo e, tra parentesi quadre, il suo intervallo di competenza (es. `1` o `14`). Questo range definisce l'intero sotto-albero logico governato da quel router.
    
    - La struttura geometrica dell'albero è definita rigidamente e staticamente all'inizio (sul Coordinatore) tramite tre parametri:

	- Rm​ **(Max Routers):** Il numero massimo di router figli che ciascun Coordinatore o Router può associare direttamente.
	- Dm​ **(Max End Devices):** Il numero massimo di dispositivi terminali (end-device) figli che ciascun Coordinatore o Router può ospitare direttamente.
	- Lm​ **(Max Depth / max-depth):** La profondità (o livello) massima dell'albero. La radice (Coordinatore) si trova a profondità d=0, mentre le foglie più lontane si trovano a d=L
---

- Come fa un end-device a ricevere messaggi dal coordinatore?
	Nei sistemi wireless a bassa potenza basati sullo standard **IEEE 802.15.4** e **Zigbee**, un **End Device** (dispositivo di tipo RFD - _Reduced Function Device_) è progettato per rimanere in modalità di sleep profondo per la maggior parte del tempo al fine di preservare la batteria. Poiché un dispositivo con la radio spenta non può ricevere alcuna comunicazione dall'esterno, **il Coordinatore (o un Router genitore) non può inviare messaggi in modalità** **push** **diretta**.

	Per risolvere questo problema, il trasferimento dati da Coordinatore a End Device è interamente **basato su un modello di polling (****pull-based****)** gestito a livello MAC. Il funzionamento dettagliato varia a seconda della modalità di accesso al canale della rete:

	Trasferimento in reti sincronizzate (Beacon-Enabled)

	Nelle reti che utilizzano una struttura di **Superframe** regolata dall'invio periodico di **Beacon** da parte del Coordinatore, il processo si articola nelle seguenti fasi:

	1. **Bufferizzazione:** Il Coordinatore riceve un messaggio destinato all'End Device, lo memorizza temporaneamente nel proprio buffer e **segnala la presenza di dati pendenti** all'interno dell'apposito campo (_Pending Address List_) del successivo **Beacon Frame**.
	2. **Ascolto del Beacon:** L'End Device si sveglia periodicamente in corrispondenza dell'inizio del Superframe per ricevere e decodificare il Beacon.
	3. **Rilevamento del dato:** L'End Device analizza il Beacon. Se identifica il proprio indirizzo nella lista dei nodi con messaggi pendenti, capisce che il Coordinatore ha dati per lui.
	4. **Richiesta Dati (Polling):** Durante il periodo ad accesso conteso (**CAP - Contention Access Period**), l'End Device invia un frame di **Data Request** al Coordinatore utilizzando il protocollo CSMA-CA slottato.
	5. **ACK del Coordinatore:** Il Coordinatore riceve la richiesta e risponde immediatamente con un pacchetto di **ACK (Acknowledgement)** per confermare la ricezione.
	6. **Invio del Messaggio:** Il Coordinatore estrae il messaggio dal buffer e lo trasmette all'End Device in uno slot successivo del CAP.
	7. **ACK del Dispositivo:** L'End Device riceve il frame dati e invia un **ACK obbligatorio** al Coordinatore. Ricevuto questo ACK, il Coordinatore elimina definitivamente il messaggio dal proprio buffer.


	 Trasferimento in reti asincrone (Non Beacon-Enabled)

	Nelle reti che non utilizzano la struttura a Superframe, il Coordinatore non trasmette Beacon. Di conseguenza, l'End Device non ha modo di sapere in anticipo se ci sono messaggi pronti per lui e la procedura si svolge in modo totalmente asincrono:

	1. **Bufferizzazione silenziosa:** Il Coordinatore riceve il messaggio per il figlio, lo memorizza in memoria e attende passivamente che sia il dispositivo a farsi vivo.
	2. **Polling periodico:** L'End Device si sveglia a intervalli temporali definiti dall'applicazione e invia "alla cieca" un frame di **Data Request** al Coordinatore, sfruttando il protocollo CSMA-CA non slottato.
	3. **ACK del Coordinatore:** Il Coordinatore risponde immediatamente con un ACK per bloccare il timeout radio dell'End Device.
	4. **Verifica e Risposta:**
	    - **Se ci sono messaggi pendenti:** Il Coordinatore estrae il dato dal buffer e lo trasmette all'End Device.
	    - **Se NON ci sono messaggi pendenti:** Il Coordinatore invia un frame vuoto (_empty packet_) con un payload nullo per notificare al dispositivo che può tornare a dormire.
	5. **Chiusura:** L'End Device conferma la corretta ricezione del messaggio inviando un ACK finale, permettendo al Coordinatore di scartare il pacchetto spedito

---

- **Binding:** Cosa è un binding? Perché si usano indirizzi a 64-bit? Cos'è l'Address Map? (Immagine: ZigBee V).
	L'**APS Binding** (collegamento) è un meccanismo del livello **Application Support Sublayer (APS)** di Zigbee che consente di connettere logicamente un Endpoint applicativo (APO) situato su un nodo a uno o più Endpoint situati su altri nodi della rete.

	Il funzionamento si basa su queste caratteristiche chiave:

	- **Unidirezionalità:** Il collegamento è strettamente unidirezionale (va da una sorgente a una o più destinazioni).
	- **Configurazione centralizzata:** Può essere configurato solo dal **Zigbee Device Object (ZDO)** del Coordinatore o di un Router (tipicamente durante la fase di deployment iniziale). La primitiva `BIND.request` accetta come input la tupla `<indirizzo sorgente, endpoint sorgente, identificatore del cluster, indirizzo destinazione, endpoint destinazione>`.
	- **Indirizzamento Indiretto:** Il binding è il pilastro su cui si fonda l'**indirizzamento indiretto**. Normalmente, un messaggio viene instradato conoscendo esplicitamente la coppia `<indirizzo di rete, endpoint>` di destinazione (indirizzamento diretto). Tuttavia, l'indirizzamento diretto è inadatto per dispositivi estremamente semplici (constrained) che non hanno memoria per tracciare i destinatari, ed è un problema critico se gli indirizzi di rete cambiano. Con il binding, un dispositivo sorgente pubblica un messaggio specificando solo il proprio Endpoint e il **Cluster ID** associato: sarà poi il router/coordinator del livello APS (sul Coordinatore o sui Router che memorizzano la tabella di binding) a risolvere la transazione traducendola nelle reali coppie `<endpoint di destinazione, indirizzo di rete di destinazione>`.


	Perché si usano gli indirizzi fisici a 64-bit? (La logica d'esame)

	Nei Binding, le associazioni logiche tra sorgenti e destinazioni vengono definite e memorizzate all'interno della **Binding Table** utilizzando esclusivamente gli **indirizzi fisici IEEE MAC a 64-bit** (scritti stabilmente nell'hardware) e non gli indirizzi logici a 16-bit.

	**Il motivo ingegneristico è la conservazione dello stato a seguito di disconnessioni:** Nelle reti Zigbee, gli indirizzi di rete a 16-bit (_short addresses_) sono dinamici e volatili. Se un dispositivo si spegne, salta la corrente o si resetta, quando effettua nuovamente il join alla rete (magari associandosi a un padre diverso) riceverà un indirizzo a 16-bit completamente differente da quello precedente.

  Se la tabella di binding utilizzasse gli indirizzi a 16-bit, a ogni reset del dispositivo **tutti i binding ad esso associati andrebbero distrutti**, richiedendo una costosa riconfigurazione manuale della rete.
  Utilizzando gli indirizzi a 64-bit (che sono statici e immutabili), **il legame logico tra i dispositivi rimane intatto per sempre**, a prescindere da quante volte i dispositivi cambino il loro indirizzo di rete a 16-bit durante il ciclo di vita della rete.

	Poiché il routing dei pacchetti a livello Network deve necessariamente avvenire utilizzando gli indirizzi logici a 16-bit (per ragioni di efficienza e overhead dei frame radio), il livello APS ha bisogno di un "traduttore" che colleghi i 64-bit dei binding ai 16-bit del routing.

	Questo traduttore è l'**APS Address Map Table**, na tabella di transizione memorizzata nel livello APS di ciascun nodo che associa l'indirizzo di rete logico a 16-bit (_NWK Address_) con l'indirizzo fisico globale a 64-bit (_IEEE MAC Address_).
	 Quando un dispositivo cambia il proprio indirizzo a 16-bit (ad esempio perché si è scollegato e ha eseguito un nuovo join):
	    1. Il dispositivo invia un **annuncio di rete**  per notificare il suo nuovo indirizzo a 16-bit abbinato al suo MAC fisso a 64-bit.
	    2. Ogni nodo della rete riceve l'annuncio e aggiorna automaticamente la propria **Address Map Table** locale.
	    3. In questo modo, la Binding Table (basata su indirizzi a 64-bit) non deve essere modificata, ma il livello APS saprà istantaneamente instradare i messaggi indiretti verso il nuovo indirizzo logico a 16-bit aggiornato
---
---
---


# Duty cycle
- **Concetto di Duty Cycle:** Cos'è il duty cycle?
	La progettazione dei dispositivi IoT è fortemente vincolata da limitazioni in termini di elaborazione, memoria, comunicazione e, soprattutto, **autonomia energetica**.
	Mentre le capacità computazionali dei processori seguono la Legge de Moore raddoppiando le prestazioni o dimezzando i consumi e le dimensioni a parità di costo ogni due anni (nel caso dei micontrollori, puntimao a diminuire le dimensione e i prezzi, poichè non usiamo processori nuovi), **le tecnologie delle batterie (es. Duracell o Li-ion) migliorano in maniera lineare e lento**. Di conseguenza, l'evoluzione hardware da sola non è sufficiente a garantire la sopravvivenza a lungo termine di un nodo sensore; è invece indispensabile implementare **strategie algoritmiche di efficienza energetica** a livello software e di protocollo.

	Analizzando il profilo di consumo energetico tipico di un sensore wireless (_Wireless Sensor Node_), si osserva che la spesa energetica è distribuita approssimativamente tra tre macro-sottosistemi:

	- **Radio (Wireless NIC):** ~40% del consumo totale.
	- **Processore e Chipset:** ~40% del consumo totale.
	- **Sensore e convertitore ADC:** ~20% del consumo totale.

	Poiché il lavoro richiesto a un nodo sensore è tipicamente periodico e ripetitivo (composto dalle fasi di **Sensing** → **Processing & Storing** → **Transmit/Receive** → **Sleep**), l'efficienza energetica si ottiene riducendo drasticamente il tempo di attività delle componenti più energivore.

	Si definisce **Duty Cycle (DC)** la frazione del periodo di attività / quello totale
	
	All'interno dello stesso dispositivo, **ogni sottosistema ha il proprio duty cycle specifico**. Ad esempio, 

	Il tempo di vita utile del dispositivo, espresso come numero massimo di cicli operativi eseguibili (Lifetime), $(B_0-L)/E_t$

	Dove B0​ è la carica iniziale della batteria ed L rappresenta l'energia persa per autoscarica chimica (_battery leaks_). Poiché le perdite per autoscarica dipendono dal tempo (tipicamente espresse come percentuale annua, es. 3% anno), la carica residua della batteria Bn​ al ciclo n può essere modellata rigorosamente tramite una **equazione alle differenze (ricorrenza)**:

	Bn​=Bn−1​(1−ϵ)−E

	Dove ϵ rappresenta la frazione di carica persa a causa delle perdite interne del buffer durante il tempo di un singolo ciclo.

	La gestione aggressiva del duty cycle (es. scendere a DC<1%) permette effettivamente di estendere la vita della batteria da pochi giorni a diversi mesi. Tuttavia, questa strategia presenta un **limite applicativo invalicabile**:

	1. **Il Trade-off tra Prestazioni e Lifetime (Utilità applicativa):** Ridurre arbitrariamente il duty cycle significa allungare il periodo di sleep e quindi rischiare di ridurre l'utilità del sensore.
	2. **La disconnessione dalla rete:** Spegnere la radio per lunghi periodi significa isolare il nodo, rendendo complessa la sincronizzazione del network e la ricezione di messaggi di controllo.

	Per raggiungere tempi di dispiegamento (_deployment_) di molti anni o potenzialmente **infiniti**, l'ottimizzazione del Duty Cycle non è più sufficiente. È necessario affiancare a tali politiche l'uso di tecnologie di **Energy Harvesting** (cattura di energia da fonti ambientali come luce solare, vibrazioni o calore), convertendo l'obiettivo di progettazione dalla massimizzazione del tempo di vita al raggiungimento della **Neutralità Energetica (Energy Neutrality)**, ovvero operare in modo tale che l'energia consumata nel ciclo sia sempre inferiore o uguale a quella incamerata dall'ambiente
---
- **Calcolo della Lifetime:** Spiega il tempo di vita (lifetime) e come si ottiene.  _Attenzione:_ È estremamente pignolo sulle unità di misura. Puoi usare i _mAh_ o i _Joule_ per l'energia, ma devi motivare il perché. Devi farlo nella formula della lifetime, poiché è data da: `Capacità della Batteria / Energia per ciclo` (ricordati di parlare anche dei "leaks", le dispersioni).



	A livello di fisica generale, l'energia (E) si misura in **Joule (**J**)** e la potenza (P) in **Watt (**W**)**. Per definizione elettromagnetica, il Joule rappresenta il lavoro compiuto da una corrente di 1 Ampere che attraversa una differenza di potenziale di 1 Volt per la durata di 1 secondo:

	1 Joule=1 Volt⋅1 Ampere⋅1 secondo=1 W⋅1 s

	Nei circuiti a corrente continua (DC) tipici dei dispositivi IoT, assumiamo che **la differenza di potenziale fornita dalla batteria (**V**) rimanga costante** (o quasi costante) per quasi tutto il periodo di scarica del dispositivo.

	Se il voltaggio V è costante, l'energia consumata dipende linearmente **solo dalla corrente accumulata nel tempo** (ovvero dalla carica elettrica Q):

	E=∫V⋅I(t)dt=V⋅∫I(t)dt=V⋅Q

	Poiché V è costante ed è un fattore comune sia alla capacità della batteria sia al consumo dei singoli componenti, **possiamo omettere la moltiplicazione per la tensione** e descrivere sia la capacità della batteria (B0​) sia l'energia consumata per ciclo (Ecycle​) direttamente in unità di carica elettrica, specificatamente in **milliampere-ora (**mAh**)**.

	- **Conversione matematica formale:** Per convertire una capacità di carica CmAh​ (es. 2000 mAh) in energia equivalente in Joule (EJoule​), dobbiamo moltiplicare la carica in Coulomb (1 mAh=3.6 Coulomb) per la tensione nominale della batteria (V): EJoule​=CmAh​⋅3.6⋅Vnominal​ _Esempio:_ Una batteria da 2000 mAh a 3.7 V nominali immagazzina un'energia pari a: 2000⋅3.6⋅3.7=26.640 Joule

	**Motivazione ingegneristica all'orale:** Esprimere i consumi in mAh evita continue e ridondanti moltiplicazioni per il voltaggio nominale (operazione che introdurrebbe approssimazioni ed errori, dato che la curva di scarica reale di una batteria non è perfettamente piatta ma presenta un lieve decadimento di tensione nel tempo). Inoltre, tutti i datasheet dei componenti IoT specificano l'assorbimento di corrente direttamente in milliampere (mA) o microampere (μA), rendendo i mAh l'unità di misura più naturale e pratica.

	Nel caso ideale, trascurando le perdite interne della batteria, il tempo di vita (espresso in ore) è semplicemente il rapporto tra la capacità totale della batteria e il consumo medio orario del dispositivo:

	Lifetime= B_0/E_t​


	Le batterie reali non sono buffer ideali: esse soffrono di autoscarica chimica spontanea (**battery leaks**), che consuma una porzione della carica anche se il dispositivo si trova in sleep profondo. 
	
	Lifetime (cicli)=B_0-L/E_t

	Dove L rappresenta la carica totale persa a causa delle dispersioni durante l'intero tempo di funzionamento del dispositivo. Poiché L dipende intrinsecamente dalla durata temporale della stessa lifetime (creando una dipendenza circolare), per calcolare la lifetime reale dobbiamo ricorrere a un'**equazione di ricorrenza (differenze finite)**.

	Definiamo ϵ come la frazione infinitesima di carica persa per autoscarica in un singolo ciclo operativo. La carica residua della batteria al ciclo n (Bn​) è data da:

	Bn​=Bn−1​(1−ϵ)−Ecycle​

	Risolvendo analiticamente questa equazione alle differenze a partire dalla carica iniziale B0​, otteniamo la formula chiusa per determinare lo stato della batteria al ciclo n:

	Bn​=B0​(1−ϵ)n−Ecycle​⋅ϵ1−(1−ϵ)n​



---


- **Analisi del codice (Immagine: duty cycle - I):** Prendendo in considerazione questo blocco di codice, parla del duty cycle in Arduino. Spiega come funziona l'accensione e lo spegnimento dei componenti e cosa implica questo per il duty cycle. C'è una relazione tra duty cycle ed energia? (Impara la formula a memoria!).  In questo ciclo ci sono molti accensioni e spegnimenti. Perché lo facciamo?
	
	1. Funzionamento dell'accensione e spegnimento dei componenti nel codice (`duty cycle - I`)

	Il blocco di codice analizzato implementa una tecnica fondamentale di risparmio energetico nota come **Dynamic Power Gating** (gestione dinamica dell'alimentazione) e **Duty Cycling**. In un sistema IoT constrained, i sensori e i ricetrasmettitori radio consumano una quantità di energia enorme rispetto al microcontrollore.

	Il codice controlla questi componenti in modo estremamente granulare nel `loop()`:

	1. **Accensione e Lettura del Sensore:** La chiamata `turnOn(analogSensor)` alimenta la scheda sensori. Viene eseguita la lettura analogica (`analogRead(A0)`) che dura 15 ms. Subito dopo, il sensore viene spento con `turnOff(analogSensor)` per azzerarne il consumo.
	2. **Elaborazione Dati:** Il processore esegue la conversione matematica in tensione, operazione che richiede 1 ms. Durante questa fase, la radio e il sensore sono spenti.
	3. **Accensione e Trasmissione Radio:** La radio viene alimentata tramite `turnOn(radioInterface)`. La trasmissione seriale/radio richiede 4 ms. Subito dopo l'invio, la radio viene spenta con `turnOff(radioInterface)`.
	4. **Sleep di Sistema:** Infine, la funzione `idle(380)` mette il microcontrollore in uno stato di sleep a basso consumo per 380 ms, spegnendo il clock della CPU e congelando le periferiche non necessarie.

	Il periodo totale di un singolo ciclo loop è: $$T_{total} = 15\text{ ms} + 1\text{ ms} + 4\text{ ms} + 380\text{ ms} = 400\text{ ms} \text{$$$$}$$


	Il **Duty Cycle (**DC**)** è la frazione di tempo in cui un componente si trova in stato attivo rispetto al periodo totale del ciclo:

		- **Processor Duty Cycle (**DCproc​**):** Il microcontrollore deve rimanere attivo per coordinare tutte le fasi di sensing, calcolo e trasmissione (15+1+4=20 ms). $$DC_{proc} = \frac{20\text{ ms}}{400\text{ ms}} = \mathbf{5\%} \text{$$$$}$$
	- **Sensor Duty Cycle (**DCsensor​**):** Il sensore è alimentato solo durante la fase di lettura (15 ms). $$DC_{sensor} = \frac{15\text{ ms}}{400\text{ ms}} = \mathbf{3.75\%} \text{$$$$}$$
		- **Radio Duty Cycle (**DCradio​**):** La radio viene accesa esclusivamente durante la trasmissione dei dati (4 ms). $$DC_{radio} = \frac{4\text{ ms}}{400\text{ ms}} = \mathbf{1\%} \text{$$$$}$$



La relazione fisica tra il duty cycle e l'energia consumata è **lineare**. Poiché nei sistemi IoT operiamo a tensione continua e costante (V), il consumo di potenza ed energia dipende unicamente dalla corrente media assorbita (Iavg​).

La formula fondamentale per calcolare la **corrente media (**Iavg​**)** di un componente (o del sistema) in un ciclo è:

$$\mathbf{I_{avg} = I_{active} \cdot DC + I_{sleep} \cdot (1 - DC)} \text{$$$$}$$

Dove:

- Iactive​: Corrente assorbita nello stato attivo.
- Isleep​: Corrente assorbita in modalità sleep/low-power.
- DC: Il duty cycle espresso in frazione (0≤DC≤1).

🧮 Calcolo del consumo medio orario del sistema (Dati del Datasheet):

- **Microprocessore (Atmega128L) [**DC=5%**]:** $$I_{proc} = 8\text{ mA} \cdot 0.05 + 0.015\text{ mA} \cdot 0.95 = 0.400 + 0.0142 = \mathbf{0.414\text{ mAh}} \text{$$$$}$$
- **Radio [**DC=3.75% **(valore slide)]:** $$I_{radio} = 20\text{ mA} \cdot 0.0375 + 0.020\text{ mA} \cdot 0.9625 = 0.750 + 0.0192 = \mathbf{0.769\text{ mAh}} \text{$$$$}$$
- **Sensore [**DC=1% **(valore slide)]:** $$I_{sens} = 5\text{ mA} \cdot 0.01 + 0.005\text{ mA} \cdot 0.99 = 0.050 + 0.0049 = \mathbf{0.055\text{ mAh}} \text{$$$$}$$
- **Consumo Totale Orario:** $I_{tot} = 0.414 + 0.769 + 0.055 = \mathbf{1.243\text{ mAh}} \text{$$}$
- **Lifetime** (con batteria da 2000 mAh, senza leak): $$\text{Lifetime} = \frac{2000\text{ mAh}}{1.243\text{ mAh}} \approx \mathbf{1609\text{ ore}} \text{ (circa 67 giorni)} \text{$$$$}$$


	Questo approccio aggressivo prende il nome di **Dynamic Power Management**. Lo facciamo perché la differenza nei consumi di corrente tra lo stato attivo e lo stato di sleep è di **3 ordini di grandezza (1000 volte)**:

	- La radio consuma 17.4−20 mA quando trasmette, ma appena 20 μA in sleep.
	- La scheda sensori consuma 5 mA attiva e solo 5 μA in sleep.

	Se lasciassimo i componenti costantemente accesi (Duty Cycle a 100%), il consumo medio sarebbe pari alla somma dei consumi attivi (8 mA+20 mA+5 mA=33 mA), scaricando la batteria in appena **60 ore** (meno di 3 giorni). Spegnendo i componenti nei millisecondi in cui non servono e addormentando il microcontrollore in `idle(380)`, abbattiamo lo spreco energetico e aumentiamo l'autonomia del dispositivo di oltre **25 volte

---

- **Stati del Processore:** Qual è la differenza tra _idle_ e _delay_ nel processore?
	La differenza fondamentale tra lo stato di **idle** (o modalità sleep Idle) e la funzione di **delay** risiede nello **stato dei clock interni della CPU** e, di conseguenza, nel **consumo di corrente** del microcontrollore:


	La funzione `delay(ms)` nativa di Arduino è una chiamata bloccante che implementa un meccanismo di **attesa attiva** (_busy-waiting_).

	- **Cosa fa l'hardware:** Durante un `delay()`, il core della CPU rimane pienamente attivo. Il clock principale (CLKCPU​) e il clock della memoria flash (CLKFLASH​) continuano a oscillare alla massima frequenza (es. 16 MHz). La CPU non si riposa, ma esegue costantemente un ciclo infinito di istruzioni assembly fittizie (_NOP_ o decrementi di un contatore) per far passare il tempo richiesto.
	- **Impatto energetico:** Il consumo di corrente rimane al suo valore nominale massimo. 
	- **Impatto sul Duty Cycle:** Poiché il processore esegue istruzioni per tutto il tempo dell'attesa, il suo **Processor Duty Cycle (**DCproc​**)** effettivo rimane fisso al **100%**, azzerando qualsiasi possibilità di risparmio energetico.


	La funzione `idle(ms)` (messa a disposizione da librerie di risparmio energetico come `LowPower.h`) o l'istruzione assembly `sleep` configurata in modalità Idle, mettono il processore in uno **stato di riposo parziale gestito dall'hardware**.

	- **Cosa fa l'hardware:** Quando viene invocata la modalità Idle, il circuito di Power Management del microcontrollore **arresta immediatamente il clock della CPU (**CLKCPU​**)** e il clock della memoria Flash (CLKFLASH​). L'esecuzione delle istruzioni nel core si ferma completamente. Tuttavia, le periferiche hardware di supporto (come i Timer interni, la seriale USART, l'interfaccia SPI e il Watchdog) rimangono alimentate e attive. Il processore si risveglierà automaticamente allo scadere di un timer interno o al sopraggiungere di un interrupt esterno.
	- **Impatto sul Duty Cycle:** L'uso di `idle()` permette di implementare un vero **Duty Cycling**. Il tempo trascorso in idle viene considerato tempo di inattività (Tinactive​), permettendo di abbassare il duty cycle del processore (es. al 5%) e moltiplicando la vita utile della batteria del dispositivo

---
---
---


# MAC Protocol

- **B-MAC (Berkeley MAC):** Come funziona questo protocollo? Spiega il meccanismo di Low Power Listening (LPL) e come aiuta a ridurre il duty cycle e il consumo energetico rispetto ad altri approcci.
	l protocollo **B-MAC (Berkeley MAC)** è un protocollo di controllo dell'accesso al mezzo (MAC) asincrono, in modalità beacon-enabled, **B-MAC non richiede alcuna sincronizzazione temporale esplicita tra i nodi della rete**. Usa un solo parametro il tempo di wakeup

	Il suo principio di funzionamento si basa interamente sul paradigma del **Preamble Sampling** (campionamento del preambolo) mediante la tecnica del **Low-Power Listening (LPL)**.
		- Il ricevitore trascorre la maggior parte del tempo in uno stato di sleep profondo per preservare la batteria.
		- Ad intervalli regolari e periodici (definiti dall'intervallo di controllo Tsleep​ o _check interval_), il ricevitore si sveglia brevemente ed esegue un'operazione di **Preamble Sampling** (campionamento del canale).
		- Questa attività di ascolto (LPL) è estremamente rapida ed economica (dura frazioni di millisecondo): il nodo accende la radio, effettua un controllo di presenza di segnale
		    1. **Se non rileva attività:** spegne immediatamente la radio e torna in sleep.
		    2. **Se rileva un preambolo attivo:** mantiene la radio accesa in modalità di ascolto continuo (_Listen mode_) fino al termine del preambolo, per poi ricevere e decodificare il pacchetto dati vero e proprio.
		
	Poiché i nodi non sono sincronizzati, il trasmettitore può decidere di inviare un pacchetto in qualsiasi momento, senza attendere una finestra temporale specifica (trasmissione _on-demand_).
    Per assicurarsi che il destinatario intercetti la trasmissione durante uno dei suoi risvegli periodici, il trasmettitore precede il pacchetto dati vero e proprio con un **preambolo radio estremamente lungo**.

	Affinché il protocollo funzioni in modo affidabile senza perdita di pacchetti, la durata temporale del preambolo (Tpreamble​) deve essere strettamente maggiore del periodo di sleep del ricevitore (Tsleep​):

	Tpreamble​>Tsleep​

	Se questa condizione non fosse soddisfatta, il ricevitore potrebbe svegliarsi, campionare il canale durante un momento di silenzio tra un preambolo e l'altro, e tornare a dormire perdendo completamente la successiva trasmissione dati.
	
	Rispetto ad approcci sincroni (come S-MAC), B-MAC offre vantaggi energetici e architetturali straordinari, pur accettando precisi trade-off:

	Zero Overhead di Sincronizzazione (Differenza con S-MAC)
	In protocolli come **S-MAC**, i nodi devono scambiarsi periodicamente pacchetti di sincronizzazione (**SYNC frames**) per allineare le proprie finestre di attività (ascolto/sleep).

	- **Il problema di S-MAC:** Questo continuo scambio di messaggi di controllo genera un overhead di traffico radio non trascurabile che consuma costantemente energia, anche quando non ci sono dati applicativi da trasmettere. Inoltre, richiede risorse di memoria per tracciare le tabelle delle pianificazioni (_schedules_) dei vicini.
	- **La soluzione di B-MAC:** Non essendoci sincronizzazione, **l'overhead di controllo di B-MAC è esattamente pari a zero**. I nodi non devono scambiarsi alcun SYNC frame né mantenere tabelle di vicinato, massimizzando il risparmio energetico in scenari a bassissimo traffico (es. 1 campione ogni 10-20 minuti).
		
	Il duty cycle legato all'attività di monitoraggio dell'etere da parte del ricevitore è infinitesimo.
	
	**Il costo energetico del trasmettitore:** Per consentire al ricevitore di dormire a lungo (es. Tsleep​=1 s), il trasmettitore deve inviare un preambolo lunghissimo e costoso (1 s di trasmissione alla massima potenza).
  ** Il problema dell'Overhearing:** Quando un trasmettitore invia un lungo preambolo, **tutti nodi nel raggio radio** che eseguono il sampling si sveglieranno e rimarranno attivi in ascolto per tutta la durata del preambolo, per poi scoprire (solo all'arrivo dell'header del pacchetto dati) che il messaggio non era indirizzato a loro, sprecando enormi quantità di energia.

	- **X-MAC:** Risolve il problema frammentando il lungo preambolo continuo in una serie di **brevi preamboli intermittenti contenenti l'ID del destinatario**. Se il ricevitore si sveglia e riconosce il proprio ID nel breve preambolo, invia immediatamente un **Early ACK** (ACK anticipato) per interrompere la trasmissione del preambolo e forzare il mittente a inviare subito il pacchetto dati, risparmiando tempo ed energia su entrambi i lati.
	- **BoX-MAC:** Evoluzione ulteriore in cui il preambolo stesso non è una stringa fittizia, ma è la **ripetizione continua del pacchetto dati stesso**. Il ricevitore si sveglia, decodifica direttamente il pacchetto applicativo, invia un ACK per spegnere il trasmettitore e conclude istantaneamente la transazione
---
---
---


# IEEE 802.15.4
    - **Analisi del Superframe (Immagine: IEEE 802.15.4 - I / Superframe):** Descrivi in dettaglio la struttura del superframe (periodo attivo, inattivo, CAP, CFP). Cosa contiene il beacon frame?
----
---
---

# Arduino

- **Librerie & Hardware:** Cosa fanno le funzioni nella libreria `lowpower.h`? Come si riaccende il microprocessore, qual è il meccanismo? (Risposta: tramite un timer o un interrupt esterno).
	
	Le funzioni principali della libreria (come `LowPower.idle()` e `LowPower.powerDown()`) hanno il compito di **transire il microcontrollore in uno stato di sleep a bassissimo consumo**.
	

	1. **Arresto dei Clock di Sistema:** Spengono selettivamente le sorgenti di clock del microcontrollore. Nella modalità **Idle**, viene arrestato il clock della CPU (CLKCPU​) e della memoria Flash (CLKFLASH​), mantenendo attivi i clock delle periferiche. Nella modalità **Power-Down** (la più aggressiva), vengono spenti tutti i clock interni e l'oscillatore esterno a cristallo, congelando l'intero chip.
	2. **Disattivazione selettiva delle Periferiche:** Permettono di passare argomenti specifici per spegnere moduli hardware interni che consumerebbero corrente anche in sleep. I parametri principali sono:
	    - `ADC_OFF`: Spegne il convertitore Analogico-Digitale.
	    - `BOD_OFF`: Spegne il circuito di _Brown-Out Detection_ (il modulo che monitora cali di tensione, che da solo assorbe circa 15 μA).
	    - `TIMERx_OFF`, `SPI_OFF`, `USART0_OFF`, `TWI_OFF`: Disattivano rispettivamente i timer interni, l'interfaccia SPI, la porta seriale UART e il bus I2C.
	3. **Configurazione del Tempo di Sleep:** Accettano costanti temporali predefinite per determinare la durata dello stato di sleep (es. `SLEEP_15MS`, `SLEEP_2S`, `SLEEP_8S` o la costante speciale `SLEEP_FOREVER` per uno sleep indefinito).

	Il microcontrollore non può riaccendersi da solo eseguendo codice, il risveglio (wake-up) avviene **esclusivamente a livello hardware tramite un segnale di Interrupt** che riattiva la generazione dei clock interni.


	 Risveglio tramite Timer Interno (Watchdog Timer - WDT)

	Quando chiamiamo una funzione impostando un tempo definito (es. `LowPower.powerDown(SLEEP_8S, ...)`), il microcontrollore sfrutta il **Watchdog Timer**.

	- **Il Meccanismo:** Il Watchdog è un timer hardware asincrono che funziona con un oscillatore interno separato da quello della CPU. Durante lo sleep, mentre tutto il resto è spento, il Watchdog continua a contare.
	- **Il Risveglio:** Al raggiungimento del timeout impostato (overflow), il Watchdog genera un **interrupt interno**. Questo interrupt hardware risveglia il circuito di Power Management, il quale riattiva l'oscillatore principale (fase di stabilizzazione del clock) e fornisce nuovamente il segnale di clock alla CPU, facendola ripartire.

	 Risveglio tramite Interrupt Esterno

	Quando il dispositivo viene addormentato senza un limite di tempo impostando il parametro **SLEEP_FOREVER**, il Watchdog viene disattivato. L'unico modo per ridare vita al microcontrollore è applicare un segnale elettrico dall'esterno.

	- **Il Meccanismo:** Prima di invocare lo sleep, il software deve abilitare un interrupt hardware esterno su uno dei pin dedicati di Arduino (es. INT0 sul pin 2, configurato con `attachInterrupt(0, wakeUp, LOW)`).
	- **Il Risveglio:** Quando un evento fisico reale (come la pressione di un pulsante o il segnale di un sensore di presenza) porta il pin 2 a livello logico **LOW**, l'hardware del microcontrollore rileva asincronamente la variazione di stato. Questa transizione genera istantaneamente un **interrupt esterno**. Il circuito di controllo riaccende immediatamente i clock di sistema, la CPU si risveglia, esegue la routine di servizio dell'interrupt (`wakeUp()`) e poi riprende l'esecuzione del codice applicativo esattamente dall'istruzione successiva a quella che aveva avviato lo sleep.
---
---
---

- **Analisi del codice (Immagine: Arduino - interrupt):** Commenta il codice dicendo cosa fanno i vari comandi.


---

- **Esecuzione e Vincoli:** Quando viene eseguito l'interrupt handler (ISR)? Cosa è permesso fare agli interrupt e perché? (Risposta: non devono interferire con l'esecuzione normale e devono essere estremamente brevi).
---

- **La keyword volatile:** A cosa serve `volatile`? Perché è necessario e perché il valore non sarebbe consistente altrimenti?
    
    - _Risposta:_ Per via dell'ottimizzazione del compilatore, il valore di una variabile potrebbe essere scritto in un registro locale invece che nella memoria RAM. Alla fine dell'interrupt, i valori dei registri tornano allo stato precedente, perdendo la modifica. `volatile` forza il compilatore a leggere/scrivere sempre direttamente in memoria.
        
----
---
---
# Energy Harvesting & Gestione della Batteria

- **Schema Harvest-Store-Use:** Spiega l'immagine dell'Energy Harvesting con il grafico della potenza immagazzinata e consumata.

---

    
- **Significato degli integrali (Immagine: 3 integrali):** L'integrale della potenza in ingresso (harvested) nel tempo rappresenta l'energia totale raccolta, mentre l'integrale della potenza in uscita (used/leaked) rappresenta l'energia consumata. Per mantenere il sistema vivo (neutralità energetica), l'integrale dell'energia raccolta deve essere maggiore o uguale a quello dell'energia consumata, tenendo conto delle perdite.

---


    
- **Caratteristiche dei Buffer:** Spiega e discuti le caratteristiche di un buffer (batteria/supercondensatore):
    
    1. _Capacità:_ Quanta energia può immagazzinare.
        
    2. _Efficienza di carica/scarica:_ Quanta energia si perde durante il trasferimento.
        
    3. _Leakage (Dispersione):_ L'energia che si perde fisiologicamente nel tempo anche senza utilizzo.
        
    
    - _Qual è la più importante a livello progettuale?_ Dipende dall'applicazione, ma nei sistemi WSN/IoT a lunghissimo termine, il **Leakage** e l'**Efficienza** sono spesso i fattori più critici.

---

    
- **Modello di Kansal:** Come viene definita la neutralità energetica (energy neutrality)? Spiega il suo algoritmo per il duty cycling dinamico e introduci il concetto di _utility_ in questo contesto.

---

- **Modello Task-based:** In cosa consiste l'approccio basato sui task per la gestione dell'energia?
---

 - **Confronto:** Quali sono le differenze principali e i compromessi tra il Task-based model e il modello di Kansal?