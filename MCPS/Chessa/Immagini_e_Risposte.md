
![[Pasted image 20260711154428.png]]
L'interoperabilità rappresenta una delle principali sfide del mondo IoT. Spesso le soluzioni sono progettate come **silos verticali**, in cui i dispositivi presenti nella rete funzionano esclusivamente con dispositivi dello stesso produttore, generando **vendor lock-in** (ovvero l'essere legati ai prodotti di un singolo venditore), e migrare su altre soluzioni risulta difficile e costoso.

La soluzione iniziale è rappresentata dall'adottare un **protocollo comune**, ma questo dipende dai bisogni economici delle aziende, che spesso puntano sulla creazione di propri standard. Quindi, alla fine, abbiamo troppi standard che ci riportano al problema dell'interoperabilità e dei silos verticali.

Di conseguenza, la soluzione consiste nell'introdurre **gateway a livello applicativo**, che hanno il compito non solo di tradurre i protocolli a basso livello, ma anche di mappare i diversi comportamenti a livello applicativo.

Ci sono diversi tipi di soluzioni:
 Un approccio si basa sull'uso di un **service gateway**, che si occupa di abilitare la comunicazione tramite la traduzione dei protocolli, permettendo la comunicazione su Internet e tra protocolli diversi. Il service gateway permette di far coesistere dispositivi di vendor diversi, a patto che comunichino con lo stesso protocollo all'interno del gateway.

Infine, come mostrato nell'immagine, esiste una soluzione che prevede l'uso di **integration gateway**, i quali permettono di far coesistere diversi vendor e diversi protocolli all'interno della rete. Non si limita a tradurre i protocolli a basso livello, ma mappa i differenti comportamenti applicativi dei vari dispositivi.
In questo specifico caso, abbiamo un **integration gateway distribuito**. Questa soluzione mappa i messaggi in un protocollo intermedio e poi nel protocollo richiesto dal vendor.
questa specifica soluzione permette di avere $2n$ possibili mapping invece di $n^2$ (come avverrebbe nel caso di una soluzione con un unico integration gateway non distribuito).
Nella **configurazione Type C** (integrazione diretta) ogni protocollo deve tradursi verso tutti gli altri, richiedendo n(n−1)≈n2 mappature punto-a-punto. Nella **configurazione Type D**, l'introduzione di un **protocollo intermedio comune** riduce la complessità a sole 2n mappature, poiché ciascun protocollo deve essere tradotto solo "da" e "verso" la lingua comune.

Le limitazioni di una soluzione non interoperabile nel mondo IoT risiedono nel fatto che manca scalabilità in termini di business, e coloro che traggono vantaggio dal mantenere la non interoperabilità sono le grandi compagnie.

---

![[Pasted image 20260714084759.png]]
La sicurezza è un punto critico delle soluzioni IoT a causa del fatto che si lavora con dispositivi limitati in termini di memoria, forza computazionale e costo; inoltre, siamo connessi a internet e ci affidiamo a un cloud. 

Di conseguenza, risulta difficile mantenere una pianificazione dei patching e applicare gli aggiornamenti in situazioni che richiedono un'operatività continua 24/7. Spesso questi dispositivi non hanno soluzioni di sicurezza adeguate al loro interno, anche perché l'obiettivo principale del vendor è minimizzare i costi di produzione ed entrare nel mercato il più presto possibile.

Ci troviamo in ambienti con situazioni eterogenee da gestire e un'unica soluzione non va bene per tutti, poiché dipende sempre dal device. La prima immagine mostra uno schema di una rete composta da piattaforme di gestione e storage (A), gateway (G), dispositivi non limitati con funzioni di sicurezza (U), dispositivi IoT non limitati senza funzioni di sicurezza (U non blu) e dispositivi limitati IoT (C).

Le principali vulnerabilità in ambito IoT includono le password hardcoded all'interno del device, che possono essere facilmente recuperate dato che spesso non sono cifrate, o l'utilizzo di reti non protette. Tutte queste situazioni sono causate dal fatto che il dispositivo è vincolato in termini di capacità.

In questo ambito esiste una raccomandazione, detta ITU-T Y.2066, che esprime i requisiti funzionali durante le operazioni dei dati nell'IoT, offrendo suggerimenti in merito alla sicurezza delle comunicazioni, della gestione dei dati e della fornitura dei servizi.

I gateway IoT svolgono funzioni di sicurezza essenziali, permettendo l'identificazione a ogni accesso.

L'autenticazione dei device è basata sui requisiti dell'applicazione e sulle capacità del dispositivo (è preferibile quella mutua) e fornisce una protezione adeguata al livello di sicurezza richiesto, coprendo i dati salvati nel gateway, i dati trasferiti tra gateway e device e quelli tra gateway e applicazioni.

Se un sensore constrained non ha capacità hardware di cifratura, è praticamente impossibile per il gateway proteggere i dati memorizzati a bordo di quel dispositivo.

Il gateway, infine, permette l'autodiagnosi, la riparazione automatica e la manutenzione remota; consente inoltre l'aggiornamento di software e firmware e aiuta a configurare le applicazioni.

Sorgono ovviamente problematiche in ambito privacy a causa della grande raccolta di dati critici, come avviene ad esempio in casa tramite videocamere o orologi intelligenti.

L'applicazione della sicurezza nei device IoT richiede inevitabilmente un tradeoff prestazionale.

---

![[Pasted image 20260714092146.png]]
I dispositivi IoT hanno l'esigenza di connettersi a internet, ma l'uso dello stack protocollare di internet, che è pensato per dispositivi ricchi di risorse, non va bene per loro. 
Anche a causa del fatto che lavorano in condizioni di rete instabile, con hardware che consuma poco e con risorse limitate, i requisiti cambiano radicalmente. 
In questo contesto si fa strada MQTT, che non nasce per soddisfare questa precisa esigenza ma cade a pennello. Message Queuing Telemetry Transport è un protocollo di messaggistica leggero di tipo publish/subscribe che si appoggia al paradigma client/server, progettato per ambienti con risorse limitate e connettività instabile.

Il paradigma publish/subscribe permette di avere disaccoppiamento spaziale (pub e sub non si conosco), temporale (non devono essere attivi contemporaneamente) e di sincronizzazione (l'esecuzione di uno non blocca l'altro). Abbiamo 3 attori: il publisher che produce contenuti, il subscriber che indica l'interesse di iscriversi a determinati topic, e il broker che rappresenta il server dell'infrastruttura, il quale si occupa di ricevere i messaggi, filtrarli per topic e distribuirli a chi è interessato , gestendo iscrizioni e disiscrizioni.

Abbiamo una scalabilità maggiore rispetto al classico client-server dato il fatto che possiamo parallelizzare l'event service (broker) e possiamo filtrare per contenuto (si guarda il payload) , tipo (in base all'evento)  o topic.

L'immagine nel dettaglio mostra la struttura di MQTT composta da un publisher che pubblica contenuti a sinistra, mentre il broker prende il messaggio e ne notifica l'arrivo.

È presente un subscriber che mostra interesse per un topic tramite il messaggio di subscribe e riceve il messaggio pubblicato dal broker se è attivo, con la possibilità di disiscriversi.

Publisher e subscriber non si conoscono, devono conoscere solo l'IP o l'hostname del broker.

Il publisher, quando pubblica, può avere conferma che il messaggio arriva al broker, ma non sa se arriverà mai a qualche subscriber.

Publish è l'operazione che permette la pubblicazione del messaggio da parte del publisher; contiene Packet ID se QoS > 0, Topic Name, QoS, payload, Retain Flag e DUP Flag.

Subscribe è composto da Packet ID, Topic e QoS (che si ripetono in lista se si hanno più topic di interesse).

Suback è la risposta a un subscribe e contiene Packet ID e Return Code.

Unsubscribe contiene Packet ID e la lista dei topic; la risposta di ricezione è UNSUBACK con il relativo Packet ID.

---
![[Pasted image 20260714095131.png]]
Instaurazione della connessione, che è TCP, avviene lo scambio del messaggio Connect che contiene: ClientID, CleanSession (con true sessione effimera altrimenti persistente), Username/PW, Will flags e KeepAlive. Il broker risponde con Connack: flag connection (Accepted/refused) e il flag session present, che indica se vi era già una sessione persistente per quel client.

A seguito di ciò, si possono eseguire le operazioni di publish e subscribe specificando un QoS.

Con QoS si indica l'accordo fra broker e client sul livello di garanzia accordato per lo scambio dei messaggi.

Abbiamo due canali comunicativi, tra publisher e broker e tra broker e subscriber; di conseguenza abbiamo due QoS differenti.

Ci sono 3 livelli di QoS:

**0 (modalità best effort)** indica nessuna garanzia che il messaggio arrivi, per questo nel publish non importa indicare il packet ID e non abbiamo nessun ACK di conferma; il broker non lo memorizza. È la soluzione ideale nel caso in cui i dati invecchino rapidamente.

**1 garantisce che il messaggio arrivi almeno una volta.** Con l'ACK (PUBACK) da parte del broker è necessario che ci sia il packet ID nel publish. Se il PUBACK non arriva entro un timeout, si rinvia il messaggio con il DUP flag vero. Il messaggio può arrivare più volte, situazione ideale per i casi in cui il messaggio deve arrivare per forza.

**2 è il livello più alto**, garantisce esattamente un arrivo grazie all'handshake a 4 messaggi che aumenta complessità e overhead.

L'handshake inizia con il publish, composto da packet ID (sicuramente inserito poiché QoS = 2), QoS, payload, retain flag, DUP flag e topic name. A seguito di questo, il broker risponde con PUBREC, che indica che il messaggio è arrivato ed è stato salvato nel broker. Successivamente il publisher, dopo aver ricevuto PUBREC, invia PUBREL e autorizza la consegna del messaggio al subscriber.

PUBCOMP conferma che la consegna è avvenuta e si può eliminare il messaggio.

Risulta necessario un double handshake rispetto a un approccio one-way poiché la prima parte garantisce l'invio del messaggio e la seconda serve a gestire lo scarto. PUBREC da solo non è sufficiente: se si perde, il messaggio viene rimandato e serve un riferimento per sapere che si tratta di un duplicato. Con PUBREL sappiamo che si può cancellare il messaggio nel broker e, ricevendo PUBCOMP, il publisher sa di non dover rimandare PUBREL.

Il QoS 2 trova applicazione, ad esempio, in un sensore di insulina.

---
![[Pasted image 20260714105716.png]]
1) A seguito dell'arrivo dei messaggi al broker avviene il filtraggio dei messaggi, che può essere per topic (data agnostic), per contenuto (controllo sul payload, impedisce crittografia) e per evento (controllo del tipo). MQTT usa tipicamente quello per topic. 
	 Il topic è una stringa gerarchica che viene usata appunto per discriminare i messaggi in base a qualche logica. La separazione a livello gerarchico è fornita dallo `/` e un esempio è `casa/primopiano/cameradaletto/presenza` per indicare i sensori di presenza. Esistono delle wildcard che sono `+`, che permette di sostituire **un singolo livello** della gerarchia, e `#`, sostituisce **qualsiasi livello rimanente** sottostante e deve essere posizionata **tassativamente alla fine del topic**. Il simbolo `$` è riservato per statistiche interne al protocollo e non può essere usato dai client per pubblicare. Non ci sono delle regole specifiche per i topic, ma delle best practice che permettono di mantenere in maniera corretta la struttura gerarchica.

