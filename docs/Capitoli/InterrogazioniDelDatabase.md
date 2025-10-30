# Interrogazioni del Database

In questo progetto, vengono utilizzati due strumenti principali per interrogare il database: **Prisma** e **Kysely**.

## Interrogazioni con Prisma Client

Prisma è l'ORM primario utilizzato per la maggior parte delle interazioni con il database. Fornisce un client type-safe generato automaticamente a partire dallo schema del database.

### Generazione del Client

Per poter interrogare il database, Prisma è in grado di generare una libreria client ad-hoc basata sullo schema del database. Questa libreria viene aggiunta automaticamente ai `node_modules` del progetto.

Per generare o aggiornare la libreria, denominata `@prisma/client`, è sufficiente lanciare il seguente comando:

```bash
npx prisma generate
```

Questo comando analizzerà il file `schema.prisma` e creerà la libreria client personalizzata all'interno della cartella `node_modules`.

### Utilizzo del Client

Nel nostro progetto, il file `GeneratedServer/src/models/prisma.ts` si occupa di inizializzare il client Prisma, connetterlo al database ed esportare l'oggetto `prisma`, che verrà poi utilizzato per eseguire le query.

Per una guida completa su come utilizzare la libreria per effettuare query, si consiglia di consultare la [documentazione ufficiale di Prisma](https://www.prisma.io/docs/orm/prisma-client/queries).

### Gestione delle Transazioni con Prisma

Prisma offre diverse funzionalità per la gestione delle transazioni, come descritto nella [documentazione ufficiale](https://www.prisma.io/docs/orm/prisma-client/queries/transactions).

#### Transazioni Interattive

In particolare, per le [transazioni interattive](https://www.prisma.io/docs/orm/prisma-client/queries/transactions#interactive-transactions), è stata creata una funzione di supporto denominata `prismaTransacionResultAndError`.

Questa funzione, che si trova sempre nel file `GeneratedServer/src/models/prisma.ts`, permette di creare una transazione equivalente a quella standard, con la differenza che la gestione degli errori non avviene tramite eccezioni, ma restituendo un oggetto di tipo `ResultAndError`.

---

## Interrogazioni con Kysely

Nel progetto è presente anche [Kysely](https://kysely.dev/), una libreria per la costruzione di query SQL in modo type-safe. Kysely è più simile alla scrittura di SQL nativo, ma garantisce che le query facciano riferimento a tabelle e proprietà realmente esistenti nel database, grazie all'inferenza dei tipi di TypeScript.

### Quando usare Kysely

È utile utilizzare Kysely quando la libreria di Prisma presenta delle limitazioni o non rende possibile sfruttare funzionalità specifiche di SQL.

### Utilizzo di Kysely

Nel file `GeneratedServer/src/models/database.ts` viene esportato l'oggetto `kysely`, già configurato e collegato al database.

Per una guida su come formulare le query, è possibile consultare gli [esempi nella documentazione ufficiale di Kysely](https://kysely.dev/docs/category/examples).

### Gestione delle Transazioni con Kysely

Per la gestione delle transazioni con Kysely, è disponibile una funzione helper nel file `GeneratedServer/src/models/database.ts`:

#### `startTransactionKyseley()`

Questa funzione restituisce un oggetto per la gestione della transazione che, dopo aver eseguito le operazioni, può essere confermato (`commit`) e successivamente rilasciato (`release`).

-   Se `release()` viene chiamato prima di `commit()`, la transazione viene annullata (rollback).
-   Se `release()` viene chiamato dopo `commit()`, non ha alcun effetto.

Questo meccanismo è particolarmente utile per creare un costrutto `try...finally` che garantisca il rollback automatico in caso di eccezioni o di uscite anticipate dalla funzione prima del commit. Questo approccio migliora l'API di base di Kysely, dove è necessario gestire manually il rollback in ogni possibile caso di errore.

La proprietà `transaction` dell'oggetto restituito dalla funzione può essere usata normalmente per eseguire le query, che verranno eseguite nel contesto della transazione.

#### Esempio di utilizzo

Ecco un esempio di come utilizzare la funzione `startTransactionKyseley()`:

```typescript
const transaction = await startTransactionKyseley();
try {
    await groupModel.makeGroupInvite(id, idUtente, transaction.transaction);

    await transaction.commit();

    // Operazioni post-commit
    await firebaseMessagingModel.sendMessageToUser(idUtente, {
        notification: {
            title: "Invito a gruppo",
            body: "Hai ricevuto un invito al gruppo ",
        }
    }, dbKysely);

    return [undefined, null];
} catch (error) {
    // La transazione verrà annullata dal blocco finally
    return [null, "Errore inaspettato " + error];
} finally {
    // Esegue il rollback se il commit non è avvenuto, altrimenti non fa nulla.
    await transaction.release();
}
```
In questo esempio, se si verifica un'eccezione all'interno del blocco `try`, il blocco `finally` viene eseguito, chiamando `release()` ed effettuando il rollback della transazione. Se invece si arriva al `commit()` senza errori, la chiamata a `release()` nel `finally` non avrà alcun effetto.

### Generazione dei Tipi per Kysely

Per funzionare correttamente, Kysely utilizza un file che descrive le tabelle del database in tipi TypeScript. Questo file si trova in `GeneratedServer/src/db/HapGree.ts`.

Il file viene generato automaticamente tramite il comando `npx prisma generate`, in quanto nello schema `prisma.schema` è stato aggiunto il seguente generatore:

```prisma
generator kysely {
  provider = "prisma-kysely"

  // Optionally provide a destination directory for the generated file
  // and a filename of your choice
  output   = "../src/db"
  fileName = "HapGree.ts"
  decimalTypeOverride = "ColumnType<string, number | string>"
}
```
