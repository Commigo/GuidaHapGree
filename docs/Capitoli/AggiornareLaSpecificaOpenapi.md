# Aggiornare la Specifica OpenAPI

Come introdotto nel capitolo [**OpenAPI**](/Capitoli/OpenApi.md), il nostro progetto utilizza la specifica OpenAPI per definire l'interfaccia delle API. Questo capitolo illustra come aggiornare tale specifica per modificare o aggiungere nuove richieste.

## Dal JSON manuale al Generatore TypeScript

Inizialmente, la specifica veniva aggiornata modificando manualmente il file `ApiDocumentation\OpenApi\OpenApi\HapGreeOpenApi.json`. Questo processo presentava diversi svantaggi:

- **Scomodità**: Essendo un file JSON, l'editor non forniva funzionalità avanzate come l'autocompletamento o la validazione in tempo reale.
- **Verbosità**: La sintassi di OpenAPI richiede la stesura di molto codice boilerplate e ripetitivo per ogni nuova operazione.

Per questi motivi, e seguendo le raccomandazioni di OpenAPI stessa, è stato sviluppato un generatore che crea la specifica a partire da file TypeScript. Questo approccio trasforma l'aggiornamento della specifica in un processo più simile alla programmazione tradizionale, offrendo numerosi vantaggi:

- **Funzioni per operazioni ripetitive**: È possibile creare utility per ridurre la duplicazione del codice.
- **Supporto dell'editor**: Si beneficia di autocompletamento, suggerimenti e controllo dei tipi.
- **Efficienza**: Richiede un lavoro significativamente minore rispetto alla modifica manuale.

