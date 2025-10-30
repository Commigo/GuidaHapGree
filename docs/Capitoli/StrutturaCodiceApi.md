# Struttura del Codice API

In questo capitolo viene esplorata l'organizzazione dei file e del codice all'interno del progetto, con un focus sull'implementazione di routes, controller, service, model e le relative interfacce.

## Organizzazione Generale

Il codice è strutturato in **gruppi logici** basati sulle funzionalità, come ad esempio `Challenge`, `Auth`, e `Mobility`. Ogni gruppo contiene i file necessari per gestire le proprie specifiche richieste:

-   Il codice che implementa le **route** di un gruppo si trova in un unico file.
-   Lo stesso principio si applica a **controller**, **service** e **interceptor**, ciascuno raggruppato nel proprio file.

Per ogni richiesta appartenente a un gruppo, esiste una funzione dedicata nei rispettivi file del controller, del service e dell'interceptor.

## Flusso di una Richiesta

Partendo dalle route, il meccanismo segue questo schema:

1.  **Route**: Per ogni gruppo, un file di routing definisce le rotte Fastify. A ciascuna rotta sono associati un interceptor e un controller specifici per quell'operazione.
2.  **Interceptor**: Un file per gruppo contiene un oggetto (`NomeGruppoInterceptors`) con metodi che fungono da interceptor per ciascuna richiesta.
3.  **Controller**: Un file per gruppo contiene un oggetto (`NomeGruppoControllers`) con metodi dedicati a ciascuna richiesta. Il compito del controller è estrarre i parametri dalla richiesta e invocarli il metodo corrispondente nel service.
4.  **Service**: Un file per gruppo contiene un oggetto (`NomeGruppoService`) che implementa la logica di business specifica per ogni richiesta.

## Ruolo delle Interfacce

Per ogni gruppo, sono presenti anche due file di interfacce:

-   **Interfaccia dei Service**: Dichiara le firme dei metodi per ogni richiesta. L'oggetto di implementazione del service (`NomeGruppoService`) implementa questa interfaccia. Ciò garantisce che l'implementazione sia completa e rispetti i parametri definiti.
-   **Interfaccia degli Interceptor**: Similmente, definisce le firme per ogni interceptor, che vengono poi implementate dall'oggetto corrispondente.

### Vantaggio delle Interfacce Generate Automaticamente

A prima vista, queste interfacce potrebbero sembrare ridondanti. Tuttavia, il loro ruolo è cruciale: i file delle interfacce vengono **generati automaticamente a partire dalla specifica OpenAPI**, a differenza dei file di implementazione.

Questo collegamento assicura che qualsiasi modifica alla specifica OpenAPI si rifletta automaticamente nelle interfacce, segnalando immediatamente ai file di implementazione la necessità di un aggiornamento.

## Esempio Pratico: Gruppo "Food"

Di seguito è riportato un esempio concreto basato sul gruppo `Food`.

### File delle Routes

```typescript
export async function FoodRoutes(fastify: FastifyInstance) {
    fastify.post('/food/foods', {preHandler: FoodInterceptors.createFoodByDescription }, FoodControllers.createFoodByDescription);
    fastify.post('/food/receipt', {preHandler: FoodInterceptors.createReceipt }, FoodControllers.createReceipt);
    fastify.delete('/food/receipt/:id', {preHandler: FoodInterceptors.deleteFoodReceipt }, FoodControllers.deleteFoodReceipt);
    fastify.get('/food/:ean', {preHandler: FoodInterceptors.foodEanGet }, FoodControllers.foodEanGet);
    fastify.get('/food/foods', {preHandler: FoodInterceptors.foodsGet }, FoodControllers.foodsGet);
    fastify.get('/food/contributi', {preHandler: FoodInterceptors.getFoodContributi }, FoodControllers.getFoodContributi);
    fastify.get('/food/dashboard', {preHandler: FoodInterceptors.getFoodDashboard }, FoodControllers.getFoodDashboard);
    fastify.get('/food/recentActivities', {preHandler: FoodInterceptors.getFoodRecentActivities }, FoodControllers.getFoodRecentActivities);
    fastify.get('/food/stats', {preHandler: FoodInterceptors.getFoodStats }, FoodControllers.getFoodStats);
    fastify.get('/food/receipt/:id', {preHandler: FoodInterceptors.getReceiptById }, FoodControllers.getReceiptById);
}
```

