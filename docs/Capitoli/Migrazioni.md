# Le Migrazioni del Database con Prisma

Abbiamo visto come creare lo schema tramite il file `.prisma`. Ma come facciamo ad aggiornare lo schema e fare in modo che un database già creato e attivo riceva queste modifiche senza doverlo ricreare da capo?

## Il Problema: Aggiornare lo Schema del Database

All'inizio del progetto, quando utilizzavamo ancora un file SQL invece di Prisma, il processo di aggiornamento era complesso e soggetto a errori. Le opzioni erano:

-   Eliminare completamente il database e ricrearlo eseguendo il nuovo script SQL.
-   Eseguire manualmente le query di aggiornamento.

Quest'ultimo approccio spesso portava a eseguire query leggermente diverse da quelle definite nel file di definizione, creando inconsistenze tra i vari database locali e quello attivo sul server. Il processo era lento, macchinoso e facilmente soggetto a errori umani.

## La Soluzione: Le Migrazioni

Il processo può essere notevolmente migliorato utilizzando le **migrazioni**, una sorta di sistema di controllo di versione per il database, simile a Git.

Ogni volta che si modifica lo schema Prisma, vengono creati automaticamente dei file SQL, ordinati in base alla sequenza dei cambiamenti. Questi file contengono le operazioni necessarie per aggiornare lo schema. Il database tiene traccia delle migrazioni già applicate e, quando ne vengono create di nuove, può essere aggiornato semplicemente eseguendo i file SQL delle migrazioni non ancora applicate.

Per esempio, se si modifica uno schema creando una nuova tabella `Food`, Prisma genera una migrazione che crea un file SQL contenente una query `CREATE TABLE Food`. Quando viene avviato il comando di migrazione su un particolare database, questo rileva la nuova migrazione e la applica, eseguendo il `CREATE TABLE`. Se il database è già aggiornato, non accade nulla.

Di conseguenza, lo schema del database non è definito solo dal file di schema corrente, ma da tutto il suo storico di cambiamenti. Questo permette a qualsiasi istanza del database non aggiornata di aggiornarsi ripartendo dal punto dello storico in cui si era fermata. Questo vale anche per un'istanza appena creata, che eseguirà tutte le migrazioni in una sola volta.

> **Nota:** Qui stiamo sempre parlando dello schema del database, non dei dati presenti al suo interno.

## Come Gestire le Migrazioni

### Generare una Nuova Migrazione

Per poter generare le nuove migrazioni una volta modificato lo schema Prisma, è sufficiente lanciare il comando:

```bash
npx prisma migrate dev
```

Questo comando chiede di dare un nome alla nuova migrazione e tenta di applicarla al database collegato nell'ambiente di sviluppo.

Nel file `package.json` è stato creato uno script apposito per semplificare l'operazione:

```json
"scripts": {
  "prismaMigrate": "npx prisma migrate dev"
}
```

Prisma si collega automaticamente al database indicato nel file di schema. Nel nostro progetto, il database collegato dipende da una variabile d'ambiente:

```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}
```
un esempio del valore della variabile DATABASE_URL=mysql://root:root@localhost/HapGreeTest, le variabili d'ambiente verranno approfondite in un capitolo successivo

### Applicare le Migrazioni Esistenti

Quando invece si vogliono solo applicare le migrazioni create al database collegato (ad esempio, in un ambiente di produzione o dopo aver pullato nuove migrazioni), bisogna usare il comando:

```bash
npx prisma migrate deploy
```
Questo comando prova ad applicare tutte le migrazioni non applicate al database e si ferma se avviene un errore. A differenza di `migrate dev`, non genera nuove migrazioni e non chiede un nome.

## Migrazioni Manuali

Oltre alle migrazioni generate automaticamente da Prisma, è anche possibile creare delle **migrazioni manuali**. Queste possono essere utili per correggere errori particolari o per aggiungere dati fissi che possono essere considerati parte dello schema.

Per esempio, nel nostro progetto è stata aggiunta una migrazione manuale per inserire i ruoli utente:

```sql
-- roles
INSERT INTO Role(id,role) VALUES
(1,'Admin'),
(2,'Utente'),
(3,'Merchant') as v ON DUPLICATE KEY UPDATE `id` = v.id;
```

Questo garantisce che il database contenga sempre questi ruoli. Questa pratica ha senso solo se i valori sono speciali e fissi, quasi come se fossero parte dello schema stesso. Non deve essere utilizzata per inserire dati di esempio, poiché quelli sono dati che dipendono dalla specifica istanza del database.

Nel nostro progetto la cartella che contiene tutte le migrazioni è: `GeneratedServer/prisma/migrations`.


## Approfondimenti

Per approfondire il concetto delle migrazioni, è possibile consultare la [documentazione ufficiale di Prisma](https://www.prisma.io/docs/orm/prisma-migrate/getting-started).
