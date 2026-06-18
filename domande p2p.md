1. **Cosa sono gli UTXO?** 
Bitcoin si basa su un modello UTXO e non account come Ethereum, questo significa che non esiste una variabile persistente per mantenere il saldo dell'account. Il saldo viene contato aggregando gli unspent transaction output. Ogni volta che avviene una transazione gli UTXO vengono distrutti e se ne creano nuovi.

Inoltre l'output deve essere speso per intero e per questo tipicamente viene creato un UTXO change che torna come resto al mittente.

Si possono aggregare UTXO diversi per raggiungere somme maggiori, rimane la regola sommatoria di input >= sommatoria output e inoltre si ha una fee per il miner che è sommatoria input - sommatoria output.

Ovviamente ogni input deriva da un UTXO.

Il mittente forma la transazione inserendo un locking script (scriptPubKey) nell'output, che rappresenta la condizione di spesa da rispettare per ottenere i fondi. Il destinatario, fornendo la prova in una transazione successiva, referenzia quell'UTXO, fornisce l'unlocking script (scriptSig) e può spendere l'UTXO.

-----------------------
2. **Cosa è la Lightning Network, perché è stata introdotta e quali sono i principi di funzionamento?**

Il trilemma della blockchain asserisce che risulta impossibile ottenere più di due delle caratteristiche tipiche di un ambiente blockchain, che sono: sicurezza, decentralizzazione e scalabilità. Ethereum e Bitcoin favoriscono sicurezza e decentralizzazione a discapito della scalabilità.

Sono state provate soluzioni on-chain per questa problematica, ad esempio l'hard fork di Bitcoin Cash che, per gestire la situazione, ha aumentato la dimensione del blocco. Tuttavia, è stato calcolato che per raggiungere lo stesso volume di transazioni al secondo di Visa servirebbero blocchi da 8 GB: una dimensione impossibile per un ambiente decentralizzato, poiché porterebbe inevitabilmente alla centralizzazione.

Per questo motivo è nata la Lightning Network, una soluzione di livello 2 (Layer 2), basata sul concetto di salvare on-chain solo l'apertura e la chiusura del canale di pagamento, effettuando ogni transazione intermedia off-chain, definita commitment transaction.

Questa soluzione prevede 3 fasi: l'apertura del canale, ovvero i fondi vengono messi in un indirizzo multisig 2-di-2 tra i due attori; a seguito abbiamo l'interazione, in cui ogni pagamento parziale crea una commitment transaction firmata da entrambi gli attori, che ridistribuisce i fondi e "straccia" la transazione precedente (non succede letteralmente nella realtà, ma viene invalidata); e alla fine del tutto la chiusura, che consiste nel fatto che uno dei due attori carica on-chain una transazione con i saldi più recenti.

Le problematiche principali risiedono nel fatto che un attore può avere un comportamento malevolo e caricare una commitment transaction con un saldo a lui maggiormente favorevole (uno stato passato). Questo è scongiurato tramite un meccanismo di revoca dei segreti, che fornisce un modo per punire la parte in caso di comportamento errato.

Questa situazione induce il fatto che gli attori devono essere sempre al corrente dello stato della catena, motivo per cui esistono servizi appositi come le watchtower. 
Un'altra problematica possibile è il fatto che un attore potrebbe sparire a seguito dell'apertura del canale; per questo esiste il meccanismo di time lock, che permette di recuperare i fondi rimasti bloccati dopo un tot di tempo di inattività.

Un altro meccanismo importante è la possibilità di effettuare pagamenti fra due attori senza aprire un canale di pagamento diretto (che è costoso), ma sfruttando altri canali già aperti con altri attori che fanno da intermediari. Questo è possibile tramite il meccanismo HTLC (Hashed Timelock Contract): il destinatario manda l'hash del segreto al mittente; il mittente, per tutti gli attori intermedi, promette la somma che deve andare al destinatario in cambio del segreto, e così alla fine il destinatario ottiene i soldi rivelando il segreto. 