2) A causa del comportamento dei dispositivi IoT che si collegano e scollegano spesso alla rete per situazioni di batteria, risparmio energetico, rete instabile, eccetera, nasce l'esigenza di mantenere salvato in qualche modo lo stato della comunicazione interrotta. Per questo motivo MQTT fornisce un meccanismo di sessione persistente che permette di salvare le sottoscrizioni del subscriber ed eventuali messaggi non consegnati. La sessione persistente è attiva quando il clean session è false nel connect, che ha in risposta il connack (con flag session present attivo).
	Le sessioni sono legate al clientID del client.
	Quindi, con sessione persistente attiva, il broker salva le sottoscrizioni dei subscriber, salva i messaggi con QoS 1 e 2 che non sono ancora stati consegnati e confermati e, diametralmente, ovviamente anche il client deve salvare lo stato localmente ovvero tutti i messaggi inviati e ricevuti dal broker e non confermati dal broker. Le sessioni persistenti vanno evitate quando possibile poiché introducono overhead; ad esempio, in caso di QoS 0 sono inutili, oppure vanno evitate nel caso in cui la ricezione di messaggi vecchi sia indesiderata.

3) Nella situazione di aggiornamenti poco frequenti da parte di un publisher può accadere che il nuovo iscritto non riceva nessun messaggio per tanto tempo; per questo esiste il meccanismo del retained message, che permette di salvare l'ultimo messaggio inviato per quel topic e inviarlo appena vi è una nuova iscrizione.

	Il retained message viene specificato nell'operazione di publish tramite il flag RetainFlag=true. Funziona anche tramite wildcard.
	C'è uan netta differenza fra i messsaggi retained e le sessioni persistenti Il broker li memorizza e li mantiene attivi sul server anche se tutti i client sono disconnessi e non ci sono sessioni persistenti attive. Vengono conservati anche dopo essere stati consegnati ai subscriber.
	Per eliminare un messaggio precedente basta basta **pubblicare un messaggio retained vuoto (con payload vuoto/zero byte) sullo stesso identico topic**.
	
	**Si integrano perfettamente con il Last Will & Testament**: se un sensore invia il proprio stato `ON` come retained, e successivamente subisce un crash improvviso, il broker può pubblicare il suo messaggio di "testamento" (LWT) configurato come messaggio retained con payload `OFF`. Questo garantisce che i futuri subscriber ricevano immediatamente lo stato reale (`OFF`) e non quello obsoleto (`ON`)

4) Il flag LastWill, attivabile sempre durante l'operazione di connect, permette di indicare un messaggio da inviare in caso di disconnessione anomala. Nella casistica di disconnessione normale viene mandato un messaggio disconnect che permette di notificare i partecipanti alla connessione.

	A causa di un'anomalia ovviamente risulta impossibile comunicare il proprio stato finale e per questo esiste tale meccanismo, che rappresenta un messaggio preconfigurato da consegnare al broker in fase di connessione e quando il broker rileva una disconnessione anomala a causa di errore I/O, mancanza di risposta durante il tempo di keep alive oppure chiusura della connessione TCP brusca, oppure la chiusura della connessione da parte del broker.

	Ovviamente in caso di disconnect il testamento viene scartato poichè il broker capisce che è una situazione voluta.

	LastWill non è altro che un normale messaggio con topic, QoS, retained flag e payload. Come anticipato si definiscono nel messaggio connect che ha lastWillTopic, lastWillQoS, lastWillMessage e lastWillRetain. Come detto prima si usa a coppia spesso con retained message.

	KeepAlive rappresenta un intervallo di tempo in secondi all'interno in cui è richiesto che ci sia qualche segnale tra client e broker, dal semplice messaggio scambiato (che azzera il timer) al pingreq (inviato in caso di mancanza di messaggi dal client al broker) con pingresp (risposta del broker), che permette di assicurarsi che il client sia sempre attivo. Se non viene mandato il pingreq dal client prima del tempo limitate dato da KeepALive, si spegne la connessione e si manda il Last Will e testamento se presente.

---
![[Pasted image 20260714120907.png]]
Zigbee è un protocollo di comunicazione a costo ridotto, impiegabile globalmente, affidabile, scalabile e facile da distribuire.

Zigbee lavora sopra lo standard IEEE 802.15.4, che si occupa del physical layer e del MAC layer, mentre lui si occupa dell'application layer e del network layer.

Essendo diviso in livelli, Zigbee permette che un livello fornisca servizi per il successivo o il precedente, e ci sono per questo motivo 4 primitive: request, che permette a un livello superiore di chiedere un servizio a un livello inferiore; indicate, che permette di far notificare un evento da un livello inferiore al superiore; response, per confermare la ricezione dell'evento e agire con una procedura da livello superiore a inferiore; confirm, per confermare che il livello inferiore ha terminato il servizio richiesto dal superiore e mandare il risultato.

Detto questo, possiamo parlare del livello rete in cui partecipano 3 attori. Gli RFD sono end device che non hanno funzionalità di routing e non possono inviare messaggi da soli, ma devono per forza associarsi a un coordinatore o a un FFD (router). Di conseguenza abbiamo gli FFD, che implementano l'intero stack di rete e hanno capacità di routing. Il coordinatore è un caso speciale di FFD che si occupa di gestire e creare la rete.

Questa immagine in particolare mostra il meccanismo di join in una rete da parte di un dispositivo, che può essere FFD o RFD. Siamo nel caso di accesso indiretto (e non diretto grazie a un router o al coordinator che lo aggiunge alla rete). In questo caso avviene che il nostro device richiede al livello rete una network discovery request, un comando che cerca PAN esistenti; di conseguenza, il livello rete effettua una scansione attiva alla ricerca di PAN esistenti e tramite confirm restituisce i PAN ID, con in aggiunta informazioni su router e coordinatori. A seguito vengono notificati al livello applicativo i possibili ID e reti PAN; con qualche logica il device sceglie la PAN e invia una richiesta di join al livello inferiore, che richiede una association request performata dal livello MAC (allego anche association a livello MAC).

A seguito avvengono l'associate confirm e il join confirm, e al nostro device arriva un indirizzo a 16 bit, il quale viene assegnato dal router o dal coordinator.

![[Pasted image 20260714151136.png]]
Questa immagine rappresenta il join a livello MAC ed è composto da un'associate request che richiede l'associazione a quello specifico PAN ID, inserendo anche l'indirizzo del coordinatore e il proprio indirizzo IEEE a 64 bit, che verrà sostituito da quello a 16 bit.

La richiesta viene inviata nel Contention Access Period e a seguito avviene lo scambio di messaggi fra i due device. Il router o coordinator risponde con un ACK e notifica al suo livello network che è arrivata una richiesta; viene quindi fornita una risposta, ovviamente controllando la disponibilità di indirizzi.

A seguito, con la response, viene fornito l'indirizzo a 16 bit. Dopo un tempo prestabilito, è il device che si vuole associare a fare polling e richiedere informazioni riguardo l'associazione con una data request tramite trasmissione indiretta.

Dopo aver recuperato la risposta dal MAC layer del device che si vuole collegare, si effettua l'associate confirm, mentre la comm.status.indication indica che l'associazione è conclusa con successo o con un errore.

---
![[Pasted image 20260714152307.png]]
**L'Application Framework**: ospita fino a **240 Application Objects (APO)**, ciascuno dei quali rappresenta un'applicazione definita dall'utente (es. il controllo di una lampadina o la lettura di un sensore). A ogni APO è associato un **endpoint (da 1 a 240)**. Gli endpoint fungono da veri e propri **"cavi virtuali"** (equivalenti ai socket Unix), consentendo la coesistenza di profili, dispositivi e punti di controllo differenti all'interno di un unico nodo fisico. Ogni specifica applicazione nella rete è identificata univocamente dalla combinazione `<indirizzo di rete a 16 bit, endpoint>`.

 **Lo Zigbee Device Object (ZDO)**: si colloca sull'**endpoint 0** ed è l'applicazione di gestione centrale del dispositivo. Lo ZDO è governato dallo _Zigbee Device Profile (ZDP)_ e ha il compito di coordinare i vari APO affinché si organizzino in un'applicazione distribuita. Fornisce quattro macro-servizi :
    - **Device & Service Discovery**: permette di recuperare gli indirizzi mac/rete dei nodi (Device) e di interrogarli per scoprire quali profili e servizi supportano (Service).
    - **Binding Management**: elabora le richieste per creare o rimuovere collegamenti logici nella Binding Table dell'APS, abilitando l'instradamento indiretto.
    - **Network & Node Management**: gestisce l'avvio della rete, le richieste di join/leave e il recupero delle tabelle di routing locali.

**L'Application Support Sublayer (APS)**: funge da **livello di trasporto leggero**. Dal punto di vista dei dati, l'APS permette lo scambio asincrono di messaggi tra i dispositivi e si occupa di **generare ACK end-to-end (APS-ACK)** per garantire l'affidabilità della consegna. Dal punto di vista della gestione, l'APS memorizza e mantiene tre strutture dati critiche per la rete: la _Binding Table_, la _Groups Table_ (per il multicast tra endpoint) e la _Address Map_.

---

![[Pasted image 20260714155036.png]]
Questa slide illustra le tre topologie di rete supportate a livello Network (NWK) da Zigbee: **Star (Stella)**, **Tree (Albero)** e **Mesh (Maglia)**. Le prime due possono sfruttare la struttura a **Superframe (modalità beacon-enabled)** per sincronizzare i nodi, mentre la terza richiede tassativamente una **comunicazione senza struttura superframe (modalità non beacon-enabled)**.

**Topologia a Stella (Star)**: Prevede un unico **Coordinatore centrale (PAN Coordinator)** al centro, a cui tutti gli altri dispositivi si collegano direttamente. In questa topologia non esiste il concetto di routing multi-hop: tutte le comunicazioni sono a singolo salto e avvengono esclusivamente tra il coordinatore e i nodi periferici. Tutti i nodi si sincronizzano con il superframe a partire dal Beacon emesso periodicamente dal coordinatore.

**Topologia ad Albero (Tree)**: La rete si organizza in una struttura gerarchica padre-figlio. Il **Coordinatore Zigbee funge da radice**, i **Router sono i nodi interni** (che inoltrano i messaggi e possono avere nodi figli) e gli **End Device rappresentano le foglie**. L'instradamento dei pacchetti avviene in modalità _Tree Routing_, risalendo e scendendo l'albero logico in base agli indirizzi assegnati geometricamente ai dispositivi. Questa topologia può usare il Superframe, ma ciò richiede che ciascun router intermedio si sincronizzi rigorosamente con il frame di beacon del nodo del salto successivo.

**Topologia a Maglia (Mesh)**: È una topologia _peer-to-peer_ arbitraria in cui ogni router (FFD) può stabilire un collegamento diretto con qualsiasi altro router nel suo raggio di copertura. Se un percorso si interrompe, la rete è in grado di ricalcolare dinamicamente una nuova rotta (_self-healing_). **La topologia Mesh non permette l'uso del Superframe**. Sincronizzare i beacon lungo maglie arbitrarie e dinamiche sarebbe computazionalmente impraticabile. Poiché la rete lavora in modalità asincrona (_non beacon-enabled_), **i router della Mesh non possono spegnere la radio e devono rimanere Sempre Accesi (Always ON)** per garantire l'instradamento immediato dei pacchetti, impedendo il risparmio energetico sui nodi di routing. Solo gli End Device (le foglie terminali) possono addormentarsi, svegliandosi saltuariamente per interrogare il proprio router di riferimento tramite polling asincrono (_Data Request_).

