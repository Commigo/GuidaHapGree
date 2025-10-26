# Fastify

Per poter implementare le API Rest, il nostro server fa uso della libreria [*Fastify*](https://fastify.dev/).

Fastify permette di definire una serie di *route* a cui il nostro server può rispondere. Per ogni route, è possibile specificare il metodo HTTP e il percorso. La libreria si occupa di gestire la deserializzazione e la serializzazione in maniera automatica, convertendo da e verso il formato JSON.

Il meccanismo di funzionamento prevede che per ogni route si definisca una funzione handler. Questa funzione riceve come parametri un oggetto `request` e un oggetto `reply`. L'oggetto `request` serve per leggere i parametri della richiesta in ingresso, mentre l'oggetto `reply` viene utilizzato per costruire e inviare la risposta, impostando il corpo (solitamente in JSON) e altri parametri come lo status code HTTP.

Consiglio di integrare quanto appena letto con la documentazione ufficiale di [*Fastify*](https://fastify.dev/).

## Esempi di Route

Nel nostro server, alcuni esempi di route sono i seguenti:

```typescript
fastify.post('/food/receipt', {preHandler: FoodInterceptors.createReceipt }, FoodControllers.createReceipt);
fastify.delete('/food/receipt/:id', {preHandler: FoodInterceptors.deleteFoodReceipt }, FoodControllers.deleteFoodReceipt);
fastify.get('/food/:ean', {preHandler: FoodInterceptors.foodEanGet }, FoodControllers.foodEanGet);
```

## Architettura del Server

Nel nostro server, ognuna di queste route è assegnata a un **Controller**, che si occupa di:
- Usare l'oggetto `request` per leggere i parametri passati.
- Passare i parametri al **Service** corrispondente.
- Usare l'oggetto `reply` per restituire la risposta in formato JSON.

Nelle sezioni future vedremo in dettaglio come il server è organizzato secondo l'architettura a **Controller**, **Service** e **Model**.

## Configurazione e Avvio

Le route del nostro server vengono definite nel file di avvio `index.ts`, insieme ad altre configurazioni. Successivamente, il server viene avviato tramite il metodo `fastify.listen()`, che lo mette in ascolto su una porta specifica per poter rispondere alle richieste HTTP.

### Configurazioni Aggiuntive

Sempre nel file `index.ts`, sono state aggiunte alcune configurazioni degne di nota a Fastify. Ad esempio, per la gestione dei file, è stata implementata una funzione che serializza i file secondo un criterio specifico. Questo fa in modo che il valore del file sia trattato come un normale parametro nell'oggetto `request`, evitando la necessità di deserializzarlo manualmente nel controller, come avverrebbe con la configurazione di default.