Tutto funziona in sicurezza grazie ad hash lock e time lock.

-----------------------
3. **Abbiamo parlato di unstructured overlay. Se lei vuole mandare informazioni su una rete non strutturata, quali meccanismi può utilizzare?**

Esistono diversi tipi di overlay, ovvero reti logiche posizionate a un livello di astrazione al di sopra di una rete fisica (ad esempio, un tunnel logico composto da decine di router e quindi punti di passaggio fisici).

Un primo esempio è un overlay di tipo **centralizzato**, in cui c'è un server che si occupa di mantenere le informazioni. Abbiamo un lookup in $O(1)$ ma un *Single Point of Failure* (SPOF), tipico del paradigma client-server.

A seguito di ciò, per evitare lo SPOF, esistono diversi overlay, come ad esempio l'**overlay non strutturato**, presente in Bitcoin, in cui i nodi sono collegati fra di loro come un grafo casuale. Questo approccio è ottimo per quanto riguarda il join dei nodi, poiché non è necessario rispettare nessuna topologia strutturata e resiste al *churn* (ingresso e uscita continua dei nodi). Inoltre, le risorse risultano distribuite fra i peer; questo è un vantaggio per risolvere lo SPOF, ma uno svantaggio nelle situazioni in cui dobbiamo fare peer discovery o ricerca delle risorse. In questo caso abbiamo un lookup in $O(N)$.

Una prima problematica è il bootstrap, che richiede di avere IP statici hardcodati, di usare un DNS noto oppure la cache del nodo (che si ricorda gli IP a cui era connesso se non si tratta della prima connessione). Tutto ciò soffre il fatto che l'IP non è statico, poiché ogni qualvolta che un peer accede o esce dalla rete l'IP cambia, e forzare un IP statico porterebbe a problemi di sicurezza.

La procedura di bootstrap funziona tramite meccanismo di ping-pong: il nodo si rivolge al primo nodo con un ping, il nodo target risponde pong e passa la richiesta tramite protocollo di gossip ai vicini, che a loro volta rispondono pong, permettendo l'aggiornamento della cache. La ricerca del contenuto è uguale: si chiede tramite attributi del contenuto creando una query, si effettua gossiping e infine il download diretto tramite una GET HTTP.

La ricerca dei contenuti rimane il punto debole con $O(N)$ nel caso peggiore. L'algoritmo base è il **flooding**, che sarebbe una ricerca in ampiezza con un TTL (*Time To Live*) che si decrementa ad ogni hop. Questo algoritmo garantisce resilienza, ma può portare a falsi negativi se il TTL è troppo piccolo. I messaggi duplicati si evitano grazie ad un ID univoco della query.

Una tecnica che cerca di evitare i falsi negativi è l'**expanding ring**, che effettua un flooding ma con TTL incrementale, partendo da un sottoinsieme (ovvero i vicini diretti). Questo permette di evitare di saturare la rete con messaggi e, inoltre, ha un meccanismo che permette di congelare l'ID della query all'ultimo nodo in cui è arrivata; in modo che, se dovesse arrivare una richiesta successiva, con stesso id e TTL maggiore entro un tempo limite, il nodo la trasporta senza elaborarla nuovamente, facendo risparmiare risorse.

Un'altra tecnica si basa sulle **catene di Markov**, ovvero processi stocastici privi di memoria in cui la probabilità del passo successivo è dipendente esclusivamente dal passo precedente. In questo approccio (Random Walk), si manda la query ad un nodo scelto a caso ad ogni passo, sempre con TTL decrescente. Ovviamente riduce il traffico, ma aumenta la latenza. Una sua generalizzazione è il **k-random walk**, che funziona allo stesso modo ma con $k$ cammini paralleli; quindi, il limite massimo del traffico generato è $k \cdot \text{TTL}$.

