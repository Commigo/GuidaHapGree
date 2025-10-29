# Generare la Specifica OpenAPI

Nel capitolo precedente abbiamo visto come scrivere il codice che verrà poi utilizzato per generare la specifica OpenAPI. Adesso vedremo come avviare il processo di generazione e come i file necessari sono strutturati all'interno del progetto.

## Struttura del Progetto

Il progetto che contiene i file per la generazione si trova nella cartella del submodule Git: `ApiDocumentation\OpenApi\OpenApi`.

Questo è un progetto TypeScript in Node.js e, come descritto nei capitoli [**NodeJs**](./NodeJs.md) e [**TypeScript**](./TypeScript.md), contiene i propri file `package.json` e `tsconfig.json`.

### Dipendenze

Fra le varie dipendenze del progetto è presente `@commigo/ts-to-openapi`, un package privato dell'organizzazione **Commigo**. Di conseguenza, per poterlo scaricare eseguendo `npm i`, è necessario prima effettuare il login a `npm` con delle credenziali che permettano l'accesso ai package dell'organizzazione.

Per maggiori dettagli, seguire questa [guida ufficiale](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry).

### Struttura dei Sorgenti

I file sorgenti si trovano nella cartella `ApiDocumentation\OpenApi\OpenApi\src` e sono organizzati come segue:

-   `ApiDocumentation\OpenApi\OpenApi\src\HapGreeServer.ts`: Esporta l'oggetto di configurazione principale.
-   `ApiDocumentation\OpenApi\OpenApi\src\Models`: Contiene i file che esportano gli oggetti *Model*.
-   `ApiDocumentation\OpenApi\OpenApi\src\Paths`: Contiene una serie di sottocartelle, ognuna rappresentante un gruppo logico di funzionalità (es. `Food`, `Mobility`, `Auth`). Ogni sottocartella contiene i file che esportano i *path* per quella specifica funzionalità.

Questa suddivisione è fondamentale, poiché il generatore si basa su di essa per raggruppare le API.

## File di Configurazione

Il generatore utilizza un file di configurazione, `ApiDocumentation\OpenApi\OpenApi\configuration.json`, strutturato in questo modo:

```json
{
    "documentFile": "./src/HapGreeServer.ts",
    "modelsDirectory": "./src/Models",
    "pathsDirectory": "./src/Paths"
}
```

-   `documentFile`: Contiene il percorso al file che esporta l'oggetto di configurazione.
-   `modelsDirectory`: Contiene il percorso alla cartella che contiene i file dei *Model*.
-   `pathsDirectory`: Contiene il percorso alla cartella che raggruppa le funzionalità.

> **Nota**: Il generatore leggerà le esportazioni **solo** dai percorsi specificati in questo file.

### Raggruppamento tramite Tag

I *path* presenti all'interno della stessa sottocartella (es. `Food`) vengono generati con un `tag` OpenAPI che corrisponde al nome della cartella stessa. Questo, come spiegato nel capitolo [**OpenApi**](./OpenApi.md), determina un raggruppamento logico sia nella specifica che nel codice client generato.

## Avvio della Generazione

Per avviare la generazione della specifica, si utilizza un tool tramite Docker, in modo simile a quanto fatto per la generazione del codice. Il tool è disponibile tramite l'immagine Docker dei package di Commigo: `ghcr.io/commigo/typescript-to-openapi-generator:latest`.

Il comando da eseguire è il seguente:

```bash
docker run --rm -v %CD%:/local ghcr.io/commigo/typescript-to-openapi-generator:latest /local/configuration.json /local/HapGreeOpenApi.json
```

Questo comando prende come parametri il file di configurazione (`configuration.json`) e il percorso del file di output (`HapGreeOpenApi.json`).

### Script di Automazione

Per semplificare il processo, è stato creato lo script `ApiDocumentation\OpenApi\OpenApi\Scripts\RunGenerator.bat`, che esegue automaticamente il comando con i parametri corretti.

> **Importante**: Prima di eseguire lo script, assicurarsi di trovarsi nella directory `ApiDocumentation\OpenApi\OpenApi`.
