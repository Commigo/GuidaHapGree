# I Models

Nei capitoli precedenti, abbiamo visto come i `Service` delegano l'effettivo lavoro a dei componenti chiamati **Models**.

In questo progetto, i models sono di vario tipo ma possono essere raggruppati in alcune categorie principali:
- Models che si occupano di interagire con il **database**.
- Models che contattano un **altro server API** gestito da un team diverso.
- Models che si interfacciano con **API esterne** (es. PayPal).
- Altri models più specifici, come quello per inviare notifiche con **Firebase**.

I models sono situati principalmente nella cartella `GeneratedServer/src/models`.

Vediamo un esempio per ogni categoria.

---

## 1. Models per il Database

Questi models gestiscono tutte le interrogazioni al database. Utilizzano principalmente due strumenti: **Kysely** e **Prisma**.

### Esempio con Kysely

Un esempio si trova nel file `GeneratedServer/src/models/group.ts`, che gestisce i gruppi. L'oggetto `groupModel` contiene metodi per interrogare il database.

Ecco la funzione `getPublicGroups` che usa **Kysely** per leggere i gruppi pubblici:

```typescript
async getPublicGroups(nameFilter = '', limit = 10, offset = 0, db: Kysely<DB>) {
    const rows = await db.selectFrom("GroupGreen as g")
        .innerJoin("User as u", "u.id", "g.fkOwner")
        .where("g.isPrivate", "=", 0)
        .where("g.name", "like", `%${nameFilter}%`)
        .orderBy("g.createdAt", "desc")
        .limit(limit)
        .offset(offset)
        .select([
            "g.id", "g.fkOwner", "g.name", "g.description", "g.tag",
            "g.isPrivate", "g.createdAt", "u.firstName", "u.lastName", "u.avatarUrl"
        ])
        .execute();

    const groups = [];
    for (let group of rows) {
        const processedGroup = {
            id: group.id,
            name: group.name,
            description: group.description,
            tag: group.tag,
            isPrivate: group.isPrivate ? true : false,
            createdAt: group.createdAt,
            owner: {
                id: group.fkOwner,
                avatarUrl: group.avatarUrl,
                firstName: group.firstName,
                lastName: group.lastName
            },
            ownerId: group.fkOwner,
            totalMembers: await this.countMembers(group.id, db),
            sampleMembers: await this.getSampleMembers(group.id, db)
        };
        groups.push(processedGroup);
    }

    return groups;
};
```
> **Nota:** È importante notare come venga passato l'oggetto `db` di Kysely, che può rappresentare sia la connessione globale al database sia una transazione specifica.

### Esempio con Prisma

Nello stesso model, troviamo un metodo che utilizza **Prisma** per leggere il numero di notifiche (richieste e inviti) relative ai gruppi.

```typescript
async getInvitationsAndRequestsNumber(userId: number, db: PrismaDB) {
    const numOfInvitations = await db.groupInvitation.count({
        where: {
            userId,
        }
    });

    const numOfRequests = await db.groupRequest.count({
        where: {
            GroupGreen: {
                fkOwner: userId
            }
        }
    });

    return numOfInvitations + numOfRequests;
}
```
> **Nota:** Anche in questo caso, l'oggetto `db` di Prisma può essere l'istanza globale o una transazione.

---

## 2. Models per API Interne

Questi models gestiscono la comunicazione con l'altro server API del progetto. Si trovano nella cartella `GeneratedServer/src/models/OtherHapGreeServer`.

Utilizzano un client HTTP basato su **Axios**, definito nel file `GeneratedServer/src/models/OtherHapGreeServer/OtherServerBaseHttp.ts`. Vengono creati due client:
- `otherServerBaseHttp`: per richieste con body in formato JSON.
- `otherServerBaseHttpFromData`: per richieste `multipart/form-data`.

L'URL di base viene impostato dinamicamente tramite variabili d'ambiente (`OTHER_API_SERVER_RELEASE` o `OTHER_API_SERVER_TEST`).

```typescript
export const otherServerBaseHttp = axios.create({
    baseURL: IsProduction() ? process.env.OTHER_API_SERVER_RELEASE : process.env.OTHER_API_SERVER_TEST,
    headers: { 'Content-Type': 'application/json' },
    timeout: 10000,
});

export const otherServerBaseHttpFromData = axios.create({
    baseURL: IsProduction() ? process.env.OTHER_API_SERVER_RELEASE : process.env.OTHER_API_SERVER_TEST,
    headers: { 'Content-Type': "multipart/form-data" }
});
```

### Esempio di Chiamata API

Il file `GeneratedServer/src/models/OtherHapGreeServer/OtherServerAuthModel.ts` contiene l'oggetto `otherServerAuthModel` con i metodi per l'autenticazione.

Ecco la funzione di `login`:

```typescript
async login(email: string, pwd: string, dispositivo: string): Promise<ResultAndError<string>> {
    try {
        const response = await otherServerBaseHttp.post("login", {
            email: email,
            device_name: dispositivo,
            password: pwd
        });
        if (response.data.data.token) {
            return [response.data.data.token, null];
        } else {
            return [null, "response not understood"];
        }
    } catch (error: any) {
        if (error.response) {
            if (axios.isAxiosError<BaseServerResponse>(error)) {
                if (error.response.data.errors) {
                    return [null, JSON.stringify(error.response.data.errors)];
                }
                return [null, error.response.data.message];
            } else {
                return [null, "errore non definito"];
            }
        } else {
            return [null, "errore di rete: " + error.message];
        }
    }
}
```