Un'altra tecnica ancora è la **Directed BFS** con l'uso dei *Routing Indices*. Semplicemente si favorisce la trasmissione della query sul vicino "migliore", in base ai risultati precedenti di ricerca e ad altri fattori. Tali informazioni si salvano nei Routing Indices, che sono strutture dati create ad hoc per questo scopo.

Altre soluzioni sono gli **overlay ibridi con super peer**, che sono peer con potenza di computazione e storage maggiori, i quali gestiscono un pool locale di peer sottoposti. In questo caso il flooding avviene esclusivamente tra i super peer, diminuendo la saturazione della banda.

Infine, abbiamo **overlay strutturati** come in Ethereum, ad esempio la **DHT** (*Distributed Hash Table*), in cui c'è un lookup garantito in $O(\log N)$. Prendono il meglio dai sistemi centralizzati e non strutturati, ovvero la capacità di scalare e di resistere al churn.

PS: gossip si usa per propagare informazioni alla rete, con massima copertura, invece flooding per mandare le query.

-----------------------
4. **Quali sono gli attacchi a Bitcoin? Se lei volesse usare un double spending attack, come lo farebbe?**

Gli attacchi possibili su Bitcoin sono: Eclipse attack, Sybil attack e Double spending.

* **Eclipse attack:** significa che l'attaccante circonda con i suoi nodi malevoli un nodo onesto e lo isola dalla rete. In Bitcoin questo risulta statisticamente difficile a causa dell'overlay non strutturato; l'attaccante dovrebbe conoscere nel dettaglio la topologia e questo risulta estremamente difficile.

* **Sybil attack:** si intende che l'attaccante crea identità fittizie per acquisire il possesso della rete. Questo tipo di attacco risulta efficiente in contesti dove, per ottenere consistenza sui dati fra i nodi distribuiti, si usano algoritmi di voto come consenso, dove avere più entità effettivamente porta un vantaggio. In Bitcoin, però, questa situazione è scongiurata grazie al fatto che ci si basa sulla potenza computazionale e non sulla presenza. Questo permette di collegarci direttamente all'attacco di double spending, che risulta statisticamente possibile ma improbabile.

* **Double spending:** in questa situazione un attaccante tenta di spendere la stessa moneta due volte. Una soluzione *naive* risulta mandare due transazioni differenti, che spendono lo stesso UTXO, a due nodi diversi e sperare che vengano registrate nella blockchain. Sappiamo che risulta impossibile poiché, a seguito di una transazione, questa viene mandata in broadcast ai nodi miner, che la validano nell'UTXO set controllando la validità delle firme e se $\sum \text{Input} \ge \sum \text{Output}$. A seguito di ciò, la transazione viene messa nella mempool che scarta tutti i doppioni; in questa situazione sicuramente non è possibile perpetrare un double spending attack. Rimane una possibilità se un attaccante spende la moneta che viene registrata sulla catena canonica, ma in modalità *stealth* prosegue una fork malevola in cui la transazione non è riportata. Questo diventa pericoloso se la fork malevola supera la catena canonica, in modo che venga applicata la *longest chain rule* e la catena canonica diventi orfana. Questo, come detto, è statisticamente possibile ma computazionalmente impossibile, poiché l'attaccante dovrebbe riuscire ad avere una forza computazionale tale da superare la catena canonica, e diventa possibile solo quando la forza computazionale posseduta diventa il 51% della forza computazionale totale. Questo non è mai accaduto, anche se nel 2014 una mining pool ha ottenuto il 38% scatenando una grande confusione nella community e creando anche danni economici. Tuttavia, l'attacco non è avvenuto, poiché Bitcoin ha un meccanismo che permette di favorire chi è onesto; questo indica che sovvertire le regole non porta a nessun maggiore guadagno economico rispetto a seguirle, e per questo i nodi rimangono onesti.
* * **Transaction malleability:** attacco che sfrutta una particolarità della firma ECDSA, in cui una transazione firmata risulta valida sia con gli attributi originali della firma (r, s), sia a causa della natura delle curve ellittiche con (r, n-s). Questo viene sfruttato per modificare l'ID della transazione (TxID), che è l'hash della transazione lasciando tutti gli altri campi uguali e alterando solo i parametri della firma senza invalidarla.
  Classico esempio: Alice manda 1 BTC a Bob con ID della transazione 1234. Bob, prima che la transazione venga registrata on-chain, effettua l'attacco e genera la stessa transazione, ma con i parametri della firma contraffatti, ottenendo un nuovo ID (es. 5678).
   A questo punto possono avvenire due situazioni: se viene registrata la prima transazione (1234), l'attacco fallisce; se invece si registra prima la transazione con ID 5678, il destinatario (Bob) può affermare di non aver mai ricevuto i fondi. Il mittente (Alice) cercherà di mostrare la ricevuta della transazione con ID 1234, che non mai stata registrata. La rete scarta sempre  una delle due poiché si renderà conto che sono un duplicato (stessi input spesi).
   *Soluzione:* Questo attacco è stato risolto con l'aggiornamento **SegWit** (Segregated Witness), che sposta i parametri della firma ECDSA all'interno del campo *witness*, il quale non viene utilizzato nel calcolo dell'hash per generare l'ID della transazione.

