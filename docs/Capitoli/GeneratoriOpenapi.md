# Generatori OpenAPI

Come menzionato nel capitolo [OpenAPI](/Capitoli/OpenApi.md), a partire dalla specifica OpenAPI del server siamo in grado di generare sia il progetto Bruno che parte dell'implementazione del server stesso.

Entrambi sono generati a partire da un tool che è un fork del generatore ufficiale di OpenAPI. La repository del fork è [questa](https://github.com/Commigo/BrunoOpenApiGenerator).

Il progetto è scritto in Java. Per poter rendere semplice l'utilizzo del tool senza dover installare Java o altre dipendenze, è stato wrappato in un container Docker. In questo modo è possibile utilizzarlo scaricando unicamente la sua immagine con Docker. L'immagine del tool è caricata sul registry di Commigo al [seguente link](https://github.com/orgs/Commigo/packages/container/package/openapi-generator) e il path dell'immagine è `ghcr.io/commigo/openapi-generator:latest`.

## Generazione Progetto Bruno

Per poter usare il tool, bisogna specificare il path del file della specifica OpenAPI, il tipo di generatore da utilizzare e dove salvare l'output generato. Nel caso di Bruno, bisogna eseguire il seguente comando:

```bash
docker run --rm -v %CD%:/local ghcr.io/commigo/openapi-generator:latest generate -g bruno -i /local/ApiDocumentation/OpenApi/OpenApi/HapGreeOpenApi.json -o /local/ApiDocumentation/GeneratedBruno
```

Spiegazione del comando:
*   `docker run --rm -v %CD%:/local ghcr.io/commigo/openapi-generator:latest`: avvia un container Docker con il tool.
    *   `--rm`: elimina il container al termine del comando.
    *   `-v %CD%:/local`: permette al container di leggere i file nella directory corrente, associandoli al path `/local` all'interno del container.
*   `generate`: è il comando del tool per la generazione.
*   `-g bruno`: specifica che si vuole generare file per Bruno.
*   `-i /local/ApiDocumentation/OpenApi/OpenApi/HapGreeOpenApi.json`: specifica il file di input.
*   `-o /local/ApiDocumentation/GeneratedBruno`: indica la cartella di output.

Per rendere questo processo più facile, è stato creato lo script `Scripts\RunBrunoCodeGen.bat` che esegue il tool automaticamente.

L'output sarà una cartella composta da tutti i file `.bru` che descrivono le richieste. In generale, i file generati non andrebbero modificati o committati, poiché dipendono unicamente dalle modifiche fatte alla specifica OpenAPI e cambiano solo se la specifica cambia.

L'unico file Bruno non generato, ma che deve essere aggiunto manualmente (anche tramite l'applicazione), è quello degli **environments**. Questo file non dipende dalla specifica OpenAPI e quindi non viene modificato o cancellato quando viene eseguito il tool di generazione.

## Generazione Codice Server

Per poter generare il codice del server, bisogna fare un procedimento simile. Il comando è praticamente lo stesso, ma bisogna cambiare il tipo di generatore. Al posto di `bruno`, infatti, si passa `typescript-node-server` e un'altra cartella di output, ovvero `\GeneratedServer\generated`.

```bash
docker run --rm -v %CD%:/local ghcr.io/commigo/openapi-generator:latest generate -g typescript-node-server -i /local/ApiDocumentation/OpenApi/OpenApi/HapGreeOpenApi.json -o /local/GeneratedServer/generated
```

### Struttura del Codice Generato

È interessante spendere qualche parola sulla struttura del contenuto generato. Come anticipato, gli elementi generati sono i controller e le interfacce dei service, ma non solo: vengono generati anche i tipi utilizzati e le interfacce per poter definire degli interceptor su ogni rotta.

L'output è così strutturato:

*   **`api`**:  descritto nel capitolo [Struttura Cartelle Api](/Capitoli/StrutturaCartelleApi.md), separati in gruppi logici in base ai tag usati nella specifica.

*   **`model`**: Sono definiti tutti i tipi generati e usati nella specifica OpenAPI.

*   **`utils`**: Contiene la definizione di `ResultAndError`, così che possa essere usata nei controller.

Inoltre, viene anche generato il file `index.ts` che imposta tutte le rotte generate su Fastify e avvia il server.
