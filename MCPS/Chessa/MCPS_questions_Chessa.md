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
    
        
    - Qual è il risultato di uno scan? Come fa un dispositivo a scegliere a quale PAN ID associarsi?
        
    - _Domanda trabocchetto:_ Perché c'è una "Data Request" dal client verso il router durante il join? (Risposta: Per comunicare serve un indirizzo di rete, il client ne è ancora sprovvisto, quindi deve essere lui a inizializzare lo scambio per farsi assegnare lo short address).
        
    - Disegna il diagramma dell'Associate Request.
        
- **Livelli e Superframe (IEEE 802.15.4):**
    
    - Architettura generale e come si trova il canale giusto a cui connettersi.
        
    - Livello di Rete (Immagine: ZigBee IV): Cosa rappresenta il grafo? Dimensionamento dell'albero (max-depth, ecc.).
        
    - **Analisi del Superframe (Immagine: IEEE 802.15.4 - I / Superframe):** Descrivi in dettaglio la struttura del superframe (periodo attivo, inattivo, CAP, CFP). Cosa contiene il beacon frame?
        
    - Come fa un end-device a ricevere messaggi dal coordinatore?
        
- **Binding:** Cosa è un binding? Perché si usano indirizzi a 64-bit? Cos'è l'Address Map? (Immagine: ZigBee V).
---

    


### 1. Duty Cycle, Energia e Lifetime (Tempo di vita)

- **Concetto di Duty Cycle:** Cos'è il duty cycle?
    
- **Analisi del codice (Immagine: duty cycle - I):** Prendendo in considerazione questo blocco di codice, parla del duty cycle in Arduino.
    
    - _Cosa dire:_ Spiega come funziona l'accensione e lo spegnimento dei componenti e cosa implica questo per il duty cycle.
        
    - _Relazione con l'energia:_ C'è una relazione tra duty cycle ed energia? (Impara la formula a memoria!)
        
    - _Power-up/down:_ In questo ciclo ci sono molti accensioni e spegnimenti. Perché lo facciamo?
        
- **Protocolli MAC e Duty Cycle:**
    
    - **B-MAC (Berkeley MAC):** Come funziona questo protocollo? Spiega il meccanismo di Low Power Listening (LPL) e come aiuta a ridurre il duty cycle e il consumo energetico rispetto ad altri approcci.
        
- **Stati del Processore:** Qual è la differenza tra _idle_ e _delay_ nel processore?
    
- **Librerie & Hardware:** Cosa fanno le funzioni nella libreria `lowbattery.h`? Come si riaccende il microprocessore, qual è il meccanismo? (Risposta: tramite un timer o un interrupt esterno).
    
- **Calcolo della Lifetime:** Spiega il tempo di vita (lifetime) e come si ottiene.
    
    - _Attenzione:_ È estremamente pignolo sulle unità di misura. Puoi usare i _mAh_ o i _Joule_ per l'energia, ma devi motivare il perché. Devi farlo nella formula della lifetime, poiché è data da: `Capacità della Batteria / Energia per ciclo` (ricordati di parlare anche dei "leaks", le dispersioni).
        

### 2. Arduino & Interrupts

- **Analisi del codice (Immagine: Arduino - interrupt):** Commenta il codice dicendo cosa fanno i vari comandi.
    
- **Esecuzione e Vincoli:** Quando viene eseguito l'interrupt handler (ISR)? Cosa è permesso fare agli interrupt e perché? (Risposta: non devono interferire con l'esecuzione normale e devono essere estremamente brevi).
    
- **La keyword volatile:** A cosa serve `volatile`? Perché è necessario e perché il valore non sarebbe consistente altrimenti?
    
    - _Risposta:_ Per via dell'ottimizzazione del compilatore, il valore di una variabile potrebbe essere scritto in un registro locale invece che nella memoria RAM. Alla fine dell'interrupt, i valori dei registri tornano allo stato precedente, perdendo la modifica. `volatile` forza il compilatore a leggere/scrivere sempre direttamente in memoria.
        

### 5. Energy Harvesting & Gestione della Batteria

- **Schema Harvest-Store-Use:** Spiega l'immagine dell'Energy Harvesting con il grafico della potenza immagazzinata e consumata.
    
- **Significato degli integrali (Immagine: 3 integrali):** L'integrale della potenza in ingresso (harvested) nel tempo rappresenta l'energia totale raccolta, mentre l'integrale della potenza in uscita (used/leaked) rappresenta l'energia consumata. Per mantenere il sistema vivo (neutralità energetica), l'integrale dell'energia raccolta deve essere maggiore o uguale a quello dell'energia consumata, tenendo conto delle perdite.
    
- **Caratteristiche dei Buffer:** Spiega e discuti le caratteristiche di un buffer (batteria/supercondensatore):
    
    1. _Capacità:_ Quanta energia può immagazzinare.
        
    2. _Efficienza di carica/scarica:_ Quanta energia si perde durante il trasferimento.
        
    3. _Leakage (Dispersione):_ L'energia che si perde fisiologicamente nel tempo anche senza utilizzo.
        
    
    - _Qual è la più importante a livello progettuale?_ Dipende dall'applicazione, ma nei sistemi WSN/IoT a lunghissimo termine, il **Leakage** e l'**Efficienza** sono spesso i fattori più critici.
        
- **Modelli di Energy Harvesting:**
    
    - **Modello di Kansal:** Come viene definita la neutralità energetica (energy neutrality)? Spiega il suo algoritmo per il duty cycling dinamico e introduci il concetto di _utility_ in questo contesto.
        
    - **Modello Task-based:** In cosa consiste l'approccio basato sui task per la gestione dell'energia?
        
    - **Confronto:** Quali sono le differenze principali e i compromessi tra il Task-based model e il modello di Kansal?