* **DNS poisoning:** se un attaccante riesce a manomettere il DNS, ovviamente può forzare un nuovo nodo a connettersi esclusivamente a nodi malevoli (facilitando, ad esempio, un Eclipse attack).

* **Network listening:** lo sniffing del traffico di rete, che però risulta inefficace grazie alla cifratura odierna.
-----------------------
5. **Distributed hash table: qual è la differenza tra un hash crittografico e un consistent hashing? Tre caratteristiche importanti dell'hash crittografico.**
Parliamo di overlay strutturati, che prendono il meglio dagli overlay centralizzati (ovvero capacità di avere lookup certi e scalabilità nelle query) e dalle reti non strutturate (da cui prendono la resistenza al churn e la decentralizzazione). Questo porta ad un lookup in $O(\log N)$ e uno spazio per le tabelle di routing occupato in $O(\log N)$.

Possiamo definire un hash crittografico come una funzione che mappa dati di lunghezza arbitraria in una lunghezza fissa, con ulteriori caratteristiche di sicurezza rispetto all'hash classico:

* **Determinismo:** stesso input, stesso output.
* **Veloce a calcolare**
* **Pre-image resistance (Resistenza alla pre-immagine):** dato un risultato, è computazionalmente difficile trovare l'input originale.
* **Avalanche effect (Effetto valanga):** il minimo cambiamento dell'input rende l'output totalmente diverso.
* **Second pre-image resistance:** dato un input noto e il risultato noto, risulta impossibile trovare un altro valore che, hashato, fornisca lo stesso risultato.
* **Resistenza alle collisioni:** deve risultare computazionalmente difficile trovare due input tali che l'hash sia uguale. Abbiamo parlato del principio dei cassetti (o delle buche dei piccioni) e del paradosso del compleanno, che formalizza il numero di tentativi per poter trovare una collisione come $2^{n/2}$ (con $n$ bit).
* **Puzzle friendliness:** il fatto che per trovare un valore che, accoppiato con un valore noto, produca un certo output (principio fondante della PoW) non abbia scorciatoie, ma richieda una computazione completa e pesante.
* **Hiding**: dato un dato r a alta min entropia e un risultato risulta impossibile ricavare informazioni su x concatenato con r che hashato produce l'output

Detto ciò, se uniamo l'hash al campo delle DHT soffriamo il problema del rehashing, poiché magari noi dobbiamo salvare informazioni su 4 server quindi abbiamo una funzione di hash che mappa i 4 server; in caso di rehashing, quindi se un server viene aggiunto o tolto, siamo costretti a rimappare circa il 99% dei dati e questo è computazionalmente impossibile per soluzioni con molti server. Per questo si usa il **consistent hashing**, che si basa sul mappare nodi e chiavi sullo stesso spazio di indirizzamento in modo che in caso di rehashing non dobbiamo mappare il 99% ma $K/N$, in cui $K$ è il numero di chiavi e $N$ il numero di nodi.

