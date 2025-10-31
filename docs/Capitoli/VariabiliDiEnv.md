# Gestione delle Variabili d'Ambiente

Nei capitoli precedenti sono state spesso menzionate le variabili d'ambiente e come alcune di queste influenzano il comportamento di tool o del codice del server.

Nel nostro progetto, diverse variabili vengono utilizzate per configurare aspetti come la connessione al database, le API di PayPal, etc.

## Configurazione

Per rendere le variabili d'ambiente disponibili e leggibili automaticamente sia dal progetto del server che da alcuni tool (come i generatori di Prisma), è necessario creare un file `.env` nella root del progetto NodeJS, ovvero in `GeneratedServer/.env`.

**Nota:** Questo file non deve essere committato in quanto contiene informazioni sensibili come le chiavi API per PayPal.

## Accesso alle Variabili

Le variabili d'ambiente possono essere lette all'interno del codice TypeScript in questo modo:

```typescript
process.env.NOME_DELLA_VARIABILE
```

## Variabili Utilizzate

Di seguito sono elencate tutte le variabili utilizzate nel progetto e il loro scopo.

### Variabili Generali

*   `PORT`: Rappresenta la porta su cui il server si deve avviare.
*   `MODE`: Può avere valore `TEST` o `RELEASE` ed indica se è un server di test o di produzione.

### Variabili di Connessione al Database

Queste variabili dipendono dal valore di `MODE`.

*   **Se `MODE=RELEASE`**:
    *   `DB_HOST_RELEASE`: Host del database a cui collegarsi.
    *   `DB_USER_RELEASE`: Utente per la connessione al database.
    *   `DB_PASSWORD_RELEASE`: Password per la connessione al database.
    *   `DB_NAME_RELEASE`: Nome del database a cui connettersi.
*   **Se `MODE=TEST`**:
    *   `DB_HOST_TEST`: Host del database a cui collegarsi.
    *   `DB_USER_TEST`: Utente per la connessione al database.
    *   `DB_PASSWORD_TEST`: Password per la connessione al database.
    *   `DB_NAME_TEST`: Nome del database a cui connettersi.

### Variabili per PayPal

Queste variabili dipendono dal valore di `MODE`.

*   **Se `MODE=TEST`**:
    *   `PAYPAL_CLIENT_ID_TEST`: ID del client di PayPal.
    *   `PAYPAL_CLIENT_SECRET_TEST`: Chiave segreta di autenticazione per PayPal.
*   **Se `MODE=RELEASE`**:
    *   `PAYPAL_CLIENT_ID_RELEASE`: ID del client di PayPal.
    *   `PAYPAL_CLIENT_SECRET_RELEASE`: Chiave segreta di autenticazione per PayPal.

### Variabili per Altre API

Queste variabili dipendono dal valore di `MODE`.

*   `OTHER_API_SERVER_TEST`: Se `MODE=TEST`, indica l'host dell'altro server API.
*   `OTHER_API_SERVER_RELEASE`: Se `MODE=RELEASE`, indica l'host dell'altro server API.
*   `ORDER_API_KEY`: Valore segreto che permette di autenticarsi con l'altro server API per la gestione degli ordini. Permette a loro di sapere che la richiesta arriva da noi.

### Altre Variabili

*   `GOOGLE_APPLICATION_CREDENTIALS`: Indica il path del file di configurazione di Firebase.
*   `DATABASE_URL`: Indica il database utilizzato da `prisma migrate` per capire a quale database connettersi.