---
![[Pasted image 20260715092114.png]]
Le relazioni genitore-figlio nel join costruiscono la topologia logica ad albero in cui il coordinator è la radice, i nodi interni i router e le foglie end device/router.

Grazie a questa struttura possiamo assegnare gli indirizzi da 16 bit; al momento della configurazione della rete nel coordinator vengono fissati tre parametri: $D_m$, $R_m$ e $L_m$.

$D_m$ rappresenta il numero massimo di end device (RFD) che ciascun router/coordinator può avere, $R_m$ il numero massimo di router che un router/coordinator può avere e $L_m$ la profondità massima raggiungibile.

Questi tre parametri permettono l'assegnazione statica degli indirizzi in maniera gerarchica.

Il router al join riceve un range di indirizzi in base al router a cui si attacca come riferimento padre, e l'end device riceve un solo ID.

Il numero massimo di indirizzi con 16 bit è 65536.

Ovviamente più in alto nell'albero si collega il dispositivo, minore è il numero di hop; anche se assegniamo gli indirizzi ad albero, la topologia potrebbe essere mesh.

Definiamo l'ampiezza dell'intervallo, ovvero il numero di indirizzi assegnati al router/coordinator incluso il proprio, come:

se $L_m=d \rightarrow C(d)=1$

altrimenti $C(d) = 1 + R_m \cdot C(d+1) + D_m$

In questa immagine abbiamo che con $R_m=2$, $D_m=2$ e $L_m=3$, viene:
- $C(3)=1$
- $C(2)=1+2 \cdot 1+2=5$
- $C(1)=1+2 \cdot 5+2=13$
- $C(0)=1+2 \cdot 13+2=29$

Di conseguenza l'intervallo del coordinatore è $[0,28]$.

Questo schema permette il **Tree Routing**, un meccanismo di instradamento efficiente e a costo di memoria nullo, poiché i nodi non necessitano di tabelle di routing. Quando un router con indirizzo A riceve un pacchetto per la destinazione D, verifica se D è un suo discendente controllando se A<D<A+C(d). 

In caso positivo inoltra il pacchetto verso il basso al rispettivo figlio; in caso negativo, si limita a passare il pacchetto verso l'alto al proprio genitore. Questa modalità consente inoltre l'uso del superframe e della sincronizzazione tramite beaconing lungo i salti.

Bisogna però sottolineare che questo modello di indirizzamento è molto **rigido**. Se un ramo dell'albero satura i propri indirizzi a causa del limite geometrico, i nuovi dispositivi riceveranno un rifiuto di associazione (_association rejected_), anche se altre porzioni della rete possiedono ampi intervalli di indirizzi inutilizzati. Per ovviare a questo limite, Zigbee permette l'adozione del **Mesh Routing**, il quale mappa una topologia fisica a maglia basandosi sul protocollo dinamico ad-hoc **AODV**, rinunciando tuttavia alla sincronizzazione tramite superframe.

---
![[Pasted image 20260715095449.png]]
Nello standard Zigbee, il **Binding** rappresenta un collegamento logico unidirezionale tra un endpoint (APO) sorgente e uno o più endpoint di destinazione (o gruppi multicast) situati su altri nodi della rete.

Questo meccanismo è fondamentale per abilitare il **Routing Indiretto (****Indirect Addressing****)**, una modalità in cui il dispositivo sorgente specifica la destinazione in modo del tutto implicito. Normalmente, i messaggi vengono instradati in modo diretto tramite la tupla `<network address, endpoint>` del destinatario. Tuttavia, per dispositivi estremamente semplici, memorizzare e mantenere aggiornate le tabelle di routing è computazionalmente proibitivo, specialmente in scenari IoT in cui gli indirizzi di rete a 16-bit vengono riassegnati dinamicamente dal parent a seguito di leave o nuovi join. 

Per ovviare a questo problema l'APS (Application Support Sublayer) gestisce e fa cooperare due tabelle fondamentali:
di volatilità, 
1. **La APS Binding Table**: Memorizzata nell'APS del coordinatore e/o dei router, viene configurata e aggiornata su esplicita richiesta dello **ZDO (Zigbee Device Object)** tramite le primitive `BIND.request` e `UNBIND.request`. La Binding Table associa le sorgenti e le destinazioni basandosi esclusivamente sui loro **indirizzi fisici IEEE MAC a 64-bit**, che sono cablati in fabbrica e non cambiano mai. Ogni voce (entry) della tabella è strutturata come una tupla contenente: _indirizzo MAC sorgente, endpoint sorgente, Cluster ID_ (la funzionalità, es. On/Off), _indirizzo di destinazione MAC_ (o indirizzo di gruppo a 16-bit per il multicast) e _endpoint di destinazione_.
2. **La APS Address Map**: È la tabella che associa in tempo reale l'indirizzo fisico IEEE MAC a 64-bit di ciascun dispositivo con il suo indirizzo di rete logico corto a 16-bit (NWK address) attualmente attivo. Se un dispositivo si disconnette e si ricollega ottenendo un indirizzo a 16-bit differente, invia un annuncio sulla rete; tutti i nodi ricevono la notifica e aggiornano la propria Address Map interna, preservando intatti i binding applicativi precedentemente configurati senza bisogno di alcun intervento manuale.

Grazie a questa architettura, quando un'applicazione su un nodo limitato deve inviare un dato, esegue un'operazione asincrona inviando un messaggio ad indirizzamento indiretto in cui specifica soltanto il **proprio endpoint locale e il Cluster ID**(**identificatore a 16 bit** che definisce univocamente quella specifica interfaccia di funzionalità all'interno di un determinato profilo applicativo) della funzionalità. 
Il pacchetto viene preso in carico dal coordinatore o router che effettua una **risoluzione in due stadi**:

- Interroga la **Binding Table** usando la tupla `<MAC sorgente, EP sorgente, Cluster ID>` per ricavare il MAC e l'endpoint del destinatario.
- Interroga la **Address Map** per tradurre quel MAC di destinazione a 64-bit nel corrispondente indirizzo di rete a 16-bit attualmente attivo, potendo così instradare fisicamente il pacchetto radio sulla rete.

---
![[Pasted image 20260715104301.png]]
Un **Cluster** rappresenta un protocollo di interazione standardizzato. Informalmente, esso fornisce l'accesso a un determinato servizio o funzionalità di un _Application Object (APO)_ (ad esempio, l'azione di spegnere una luce). Nel caso più semplice può ridursi a un singolo messaggio. Ogni Cluster è identificato da un **ID a 16 bit** e definisce due elementi fondamentali: **attributi:** che memorizzano e mostrano lo stato del dispositivo in quel cluster (es. lo stato On/Off) e i **comandi:** che causano un'azione sul dispositivo manipolandone gli attributi.

Un **Application Profile** è invece la specifica del comportamento di un'intera classe di applicazioni che cooperano su più dispositivi Zigbee (es. _Home Automation_, con Profile ID `0x0104`). Ogni profilo ha un ID univoco a 16 bit assegnato dalla Zigbee Alliance. Profili differenti possono coesistere contemporaneamente sullo stesso dispositivo fisico.