Tipicamente ad anello circolare come in Chord oppure ad albero come in Kademlia


-----------------------
6. **Come funziona Chord?**

Chord è una DHT che mappa nodi e chiavi in uno spazio circolare. Sappiamo che il numero di indici è maggiore del numero dei  nodi e vanno, ad esempio, da $0$ a $m$. Si costruisce una struttura circolare con $m$ indici e, a seguito, si cominciano a mappare i nodi nello spazio di indirizzamento. È importante dire che le chiavi verranno mappate all'interno del nodo se l'output coincide con l'indice del nodo, oppure al prossimo primo nodo disponibile. Importante è la funzione `succ()`, che gira in senso orario trovando il primo nodo disponibile. 

Il *churn* non arreca problemi poiché, in caso di rimozione di un nodo, tutti le chiavi vengono spostati al successore (`succ()`) e, in caso di aggiunta di un nodo, vengono rimappate le chiavi che cadono nel range dopo il nodo precedente fino al nodo aggiunto (Rehashing locale).

La problematica di questo approccio, senza alcuna struttura dati aggiuntiva, è il fatto che abbiamo un lookup in $O(N)$ nel caso peggiore. Per questo si utilizza una cosiddetta **finger table**, che permette di salvare in ciascun nodo dei percorsi veloci tramite puntatori a distanza esponenziale. La formula è:
$$\text{succ}(n + 2^{i-1})$$In questo modo, con $i$ che va da $1$ a $m$, avrò puntatori a $\text{succ}(n+1)$, $\text{succ}(n+2)$, $\text{succ}(n+4)$... In questo modo il costo di lookup cala a $O(\log N)$.

-----------------------

7. **Quali sono le caratteristiche di un hash crittografico? Non-invertibilità, collision resistance.**
Le caratteristiche principali dell'hash crittografico sono:
- **Determinismo**
- **Fast computation**
- **Pre-image resistance:** incapacità di riuscire a capire l'input dall'output.
- **Collision resistance:** dati due input deve essere computazionalmente difficile trovare una collisione (paradosso dei compleanni, ecc.).
- **Second pre-image resistance:** dato un input noto e il risultato noto, risulta impossibile trovare un altro valore che, hashato, fornisca lo stesso risultato.
- **Puzzle friendliness**:  il fatto che per trovare un valore che, accoppiato con un valore noto, produca un certo output (principio fondante della PoW) non abbia scorciatoie, ma richieda una computazione completa e pesante.
- **Hiding**: dato un dato r a alta min entropia e un risultato risulta impossibile ricavare informazioni su x concatenato con r che hashato produce l'output
- **Effetto valanga**: il minimo cambiamento dell'input rende l'output totalmente diverso.
-----------------------
8. **Qual è la regola utilizzata da Kademlia per mappare oggetti e nodi? Come sono fatte le tabelle di Kademlia?**

Kademlia rappresenta un'applicazione di overlay strutturato che si distingue da Chord grazie alla possibilità di avere parallel routing, un'aggiornamento automatico con il lookup e lo XOR. Inoltre, invece di avere una struttura ad anello, utilizza una struttura a **TRIE binario**. 

Essendo una DHT (*Distributed Hash Table*) con *consistent hashing*, nodi e chiavi sono mappati nello stesso spazio di indirizzamento; quindi, per creare il TRIE basta posizionare un albero binario completo con gli indici e, a seguito, mappare i nodi con il loro hash. 

Dopo aver fatto questo risulta possibile mappare le chiavi: siamo nella situazione in cui la chiave viene assegnata al nodo con il prefisso comune più lungo (*lowest common anchestor*). In caso di pareggio, si guarda come spareggio il bit che differisce tra i prefissi dei due nodi e si sceglie quello che ha il bit in comune con la chiave. 

