# Dove si trova il server
I file che compongono il server si trovano nella cartella /GeneratedServer

# Cosa fa il nostro Server?

Il server fa tre cose principali:
1.  **API REST**: Gestisce le richieste che arrivano da altre applicazioni.
2.  **Task Pianificati**: Avvia dei processi in automatico a intervalli di tempo regolari.
3.  **File Statici**: Rende disponibili dei file (come immagini o documenti).

## le API Rest

Le API REST sono la parte fondamentale del server. Tramite queste, le altre applicazioni (come l'app mobile) possono "dialogare" con il nostro sistema. Funziona così: un'app fa una richiesta HTTP al server e lui risponde. La risposta può contenere dati (solitamente in formato JSON), confermare la creazione di nuove risorse o l'esecuzione di determinate azioni.

Ad esempio, se l'app mobile vuole sapere quali sono le challenge attive, fa una richiesta GET all'indirizzo `/challenge/getActiveChallenges`. Il server, a sua volta, risponde con un documento JSON strutturato in questo modo:

```json
{
  "giornaliere": [
    {
      "timeframe": "2025-10-25",
      "idTemplate": 13,
      "isGroupChallenge": 0,
      "id": 335,
      "reward": 0,
      "timeframe_type": "giorno",
      "category": "mobilita_sostenibile",
      "createdAt": "2025-07-24T12:27:36.000Z",
      "updatedAt": "2025-07-24T12:27:36.000Z",
      "vehicleType": "gasolineCar",
      "numberOfTrips": 1,
      "progress": {
        "tripsNeeded": 1,
        "progress": 0,
        "tripsMade": 0
      },
      "timeframeType": "giorno",
      "collected": false
    },
    {
      "timeframe": "2025-10-25",
      "idTemplate": 8,
      "isGroupChallenge": 0,
      "id": 334,
      "reward": 0,
      "timeframe_type": "giorno",
      "category": "inserimento_cibo",
      "createdAt": "2025-07-24T12:27:36.000Z",
      "updatedAt": "2025-07-24T12:27:36.000Z",
      "gramsTreshold": 100,
      "foodCategory": "a",
      "progress": {
        "progress": 0,
        "gramsEaten": 0,
        "gramsNeeded": 100
      },
      "timeframeType": "giorno",
      "collected": false
    },
    {
      "timeframe": "2025-10-25",
      "idTemplate": 18,
      "isGroupChallenge": 0,
      "id": 336,
      "reward": 0,
      "timeframe_type": "giorno",
      "category": "passi",
      "createdAt": "2025-08-10T21:12:19.000Z",
      "updatedAt": "2025-08-10T21:12:19.000Z",
      "stepsNeeded": 8000,
      "progress": {
        "stepsDone": 0,
        "stepsNeeded": 8000,
        "progress": 0
      },
      "timeframeType": "giorno",
      "collected": false
    }
  ],
  "settimanali": [
    {
      "timeframe": "2025-W43",
      "idTemplate": 30,
      "isGroupChallenge": 0,
      "id": 301,
      "reward": 0,
      "timeframe_type": "settimana",
      "category": "mobilita_sostenibile",
      "createdAt": "2025-08-10T21:12:19.000Z",
      "updatedAt": "2025-08-10T21:12:19.000Z",
      "vehicleType": "gasolineCar",
      "numberOfTrips": 5,
      "progress": {
        "tripsNeeded": 5,
        "progress": 0,
        "tripsMade": 0
      },
      "timeframeType": "settimana",
      "collected": false
    },
    {
      "timeframe": "2025-W43",
      "idTemplate": 7,
      "isGroupChallenge": 0,
      "id": 299,
      "reward": 0,
      "timeframe_type": "settimana",
      "category": "inserimento_cibo",
      "createdAt": "2025-07-24T12:27:36.000Z",
      "updatedAt": "2025-07-24T12:27:36.000Z",
      "gramsTreshold": 800,
      "foodCategory": "a",
      "progress": {
        "progress": 0,
        "gramsEaten": 0,
        "gramsNeeded": 800
      },
      "timeframeType": "settimana",
      "collected": false
    },
    {
      "timeframe": "2025-W43",
      "idTemplate": 19,
      "isGroupChallenge": 0,
      "id": 303,
      "reward": 0,
      "timeframe_type": "settimana",
      "category": "passi",
      "createdAt": "2025-08-10T21:12:19.000Z",
      "updatedAt": "2025-08-10T21:12:19.000Z",
      "stepsNeeded": 50000,
      "progress": {
        "stepsDone": 15914,
        "stepsNeeded": 50000,
        "progress": 0.31828
      },
      "timeframeType": "settimana",
      "collected": false
    }
  ],
  "mensili": []
}
```

## Task pianificati
Il server si occupa anche di eseguire azioni a intervalli regolari, ad esempio inviare a tutti gli utenti una notifica alle 8 di ogni giorno, oppure controllare ogni 5 minuti se l'elaborazione degli scontrini è completata, ecc.

## File statici
Infine, si occupa di rendere disponibili file in modo pubblico, come ad esempio loghi, immagini di default o altri ancora.