### Implementazione dei Controller

L'oggetto `FoodControllers` contiene i metodi per gestire le richieste.


```typescript
export const FoodControllers = {

    async createFoodByDescription(request: FastifyRequest, replay: FastifyReply): Promise<void>
    {
        try {
            let errors : string[] = [];
            //query
            const descriptionRaw = (request.query as any).description;
            const descriptionSchema = z.coerce.string();
            const descriptionParsingResult = descriptionSchema.safeParse(descriptionRaw);
            if(!descriptionParsingResult.success)
            {
                errors.push("il parametro di query description passato ha un problema: " + JSON.parse(descriptionParsingResult.error.message)[0].message);
            }
            const description = descriptionParsingResult.data!
            //path
            //body
            let BeaererToken : BearerAuthToken | undefined;
            if(request.headers.authorization && request.headers.authorization.startsWith("Bearer "))
            {
                const BeaererTokenRaw = request.headers.authorization.substring(7);
                const BeaererTokenParsingResult = BearerAuthTokenSchema.safeParse(BeaererTokenRaw);
                if(!BeaererTokenParsingResult.success)
                {
                    errors.push("token di autorizzazione ha il formato sbagliato: " + JSON.parse(BeaererTokenParsingResult.error.message)[0].message);
                }
                BeaererToken = BeaererTokenParsingResult.data!;
            }
            else {
                errors.push("token di autorizzazione mancante");
            }

            if(errors.length !== 0)
            {

                replay.status(400).send({errors : errors });
                return;
            }
            const [result, error] = await FoodService.createFoodByDescription(description,BeaererToken!,);
            if (error !== null) {
                replay.status(500).send({errors : [error]})
                return;
            }
            replay.status(200).send(result);
            return;


            } catch (e) {
                replay.status(500).send({errors : ["Errore inaspettato " + e]})
                return;
            }


    },


    async createReceipt(request: FastifyRequest, replay: FastifyReply): Promise<void>
    {
        try {
            let errors : string[] = [];
            //query
            //path
            //body

             const BodyRequestParameterRaw : any = request.body;
             const BodyRequestParameterSchema = CreateFoodReceiptBodyUsable;
             const BodyRequestParameterParsingResult = BodyRequestParameterSchema.safeParse(BodyRequestParameterRaw);
             if(!BodyRequestParameterParsingResult.success)
             {
                errors.push("il body della richiesta ha un problema: " + JSON.parse(BodyRequestParameterParsingResult.error.message)[0].message);
             }
             const BodyRequestParameter = BodyRequestParameterParsingResult.data!;



            let BeaererToken : BearerAuthToken | undefined;
            if(request.headers.authorization && request.headers.authorization.startsWith("Bearer "))
            {
                const BeaererTokenRaw = request.headers.authorization.substring(7);
                const BeaererTokenParsingResult = BearerAuthTokenSchema.safeParse(BeaererTokenRaw);
                if(!BeaererTokenParsingResult.success)
                {
                    errors.push("token di autorizzazione ha il formato sbagliato: " + JSON.parse(BeaererTokenParsingResult.error.message)[0].message);
                }
                BeaererToken = BeaererTokenParsingResult.data!;
            }
            else {
                errors.push("token di autorizzazione mancante");
            }

            if(errors.length !== 0)
            {

                replay.status(400).send({errors : errors });
                return;
            }

            const [result, error] = await FoodService.createReceipt(BodyRequestParameter,BeaererToken!,);
            if (error !== null) {
                replay.status(500).send({errors : [error]})
                return;
            }
            replay.status(200).send(result);
            return;


            } catch (e) {
                replay.status(500).send({errors : ["Errore inaspettato " + e]})
                return;
            }


    },

    .......................

}
```

### Interfaccia dei Service

Definisce il contratto che l'implementazione del service deve rispettare.