Questa tecnica, in caso di ricerca di nodi vicini alle chiavi, permette di eseguire due approcci: una ricerca *brute force* dei nodi, oppure una ricerca tramite lo XOR. L'operatore **XOR** permette di calcolare la distanza logica fra due identificatori; questa non è una distanza geografica, ma una distanza metrica determinata dalla dislocazione dei nodi all'interno dell'albero. XOR è simmetrica d(x,y) = d(y,x).

All'interno di ogni nodo abbiamo delle routing table divise in M segmenti, denominate **k-bucket**, che contengono distanze $$[2^{i-1}, 2^i)$$ Ogni k-bucket contiene k contatti (ID, IP, porta UDP), ordinati in base all'ultima visualizzazione e grazie ai k-bucket ho la possibilità di avvicinarmi di un bit ad ogni passo.

I nodi vecchi vengono preferiti ai nuovi, poichè statisticamente risultano più affidabili.

Come anticipato con Kademlia abbiamo routing parallelo e l'interazione parallela e iterativa termian quando non si ricevono conoscenza migliori rispetto alle precedenti. O(log n).

	Primitive  UDP: PING, STORE, FIND_NODE, FIND_VALUE

- Join: con bootstrap node e interrogando se stessi e a seguito vicini sempre più lontani in modo da riempire le k-bucket.
- Store: Si inserisce il valore associato alla chiave tramite STORE ai k nodi con ID più vicini, serve refresh periodico, perchè i dati sono soft state.
- Leave: se un nodo abbandona, la k-bucket viene silenziosamente dismessa, senza dover fare nulla.

-----------------------
9. **Bitcoin: Cos'è un escrow?**
L'escrow rappresenta una situazione in cui ci sono due attori che vogliono fare uno scambio (ad esempio denaro-bene) in un ambiente non fidato; per fare ciò si affidano ad una terza persona che è onesta e fidata, che faccia da giudice. Questa situazione può avvenire anche in Bitcoin ed è fattibile grazie agli indirizzi multisig. Gli indirizzi multisig, detti multifirma, sono fondi che possono essere bloccati e mossi al raggiungimento di M su N firme richieste, situazione tipica dei conti cointestati 2-di-2.

Il funzionamento dell'escrow è spiegabile col seguente esempio: Alice vuole comprare un libro da Bob e Judy fa da giudice.

Viene creato un indirizzo multifirma 2-di-3 in cui Alice crea e pubblica una funding transaction. A seguito, possono accadere 3 situazioni:
- Alice riceve il libro spedito da Bob, tutto è andato bene: Alice e Bob firmano la tx e Bob riceve i soldi.
- Alice riceve il libro ma non vuole pagare: in questa situazione Bob e Judy firmano, così i fondi vanno a Bob.
- Alice non riceve il libro e in questo momento i soldi sono fermi: Alice e Judy firmano per poter far riottenere i soldi ad Alice.

Tutto questo è possibile solo se Judy è fidata e non ha comportamenti bizantini.

-----------------------
10. **Cos'è un consistent hashing?**
**Consistent hashing**, che si basa sul mappare nodi e chiavi sullo stesso spazio di indirizzamento in modo che in caso di rehashing non dobbiamo mappare il 99% ma $K/N$, in cui $K$ è il numero di chiavi e $N$ il numero di nodi.

Si differenzia dall'hashing normale proprio per il numero di chiavi rimappate.

-----------------------
11. **Cos'è il Merkle Tree? Quando si usa il Merkle Tree Root e una Merkle Proof in Bitcoin? Voleva sapere degli SPV nodes, i light client.**
Struttura dati ad albero basata su hash crittografico che permette di essere *tamper-evident* (a prova di manomissione).

