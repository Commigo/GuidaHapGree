Il nostro server utilizza node js come ambiente di runtime

## Node.js

Node.js è un ambiente di runtime che permette di eseguire codice JavaScript lato server, anziché solo nel browser.

### Il file package.json

Ogni progetto Node.js è corredato da un file di configurazione denominato `package.json`, che assolve a due compiti fondamentali:

-   **Definizione delle dipendenze**: elenca tutte le librerie esterne necessarie al corretto funzionamento del progetto.
-   **Creazione di script**: consente di definire comandi personalizzati per automatizzare attività ricorrenti, come l'avvio del server, l'esecuzione di test o l'aggiornamento delle dipendenze.

Il file `package.json` del nostro progetto è situato in `/GeneratedServer/package.json`. Al suo interno sono specificate le librerie utilizzate e gli script disponibili, che verranno analizzati in dettaglio nelle sezioni successive.

### Gestione delle dipendenze con npm

Per l'installazione e la gestione delle dipendenze, ci avvaliamo di npm (Node Package Manager), il gestore di pacchetti ufficiale di Node.js.

Eseguendo il comando `npm install`, npm analizza il file `package.json` e procede al download delle librerie richieste, collocandole all'interno della cartella `node_modules`. Questa cartella viene deliberatamente ignorata da git (tramite il file `.gitignore`), in quanto il suo contenuto può variare in base alla versione di Node.js e al sistema operativo in uso. Di conseguenza, ogni ambiente di sviluppo (PC, server, container Docker) necessita di eseguire nuovamente il comando `npm install` per generare la propria cartella `node_modules`.

### Esecuzione di uno Script con npm

Per eseguire uno script specificato nel file `package.json`, utilizziamo il comando `npm run <nomeScript>`.
Inoltre la maggior parte degli editor possono runnare gli script semplicemente premendo sopra al suo nome nel package.json

### Esecuzione di un programma javascript

Per eseguire un file javascript usando Node.js è sufficiente utilizzare il comando `node <pathFile>.js`. Inizialmente, il nostro server veniva avviato tramite il comando `node index.js`, che eseguiva il file `index.js`, punto di ingresso principale dell'applicazione.

Successivamente, siamo passati a TypeScript, che introduce una fase di compilazione aggiuntiva. Prima di poter essere eseguito, il codice TypeScript deve essere compilato in JavaScript. Questo processo verrà approfondito nella prossima sezione.

### Esecuzione di package con npx

All'interno del file `package.json` sono presenti alcuni script che utilizzano il comando `npx`, come ad esempio `"prismaGenerate": "npx prisma generate"`.

`npx` è uno strumento che consente di eseguire package Node.js eseguibili senza la necessità di installarli preventivamente. Questo si rivela particolarmente utile per quei package che, oltre a fornire librerie da importare nel codice, mettono a disposizione anche veri e propri tool da riga di comando.