Per le chiamate autenticate, viene passato un token nell'header `Authorization`, come nel metodo `logout`:
```typescript
async logout(token: string): Promise<ResultAndError<void>> {
    try {
        await otherServerBaseHttp.post("logout", {}, {
            headers: {
                Authorization: `Bearer ${token}`
            }
        });
        return [undefined, null];
    } catch (error: any) {
        if (error.response) {
            if (axios.isAxiosError<BaseServerResponse>(error)) {
                return [null, error.response.data.message];
            } else {
                return [null, "errore non definito"];
            }
        } else {
            return [null, error.message];
        }
    }
}
```

---

## 3. Models per API Esterne (PayPal)

Un esempio di model che contatta API esterne è `GeneratedServer/src/models/PaypalModel/PaypalModel.ts`. Anche questo utilizza un client **Axios** dedicato.

L'host di PayPal cambia in base all'ambiente (produzione o sandbox).
```typescript
const paypalHost = IsProduction() ? "api-m.paypal.com" : "api-m.sandbox.paypal.com";

export const toPaypalServerClient = axios.create({
    baseURL: `https://${paypalHost}/v2`,
    headers: { 'Content-Type': 'application/json' },
});
```

### Esempio di Creazione Ordine PayPal

Il metodo `createOrder` invia una richiesta per creare un nuovo ordine di pagamento.

```typescript
async createOrder(amount: number, token: string): Promise<ResultAndError<PaypalCreatedOrder>> {
    try {
        const response = await toPaypalServerClient.post("checkout/orders", {
            intent: "CAPTURE",
            payment_source: {
                paypal: {
                    experience_context: {
                        return_url: "https://api.hapgree.com/user/paypal/waitForPayment",
                        cancel_url: "https://api.hapgree.com/user/paypal/waitForPayment",
                        user_action: "PAY_NOW"
                    }
                }
            },
            purchase_units: [
                {
                    amount: {
                        currency_code: "EUR",
                        value: amount.toFixed(2)
                    }
                }
            ]
        }, {
            headers: {
                "Authorization": "Bearer " + token
            }
        });
        if (response.data.links[1].href) {
            const returnValue = {
                link: response.data.links[1].href,
                id: response.data.id,
            };
            const parseResult = PaypalCreatedOrderSchema.safeParse(returnValue);
            if (!parseResult.success) {
                return [null, parseResult.error.message];
            }
            return [{
                link: response.data.links[1].href,
                id: response.data.id,
            }, null];
        } else {
            return [null, "response not understood"];
        }
    } catch (error: any) {
        if (error.response) {
            if (axios.isAxiosError<BaseServerResponse>(error)) {
                return [null, error.response.data.message ?? "errore non definito"];
            } else {
                return [null, "errore non definito"];
            }
        }
        else {
            return [null, error.message];
        }
    }
}
```

Il token di autenticazione per le API di PayPal viene ottenuto tramite il metodo `getPaypalAccessToken`, che effettua una chiamata specifica per l'autenticazione.

```typescript
export const getPaypalAccessToken = async function (): Promise<ResultAndError<string>> {
    const client = axios.create({
        baseURL: `https://${paypalHost}/v1/oauth2/token`,
        headers: {
            'Content-Type': 'x-www-form-urlencoded',
        },
        auth: {
            username: IsProduction() ? process.env.PAYPAL_CLIENT_ID_RELEASE! : process.env.PAYPAL_CLIENT_ID_TEST!,
            password: IsProduction() ? process.env.PAYPAL_CLIENT_SECRET_RELEASE! : process.env.PAYPAL_CLIENT_SECRET_TEST!,
        },
    });
    try {
        const form = new URLSearchParams({
            grant_type: 'client_credentials',
        })
        const result = await client.post("", form);
        if (result.data.access_token) {
            return [result.data.access_token, null];
        }
        return [null, "formato della risposta non previsto"];
    }
    catch (error: any) {
        if (error.response) {
            if (axios.isAxiosError<BaseServerResponse>(error)) {
                return [null, error.response.data.message ?? "errore non definito"];
            } else {
                return [null, "errore non definito"];
            }
        }
        else {
            return [null, error.message];
        }
    }
}
```

---

## 4. Models per Notifiche (Firebase)

Per l'invio di notifiche push, viene utilizzato un model specifico per **Firebase Cloud Messaging**, definito in `GeneratedServer/src/models/notification/FirebaseMessagingModel.ts`.

Il metodo principale è `sendMessageToUser`:

```typescript
async sendMessageToUser(userId: number, message: BaseMessage, transaction: Kysely<DB>) {
    const fireBaseTokens = await fireBaseTokenModel.getFireBaseTokensByUserId(userId, transaction);
    if (fireBaseTokens.length === 0) {
        return [];
    }
    const messagesSent: Promise<String>[] = [];
    for (const token of fireBaseTokens) {
        const newMessage: TokenMessage = extend(message, "token", token.token);
        messagesSent.push(fireBaseMessaging.send(newMessage));
    }
    return Promise.all(messagesSent);
}
```

Questo model utilizza un'istanza di `fireBaseMessaging` inizializzata nel file `GeneratedServer/src/models/notification/FirebaseMessaging.ts`. Le credenziali per l'autenticazione con i servizi Google vengono caricate da un file di configurazione specificato nella variabile d'ambiente `GOOGLE_APPLICATION_CREDENTIALS`.

---

## Conclusione

Come abbiamo visto, i models sono componenti versatili che incapsulano la logica di accesso ai dati e l'interazione con servizi esterni. Questa separazione delle responsabilità rende i `Service` più snelli e focalizzati sulla logica di business principale.