```typescript
export interface FoodServicesInterface{
    createFoodByDescription : (description : string,BeaererToken : BearerAuthToken,) => Promise<ResultAndError<Food>>
    createReceipt : (BodyRequestParameter  : CreateFoodReceiptBody,BeaererToken : BearerAuthToken,) => Promise<ResultAndError<ManualReceipt>>
    deleteFoodReceipt : (id : number,BeaererToken : BearerAuthToken,) => Promise<ResultAndError<void>>
    foodEanGet : (ean : string,BeaererToken : BearerAuthToken,) => Promise<ResultAndError<Food>>
    foodsGet : (name : string,page : number | undefined,BeaererToken : BearerAuthToken,) => Promise<ResultAndError<FoodPaginatedResponse>>
    getFoodContributi : (BeaererToken : BearerAuthToken,) => Promise<ResultAndError<FoodContributes[]>>
    getFoodDashboard : (BeaererToken : BearerAuthToken,) => Promise<ResultAndError<FoodDashboardResponse>>
    getFoodRecentActivities : (page : number | undefined,BeaererToken : BearerAuthToken,) => Promise<ResultAndError<FoodRecentActivitiesPaginatedResponse>>
    getFoodStats : (BeaererToken : BearerAuthToken,) => Promise<ResultAndError<Stats>>
    getReceiptById : (id : number,BeaererToken : BearerAuthToken,) => Promise<ResultAndError<ManualReceipt>>
}
```

### Implementazione dei Service

L'oggetto `FoodService` contiene la logica di business. L'uso di `satisfies FoodServicesInterface` garantisce che tutti i metodi dell'interfaccia siano implementati.


```typescript
export const FoodService = {
    createReceipt: async function (BodyRequestParameter: CreateFoodReceiptBody, BeaererToken: BearerAuthToken): Promise<ResultAndError<ManualReceipt>> {
        const tokenInf = GetValuesFromToken(BeaererToken);
        const connection = await startTransactionKyseley();

        try {
            const receiptId = await receiptModel.createReceipt({
                fkUser: tokenInf.userId,
                type: "food",
                processed: 1,
            }, connection.transaction);
            if (receiptId === undefined) {
                return [null, "Errore durante la creazione della receipt"];
            }

            const foodEmissions: Food[] = [];

            // Processa ogni alimento nell'array
            for (const obj of BodyRequestParameter.foods) {

                // Ottieni i dati del prodotto dall'API esterna, passando il token
                const [result, error] = await otherServerFoodModel.getFoodByEan(tokenInf.apiToken, obj.ean);
                if (error !== null) {
                    return [null, `Prodotto con EAN ${obj.ean} non trovato`];
                }

                // Prendi il primo risultato
                const product = result;

                // Determina la categoria in base al product_category
                const categoryId = mapExternalToInternalCategory(product.category);

                const co2Emitted = result.quantity * product.co2KgForProductKg;
                var emissionId = product.id || 0;

                const foodId = await foodEmissionModel.insert({
                    category: product.emissionCategory,
                    fkReceipt: Number.parseInt(receiptId.toString()),
                    fkCategory: categoryId,
                    fkUser: tokenInf.userId,
                    emissionId: emissionId,
                    name: product.name,
                    quantityGrams: result.quantity * 1000,
                    cO2Emitted: co2Emitted,
                    foodEan: obj.ean
                }, connection.transaction);

                if (foodId === undefined) {
                    return [null, "Errore durante l'inserimento del foodEmission"];
                }

                foodEmissions.push(product);
            }

            // Commit della transazione
            await connection.commit();
            const co2Emitted = await foodEmissionModel.getTotalCO2ForReceipt(Number(receiptId), dbKysely);
            return [{
                cO2Emitted: co2Emitted,
                id: Number(receiptId),
                foods: foodEmissions,
                type: "manual",
                createdAt: (new Date()).toISOString(),
                elementsNumber: foodEmissions.length
            }, null];
        } catch (error) {
            // Rollback in caso di errore
            console.error('Errore nella transazione di inserimento alimenti:', error);
            return [null, "Errore inaspettato " + error];
        }
        finally {
            await connection.release();
        }
    },
    deleteFoodReceipt: async function (id: number, BeaererToken: BearerAuthToken): Promise<ResultAndError<void>> {
        const tokenInf = GetValuesFromToken(BeaererToken);
        try {
            const isReceiptOfUser = await foodEmissionModel.isReceiptOfUser(id, tokenInf.userId, dbKysely);
            if (!isReceiptOfUser) {
                return [null, "Non puoi eliminare questa receipt"];
            }
            await foodEmissionModel.deleteReceipt(id, dbKysely);
            return [undefined, null];
        } catch (error) {
            console.error('❌ Errore nella cancellazione del receipt:', error);
            return [null, `Errore ${error}`];
        }
    },

    .............
} satisfies FoodServicesInterface;
```

