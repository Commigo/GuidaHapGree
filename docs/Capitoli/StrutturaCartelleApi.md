# Struttura delle Cartelle API

Dopo aver analizzato nel capitolo precedente la struttura del codice per l'implementazione delle API, vediamo ora come i vari file sono organizzati all'interno delle cartelle del progetto.

## Codice Generato e Codice Manuale

Come già accennato, parte del codice viene generato automaticamente a partire dalla specifica OpenAPI (un concetto che verrà approfondito in seguito).

-   Tutti i file generati si trovano nella cartella `GeneratedServer/generated`.
-   Tutto il codice scritto manualmente si trova invece nella cartella `GeneratedServer/src`.

## Organizzazione Logica

All'interno di queste due directory, i file sorgente sono raggruppati in base a gruppi logici di funzionalità, come ad esempio `Mobility`, `Auth`, `Compensation`, `Gruppi`, etc.

Infatti, sia in `GeneratedServer/generated/api` che in `GeneratedServer/src/api` sono presenti tante cartelle quanti sono i gruppi di funzionalità.

### Contenuto delle Cartelle Generate

In ogni cartella di un gruppo, all'interno di `GeneratedServer/generated/api`, sono presenti i seguenti file:
-   Interfacce dei Service
-   Controller
-   Interfacce degli Interceptor
-   Routes

### Contenuto delle Cartelle Manuali

In ogni cartella di un gruppo, all'interno di `GeneratedServer/src/api`, sono presenti i file di implementazione:
-   Dei Service
-   Degli Interceptor

La struttura di questi file è stata descritta nel capitolo [precedente](/Capitoli/StrutturaCodiceApi.md).

## Aggiunta di un Nuovo Gruppo di Funzionalità

Se si desidera aggiungere un nuovo gruppo di funzionalità, è necessario:
1.  Aggiornare la specifica OpenAPI, come spiegato nei capitoli successivi.
2.  Creare manualmente la cartella con i file di implementazione per il nuovo gruppo all'interno di `GeneratedServer/src/api`.
