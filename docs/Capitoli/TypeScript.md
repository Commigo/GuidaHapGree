# TypeScript

Il nostro progetto utilizza `typescript` al posto di `javascript`. TypeScript non è altro che JavaScript con la possibilità di annotare variabili e funzioni con dei tipi, in modo che il compilatore possa controllare la presenza di errori e suggerire proprietà o metodi durante la scrittura del codice.

## Perché TypeScript?

Inizialmente, il progetto utilizzava JavaScript puro, ma questo portava a frequenti errori a runtime, come:
*   Passaggio di variabili di tipo errato.
*   Utilizzo di metodi inesistenti.
*   Nomi di proprietà scritti in modo errato che restituivano `null` invece di segnalare l'errore.

Questi problemi erano difficili da debuggare e si manifestavano solo durante l'esecuzione. Inoltre, l'editor di codice non era in grado di fornire suggerimenti o autocompletamento, non conoscendo i tipi delle variabili, rendendo l'esperienza di sviluppo frustrante.

## Approfondimenti

Per approfondire il funzionamento di TypeScript, è fortemente consigliata la lettura della guida ufficiale: [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html).

## Come funziona TypeScript?

TypeScript non è un linguaggio a sé stante. Il codice TypeScript viene compilato in codice JavaScript, che viene poi eseguito normalmente. Questo approccio comporta alcune limitazioni: i tipi aggiunti non esistono a livello di esecuzione del codice.

Ciò significa che possono verificarsi situazioni in cui il compilatore non rileva problemi, ma a runtime una variabile o un parametro riceve un valore di tipo diverso da quello atteso. Questo accade spesso quando si ricevono dati da API, di cui non si conosce con certezza il tipo.

## Configurazione: tsconfig.json

Ogni progetto TypeScript include un file di configurazione chiamato `tsconfig.json`. Nel nostro progetto, questo file si trova in `/GeneratedServer/tsconfig.json`.

Questo file specifica:
*   I file di input TypeScript da compilare.
*   La directory di output per i file JavaScript generati.
*   Altre opzioni del compilatore, come il livello di rigore nel controllo dei tipi e le modalità di importazione delle librerie.

Nel nostro `tsconfig.json` troverete le seguenti proprietà principali:

```json
{
  "include": ["./**/*.ts"],
  "outDir": "./build"
}
```

*   `"include": ["./**/*.ts"]`: Indica che tutti i file con estensione `.ts` nel progetto devono essere compilati.
*   `"outDir": "./build"`: Specifica che l'output della compilazione deve essere salvato nella cartella `./build`.

## Compilazione

Per compilare il progetto, è sufficiente eseguire il comando `npx tsc`. Il compilatore `tsc` (installato come dipendenza tramite npm) leggerà la configurazione dal file `tsconfig.json` per individuare i file di input e generare i file di output, o per mostrare eventuali errori di compilazione.

L'editor di codice è già configurato per segnalare gli errori TypeScript in tempo reale, quindi non è necessario eseguire manualmente il compilatore per individuarli: appariranno in rosso direttamente nell'editor.

## Esecuzione

I file che vengono effettivamente eseguiti sono i file JavaScript generati dalla compilazione. Per avviare il server, bisogna eseguire il file `index.js` nella cartella `build`, compilato a partire dal file TypeScript sorgente `/GeneratedServer/generated/index.ts`.

Il comando per avviare il server è:
```bash
node ./build/generated/index.js
```

## Script NPM

I comandi di build e di avvio del server sono stati aggiunti come script al file `package.json` del progetto per semplificarne l'utilizzo.

```json
{
  "scripts": {
    "start": "node ./build/generated/index.js",
    "build": "npx tsc",
    "buildAndRun": "npm run build && npm run start"
  }
}
```

Questo permette di eseguire i comandi in modo più semplice:
*   `npm run build` per compilare il progetto.
*   `npm start` per avviare il server.
*   `npm run buildAndRun` per fare entrambi in un solo comando.



