# Definizione dello Schema del Database con Prisma

In questo capitolo, vedremo come definire lo schema del nostro database (DB) utilizzando Prisma.

## L'approccio Iniziale e i Suoi Limiti

All'inizio del progetto, lo schema del database era definito in un file `.sql` che conteneva tutte le query per la creazione delle tabelle. Questo approccio presentava diversi problemi:

- Le query di creazione venivano validate solo a livello di sintassi SQL, ma non a livello di coerenza logica complessiva.
- Se una query `CREATE TABLE` faceva riferimento a una tabella non ancora creata, l'errore non veniva rilevato fino all'esecuzione del file SQL.
- Il file era più un insieme di operazioni da eseguire in sequenza che una vera e propria definizione dichiarativa dello schema.

## Miglioramento con Prisma

Per superare questi limiti, abbiamo adottato la libreria Prisma. Questo strumento permette di definire lo schema in uno o più file con estensione `.prisma`, utilizzando un linguaggio di definizione specifico.

I vantaggi principali di questo approccio sono:

- **Autocompletamento**: Supportato dall'editor di codice.
- **Validazione Logica**: Riconosce errori logici nella definizione dello schema in tempo reale, durante la scrittura.
- **Semplicità**: Rende la definizione dello schema più semplice e intuitiva, supportando anche costrutti come le `enum` in modo separato.

### Risorse Aggiuntive

Per imparare a scrivere uno schema Prisma, è possibile consultare la [documentazione ufficiale](https://www.prisma.io/docs/concepts/components/prisma-client/schema-definition).

### Posizione dello Schema

Nel nostro progetto, il file di schema si trova in: `GeneratedServer/prisma/schema.prisma`