Il **Device ID** rappresenta una codifica "human-readable" del tipo di dispositivo (ad esempio, ci dice se l'hardware è una lampada o un forno). È fondamentale sottolineare che **il Device ID non dice in alcun modo come comunicare con il dispositivo; questo ruolo è esclusivamente dei Cluster ID e Profile ID**. Infatti, Zigbee effettua il _Service Discovery_ nella rete basandosi rigorosamente su profili e cluster, ignorando il Device ID.
![[Pasted image 20260715105314.png|315]]

Per evitare che gli sviluppatori debbano riprogrammare da zero funzioni comuni, la **Zigbee Cluster Library (ZCL)** funge da repository centralizzato di funzionalità standardizzate, garantendo l'interoperabilità tra dispositivi di produttori diversi. 
La ZCL adotta un **paradigma Client-Server**:

- Il **Server** è il dispositivo che memorizza fisicamente gli attributi di stato (es. la lampadina).
- Il **Client** è il dispositivo che interroga o manipola tali attributi inviando comandi (es. l'interruttore). La ZCL definisce inoltre messaggi generici per leggere/scrivere attributi, configurare report di aggiornamento automatico o scoprire quali attributi sono supportati dal server.

**Analisi dell'immagine (Esempio On/Off Lighting):** Il diagramma mostra l'applicazione reale di questi concetti nel dominio funzionale del _Lighting_. Abbiamo un interruttore (_On/Off Switch_) accoppiato a una lampada semplice (_Simple Lamp_), e un regolatore (_Dimmer Switch_) accoppiato a una lampada dimmerabile (_Dimmable Lamp_).

- Le lampade fungono da **ZCL Server** poiché memorizzano lo stato On/Off e il livello di luminosità.
- Gli switch fungono da **ZCL Client** poiché inviano i comandi applicativi.
- Il **Configuration Tool** interviene nella fase di installazione agendo come un client di configurazione: il suo compito è **popolare la Binding Table** del coordinatore o dei router.
---
![[Pasted image 20260715111736.png]]


I dispositivi IoT sono tipicamente caratterizzati da forti vincoli in termini di memoria, potenza di calcolo, energia e larghezza di banda. Progettare in questo ambito impone il superamento di sfide quali:

- **L'efficienza energetica**, per massimizzare la vita utile del dispositivo in assenza di alimentazione fissa.
- **L'adattabilità**, per rispondere al cmabiamento delle condizioni ambientali e di rete.
- **La bassa complessità protocollare**, riducendo al minimo l'overhead.
- **La comunicazione multi-hop**, necessaria a causa della bassa potenza trasmissiva dei nod.
- **Le tecniche di local storage e pre-processing**, utili a elaborare i dati a bordo del nodo per ridurre al minimo le trasmissioni radio.

Spesso ci si chiede se l'evoluzione tecnologica legata alla **Legge di Moore** (che prevede il raddoppio del numero di transistor integrabili economicamente in un chip ogni due anni) possa risolvere questi vincoli. La risposta è: non necessariamente. Nell'ambito IoT, la legge di Moore si applica secondo tre diverse interpretazioni:

1. Le performance raddoppiano a parità di costo (tipico dei server/desktop).
2. La dimensione del chip si dimezza a parità di costo, riducendo anche i consumi energetici.
3. La dimensione e la potenza rimangono costanti, ma il costo si dimezza.

Nello scenario IoT, l'enfasi ricade prevalentemente sulla seconda e sulla terza interpretazione. A causa dell'effetto scala (legato all'altissimo numero di dispositivi prodotti), si preferisce utilizzare microcontrollori non all'avanguardia, bensì chip maturi, estremamente economici e miniaturizzati, che consumino il meno possibile.

Il vero collo di bottiglia fisico dell'IoT risiede nel confronto **"Intel vs Duracell"**: mentre la capacità di calcolo cresce in modo esponenziale, la densità energetica delle batterie aumenta solo linearmente. La soluzione ingegneristica consiste quindi nel minimizzare sistematicamente lo spreco energetico sfruttando la natura tipicamente ciclica e ripetitiva dei task applicativi (campionamento, elaborazione locale, memorizzazione e trasmissione/ricezione).

Questa alternanza tra periodi di attività e di inattività consente di formalizzare il Duty Cycle (**dc**), definito come la frazione del periodo totale T in cui un dispositivo o un suo singolo componente si trova in stato attivo.

Poiché i consumi cambiano drasticamente a seconda dello stato, l'obiettivo è mantenere il duty cycle applicativo il più basso possibile, congelando le componenti del sistema quando non necessarie.

**La formalizzazione matematica dell'energia e della vita utile:**

Nei sistemi CPS in corrente continua la differenza di potenziale (tensione in Volt) è pressoché costante. Di conseguenza, la potenza e l'energia dipendono linearmente solo dalla corrente (espressa in Ampere). Possiamo quindi misurare l'energia consumata in milliampere-ora (mAh).

La spesa energetica totale per ciclo ($E_t$​) è data dalla somma dei contributi energetici di ogni singolo sottosistema:

$E_t = E_{\text{processore}} + E_{\text{sensore}} + E_{\text{radio}} + E_{\text{logger}}$

Ciascun contributo viene calcolato pesando l'energia spesa nello stato attivo ($C_{\text{attivo}}$​) e in quello di sleep ($C_{\text{sleep}}$) per i rispettivi duty cycle. Ad esempio, per il processore:

$E_{\text{processore}} = C_{\text{attivo}} \cdot DC_{\text{attivo}} + C_{\text{sleep}} \cdot (1 - DC_{\text{attivo}})$

Per componenti con stati attivi multipli, come la radio (che prevede trasmissione, ricezione e sleep):

$E_{\text{radio}} = C_{\text{tx}} \cdot DC_{\text{tx}} + C_{\text{rx}} \cdot DC_{\text{rx}} + C_{\text{sleep}} \cdot (1 - DC_{\text{tx}} - DC_{\text{rx}})$

Una volta calcolato il consumo totale $E_t$​, possiamo stimare la vita utile del dispositivo (**Lifetime**, LT) espressa in cicli di lavoro:

$LT =\frac{B _0 - L}{E_{\text{totale}}}$

Dove B0​ è la capacità iniziale della batteria (in mAh) e L rappresenta la perdita complessiva dovuta alle autoscariche (leakage) della batteria stessa durante il suo funzionamento.

 Nella realtà, L non è una costante ma dipende strettamente dal tempo di vita stesso del dispositivo. Per questo motivo, l'autoscarica può essere modellata più fedelmente tramite un'equazione di ricorrenza in cui la carica residua al ciclo n (Bn​) tiene conto di una frazione di perdita costante per singolo ciclo (ϵ):

$B_n=B_{\text{n-1}} * (1-ϵ)-E$

Risolvendo questa ricorrenza, la vita utile del dispositivo in numero di cicli corrisponderà al valore di n. Anche se in realtà si ferma prima a causa dei limiti operativi.

$B_n=B_0*(1-ϵ)^{n-1} + \frac{E * ((1-ϵ)^n)-1}{ϵ}$


---
![[Pasted image 20260715114750.png]]
Questa immagine evidenzia come la riduzione aggressiva del **Duty Cycle (DC)** – ad esempio passando dal 100% al 5% – consenta di estendere in modo straordinario la vita utile di un dispositivo IoT a parità di capacità di batteria.

Per ottenere questo risultato è indispensabile implementare uno **scheduling rigoroso dei task applicativi** e un approccio al DC aggressivo. Tuttavia, la riduzione del duty cycle non è un'operazione arbitraria e incontra il limite  **dell'utilità applicativa:** Il duty cycle non deve essere abbassato al punto da compromettere lo scopo del sistema. Ad esempio, in un sensore antintrusione, effettuare un campionamento ogni 5 millisecondi garantisce la cattura di qualsiasi movimento umano; impostare un intervallo di un minuto per risparmiare energia renderebbe il sensore del tutto inutile.

 **Spegnere il processore è una decisione locale:** Lo scheduler interno al nodo conosce perfettamente i propri task e può decidere in autonomia quando congelare (_freeze_) la CPU senza impattare gli altri nodi.
 **Spegnere la radio è una decisione globale:**  spegnere la radio isola il dispositivo e introduce un trade-off stringente tra **risparmio energetico e latenza di comunicazione (Latency vs Energy)**.

Per risolvere questa complessa sfida globale e consentire ai nodi di dormire senza perdere la connettività di rete, si ricorre a **protocolli MAC specifici per IoT** questi protocolli orchestrano lo stato della radio sfruttando principalmente tre approcci:

1. La **sincronizzazione temporale** (come in **S-MAC**), in cui i nodi condividono un ciclo di _listen/sleep_ comune per scambiarsi i dati in slot predefiniti.
2. Il **Preamble Sampling / Low Power Listening** (come in **B-MAC**), dove il ricevitore esegue rapidissimi controlli di portante a intervalli regolari e il trasmettitore invia un lungo preambolo per svegliarlo.
3. Il **Polling** (come in **IEEE 802.15.4**), in cui i dispositivi constrained dormono liberamente e interrogano attivamente il parent solo quando necessitano di scaricare dati pendentià
Permettendo che i nodi si sveglino la radio quando è strettamente necessario senza sprecare risorse.

---

![[Pasted image 20260715145138.png]]
Il protocollo MAC nel contesto IoT non arbitra solo l'accesso al mezzo, ma permette di aumentare l'efficienza energetica grazie a tecniche che consentono la riduzione del duty cycle della radio, la componente più energivora.

Ha tre tipi di approccio: la sincronizzazione dei nodi (dove i nodi si svegliano per comunicare contemporaneamente), la sincronizzazione tramite preambolo e il polling (ovvero un master che coordina i nodi slave tramite beacon periodici).

In questa immagine si vede il protocollo S-MAC che permette la sincronizzazione dei nodi; quello che succede è che si accordano su uno schedule comune e si svegliano in contemporanea per comunicare. Quando un nuovo nodo si unisce alla rete, se ha esigenze specifiche propone uno schedule, altrimenti si unisce seguendo uno schedule già esistente. Il messaggio inviato per comunicare lo schedule è il frame SYNC in broadcast. Un nodo può cambiare il suo schedule se nessuno lo segue più.

Un nodo può inviare frame solo nel periodo di ascolto del vicino, quindi la radio si accende in ricezione nel proprio periodo di ascolto e in trasmissione nel periodo di ascolto del destinatario; ciò indica che il nodo deve conoscere lo schedule di tutti i suoi vicini nel range di comunicazione.

Prima di comunicare si esegue il carrier sense per vedere se il canale è occupato: se lo è, si posticipa l'invio al periodo di ascolto successivo. Per evitare collisioni si usa RTS/CTS con l'ottimizzazione dell'adaptive duty cycle, per cui chi riceve un RTS rimane acceso fino al termine della trasmissione poiché, trovandoci in un protocollo che usa una comunicazione multi-hop, potrebbe essere il prossimo hop.

La debolezza di questo approccio è data dalla latenza. Per arrivare dal punto A al punto D devo passare per B e C, e di conseguenza potrei, in caso di schedule non condiviso fra i nodi, dover aspettare il periodo di ascolto successivo. Questa situazione in realtà è mitigata dal fatto che i nodi tendono a convergere verso lo stesso schedule il più possibile. Formalizziamo quindi la latenza massima come $n \cdot T_{listen}$; con duty cycle bassi, questa latenza può diventare inaccettabile in sistemi real-time.

Inoltre, altre problematiche sono la desincronizzazione causata dal clock drift di questi dispositivi low-cost e il fatto che, in caso di topologie complesse, potrebbe risultare impossibile per un nodo avere un periodo di ascolto compatibile con i suoi vicini simultaneamente, per questo si può cambaire schedule o averne di più di uno.

---
![[Pasted image 20260715150811.png]]
Una soluzione diversa da un approccio basato sulla sincronizzazione dei nodi è quella data dal preamble sampling, in questo caso B-MAC, che elimina la necessità di coordinazione temporale.
B-MAC è più semplice di S-MAc perchè usa un solo parametro il periodo di sleep prima del sampling (wake up interval).

Si sfrutta l'LPL (Low Power Listening), ovvero il ricevitore si attiva temporaneamente e, se rileva il preambolo, rimane attivo, altrimenti torna in sleep.

Il mittente, in questo caso, manda un preambolo che deve essere più lungo del tempo di sleep dei nodi, in modo che il nodo destinatario possa captarlo e rimanere attivo.

In questo caso, il ricevitore si attiva solo per fare brevi campionamenti periodici, che sono minori in termini di costo energetico rispetto all'accendere stabilmente il sensore in ricezione; il trasmettitore, invece, invia il preambolo e il messaggio, e quindi spende di più.

Possiamo formalizzare il modello energetico come segue:

- $f_{\text{data}}$: la frequenza di trasmissione in Hz
    
- $f_{\text{check}}$: la frequenza di campionamento
    
- $P_{\text{tx}}$: potenza di trasmissione
    
- $P_{\text{rx}}$: potenza in ricezione
    
- $P_{\text{sleep}}$: potenza consumata in sleep
    

**Duty cycle trasmettente:**

$DC_{\text{tx}} = f_{\text{data}} \cdot (t_{\text{preamble}} + t_{\text{data}})$

$DC_{\text{check}} = f_{\text{check}} \cdot t_{\text{check}}$ (fa anche lui preamble sampling, è un nodo normale)

**Energia consumata trasmettitore:**

$E_t(T) = T \cdot [ P_{\text{tx}} \cdot DC_{\text{tx}} + P_{\text{rx}} \cdot DC_{\text{check}} + P_{\text{sleep}} \cdot (1 - DC_{\text{tx}} - DC_{\text{check}}) ]$

**Duty cycle ricevente:**

$DC_{\text{rec}} = f_{\text{data}} \cdot (\frac{1}{2} t_{\text{preamble}} + t_{\text{data}})$

Statisticamente, il ricevitore si sveglia in un momento casuale e non coordinato durante la trasmissione del preambolo. Di conseguenza-, in media, **si accende esattamente a metà del preambolo**, rimanendo in ascolto attivo solo per la restante metà (21​tpreamble​) prima che inizi l'effettivo frame dati.

$DC_{\text{rx}} = f_{\text{check}} \cdot t_{\text{check}}$

**Energia consumata ricevitore:**

$E_r(T) = T \cdot [ P_{\text{rx}} \cdot DC_{\text{rec}} + P_{\text{rx}} \cdot DC_{\text{rx}} + P_{\text{sleep}} \cdot (1 - DC_{\text{rec}} - DC_{\text{rx}}) ]$

**Lifetime (Durata della batteria):**

$LT_{\text{trasmitter}} = \frac{\text{batterycharge}}{E_t(1)}$

$LT_{\text{receiver}} = \frac{\text{batterycharge}}{E_r(1)}$

Punti di forza sono che non necessitàdi una topologia particolare e glis erve un solo parametro, i limiti sono che preamboli lunghi sono costosi e all'aumentare del $t_{check}$ il premabolo deve aumentare, con aumento del costo della trasmissione. 
In caso di traffico elevato può risultare meno efficiente della sincronizzazione.


---
![[Pasted image 20260715155003.png]]
Il grafico analizza l'aspettativa di vita del nodo trasmettitore in funzione del periodo di check ($t_{check}$), che corrisponde all'inverso della frequenza di campionamento del canale.

Prendendo in esame la curva relativa a un carico di traffico intenso, pari a una trasmissione al minuto, possiamo notare come l'andamento del tempo di vita sia dettato da un netto compromesso tra due costi energetici:

**Regime di $t_{check}$ piccoli (Crescita iniziale):** Per valori di $t_{check}$ molto bassi, l'aspettativa di vita parte da valori minimi. Questo non è dovuto alle trasmissioni, ma al fatto che il nodo si sveglia troppo frequentemente per controllare il canale. Il continuo accendere e spegnere la radio consuma rapidamente la batteria..
 
 **Il punto di ottimo:** Aumentando il $t_{check}$, il nodo riposa più a lungo e il tempo di vita sale rapidamente, raggiungendo un picco di massima efficienza (attorno ai 100-150 ms per questa specifica curva).

 **Regime di $t_{check}$ ampi (Decrescita):** Superato il punto ottimale, la curva inizia a crollare. Questo accade per una regola strutturale del B-MAC: il trasmettitore deve generare un preambolo che sia strettamente più lungo del periodo di sleep del ricevitore. Di conseguenza, all'aumentare del $t_{check}$, il nodo è costretto a inviare preamboli sempre più lunghi. Dovendo trasmettere un pacchetto ogni minuto, l'enorme spesa energetica in trasmissione finisce per dominare il bilancio complessivo, distruggendo i benefici del lungo riposo e abbattendo l'aspettativa di vita del nodo.

Nel caso del ricevitore diametralmente mentre nel trasmettitore il decadimento ad alti valori di $t_{check}$ è dovuto al costo di _invio_ di preamboli lunghissimi, nel **ricevitore** è legata al costo di ricezione.
 **A bassi valori di $t_{check}$ (overhead di controllo):** Il ricevitore consuma molto e ha una vita utile bassa perché si sveglia troppo spesso per campionare il canale (le accensioni della radio a vuoto, dette _idle listening_, dominano il consumo).
 **Al punto di ottimo:** Si trova il perfetto bilanciamento in cui la radio si sveglia abbastanza raramente da risparmiare energia, ma non così di rado da costringere a preamboli eccessivi.
**Ad alti valori di $t_{check}$ (la penalità del preambolo lungo):** Qui entra in gioco l'asimmetria del protocollo. Poiché il trasmettitore deve inviare un preambolo lungo almeno quanto il $t_{check}$ del ricevitore per farsi sentire, all'aumentare di $t_{check}$ la lunghezza del preambolo cresce linearmente.
In media, un ricevitore si sveglierà **a metà della trasmissione del preambolo** ($\frac{1}{2} t_{preamble}$).e il ricevitore deve tenere la radio accesa in modalità ricezione attiva per ascoltare il resto del preambolo, attendere l'inizio del pacchetto e infine ricevere il messaggio vero e proprio.

---
![[Pasted image 20260715162451.png]]

X-MAC risolve le problematiche di B-MAC date dal costo dell'invio di lunghi preamboli usando piccoli preamboli con all'interno l'indirizzo del destinatario e, in caso di ricezione di un preambolo da parte del ricevitore, viene inviato un ACK e si comunica il dato.

Viene mandato il corto preambolo con l'indirizzo del destinatario, i nodi non destinatari ignorano il preambolo e il destinatario al primo campionamento riconosce il suo ID, interrompe il campionamento con un ACK e richiede il frame.

Il mittente smette di inviare il preambolo e invia il frame.

Con B-MAC il preamble deve durare almeno T_sleep, invece con X-MAC la durata effettiva è solo il tempo che intercorre tra l'inizio della trasmissione e il momento in cui il ricevitore fa campionamento, in media 1/2 T_sleep.

**BoX-MAC**

Invece di inviare l'ID, invio direttamente una sequenza ripetuta del frame dati reale.

Si invia header + payload. Il ricevitore intercetta un'istanza del frame, se è il destinatario aspetta il pacchetto intero e invia ACK per fermare chi trasmette.

Soluzione ottima con frame di piccole dimensioni.


---
![[Pasted image 20260715164742.png]]
Lo standard IEEE 802.15.4 definisce le specifiche a livello fisico e MAC per le Low-Rate Personal Area Network (LR-PAN).

Il livello fisico eroga servizi dati con la trasmissione e ricezione delle PPDU, e servizi di management tramite l'energy detection (che identifica i canali liberi e serve al carrier sensing), l'attivazione e disattivazione della radio per il risparmio energetico, il link quality indicator (che misura la qualità dei pacchetti ricevuti da un nodo), la channel selection (che determina il canale su cui trasmettere tramite energy detection e/o carrier sense) e la configurazione della PHY-PIB.

La PPDU è il frame del livello fisico, composto da tre sezioni: la parte relativa alla sequenza di preambolo, la parte di header (in cui si specifica la grandezza) e il payload.

Detto questo, possiamo parlare del MAC layer, che si occupa di fornire meccanismi di accesso al mezzo comunicativo. Offre servizi dati con la trasmissione delle MPDU ed effettua la gestione dei servizi e delle funzioni di sicurezza.

Abbiamo sempre gli RFD, che non implementano funzionalità di routing e quindi possiedono solo un sottoinsieme dello stack MAC; possono collegarsi, ma non possono creare reti o inviare messaggi in autonomia. Gli FFD, invece, possono fungere da coordinatori di PAN o da coordinatori di un gruppo di RFD.

Le topologie che si possono avere includono quella a stella, con un FFD che funge da PAN coordinator e tutti gli altri dispositivi come RFD. Il PAN coordinator sincronizza tutte le comunicazioni nella rete. Nella topologia peer-to-peer, un FFD comunica con qualunque altro dispositivo nel suo range comunicativo, permettendo la formazione di reti mesh o ad albero.

L'accesso al canale avviene con o senza superframe. In caso di superframe, si divide il tempo in due periodi: quello attivo e quello di sleep. Quello attivo viene diviso in 16 slot; il primo slot è dedicato al coordinator che invia il beacon, il quale permette la sincronizzazione. Di seguito abbiamo due periodi, CAP e CFP. Il CAP è composto al massimo da 15 slot, nei quali si accede al canale tramite CSMA/CA slottato, competendo con gli altri nodi che vogliono trasmettere. A seguito dell'ottenimento del canale, si manda l'intero frame occupando il mezzo; se lo si trova occupato, si applica il binary exponential backoff e si ritenta dopo un numero casuale di slot.

Il CFP, invece, viene assegnato da un coordinatore a una specifica applicazione e permette di garantire l'accesso al canale per determinati slot. Questo periodo è suddiviso in GTS (Guaranteed Time Slot); ogni CFP ha a disposizione al massimo 7 GTS, ciascuno composto da uno o più slot, garantendo un accesso al canale non conteso. Il CAP è necessario anche per le operazioni di join o leave dei nodi e per le richieste di GTS. Inoltre, un dispositivo che comunica nel GTS deve concludere la sua trasmissione entro la fine del GTS stesso.

Nelle modalità in cui non si usa il beacon, si effettua l'accesso con CSMA/CA non slottato, e i coordinatori devono essere sempre attivi per ricevere i segnali dagli end device. Se c'è un messaggio dal coordinator all'end device, la comunicazione avviene tramite polling: l'end device interroga periodicamente il coordinator per sapere se ci sono messaggi per lui e, in caso affermativo, quest'ultimo glieli invia.

---

![[Pasted image 20260715172158.png]]

Lo standard IEEE 802.15.4 definisce **tre diverse direzioni per il trasferimento dei dati** a livello MAC:

1. **Da End-Device a Coordinatore** (o Router).
2. **Da Coordinatore** (o Router) **a End-Device**.
3. **Peer-to-Peer** (tra due dispositivi FFD, come due router).

Sotto il profilo topologico, la topologia a **Stella** supporta esclusivamente i trasferimenti di tipo 1 e 2, poiché ogni comunicazione deve necessariamente convergere sul PAN Coordinator centrale. Al contrario, la topologia **Peer-to-Peer** (ad esempio in strutture mesh o tree) è più versatile e **supporta tutte e tre le modalità di trasferimento**.


Beacon enabled:
**Da End-Device a Coordinatore**: Il dispositivo si sveglia e attende il **Beacon** per sincronizzarsi con la struttura del superframe.
Se il coordinatore gli ha riservato un canale dedicato (**GTS** nel CFP), il device trasmette direttamente.
Sennò il device deve contendere il canale durante il **CAP (Contention Access Period)** applicando il protocollo **CSMA-CA slottato**. Una volta trasmesso il dato, il coordinatore può rispondere con un **ACK opzionale**.

**Da Coordinatore a End-Device (Trasmissione Indiretta)**: Il coordinatore non invia il dato direttamente, ma lo memorizza temporaneamente in un buffer e segnala la presenza di un messaggio pendente inserendo l'indirizzo del device all'interno del frame di **Beacon**. L'end-device, che dorme per la maggior parte del tempo, si sveglia periodicamente per ricevere il Beacon. Se rileva di avere un messaggio pendente: Invia un frame di **Data Request** nel CAP usando il CSMA-CA slottato. Il coordinatore risponde con un **ACK immediato** per confermare la richiesta e invia il dato nel CAP successivo e il device manda ack.

 **Peer-to-Peer (tra due Router/Coordinatori)**: Non è possibile far comunicare direttamente due end-device RFD. Se la comunicazione avviene tra due router FFD, per potersi scambiare i dati il mittente deve prima sincronizzarsi con il beacon emesso dal router destinatario.

Non beacon enabled:
**Da End-Device a Coordinatore**: si usa il protocollo **CSMA-CA non slottato (unslotted)**. Il coordinatore deve essere **Sempre ON**.
**Da Coordinatore a End-Device**: Il coordinatore memorizza il messaggio e attende passivamente. È l'end-device che (con i propri tempi) si sveglia e interroga il coordinatore inviando un frame di **Data Request** tramite CSMA-CA non slottato. Il coordinatore risponde con un ACK, seguito immediatamente dal pacchetto dati (o da un messaggio vuoto se non ci sono dati pendenti), e il device conclude lo scambio con un ACK di conferma.
**Peer-to-Peer**: Ogni dispositivo FFD può comunicare direttamente con qualsiasi altro FFD nel suo raggio di copertura. Per farlo, i dispositivi devono rimanere costantemente attivi (Always ON) oppure coordinarsi tramite protocolli di sincronizzazione proprietari gestiti interamente dai **livelli superiori** dello stack.

---
![[Pasted image 20260715173712.png]]
L'**Associazione** è il processo a livello MAC attraverso cui un dispositivo entra a far parte di una PAN già esistente. È una procedura pensata per reti _beacon-enabled_ e si articola in una fase lato end-device (child) e una lato coordinatore (parent).

Prima di associarsi, il dispositivo deve identificare le PAN attive nell'area tramite un servizio di **scansione attiva** (`SCAN.request` / `SCAN.confirm`). Una volta selezionata la PAN desiderata, il livello NWK del device invoca la primitiva locale **ASSOCIATE.request**. Il livello MAC genera quindi un frame di **Association request**, che viene trasmesso durante il **CAP (Contention Access Period)** applicando il protocollo CSMA-CA slottato. Questo frame contiene il PAN ID della rete a cui connettersi, l'indirizzo del coordinatore e l'indirizzo fisico esteso IEEE a 64 bit del device.

Il coordinatore riceve la richiesta e risponde immediatamente con un **ACK** a livello MAC. Questo ACK indica esclusivamente la corretta ricezione fisica del frame, ma **non significa ancora che l'associazione sia stata accettata**. 
Il livello MAC del coordinatore inoltra la richiesta al proprio livello NWK tramite la primitiva **ASSOCIATE.indication**. Il livello NWK del coordinatore valuta la richiesta (verificando la disponibilità di indirizzi corti e lo spazio in memoria per nuovi figli):

Se la richiesta è accettata, il NWK assegna al nuovo dispositivo un **indirizzo logico a 16 bit** e invoca la primitiva **ASSOCIATE.response**.
Il MAC del coordinatore prende in carico la risposta, ma non la trasmette subito; la memorizza nel proprio buffer per abilitare una **trasmissione indiretta**.

Nel frattempo, il dispositivo che richiede l'associazione attende per un tempo prestabilito (**Pre-defined waiting time**), durante il quale può spegnere la radio per risparmiare energia. Trascorso questo tempo, il device si sveglia e interroga attivamente il coordinatore inviando un frame di polling chiamato **Data request**. A questo punto avviene la consegna:

1. Il coordinatore riceve il `Data request` e risponde con un **ACK**.
2. Subito dopo, il coordinatore estrae dal buffer e trasmette il frame di **Association response** contenente la decisione e l'indirizzo corto a 16 bit assegnato.
3. Il device riceve il frame e risponde con un **ACK** di conferma finale.

Con lo scambio concluso:

- Lato end-device, il livello MAC emette la primitiva **ASSOCIATE.confirm** verso il proprio livello NWK per confermare che l'associazione è avvenuta e comunicare il nuovo indirizzo a 16 bit da utilizzare per le future comunicazioni.
- Lato coordinatore, il livello MAC emette la primitiva **COMM-STATUS.indication** verso il proprio livello NWK per notificare che la transazione indiretta si è conclusa con successo


---

![[Pasted image 20260715174344.png]]

I sistemi embedded rappresentano dispositivi che hanno una funzione specifica e sono realizzati spesso in co-design hardware e software.

Un microcontrollore è un chip che integra microprocessore, memoria e interfacce I/O.

Ovviamente sono dispositivi limitati e per questo non hanno al loro interno un S.O., ma solo un'applicazione che gestisce gli interrupt. Molti di questi dispositivi richiedono una correttezza temporale in modo da poter lavorare in real time e tipicamente richiedono una grande affidabilità (tipicamente 99.99%). Sappiamo che la memoria è poca e costosa da aggiungere; per questo è necessario produrre codice che riesca a lavorare in tale contesto, a volte anche in maniera ingegnosa.

È ovviamente importante anche il risparmio energetico, poiché dispongono di batterie. Non avendo un S.O., la compilazione è cross, con un host che si occupa di iniettare nel firmware del microcontrollore codice già compilato e linkato staticamente con le librerie. Sappiamo che il lavoro di tali dispositivi è tipicamente ciclico; per questo è importante gestire il duty cycle per permettere l'efficienza energetica.

Tipicamente abbiamo un loop che esegue all'infinito l'operazione e una parte di setup iniziale che permette di definire gli elementi necessari al programma.

Detto questo, possiamo introdurre il modello Arduino, in cui abbiamo un singolo thread e una gestione degli eventi sincrona; ovvero, se dobbiamo fare qualche operazione di I/O, ci fermiamo e aspettiamo che termini tramite dei delay. Quindi non c'è sospensione del thread né context switch, e si mette solo il thread in attesa per il delay().

Questo modello lavora bene con attività di sensing e controllo. Per comunicazioni asincrone si usano librerie.

---

![[Pasted image 20260716102840.png]]
A differenza del modelloArduino (caratterizzato da attese bloccanti e delay), il modello di **TinyOS** è un modello **asincrono, event-driven e basato su componenti**, specificamente progettato per operare su nodi sensori con scarse risorse di memoria e calcolo.

L'architettura software di TinyOS si fonda su tre concetti:

1. **I Comandi:** Funzioni invocabili dall'alto verso il basso (dal componente applicativo verso l'hardware/OS) utilizzate per configurare e attivare l'hardware (es. per avviare la lettura di un sensore). I comandi sono non bloccanti e ritornano immediatamente il controllo al chiamante.
2. **Gli Eventi (Events):** Rappresentano l'astrazione a livello applicativo delle interruzioni hardware. Funzionano come delle **upcall** (dal basso verso l'alto). Gli eventi devono essere estremamente veloci e brevi poiché possono modificare strutture dati e impartire comandi diretti all'hardware. Possono interrompere i task applicativi.
3. **I Task:** Porzioni di codice che gestiscono l'elaborazioni complesse o lunghe differite nel tempo, evitando di tenere impegnati gli event handler. I task sono  non-preemptive l'un l'altro e vengono eseguiti sequenzialmente secondo una politica FIFO. Poiché un task non può mai mettersi in stato di attesa attiva o bloccarsi, TinyOS non ha bisogno di mantenere contesti e stack separati per ogni thread: **tutto il sistema gira su un unico stack di esecuzione condiviso**, abbattendo drasticamente l'occupazione di memoria RAM. Se un event handler riceve dati che richiedono elaborazioni pesanti, per non bloccare il sistema si limita a postare un task (post task) nella coda dello scheduler e termina immediatamente. Quando la coda dei task è vuota, la CPU entra automaticamente in **idle mode** per risparmiare energia.

Lo schema in figura illustra perfettamente questo paradigma asincrono:

1. Inizializzazione (init): Il sistema si avvia, esegue il setup del device e invoca il comando `Set timer` per programmare un timer hardware.
2. Fase di Triggering (Timer fires): Allo scadere del tempo, l'hardware genera un interrupt che TinyOS trasforma nell'evento gestito dal **Timer handler**.
3. Inizio Lettura : All'interno del `Timer handler`, l'applicazione invoca il comando non bloccante **Start read** per attivare il sensore. Il `Timer handler` si conclude all'istante, liberando subito la CPU.
4. Fine Lettura: Quando il sensore termina fisicamente la conversione analogico-digitale, genera l'evento **Read done** gestito dal **Read handler**.
5. Elaborazione in differita (Task): Il `Read handler` deve elaborare i dati letti. Per non bloccare l'arrivo di altri interrupt urgenti sul sistema, il `Read handler` esegue l'operazione locale di **post task**, riprogramma il timer per il ciclo successivo (`Set timer`) e termina.
6. **Esecuzione del Task applicativo:** Lo scheduler di TinyOS estrae il task dalla coda ed esegue l'elaborazione dei dati in background (`Data processing happens here`). Una volta pronti, i dati vengono inviati alla radio tramite il comando asincrono `Send data`.

**Regole di Concorrenza (I Modelli di Esecuzione in Figura):** Essendo un sistema asincrono, vi è il rischio di interferenze se più eventi accedono contemporaneamente alle strutture dati del sistema operativo. Per questo motivo:

- Gli handler devono essere brevissimi e non devono invocare comandi diretti allo stesso modulo hardware che li ha generati, onde evitare stati di inconsistenza.
- Il preemption model permette agli eventi (upcall hardware) di interrompere l'esecuzione di un task, ma garantisce che i task non possano mai prelazionarsi a vicenda, preservando l'integrità del sistema in modo deterministico e con overhead minimo


---
![[Pasted image 20260716111853.png]]

Lo sketch in analisi illustra l'applicazione del modello di programmazione asincrono in Arduino tramite l'utilizzo degli **interrupt esterni**, una tecnica di Arduino, che rende il comportamento simile a quello di TinyOS consentendo di gestire eventi asincroni non appena si presentano.

Arduino supporta tre tipologie di interrupt:

- **Interni di tipo Timer:** gestiti autonomamente dal runtime di Arduino per la scansione temporale.
- **Interni di tipo Device:** generati da periferiche integrate (es. ADC, linea seriale).
- **Esterni:** segnali provenienti da pin fisici esterni. **INT0** (mappato sul pin digitale 2) e **INT1** (mappato sul pin digitale 3).

L'abilitazione avviene tramite la funzione: `attachInterrupt(interrupt#, func-name, mode)` Dove `mode` definisce il tipo di trigger hardware interpretato direttamente dai circuiti del microcontrollore, garantendo tempi di risposta estremamente rapidi. Le modalità supportate sono `RISING` (passaggio da LOW a HIGH), `FALLING` (da HIGH a LOW), `CHANGE` (qualsiasi variazione) e gli stati logici costanti `LOW` o `HIGH`.

Nel codice vengono dichiarate due variabili globali: `greenLed` (pin 7) e `count`. Entrambe presentano la keyword `volatile`.

La direttiva **volatile** è un'istruzione fondamentale per il compilatore: gli impone di caricare e salvare il valore della variabile **sempre direttamente dalla memoria RAM** e mai dai registri di storage interni della CPU. Questo è obbligatorio per tutte le variabili condivise e modificate sia all'interno della ISR che nel `loop()`, come `count`, per evitare che il thread principale legga un valore obsoleto memorizzato in un registro.

**Fase di** **setup()****: Viene inizializzata la comunicazione seriale a 9600 bps, impostato il pin del LED (`greenLed`) come output e posto inizialmente a livello logico `LOW` (spento). Infine, viene registrato l'interrupt esterno: si associa `INT0` (ovvero il pin digitale 2) alla funzione di callback **interruptSwitchGreen**, configurando il trigger hardware su **RISING** (il fronte di salita tipico della pressione di un pulsante).

**Il** **loop()** **principale:** Ad ogni iterazione, la variabile `count` viene incrementata e il thread entra in uno stato di attesa tramite `delay(1000)`.

- _Comportamento dell'attesa:_ Se viene premuto il pulsante durante l'esecuzione del `delay(1000)`, l'interrupt viene ricevuto ed eseguito immediatamente. Al termine dell'esecuzione dell'ISR, la CPU riprende l'attesa del `delay()` dal punto esatto in cui era stata interrotta. Successivamente, il valore di `count` viene stampato sulla seriale. Se `count` raggiunge il valore di 10, il contatore viene azzerato, il LED viene spento e viene inviato il messaggio "now off".

**L'Interrupt Service Routine (***interruptSwitchGreen****): Quando l'evento hardware viene catturato, il flusso principale del `loop()` viene sospeso e viene eseguita l'ISR. Questa accende immediatamente il LED (`HIGH`), azzera la variabile condivisa `count` (in modo da resettare la temporizzazione dei 10 secondi nel loop) e stampa a schermo "now on".

Durante l'esecuzione di un interrupt handler, gli altri interrupt vengono temporaneamente disabilitati a livello hardware. Di conseguenza:

- **delay()** **non funziona all'interno di una ISR:** questo accade perché la funzione `delay()` si basa su un timer interno che incrementa il contatore tramite interrupt dedicati; essendo questi ultimi bloccati, la funzione causerebbe un congelamento infinito del microcontrollore.
- **millis()** **non viene incrementato** per la medesima ragione.
- Le ISR devono essere perciò **il più brevi e rapide possibile**.

Infine, il runtime di Arduino mette a disposizione le funzioni `noInterrupts()` per disabilitare globalmente la cattura di qualsiasi interrupt, `interrupts()` per riabilitarli, e `detachInterrupt(interrupt#)` per scollegare una callback specifica da un pin di interrupt esterno

---

![[Pasted image 20260716114224.png]]
Alimentare un dispositivo a batteria costringe a ragionare in termini di trade-off tra prestazioni e durata. Ovviamente, una grande batteria aumenta il ciclo di vita del dispositivo ma ne fa lievitare dimensioni e prezzo; viceversa, un processore a basso consumo estende la durata della batteria ma riduce la potenza di calcolo e la portata radio. Per questo motivo si sfrutta l'**energy harvesting**, una tecnica che consiste nel raccogliere energia da una fonte ambientale e trasformarla in corrente per il device, riducendo la dipendenza dalla batteria.
Abbiamo quindi tre elementi principali: **Sorgente di energia:** La fonte primaria ambientale (solare, eolica, ecc.), **harvester:** Il componente fisico che converte la sorgente in energia elettrica. La sua produzione è influenzata da fattori spesso fuori dal controllo del progettista.
Load rappresenta l'energia consumata dai componenti del dispositivo IoT, ciascuno con stati e consumi differenti.

La chiave di funzionamento di questi sistemi è il **disaccoppiamento tra produzione e consumo**, poiché la sorgente produce quando può, mentre il dispositivo consuma quando deve eseguire determinati task. Ci sono due approcci principali per gestire questo aspetto: adattare il carico alla disponibilità della sorgente energetica oppure usare un buffer energetico (spesso usati in combinazione).

1)  **Modello Harvest-Use**

