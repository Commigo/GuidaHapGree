# Architettura Controller-Service-Model

Il nostro server implementa le API REST secondo una struttura a tre livelli formata da **Controller-Service-Model**.

- I **Controller** si occupano di astrarre il mezzo con cui è stata fatta la richiesta al server.
- I **Service** implementano la logica di business specifica dell'app.
- I **Model** si occupano di astrarre la comunicazione con servizi esterni o particolari, come database, notifiche di Firebase, o altre chiamate API.
- In alcuni casi, prima di raggiungere il controller, la richiesta viene intercettata da un *interceptor*. Questo componente esegue controlli trasversali, ovvero validazioni generiche e riutilizzabili che non dipendono dalla singola operazione.


## Controller

I Controller, in questo caso specifico, astraggono il protocollo HTTP estraendo dall'oggetto `request` di Fastify tutti i parametri presenti nella query, nel body e nel path, oltre all'autorizzazione.

- Verificano che siano presenti tutti i valori necessari e che siano del formato giusto.
- Se i controlli sono corretti, passano i valori al Service corrispondente.
- Prendono il risultato del Service e lo impacchettano utilizzando l'oggetto `response` di Fastify per creare la risposta HTTP.

## Service

I Service ricevono i parametri senza sapere che sono originariamente arrivati come richiesta HTTP (quindi sono potenzialmente riutilizzabili in altri contesti). Si occupano di implementare la logica specifica dell'applicazione, delegando il lavoro ai Model quando devono interrogare il database o eseguire altre operazioni.

## Model

I Model si occupano di implementare richieste verso servizi esterni o particolari, astraendone il funzionamento. Ad esempio, eseguono un'interrogazione o un aggiornamento del database.

---

## Esempio Concreto

Vediamo un esempio con l'API per creare un invito a un gruppo: `/group/:id/invitations?idUtente=1`.

Questa API ha due parametri:
- Uno di path: `:id`
- Uno di query: `idUtente`

### Interceptor
la richiesta viene intercettata dall'interceptor authMiddleware che verifica che il token di autorizzazione sia presente e valido.


### Controller

Il controller `makeGroupInvitationById` si occupa di estrarre e validare tutti i parametri dalla richiesta HTTP.

#### Estrazione dei parametri

Innanzitutto, estrae i parametri dal path e dalla query. Utilizza la libreria `zod` per validare che siano del tipo corretto (in questo caso, numeri). Eventuali errori vengono accumulati in un array `errors`.

```typescript
// All'interno della funzione del controller...
let errors: string[] = [];

// Estrazione e validazione parametro di query 'idUtente'
const idUtenteRaw = (request.query as any).idUtente;
const idUtenteSchema = z.coerce.number();
const idUtenteParsingResult = idUtenteSchema.safeParse(idUtenteRaw);
if (!idUtenteParsingResult.success) {
    errors.push("Il parametro di query 'idUtente' ha un problema: " + JSON.parse(idUtenteParsingResult.error.message)[0].message);
}
const idUtente = idUtenteParsingResult.data!;

// Estrazione e validazione parametro di path 'id'
const idRaw = (request.params as any).id;
const idSchema = z.coerce.number();
const idParsingResult = idSchema.safeParse(idRaw);
if (!idParsingResult.success) {
    errors.push("Il parametro di path 'id' ha un problema: " + JSON.parse(idParsingResult.error.message)[0].message);
}
const id = idParsingResult.data!;
```

#### Estrazione dell'autorizzazione

Successivamente, estrae e valida il token di autorizzazione dall'header `Authorization`. Verifica che il token sia presente e nel formato "Bearer".

```typescript
// Estrazione e validazione del token di autorizzazione
let BeaererToken: BearerAuthToken | undefined;
if (request.headers.authorization && request.headers.authorization.startsWith("Bearer ")) {
    const BeaererTokenRaw = request.headers.authorization.substring(7);
    const BeaererTokenParsingResult = BearerAuthTokenSchema.safeParse(BeaererTokenRaw);
    if (!BeaererTokenParsingResult.success) {
        errors.push("Il token di autorizzazione ha un formato sbagliato: " + JSON.parse(BeaererTokenParsingResult.error.message)[0].message);
    }
    BeaererToken = BeaererTokenParsingResult.data!;
} else {
    errors.push("Token di autorizzazione mancante");
}
```

#### Chiamata al Service e creazione della risposta

Infine, se non ci sono stati errori di validazione, il controller invoca il metodo del `Service` corrispondente, passando i dati "puliti". In base al risultato del service, costruisce e invia la risposta HTTP, gestendo sia i casi di successo che di errore.

```typescript
// Se ci sono errori, non procedere
if (errors.length > 0) {
    replay.status(400).send({ errors });
    return;
}

// Passa i parametri al service e crea la risposta HTTP
const [result, error] = await GruppiService.makeGroupInvitationById(idUtente, id, BeaererToken!);
if (error !== null) {
    replay.status(500).send({ errors: [error] });
    return;
}

replay.status(204).send(result);
return;
```

### Service

Il service `makeGroupInvitationById` si occupa della logica di business. Verifica che chi sta creando l'invito sia il proprietario del gruppo e che chi lo riceve non ne faccia già parte.

Per l'effettiva verifica che richiede l'interrogazione del database, il Service si affida ai **Model**.

```typescript
async function makeGroupInvitationById(idUtente: number, id: number, BeaererToken: BearerAuthToken): Promise<ResultAndError<void>> {
    // Verifica che l'utente che fa la richiesta sia il proprietario del gruppo
    const isOwner = await groupModel.isUserGroupOwner(userInfo.userId, id, dbKysely);
    if (!isOwner) {
        return [null, "Non sei l'admin del gruppo"];
    }

    // Verifica che l'utente invitato non sia già nel gruppo
    const isInGroup = await groupModel.isUserInGroup(idUtente, id, dbKysely);
    if (isInGroup) {
        return [null, "L'utente è già parte del gruppo"];
    }
    
    // ...
}
```
Successivamente, crea l'invito e invia una notifica push alla persona invitata, sempre usando i Model.

```typescript
// Creazione dell'invito nel database
await groupModel.makeGroupInvite(id, idUtente, transaction.transaction);
await transaction.commit();

// Invio notifica push con Firebase
await firebaseMessagingModel.sendMessageToUser(idUtente, {
    notification: {
        title: "Invito a gruppo",
        body: "Hai ricevuto un invito al gruppo",
    }
}, dbKysely);
```

Come si vede da questo esempio, il Service si occupa solo dei controlli di business e delega l'effettiva implementazione ai Model.
