


# Reti wireless
![[Pasted image 20260623162111.png]]
Il problema del **Terminale Nascosto** (Hidden Terminal) si verifica a causa di limitazioni fisiche intrinseche della propagazione radio, dovute principalmente a due fattori: il **path loss** (attenuazione legata alla distanza) o la presenza di **ostacoli fisici**.

Path loss p formalizzabile come $1/(fd)^n$. Tipicamente n=2 nel vuoto

Come illustrato nell'immagine, si possono distinguere due scenari:
1. **Terminale nascosto per Path Loss (Scenario a):** Il nodo A rientra nel raggio di copertura di B (e viceversa), e lo stesso vale per B e C. Tuttavia, a causa dell'eccessiva distanza tra A e C, il segnale si attenua (path loss) al punto che A e C non riescono a percepirsi reciprocamente.
	Se C tenta di comunciare con B, A è il terminale nascosto
2. **Terminale nascosto per Ostacoli (Scenario b):** A e C sarebbero teoricamente abbastanza vicini da potersi sentire, ma un oggetto fisico interposto tra loro (es. un muro o una montagna) blocca completamente la comunicazione diretta.

**La conseguenza (La Collisione):**
In entrambi i casi, la dinamica distruttiva è la medesima. Se A sta già trasmettendo dati a B, il nodo C (non potendo sentire A) dedurrà erroneamente che il canale sia libero. Iniziando la sua trasmissione verso B, i segnali di A e di C si andranno a sovrapporre fisicamente sull'antenna del nodo B, creando una **collisione catastrofica** che rende il messaggio incomprensibile.

---
![[Pasted image 20260623170214.png]]
Il problema del **Terminale Esposto** (Exposed Terminal) rappresenta una situazione di inefficienza speculare a quella del Terminale Nascosto. In questo caso, il problema non è una collisione, ma uno **spreco di capacità** (sotto-utilizzo del canale).