Esempio di salvare dati sul cloud e controllare che siano originali:
* **Strategia naive:** mantenere in locale la copia; non ha senso se non si dispone di spazio.
* **Strategia hash:** effettuo l'hash totale del file, ma tutte le volte devo scaricare il contenuto intero per controllare se vi sono state modifiche.
* **Soluzione maggiormente efficiente:** Merkle tree con Merkle proof in $O(\log N)$ e $O(1)$ di spazio occupato per la root.

In poche parole, un Merkle tree è un albero che nasce con una strategia *bottom-up*, in cui si parte dalla suddivisione del dato in porzioni più piccole chunk per favorire una ricerca granulare. Si effettua l'hash della risorsa, il digest viene concatenato con l'hash del fratello e viene hashato nuovamente creando il padre, che rappresenta l'impegno crittografico dei figli. Salendo, otteniamo gradualmente la radice che rappresenta l'impegno crittografico di tutto l'albero e salviamo la root.

Questo ci permette di fare la Merkle proof, che consiste nel chiedere un dato di interesse al provider, che ce lo fornisce e a seguito ci fornirà anche il fratello hashato, così siamo in grado di formare il padre. A seguito verrà fornito il fratello del padre e così salendo fino ad ottenere una seconda Merkle root che confronteremo con quella originale in memoria. Questo garantisce che non ci siano manipolazioni a un basso costo computazionale e di storage.

La proof di non inclusività consiste nel testare mettendo in ordine un dato e testandolo con il dato a sx e a dx; se non appartiene, ovviamente la nuova radice è differente dalla vecchia.

Classico uso in bitcoin è per salvare le transazione all'interno di un blocco e oppure nel csaso degli svp client 
. Infatti ci sono diversi attori che partecipano a bitocoin: 
- Bitcoin core: wallet, mining, blockchain completa e routing P2P
- Full node: blockachian compelta e routing P2P
- Solo miner: minign, blockchain completa e routing P2P
- SVP wallet: wallet e routing P2P
SVP non ha tutta la blockchain scaricata a casua del poco spazio e scarica solo i block header, sfruttando un filtro di bloom per la ricerca efficace e per la privacy, e usa merkle proof per controllare se una trnasazione appartiene al blocco.

-----------------------
12. **Bitcoin: proof of work, time distance between blocks, scripts.**

Bitcoin basa il suo consenso sulla PoW, basata sulla forza computazionale e non sullo stake come in Ethereum. Il funzionamento è il seguente: si effettua un doppio hash SHA-256 del block header e del *nonce* (con il *nonce* che è una variabile casuale), finché non otteniamo un hash che rientra al di sotto di un target, definito *difficulty target*.

La prova è elegante perché, grazie al principio di *puzzle friendliness*, non è possibile ottenere una soluzione con scorciatoie; si può vincere la PoW solo con una computazione *brute force* (nel 2015 sono stati stimati circa $2^{70}$ tentativi richiesti). A seguito della PoW è possibile essere selezionati per proporre un blocco.

La difficoltà del target può essere aggiustata con il fine di mantenere il *block rate* a ogni 10 minuti (effetuando tuning della difficoltà ogni 2016 blocchi); questa scelta non è casuale, permette di evitare la creazione di troppi fork, situazione che comunque avviene e mina momentaneamente la consistenza della chain.

I Bitcoin sono prefissati a 21.000.000 e finiranno nel 2140; la pressione deflazionistica è garantita dal dimezzamento delle fee guadagnate dalla creazione del blocco ogni 4 anni.

Il block header è composto da: Merkle root, puntatore all'hash del blocco precedente, version, timestamp, nonce, difficoltà del target. 

Detto questo, la transazione è composta da: version, input (script di sblocco, lista utxo), output (satoshi e script di blocco).

Gli script rappresentano una parte interessante di Bitcoin; sono scritti in un linguaggio *Forth-like* non Turing completo e *stack-based*, senza uno stato persistente e con totale assenza di cicli.

Lo script di blocco rappresenta il destinatario della tx, poiché indica le condizioni di sblocco per muovere e spendere in una tx successiva quell'UTXO.

