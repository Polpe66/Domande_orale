
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