Come si evince dallo schema:
* Il nodo B sta attualmente trasmettendo dati verso il nodo A.
* Il nodo C (che si trova nell'area di copertura di B) vorrebbe avviare una comunicazione indipendente verso il nodo D.
* Applicando le classiche regole di ascolto del canale (Carrier Sense), il nodo C rileva il segnale di B e conclude erroneamente che il canale sia occupato, astenendosi dal trasmettere.

Tuttavia, questa attesa è inutile: se C trasmettesse verso D, il suo segnale non arriverebbe ad A (che è troppo lontano da C), e la trasmissione di B non disturba D (che è fuori dalla portata di B). Le due comunicazioni (B→A e C→D) potrebbero quindi avvenire perfettamente in parallelo. 
C si definisce "esposto" perché sente la trasmissione di B e ne viene bloccato, pur non essendo in reale conflitto.

Nella comunicazione fra A e B, C risulta il terminale esposto.

Questo scenario, unito a quello del terminale nascosto, dimostra definitivamente che i protocolli di accesso basati sul puro ascolto del trasmettitore (come il **CSMA/CD** usato in Ethernet) sono inefficaci sulle reti wireless.

---

![[Pasted image 20260623175329.png]]
Il protocollo **MACA** (*Multiple Access with Collision Avoidance*) nasce per superare le limitazioni del CSMA/CD nelle reti wireless, risolvendo in modo elegante sia il problema del terminale nascosto sia quello del terminale esposto.

Invece di rilevare le collisioni a posteriori, MACA le **previene** utilizzando due pacchetti di controllo per "prenotare" il canale radio e silenziare preventivamente i terminali vicini. La comunicazione si articola in tre fasi principali:

1. **Fase RTS (Request To Send):** Quando il nodo A vuole comunicare con B, trasmette un breve pacchetto RTS. Questo pacchetto contiene l'ID del mittente, l'ID del destinatario e, soprattutto, la **durata** prevista per la trasmissione del frame dati. Tutti i nodi nel raggio di A ricevono questo messaggio.
2. **Fase CTS (Clear To Send):** Se il nodo B è libero, risponde con un pacchetto CTS, copiando al suo interno la durata richiesta nell'RTS. Tutti i nodi nel raggio di B ricevono questo messaggio.
3. **Fase Dati:** Ottenuto il "via libera" (CTS), A inizia a trasmettere il frame dati vero e proprio in totale sicurezza.

**Gestione delle Collisioni Residue (RTS, Backoff e NAV):**
Anche con questo meccanismo, le collisioni non sono eliminate del tutto, ma confinate ai soli pacchetti di controllo. Possono infatti avvenire **collisioni fra pacchetti RTS** (es. se due nodi vicini provano ad avviare una comunicazione contemporaneamente).
Se il destinatario riceve RTS collusi (sovrapposti e incomprensibili), semplicemente non risponde inviando alcun CTS. I mittenti, non ricevendo il CTS, comprendono che c'è stata una collisione e applicano l'algoritmo di **Binary Exponential Backoff**: raddoppiano la loro finestra di attesa casuale prima di provare a inviare nuovamente l'RTS. 
I nodi estranei che invece sentono con successo un RTS o un CTS altrui, leggono il campo "durata" e aggiornano il loro **NAV (Network Allocation Vector)**, un timer interno che impone loro di restare in silenzio finché la comunicazione in corso non sarà terminata.
La perdita di un apcchetto RTS non porta perdita di informazioni.

**Gestione del Terminale Nascosto (Nodo D):**
Il nodo D si trova nel raggio di B, ma non in quello di A. Di conseguenza, non sente l'RTS di A, ma **riceve il CTS inviato da B**. Leggendo il CTS, D aggiorna il suo NAV e deduce che B sta per ricevere dei dati; pertanto, si auto-silenzia disciplinatamente per l'intera durata indicata nel pacchetto, evitando così di interferire e causare una collisione su B.

**Gestione del Terminale Esposto (Nodo C):**
Il nodo C si trova nel raggio di A, ma non in quello di B. Quando A inizia la procedura, **C riceve l'RTS, ma non riceverà mai il successivo CTS** di B (poiché è troppo lontano). Da questa asimmetria, C deduce di essere un terminale esposto: capisce che la comunicazione avverrà fuori dalla sua area di interferenza e, in caso di necessità, sarà libero di avviare una propria trasmissione parallela verso altri nodi senza causare collisioni.

---
# Mobile Network
![[Pasted image 20260623181841.png]]
# FOURIER
![[Pasted image 20260623094401.png]]
Questa immagine illustra l'intuizione alla base della **Serie di Fourier**: mostra come un segnale complesso periodico $s(t)$ possa essere decomposto in una somma infinita di componenti sinusoidali elementari. Modificando i "pesi" (ovvero le ampiezze e le fasi) di queste componenti, è possibile costruire o approssimare qualsiasi segnale periodico. 

Nello specifico, il grafico evidenzia:
* **$s_0(t)$**: la componente costante (o valore medio del segnale).
* **$s_1(t)$**: la componente fondamentale, che ha la stessa frequenza del segnale originale.
* **$s_2(t), s_3(t), \dots$**: le armoniche successive, le cui frequenze sono multipli interi della fondamentale.
---

![[Pasted image 20260623101956.png]]
La **Serie di Fourier** permette di rappresentare un segnale continuo e periodico (sviluppato in questo caso nell'intervallo $[-\pi, \pi]$ con un periodo $T = 2\pi$) come una somma infinita di componenti sinusoidali ortogonali.

La formula generale del segnale è:
$$s(t) = a_0 + \sum_{n=1}^{\infty} [a_n \cos(nt) + b_n \sin(nt)]$$

Questa rappresentazione si basa su tre componenti fondamentali, i cui "pesi" (coefficienti) si calcolano tramite i seguenti integrali:

* **La componente costante ($a_0$):** rappresenta il valore medio del segnale nell'arco del periodo 
  $$a_0 = \frac{1}{2\pi} \int_{-\pi}^{\pi} s(t) dt$$

* **I coefficienti $a_n$:** rappresentano le ampiezze delle componenti cosinusoidali (funzioni pari). Indicano quanta parte di coseno è presente nel segnale.
  $$a_n = \frac{1}{\pi} \int_{-\pi}^{\pi} s(t) \cos(nt) dt$$

* **I coefficienti $b_n$:** rappresentano le ampiezze delle componenti sinusoidali (funzioni dispari). Indicano quanta parte di seno è presente nel segnale.
  $$b_n = \frac{1}{\pi} \int_{-\pi}^{\pi} s(t) \sin(nt) dt$$

I termini **$\cos(nt)$** e **$\sin(nt)$** rappresentano le **armoniche** del segnale, ovvero le funzioni sinusoidali di base le cui frequenze sono multipli interi ($n$) della frequenza fondamentale.

---
![[Pasted image 20260623111943.png]]
Nei sistemi di telecomunicazione moderni si elaborano dati digitali discreti; pertanto, un segnale analogico continuo deve essere trasformato tramite un processo di conversione analogico-digitale, che si divide in due fasi principali: **campionamento** e **quantizzazione**. 

Durante questa trasformazione è fondamentale controllare la potenziale perdita di informazioni.

* **Campionamento (Discretizzazione nel tempo):** Consiste nell'estrarre valori dal segnale continuo in istanti di tempo regolarmente distanziati, definiti dal periodo di campionamento $T_c$ (o, equivalentemente, dalla frequenza di campionamento $f_c = 1/T_c$). 
	Sotto le opportune ipotesi - dettate dal **Teorema di Nyquist-Shannon** (campionare a una frequenza almeno doppia rispetto alla banda del segnale) - questa operazione è idealmente reversibile e non comporta perdita di informazione.

* **Quantizzazione (Discretizzazione nell'ampiezza):** Mentre il campionamento opera sul tempo, la quantizzazione lavora sulle ampiezze. Consiste nell'approssimare il valore reale (continuo) di ogni campione al valore discreto più vicino tra un insieme di livelli predefiniti. Ciascun livello viene poi codificato con un numero $R$ di bit. 

A differenza del campionamento, la quantizzazione introduce inevitabilmente una perdita di informazione, nota come **errore (o rumore) di quantizzazione**. 

**Analisi dell'immagine:**
Il grafico illustra esattamente questo fenomeno:
1. **In alto:** si osserva il segnale analogico originale (curva continua) sovrapposto al segnale quantizzato (l'andamento "a gradinata").
2. **In basso:** viene tracciato l'errore di quantizzazione nel tempo, dato dalla differenza tra il segnale originale e quello quantizzato ($e(t) = s(t) - s_q(t)$).

Per ridurre questo errore e migliorare la fedeltà del segnale, è necessario aumentare il numero di livelli (e quindi il numero di bit $R$). Tuttavia, questo comporta un costo in termini di risorse di rete: aumenta il **bitrate** (frequenza di cifra), calcolato con la formula $R_b = f_c \cdot R$.

---

![[Pasted image 20260623113804.png]]

L'**Aliasing** è un fenomeno di distorsione che si verifica durante il processo di conversione analogico-digitale quando il *sampling rate* (frequenza di campionamento) è insufficiente a catturare le repentine variazioni del segnale.

Questo problema si presenta quando non viene rispettato il **Teorema di Nyquist-Shannon**, il quale impone la seguente condizione:
$$f_c \ge 2 \cdot F_m$$
dove $f_c$ è la frequenza di campionamento e $F_m$ è la frequenza massima contenuta nello spettro di ampiezza del segnale analogico.

**Cosa succede nel dominio della frequenza:**
Se il periodo di campionamento $T_c$ è troppo grande (e, in modo equivalente, la frequenza $f_c$ è troppo bassa), la distanza tra le repliche dello spettro generato dal campionamento si riduce criticamente. Questo porta a una **sovrapposizione** delle repliche: le alte frequenze di una replica si mescolano inesorabilmente con le basse frequenze di quella adiacente. A causa di questa sovrapposizione, l'informazione originale viene corrotta, rendendo il segnale originario **indistinguibile** e impossibile da ricostruire in fase di ricezione.

---
![[Pasted image 20260623105342.png]]
La **Trasformata Discreta di Fourier (DFT)** è l'adattamento della trasformata di Fourier per l'elaborazione digitale. Mentre la Trasformata Continua (CFT) si applica a frequenze da $-\infty$ a $+\infty$ (ideale per la teoria "sulla carta"), i computer hanno memoria finita e possono operare solo su insiemi di dati limitati.

Per superare questo limite, la DFT si basa sui seguenti principi:
1. **Elaborazione a blocchi:** Prende in input un blocco finito di **$N$ campioni discreti** ($s_k$). 
2. **Finestra temporale e periodicità:** Dedotta la durata del blocco di acquisizione $T_w = N/f_c$ (dove $f_c$ è la frequenza di campionamento), l'algoritmo tratta questo segnale discreto come se fosse **periodico**, assumendo implicitamente che questo blocco di $N$ campioni si ripeta identico all'infinito.
3. **Operazioni discrete:** Sostituisce l'integrale continuo con una singola **sommatoria**.

In output, la DFT restituisce **$N$ valori complessi** ($S_n$). L'uso dei numeri complessi è fondamentale perché per descrivere ogni componente in frequenza servono due informazioni: la sua ampiezza (modulo) e il suo ritardo (fase).

Le formule che governano questa trasformazione sono:

* **DFT (Dal tempo alla frequenza):**
  $$S_n = \sum_{k=0}^{N-1} s_k e^{-j \frac{2\pi}{N} k n} \quad \text{per } n=0, 1, \dots, N-1$$

* **IDFT - Trasformata Inversa (Dalla frequenza al tempo):** Permette di ricostruire i campioni temporali del segnale partendo dai suoi valori spettrali.
  $$s_k = \frac{1}{N} \sum_{n=0}^{N-1} S_n e^{j \frac{2\pi}{N} k n} \quad \text{per } k=0, 1, \dots, N-1$$



