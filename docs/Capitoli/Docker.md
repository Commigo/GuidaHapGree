# Docker

Per evitare problemi legati a differenze negli ambienti di esecuzione (come versioni di Node.js diverse, file mancanti non tracciati, ecc.), utilizziamo [Docker](https://www.docker.com/). Questo strumento permette di costruire un ambiente di esecuzione standardizzato, chiamato **immagine**, per il nostro server, che può essere eseguito su qualsiasi macchina dotata di Docker.

Nei capitoli successivi ci sono dei riassunti delle funzionalità di docker e come usarle, però è probabile che non bastino per capire tutto, 
l'obiettivo della guida è di capire come queste cose sono applicate al nostro server e non di spiegare
il funzionamento di docker in maniera esaustiva, quindi se qualcosa sul funzionamento di docker stesso non è chiaro consulare la documentazione ufficiale [Docker](https://www.docker.com/)

## Immagini e Container

Il concetto fondamentale di Docker si basa su due elementi principali:

- **Immagine**: Un'immagine è un pacchetto eseguibile che include tutto il necessario per avviare un'applicazione: il codice, un ambiente di esecuzione (runtime), le librerie, le variabili d'ambiente e i file di configurazione. Le immagini rappresentano la struttura e lo stato dell'ambiente di esecuzione.

- **Container**: Un container è un'istanza in esecuzione di un'immagine. È un ambiente isolato e leggero che esegue l'applicazione. È possibile avviare più container dalla stessa immagine, ognuno con il proprio stato indipendente.

## Il Dockerfile

Per creare un'immagine, si utilizza un file di testo chiamato `Dockerfile`. Questo file contiene una serie di istruzioni che Docker esegue in sequenza. È possibile partire da un'immagine di base (ad esempio, una con Node.js preinstallato) o da un ambiente vuoto (`scratch`).

Ecco un esempio di un `Dockerfile` che usavamo inizialmente per il nostro server:

```dockerfile
# Parte da un'immagine di base con Node.js e Alpine Linux
FROM node:alpine

# Imposta la directory di lavoro all'interno del container
WORKDIR /usr/src/app

# Copia i file del server nella directory di lavoro
COPY ./Server ./

# Installa le dipendenze di Node.js
RUN npm ci

# Esponi la porta su cui il server sarà in ascolto
EXPOSE 3000

# Comando da eseguire all'avvio del container per far partire il server
CMD [ "npm", "run" ,"start" ]
```

### Esecuzione Consistente

Una volta generata, un'immagine garantisce che l'ambiente interno del container sia **identico** su qualsiasi macchina venga eseguito. Problemi comuni come versioni di software differenti, librerie mancanti o dipendenze non aggiornate vengono eliminati, garantendo che se un'applicazione funziona su una macchina, funzionerà allo stesso modo su tutte le altre.

## Interagire con i Container

Nonostante l'ambiente interno sia consistente, è possibile influenzare il comportamento di un container dall'esterno al momento dell'avvio. I due modi principali per farlo sono tramite le **variabili d'ambiente** and i **volumi**.

### Variabili d'Ambiente

È possibile impostare variabili d'ambiente specifiche per un container al suo avvio. Questo permette di modificare il comportamento dell'applicazione senza dover cambiare l'immagine.

Ad esempio, possiamo usare la stessa immagine per l'ambiente di produzione e di test, ma passare variabili d'ambiente diverse:
- Il container di **produzione** riceverà le credenziali per connettersi al database di produzione.
- Il container di **test** riceverà le credenziali per il database di test.

### Volumi

Di default, i container sono **effimeri**: qualsiasi modifica apportata al loro filesystem (creazione o modifica di file) viene persa quando il container viene fermato e rimosso.

Per salvare dati in modo persistente, si utilizzano i **volumi**. Un volume mappa una directory della macchina host (la macchina che esegue Docker) a una directory all'interno del container. Tutte le modifiche apportate ai file in quella directory del container vengono riflesse sulla macchina host e persistono anche dopo la distruzione del container.

Questo meccanismo può influenzare il comportamento del container, poiché il contenuto delle cartelle mappate dipende dalla macchina host e non dall'immagine originale.

## Conclusione

Docker risolve in modo efficace i problemi di consistenza dell'ambiente di esecuzione. Tuttavia, è fondamentale essere consapevoli che il comportamento di un container può essere influenzato da configurazioni esterne come le **variabili d'ambiente** e i **volumi**. Una gestione attenta di questi elementi è essenziale per garantire che i container si comportino sempre come previsto.
