# Introduzione al Database

Nel nostro progetto, utilizziamo un database MySQL per la persistenza dei dati.

Per gestire correttamente il database all'interno del progetto, è fondamentale comprendere tre concetti chiave:

1.  **Definizione e modifica dello schema del database:** Sapere come creare e aggiornare la struttura del database.
2.  **Allineamento dello schema tra le versioni:** Assicurarsi che lo schema del database sia consistente in tutti gli ambienti di deploy.
3.  **Interrogazione del database dall'applicazione:** Sapere come interagire con il database tramite il codice della nostra applicazione.

## Utilizzo di Prisma ORM

Per gestire questi aspetti, ci affidiamo a [**Prisma**](https://www.prisma.io/), un ORM (Object-Relational Mapping) che semplifica l'interazione con il database.

Prisma svolge tre funzioni principali:

*   **Definizione dello schema:** Permette di definire lo schema del database in un unico file principale.
*   **Migrazioni:** Tiene traccia delle modifiche allo schema e genera automaticamente le migrazioni necessarie per aggiornare la struttura del database in modo sicuro e controllato in tutti gli ambienti.
*   **Client per le query:** Fornisce una libreria client per il server che facilita l'interrogazione del database in modo type-safe.