Siamo nella situazione in cui il nostro dispositivo è collegato direttamente a un harvester che trasforma l'energia in corrente e la manda al device. L'energia viene consumata nel momento stesso in cui viene raccolta.

Per poter funzionare, deve valere la condizione:

$P_s(t) \ge P_c(t)$

Questo modello porta a due criticità:
- **Spreco:** Se $P_s(t) > P_c(t)$, la potenza in eccesso ($P_s(t) - P_c(t)$) viene persa.
- **Deficit:** Se $P_s(t) < P_c(t)$, il dispositivo si spegne per mancanza di energia.
 
 **Modello Harvest-Store-Use**


In questa situazione, oltre all'harvester e al device, abbiamo un **energy buffer** (come una batteria ricaricabile o un supercondensatore) in cui si immagazzina l'energia in eccesso. In questo modo disaccoppiamo temporalmente produzione e consumo.

Se ci basiamo su un buffer ideale con efficienza di carica $\eta = 1$ e nessuna perdita di potenza data dall'autoscarica, possiamo formalizzare il sistema  in modo che il device funziona se:

$$\int_0^T P_c(t) dt \le \int_0^T P_s(t) dt + B_0 \quad \forall T \in [0, \infty)$$

Dove $B_0$ è la carica iniziale del buffer. La batteria accumula la potenza in eccesso, ovvero $[P_s(t) - P_c(t)]^+$, e compensa i momenti di deficit fornendo l'energia mancante, ovvero $[P_c(t) - P_s(t)]^+$