### Interfaccia degli Interceptor

```typescript
export interface FoodInterceptorsInterface {
    createFoodByDescription : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    createReceipt : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    deleteFoodReceipt : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    foodEanGet : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    foodsGet : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    getFoodContributi : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    getFoodDashboard : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    getFoodRecentActivities : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    getFoodStats : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
    getReceiptById : (request : FastifyRequest,replay : FastifyReply) => Promise<void>
}
```

### Implementazione degli Interceptor

L'oggetto `FoodInterceptors` implementa l'interfaccia `FoodInterceptorsInterface` e associa a ogni operazione l'interceptor desiderato.

```typescript
export const FoodInterceptors  = {
    createReceipt: AuthMiddleware([1, 2, 3]),
    deleteFoodReceipt: AuthMiddleware([1, 2, 3]),
    foodEanGet: AuthMiddleware([1, 2, 3]),
    foodsGet: AuthMiddleware([1, 2, 3]),
    getFoodDashboard: AuthMiddleware([1, 2, 3]),
    getFoodStats: AuthMiddleware([1, 2, 3]),
    getReceiptById: AuthMiddleware([1, 2, 3]),
    createFoodByDescription: AuthMiddleware([1, 2, 3]),
    getFoodRecentActivities: AuthMiddleware([1, 2, 3]),
    getFoodContributi: AuthMiddleware([1, 2, 3]),

} satisfies FoodInterceptorsInterface;
```

> **Nota**: Come si può osservare, in questo caso viene utilizzato lo stesso interceptor `AuthMiddleware([1, 2, 3])` per tutte le richieste.

## Gestione degli Errori con `ResultAndError<T>`

Come si può notare nelle firme dei metodi dei `Service`, viene utilizzato il tipo `ResultAndError<T>`. Questo è un tipo generico personalizzato introdotto per migliorare la gestione degli errori nel progetto.

### Motivazione

In precedenza, la gestione degli errori si basava sull'uso di eccezioni (`try-catch`), un approccio che può rendere il flusso di controllo meno trasparente. Il tipo `ResultAndError<T>` introduce un pattern di ritorno esplicito, dove ogni funzione comunica chiaramente la possibilità di un fallimento.

### Struttura del Tipo

`ResultAndError<T>` è definito come una **tupla** che può rappresentare due stati:

-   **Successo**: Una tupla della forma `[T, null]`, dove il primo elemento è il valore di ritorno atteso (di tipo `T`) e il secondo è `null`.
-   **Errore**: Una tupla della forma `[null, string]`, dove il primo elemento è `null` e il secondo è una stringa che descrive l'errore.

### Utilizzo Pratico

Questo approccio semplifica la gestione del risultato di una funzione.

#### "Spacchettare" il Risultato

È possibile destrutturare la tupla di ritorno per accedere sia al risultato che all'eventuale errore, rendendo il codice più leggibile:

```typescript
const [result, error] = await FoodService.someMethod(...);
```

#### Verificare la Presenza di Errori

Il controllo dell'errore diventa semplice e lineare, senza la necessità di un blocco `try-catch` per la logica di business.

```typescript
if (error !== null) {
    // Gestisci l'errore (es. log, invio di una risposta HTTP di errore)
    console.error(error);
    replay.status(500).send({ errors: [error] });
    return;
}
// Se non ci sono errori, 'result' contiene il valore atteso e può essere usato in sicurezza.
replay.status(200).send(result);
```

#### Esempi di Ritorno da un Service

##### Successo

Per restituire un valore di successo da una funzione che ritorna `ResultAndError<number>`:

```typescript
return [5, null];
```

##### Errore

Per restituire un errore:

```typescript
return [null, "Si è verificato un errore perché..."];
```