Ne esistono di diversi tipi: P2PK, P2PKH, P2SH e quelli SegWit. Lo scipt di sblocco è la prova fornita dal destinatario per poter muovere i fondi.

-----------------------
13. **What is the biggest difference between Ethereum and Bitcoin?** 
Le più grandi differenze tra i due sono il fatto che Bitcoin sia semplicemente un *ledger* distribuito, invece Ethereum condivide pure la computazione, quindi è un vero e proprio computer distribuito. Questo porta ad una gestione diversa dei servizi, poiché avendo a disposizione una EVM è possibile fare diverse cose grazie anche a Solidity, che rappresenta un linguaggio Turing completo. L'ecosistema Ethereum fornisce persistenza dello stato, cosa che Bitcoin non ha. A livello di astrazione possiamo immaginarci Bitcoin come una macchina a stati con uno stato che viene modificato da una transazione; invece, con Ethereum abbiamo uno stato globale persistente composto da contratti e transazioni, che può essere modificato dalle transazioni, con l'uso di questo potente strumento detto EVM. Inoltre ethereum è account based, invece bitcoin è utxo based. Bitcoin ha un overlay nons trutturato e ehtreum è strutturato, il consenso è diverso PoS e PoW
-----------------------
14. **Blockchain trilemma**
Indica la difficiltà di ottenere all'intero di una blockchain tutte e tre le proprietà, al massimo due. Scalabilità, sicurezza e decentralizzazione.
-----------------------
15. What are some ways to solve scalability in Bitcoin and Ethereum?(talked about the lightning network, block size, time intervals) 
Sappiamo che la scalabilità rappresenta un problema, come specificato anche nel trilemma della blockchain; la soluzione risulta modificare qualche caratteristica per cercare di migliorare la situazione. Un tipico esempio è la Lightning Network, che consiste nel limitare le tx on-chain e registrare semplicemente l'apertura di un canale e la chiusura, non registrando le transazioni intermedie.  

Aumentare il *block size* non rappresenta una soluzione ottimale poiché è stimato che, per raggiungere la scalabilità di Visa, servirebbero blocchi da 8 GB e questo, inevitabilmente, creerebbe centralizzazione. Segwit ha aumentato di poco il block size.
Dal lato di Ethereum abbiamo un *block rate* migliore (un blocco ogni slot di 12 secondi), ma persiste il fatto che non si può raggiungere una scalabilità tale da competere con altri canali di pagamento tradizionali.



-----------------------
16. How are data organized in Ethereum? What is the structure of the Merkle Patricia Trie.
Per quanto riguarda il livello data link, Ethereum dispone di 4 strutture dati fondamentali:

* **World State Trie**: rappresenta il trie che contiene il riferimento a ciascun account (EOA o contratto); contiene balance, nonce, hashcode (in caso di bytecode del contratto) e storageRoot (account storage trie).
* **Account storage trie:**  trie che mantiene lo stato di storage dello smart contract (in caso di EOA è vuoto).
* **Receipt Trie**: contiene tutte le ricevute delle tx, con cui si forma un log e un log bloom per favorire ricerche efficienti su parametri di filtraggio; rappresenta un punto di contatto tra on-chain e off-chain, cosa che in Bitcoin è assente.
* **Transaction Trie**: contiene il trie con tutte le tx, utile per trovare la validità di una tx tramite Merkle proof.
Il **Merkle Patricia Trie** è questo trie maggiormente economico rispetto al trie classico, che è composto da un bit per arco. Qua abbiamo un raggruppamento di prefissi simili e, a seguito, viene effettuato l'hash. Abbiamo diversi nodi importanti: il nodo foglia (leaf node), che rappresenta il contenuto; il nodo che rappresenta l'arco (extension node) e quindi i prefissi comuni; e il nodo che permette di ramificare (branch node). L'informazione viene divisa in nibble per avere maggiore granularità
-----------------------