2) Possiamo classificare le fonti di energia in base a due metriche principali:

- **Controllabili vs Non Controllabili:** Una fonte è controllabile se è possibile ottenere energia su richiesta (es. sorgenti RF dedicate o torce "shake-to-power"). È non controllabile se si può raccogliere energia solo quando l'ambiente la rende disponibile.
    
- **Prevedibili vs Non Prevedibili:** Una fonte è prevedibile se si basa su modelli affidabili (es. il ciclo giorno-notte per l'energia solare). È non prevedibile se non abbiamo modelli certi per stimarne l'arrivo (es. il vento).
    

Per **neutralità energetica** si intende la condizione di equilibrio in cui il nostro dispositivo non si spegne mai (la batteria o il buffer non si scaricano mai completamente) riuscendo contemporaneamente a massimizzare le sue prestazioni, ovvero la sua utilità (utility). Raggiungere questo stato è l'obiettivo finale della progettazione di un sistema IoT basato su energy harvesting.

---
![[Pasted image 20260716121745.png]]
Con un buffer ideale, quindi, non c'è potenza persa a causa dell'autoscarica, l'efficienza di carica è $\eta = 1$ e abbiamo capienza infinita. Possiamo formalizzare il funzionamento corretto se:

$$\int_0^T P_c(t) dt \le \int_0^T P_s(t) dt + B_0$$

Introducendo il fatto che il buffer non è ideale, abbiamo una perdita di potenza data dall'autoscarica, un $\eta$ che può essere minore di 1 e una capacità finita. Esprimiamo la Condizione Necessaria e Sufficiente (C.N.S.) per far sì che ci sia conservazione dell'energia come:

$$B_T = B_0 + \eta \int_0^T [P_s(t) - P_c(t)]^+ dt - \int_0^T [P_c(t) - P_s(t)]^+ dt - \int_0^T P_{\text{leak}}(t) dt \ge 0$$

Necessaria e sufficiente poichè se e violiamo la condizione il dispositivo si spenge.

Definiamo poi la Condizione Sufficiente (C.S.) della capacità finita, ovvero:

$$B_{\max} \ge B_0 + \eta \int_0^T [P_s(t) - P_c(t)]^+ dt - \int_0^T [P_c(t) - P_s(t)]^+ dt - \int_0^T P_{\text{leak}}(t) dt$$

Questa condizione è sufficiente e non necessaria poichè il violare di tale condizione ovvero con un  surplus enegetico, il dispostivo semplciemtne smette di caricare la batteria, quindi violare tale condizione non compromette il sistema.

Questa immagine (caso buffer ideale) mostra che, date tutte queste condizioni, in caso di eccesso di potenza consumiamo l'energia prodotta senza intaccare il buffer; quando invece abbiamo un deficit, usufruiamo dell'energia del buffer, e l'energia prodotta è immediatamente consumata in aggiunta a quella del buffer.

---
![[Pasted image 20260716151603.png]]
Le batterie sono celle di immagazzinamento in cui la reazione chimica interna è reversibile, quindi si possono ricaricare. Abbiamo visto che le preferite nei sistemi IoT sono le litio-ionio, che hanno un'efficienza altissima. I supercondensatori (super capacitori), invece, sono sistemi che non richiedono componenti di carica complessi e sono ideali quando l'energia è disponibile ad intervalli regolari o quando la fonte di energia è molto variabile.

Tutte le tecniche di gestione della batteria nell'energy harvesting assumono che il device abbia informazioni costantemente aggiornate sulla carica della batteria e sulla produzione attuale della fonte di harvesting. Sono due misurazioni complicate, ma ci sono dei metodi per ottenerle.

### Stima dell'Energia Prodotta (Metodo Indiretto)

Assumiamo di riuscire a misurare la carica della batteria $E_b(t)$ al tempo $t$ e che la potenza consumata $P_c(t)$ tra $t_1$ e $t_2$ sia nota.

Assumendo un buffer ideale, l'energia prodotta $E_e$ nel lasso di tempo tra $t_1$ e $t_2$ è pari a:

$$E_e = \left[ \int_{t_1}^{t_2} P_c(t) dt + E_b(t_2) - E_b(t_1) \right]^+$$

Questo approccio indiretto presenta tuttavia dei limiti:

- Se si sbaglia la misurazione della carica della batteria, si introduce direttamente un errore nella stima dell'energia prodotta.
    
- Se l'intervallo tra $t_1$ e $t_2$ è troppo grande, si perde risoluzione ed è difficile capire esattamente _quando_ l'energia era effettivamente disponibile.
    
- Stimare in modo preciso la potenza consumata dal dispositivo non è affatto semplice.
    

### Misurazione Diretta tramite Tensione e ADC

Una strada migliore e più precisa è usare un circuito elettronico dedicato per misurare direttamente il flusso di corrente e il voltaggio in uscita dalla fonte di harvesting.

Sappiamo che la carica della batteria e il voltaggio sono strettamente legati (il voltaggio cala man mano che la batteria si scarica) e, per semplificare, possiamo assumere che questa relazione sia lineare.

Considerando un device che misura il voltaggio tramite un convertitore analogico-digitale (ADC) a $d$ bit:

- Siano $B_{\min}$ e $B_{\max}$ i livelli di batteria minima e massima.
- Notare che tale carica minima, non  è una proprietà della batteria, bensì del sistema

- A questi livelli corrispondono i rispettivi valori di tensione $V_{\min}$ e $V_{\max}$.

- $X_{\min}$ e $X_{\max}$ sono i corrispondenti valori digitalizzati (interi) generati dall'ADC.


Possiamo definire il valore digitale massimo misurabile ($X_{\max}$) in base ai bit del convertitore e ricavare il valore digitale minimo ($X_{\min}$) tramite proporzione:

$$X_{\max} = 2^d - 1$$

$$X_{\min} = \text{round}\left( \frac{V_{\min}}{V_{\max}} \cdot X_{\max} \right)$$

La tabella proiettata illustra l'applicazione di questo formalismo su tre note piattaforme IoT, assumendo una capacità minima di sicurezza Bmin​=300 mAh per una batteria da 3000 mAh:

1. **Raspberry Pi (RPI):** Opera in un range di tensione tra 3.5 V (Vmin​) e 5 V (Vmax​). Avendo un ADC a 10 bit, lo zero logico di sicurezza xmin​ è calcolato come round(53.5​⋅1023)=716. Se l'ADC legge un valore pari a 1000 (tensione vicina al massimo), la formula restituisce una carica residua pari a 2798 mAh, coerente con una batteria quasi del tutto carica.
2. **Arduino:** Lavora a tensioni operative nettamente più elevate (7 V - 9 V). Con un ADC a 10 bit, il valore minimo di funzionamento è xmin​=round(97​⋅1023)=795. Una lettura digitale pari a 800 (estremamente vicina alla soglia minima) indica che la batteria è quasi scarica, stimando una capacità residua di appena 359 mAh.
3. **Tmote:** Dispositivo a bassissimo consumo operante a tensioni molto ridotte (1.8 V - 3 V). Utilizza un convertitore ADC a **12 bit** di elevata precisione (xmax​=4095), per cui la soglia di arresto è xmin​=round(31.8​⋅4095)=2457. Una lettura pari a 3277 (esattamente a metà dell'intervallo dinamico dell'ADC) restituisce uno stato di carica stimato pari a 1652 mAh, corrispondente a circa il 50% della carica utile reale del sistema

---

![[Pasted image 20260716172301.png]]

Il problema della gestione dell'energia è strettamente collegato alla natura della fonte dell'energia. L'approccio di Kansal considera fonti non controllabili (perché sennò non ci sarebbero problemi) e prevedibili (perché sennò sarebbe impossibile la gestione). Detto questo, l'obiettivo è la neutralità energetica, che consiste nel far sì che il dispositivo non si spenga mai e riesca a massimizzare l'utilità dei task, puntando quindi a una massimizzazione delle prestazioni. Quello che si fa è considerare il livello attuale e atteso della batteria e modulare il carico in modo da garantire che non scenda mai sotto le performance garantite, sia in termini di utility che dei limiti operativi della batteria.

Quindi, la neutralità energetica consiste nel mantenere le performance desiderate per sempre, differenziandosi dall'approccio di massimizzare semplicemente la vita del device.

Il dispositivo IoT è composto da diverse componenti che consumano energia; possiamo decidere se tenerle attive o spente.

Vediamo l'immagine dell'architettura harvest-store-use rivisitata, con una componente che permette di effettuare la predizione dell'energia futura.

Quindi definiamo la Condizione 1, la produzione di energia: ovvero la quantità totale di energia prodotta nell'intervallo $[0, T]$ è l'integrale da $0$ a $T$ di $P_s(t) dt$, dove $P_s$ è la potenza istantanea al tempo $t$. Assumiamo che tale potenza sia limitata da un corridoio lineare e, date due costanti reali $\rho_s$ che indica il tasso medio di generazione e $\sigma_s$ la variabilità della sorgente, definiamo che:

$$\rho_s \cdot T - \sigma_s \le E_T \le \rho_s \cdot T + \sigma_s$$

La Condizione 2 esprime il fatto che il carico, definito come l'integrale da $0$ a $T$ di $P_c(t) dt$, è limitato da:

$$\int_0^T P_c(t) dt \le \rho_c \cdot T + \delta$$

in cui $\rho_c$ rappresenta la media della potenza consumata e $\delta$ la variabilità del carico del sistema.

Definiamo il teorema di Kansal, dopo aver definito le condizioni 1 e 2, come una Condizione Sufficiente (C.S.) per la neutralità energetica se:

- $\eta \cdot \rho_s \ge \rho_c + P_{\text{leak}}$
    
- $B_0 \ge \eta \cdot \sigma_s + \delta$
    
- $B_{\max} \ge B_0$
    

Con la prima garantiamo un trend di produzione, con la seconda che la carica iniziale sia sufficiente nel caso peggiore, e con la terza definiamo la carica ammissibile iniziale rispetto alla capacità massima.

La Condizione 2 può essere soddisfatta modulando il duty cycle, ricordandosi il fatto che ridurre il duty cycle riduce il consumo energetico ma anche l'utility. Definiamo l'utility come una funzione che è:

- $0$ se $DC < DC_{\min}$
    
- Lineare se $DC$ è compreso tra $DC_{\min}$ e $DC_{\max}$
    
- Massima se $DC > DC_{\max}$
    

Detto questo, Kansal formalizza tutte queste dinamiche (produzione, carico e carica della batteria) come un problema di ottimizzazione e, invece di lavorare nel continuo, **discretizza in slot di tempo** di uguale durata. Questo approccio semplifica lo scheduling e rende il sistema più stabile. Si assegna un Duty Cycle (DC) per ogni slot, lo scheduler viene eseguito una volta al giorno e le previsioni meteo forniscono delle stime di produzione per ciascuno slot.


Definiamo i seguenti parametri:

- $k$: numero di slot nella giornata.
    
- $B(i)$: carica della batteria all'inizio dello slot $i$.
    
- $B(k+1)$: carica della batteria alla fine dello slot $k$ (ovvero alla fine della giornata).
    
- $\hat{P}_s(i)$: produzione di potenza attesa (stimata) per lo slot $i$.
    

Per ogni slot (da $1$ a $k$) si assegna un duty cycle (e conseguentemente una funzione di utilità) richiedendo che l'energia si conservi a fine ciclo:

$$B(k+1) \ge B(1)$$


Per stimare la produzione futura, Kansal utilizza un filtro **EWMA** (Exponentially Weighted Moving Average). L'ipotesi alla base è che, nello stesso slot della giornata successiva, le condizioni ambientali saranno simili allo slot della giornata odierna.

Sia $P_{s, j}(i)$ la produzione _misurata_ nello slot $i$ del giorno $j$, e $\hat{P}_{s, j}(i)$ la produzione _attesa_. Stimiamo la produzione attesa per il giorno successivo ($j+1$) come:

$$\hat{P}_{s, j+1}(i) = \alpha \cdot \hat{P}_{s, j}(i) + (1 - \alpha) \cdot P_{s, j}(i)$$

dove $\alpha$ è un parametro di peso ($0 \le \alpha \le 1$).


Data $P_{\max}$, ovvero la massima potenza consumata operando al duty cycle massimo ($DC_{\max}$), suddividiamo l'insieme degli slot da $1$ a $k$ in due categorie:

- **Sun Slot:** se $\hat{P}_s(i) \ge P_{\max}$ (la potenza attesa è sufficiente per operare al massimo).
    
- **Dark Slot:** se $\hat{P}_s(i) < P_{\max}$ (la potenza attesa non copre il consumo massimo).
    


Come facciamo a garantire l'ottimalità della soluzione? Una prima soluzione ingenua sarebbe assegnare il massimo duty cycle ai "sun slot" e il minimo ai "dark slot". Questo però non garantisce l'ottimalità globale, poiché a fine giornata potremmo trovarci con un surplus o un deficit di potenza rispetto al vincolo $B(k+1) \ge B(1)$.

L'algoritmo interviene in questo modo:

- **Gestione del Surplus:** Se a fine giornata è previsto un avanzo energetico (batteria piena e spreco), l'algoritmo alza il duty cycle in alcuni _dark slot_ (partendo da quelli con indice minore) finché c'è surplus, in modo da massimizzare l'utilità ed evitare sprechi.
    
- **Gestione del Deficit (Sottoproduzione):** Se è previsto un deficit, l'algoritmo riduce uniformemente una porzione di DC da tutti i _sun slot_, garantendo così di riuscire a completare i task della giornata senza spegnere il dispositivo.

**Vantaggi:** Permette di ottenere ottimalità e semplicità implementativa, richiede memoria ridotta ed è in grado di compiere un adattamento dinamico alle variazioni.

 **Svantaggi:** Applica uno scaling unicamente lineare al duty cycle. Modellando esclusivamente il DC, l'algoritmo non è adatto a scenari con trasduttori alternativi o dispositivi che presentano comportamenti energetici complessi.
 
---

![[Pasted image 20260716191245.png]]
Il modello classico di Kansal assume che l'unico modo per modulare il carico energetico sia variare in modo continuo e lineare il duty cycle. Nella realtà ingegneristica, questo approccio presenta due severi limiti:

1. **Irregolarità dei campionamenti:** la variazione continua del duty cycle slot per slot genera frequenze di campionamento instabili, rendendo inapplicabili i classici algoritmi di elaborazione digitale dei segnali (DSP).
2. **Monotonia della modulazione:** non permette di descrivere scelte applicative discrete e multi-modali.

Un'applicazione IoT reale esegue ciclicamente una pipeline strutturata in **4 fasi fondamentali**:

1. **Sensing** (Acquisizione dei dati fisici)
2. **Storing** (Salvataggio in memoria locale)
3. **Processing** (Elaborazione/Compressione locale a bordo del nodo)
4. **Transmitting** (Invio del dato o del risultato via radio)

Ciascuna di queste fasi ammette **implementazioni alternative** caratterizzate da differenti trade-off prestazione/consumo. Ad esempio, il dispositivo può scegliere se attivare un sensore ad alta precisione ma energivoro o uno a basso consumo, oppure se trasmettere tutti i dati grezzi direttamente alla nuvola (alto consumo radio, basso calcolo) o eseguire un classificatore locale inviando solo il risultato (basso consumo radio, alto calcolo).

Per formalizzare questa flessibilità, definiamo **Task (**Tj​**)** una specifica combinazione di implementazioni delle 4 fasi applicative. Il dispositivo ha a disposizione un insieme discreto di n task alternativi, ciascuno caratterizzato da:

- Un costo energetico unitario cj​ (consumo medio in mAh durante lo slot).
- Un'utilità applicativa discreta uj​ associata alla qualità del servizio offerto.

Dividiamo la giornata in k slot temporali di pari durata (tipicamente 24 slot da 1 ora). Per mappare la decisione dello scheduler a mezzanotte, introduciamo la **variabile decisionale binaria (booleana)**:

xi,j​={10​se il task j eˋ assegnato allo slot ialtrimenti​

Valgono i seguenti parametri energetici per ogni slot i:

- P^s​(i): la produzione energetica stimata/attesa nello slot i (fornita dall'energy predictor).
- Pc​(i)=∑j=1n​xi,j​⋅cj​: la potenza consumata nello slot i, determinata unicamente dal task selezionato.

Poiché lo scheduler deve assegnare **tassativamente un solo task per ciascuno slot**, imponiamo il vincolo di unicità:

j=1∑n​xi,j​=1∀i∈[1,k]

A seconda del bilancio istantaneo tra l'energia stimata in ingresso P^s​(i) e il costo del task selezionato Pc​(i), definiamo le variabili di raddrizzamento (tramite la funzione [x]+=max(x,0)):

- **Surplus Energetico (**ps+​(i)**):** l'energia prodotta in eccesso che viene convogliata per ricaricare il buffer: ps+​(i)=[P^s​(i)−j=1∑n​xi,j​⋅cj​]+
- **Deficit Energetico (**pc−​(i)**):** la potenza mancante che il dispositivo deve necessariamente estrarre dalla batteria per completare il task: pc−​(i)=[j=1∑n​xi,j​⋅cj​−P^s​(i)]+

Introducendo l'efficienza di carica della batteria η<1, la carica residua B(i+1) all'inizio dello slot successivo viene aggiornata tramite l'**equazione dinamica di stato**:

B(i+1)=min{Bmax​,B(i)+η⋅ps+​(i)−pc−​(i)}∀i∈[1,k]

_Dettaglio d'esame:_ L'operatore di minimo (min) con la capacità massima Bmax​ modella realisticamente la saturazione fisica dell'accumulatore. Se la batteria è completamente carica, qualsiasi ulteriore surplus energetico non viene accumulato ma viene dissipato (spreco energetico), evitando sovrastime matematiche nel modello.

L'obiettivo dello scheduler è trovare l'assegnamento ottimo dei task nei k slot che **massimizzi l'utilità cumulativa** del sistema su tutto l'orizzonte temporale, sotto stringenti vincoli di sopravvivenza energetica. Il problema viene formalizzato come segue:

maxi=1∑k​j=1∑n​xi,j​⋅uj​

**Soggetto ai vincoli:**

1. j=1∑n​xi,j​=1∀i∈[1,k] (Esclusività del task per slot).
2. B(i)≥Bmin​∀i∈[1,k] (Vincolo di sopravvivenza: la batteria non deve mai scendere sotto la soglia hardware di cut-off, pena lo spegnimento improvviso del nodo).
3. B(k+1)≥B(1) (Vincolo di Neutralità Energetica a lungo termine: la carica residua a fine giornata deve essere almeno pari a quella iniziale, garantendo l'autosostentamento perpetuo del nodo).

Questo problema di ottimizzazione è **NP-Hard** (riconducibile al problema del Knapsack multi-periodo). Tuttavia, definendo lo stato del sistema tramite la coppia (i,b)—dove i rappresenta lo slot corrente e b il livello di batteria all'inizio dello slot—è possibile trovare la soluzione ottima globale in tempo pseudo-polinomiale applicando la **Programmazione Dinamica retrograda (Backward Dynamic Programming)**:

- **All'ultimo slot** k **(Condizione al contorno guidata dalla neutralità a lungo termine):** opt(k,b)=j=1,…,nmax​{uj​:b+η⋅ps+​(k)−pc−​(k)≥B(1)} _(Selezioniamo il task_ j _a massima utilità che garantisca di terminare la giornata rispettando il vincolo di non-depauperamento della carica iniziale_ B(1)_)_.
- **Per gli slot intermedi** i **(da** k−1 **a ritroso fino a** 1**):** opt(i,b)=j=1,…,nmax​{uj​+opt(i+1,B(i+1)):B(i+1)≥Bmin​} _(Massimizziamo la somma dell'utilità immediata del task_ j _e dell'utilità ottima cumulativa calcolata per lo slot successivo, a patto che la transizione mantenga la batteria residua_ B(i+1) _sopra la soglia di sicurezza_ Bmin​_)_.

**Fattibilità Hardware su Dispositivi Constrained (Tmote, Arduino Uno):** La complessità computazionale dell'algoritmo è pari a:

O(k⋅∣B∣⋅n)

Dove ∣B∣ rappresenta la cardinalità dello spazio degli stati della batteria. Sebbene la carica della batteria sia una variabile reale e teoricamente infinita, nella realtà **lo stato della batteria** ∣B∣ **viene discretizzato sfruttando direttamente i livelli di quantizzazione del convertitore ADC** del microcontrollore.

Ad esempio, per un **Arduino Uno** con ADC a 10 bit (1024 livelli di tensione), restringendo l'intervallo operativo della batteria alle tensioni di sicurezza (es. da 3.4 V a 4.2 V), lo spazio degli stati ∣B∣ si riduce a soli **197 livelli discreti** (da 828 a 1024). Con soli 197 stati e k=24 slot, l'algoritmo richiede pochissimi kilobyte di memoria RAM (perfettamente compatibili con i miseri 2 KB di SRAM dell'ATmega328P) e viene eseguito in **meno di 0.2 secondi**, consentendo uno scheduling real-time asincrono ed estremamente preciso direttamente a bordo del nodo embedded