La generazione si basa sulla libreria `@commigo/ts-to-openapi`, il cui codice sorgente è disponibile in [questo repository](https://github.com/Commigo/TypeScriptToOpenApi).

Questo capitolo descriverà sia la libreria sia il suo impiego pratico nel progetto. Per una comprensione più approfondita, si consiglia di analizzare i file già implementati.

## Struttura del Generatore

Il generatore interpreta file TypeScript suddivisi in tre sezioni logiche:

1.  **Model**: Descrivono la struttura dei dati utilizzata nelle richieste e nelle risposte.
2.  **Path**: Contengono una route e le relative operazioni (GET, POST, DELETE, etc.).
3.  **Configurazione**: Un file principale che definisce le proprietà generali del documento (titolo, versione, server, etc.).

Ogni file esporta un oggetto (o una lista di oggetti) tramite l'istruzione [`export default`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export). Il generatore legge questi export per costruire la specifica OpenAPI in formato JSON.

## Definizione dei Componenti

Vediamo nel dettaglio come definire i `Model`, i `Path` e il file di configurazione.

### 1. Definire i `Model`

Per la definizione dei `Model` si utilizza la libreria [**Zod**](https://zod.dev/), che permette di creare schemi per la validazione di strutture dati a runtime e di convertirli in JSON Schema. È consigliata la lettura della documentazione ufficiale di Zod.

Un `Model` viene creato con la funzione `NewModel`, che accetta i seguenti parametri:
- Uno schema Zod.
- Un nome per il modello.
- Una serie di opzioni, tra cui un valore di esempio e l'eventuale modello padre da estendere.

**Esempio:**

```typescript
import { NewModel } from "@commigo/ts-to-openapi";
import z from "zod";

export const FoodSchema = NewModel(z.object({
    id: z.number().int(),
    ean: z.string().nullable(),
    name: z.string(),
    category: z.string().optional(),
    co2KgForProductKg: z.number(),
    emissionCategory: z.enum(["a", "b", "c", "d", "e"]),
    quantity: z.number(),
    imageUrl: z.string().url().nullable()
}), "Food", {
    example: {
        id: 234,
        ean: "3017620425035",
        quantity: 10,
        name: "Nutella",
        category: "Spalmabili",
        co2KgForProductKg: 5.3,
        emissionCategory: "d",
        imageUrl: "https://images.openfoodfacts.org/images/products/301/762/042/1006/front_fr.330.full.jpg"
    }
});

export default FoodSchema;
```
Come si nota, il modello creato con `NewModel` viene esportato tramite `export default`.

### 2. Definire i `Path`

Un oggetto di tipo `Path` descrive una route e le operazioni HTTP associate.

**Esempio:**

```typescript
const Foods: Path = {
    path: "/food/foods",
    get: {
        description: "Ottiene una lista di cibi filtrati per nome",
        operationId: "FoodsGet",
        security: Security.BearerToken,
        response: FoodPaginatedResponse,
        queryParams: [
            parameter({
                model: z.string(),
                name: "name",
                required: true,
                description: "Il nome dei prodotti da ricercare",
                example: "Nutella"
            }),
            parameter({
                model: z.number().int(),
                name: "page",
                required: false,
                description: "La pagina dei risultati",
                example: 1
            })
        ],
        pathParams: []
    },
    post: {
        body: null,
        description: "Permette di creare e ottenere un cibo data una sua descrizione",
        operationId: "CreateFoodByDescription",
        security: Security.BearerToken,
        response: FoodSchema,
        queryParams: [
            parameter({
                model: z.string(),
                name: "description",
                required: true,
                description: "La descrizione del cibo",
                example: "Barilla pasta 500g"
            })
        ],
        pathParams: []
    }
};

export default Foods;
```

Ogni operazione (`get`, `post`, `put`, `delete`) ha una struttura simile:
- `description`: Descrive lo scopo dell'operazione.
- `operationId`: Un identificatore univoco, usato spesso come nome del metodo nei client generati.
- `security`: Il tipo di autenticazione richiesta (es: `Security.BearerToken`).
- `response`: Il `Model` che descrive la struttura della risposta.
- `queryParams` / `pathParams`: Una lista di parametri definiti con la funzione `parameter`, specificando tipo, nome, obbligatorietà e descrizione.
- `body`: Descrive il corpo della richiesta (assente in `GET`). Richiede il `contentType` (es: JSON) e il `Model` dei dati.

### 3. File di Configurazione

Il file di configurazione esporta un oggetto `OpenApiDocument` con le informazioni generali della specifica.

**Esempio:**

```typescript
import { OpenApiDocument } from "@commigo/ts-to-openapi";

const hapGreeDocument: OpenApiDocument = {
    title: "HapGreeServer",
    description: "Il Server delle API di HapGree",
    version: "1.0.0",
    servers: [{
        url: "http://localhost:3000/api"
    }]
};

export default hapGreeDocument;
```


### Esportazione dei `Model`
I `Model` non devono essere necessariamente esportati in file separati. Se un `Model` è utilizzato nella definizione di un `Path` esportato (ad esempio, per il `body` o la `response`), verrà incluso automaticamente nella specifica.

Questo è utile per modelli specifici di una singola operazione.

**Esempio:**

```typescript
const CreateGroupSchema = NewModel(z.object({
    name: z.string(),
    description: z.string(),
    isPrivate: z.boolean(),
    userIds: z.number().int().array()
}), "CreateGroup", {
    example: {
        name: "Nuovo Gruppo",
        description: "È un nuovo gruppo di prova",
        isPrivate: false,
        userIds: [1, 2, 3, 4]
    }
});

const Group: Path = {
    path: "/group",
    post: {
        body: {
            contentType: BodyContentType.JSON,
            content: CreateGroupSchema
        },
        description: "Crea un nuovo gruppo",
        operationId: "CreateGroup",
        security: Security.BearerToken,
        response: GroupSchema,
        queryParams: [],
        pathParams: []
    }
};

export default Group;
```
In questo caso, `CreateGroupSchema` viene incluso nella specifica perché è usato nel `Path` `Group`, che è esportato.

### Gestione degli Export
- **File non esportati**: Gli oggetti non esportati con `export default` vengono ignorati. Il generatore emette un avviso se rileva un file che dovrebbe esportare una definizione ma non lo fa.
- **Export nullo**: Se un file non deve contribuire alla specifica, è possibile aggiungere `export default null;` per evitare avvisi.

### Estensione dei Modelli
Per creare un modello che estende un altro, si usa l'opzione `parent`.

**Esempio:**

```typescript
export const PublicUserSchema = NewModel(z.object({
    id: z.number().int(),
    firstName: z.string(),
    lastName: z.string(),
    avatarUrl: z.string().url().nullable(),
}), "PublicUser", {
    example: {
        id: 0,
        firstName: "Public",
        lastName: "User",
        avatarUrl: ""
    }
});

export const InvitablePublicUserSchema = NewModel(z.object({
    isAlreadyInvited: z.boolean(),
}), "InvitablePublicUser", {
    parent: {
        model: PublicUserSchema,
    },
    example: {
        isAlreadyInvited: true,
    }
});
```
Nell'esempio, `InvitablePublicUserSchema` eredita tutti i campi di `PublicUserSchema` e aggiunge `isAlreadyInvited`.

### Paginazione
Per i modelli che rappresentano risposte paginate, si utilizza la funzione helper `NewPaginationModel`. Questo permette al generatore di codice client (es. Flutter) di creare modelli che estendono una classe base di paginazione.

**Esempio:**

```typescript
const PaginatedPublicGroupsResponse = NewPaginationModel(
    GroupActiveInformationSchema,
    "PaginatedPublicGroupsResponse", {
        elements: [
            GroupActiveInformationSchema.meta()?.example,
            GroupActiveInformationSchema.meta()?.example,
        ]
    }
);
```
La funzione prende come input il modello da paginare, il nome del nuovo modello paginato e un esempio di risposta. In questo modo, nella specifica OpenAPI verranno aggiunti tag speciali che i generatori di codice possono interpretare per gestire correttamente la paginazione.
