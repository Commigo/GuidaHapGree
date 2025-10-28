# OpenAPI

Nel nostro progetto facciamo uso dello standard [*OpenAPI*](https://learn.openapis.org/).

È consigliato approfondire l'argomento consultando il link sopra, ma ecco una breve descrizione.

## Cos'è OpenAPI?

Lo standard OpenAPI è un documento che descrive l'interfaccia delle nostre API. In sostanza, descrive quali route esistono, quali parametri accettano, quale autorizzazione utilizzano e che tipi di valori restituiscono.

Un piccolo esempio, per una nostra API che permette di invitare un utente a un gruppo, è il seguente:

```json
"/group/{id}/invitations": {
  "post": {
    "description": "permette di creare un invito per entrare nel gruppo",
    "operationId": "MakeGroupInvitationById",
    "tags": [
      "Gruppi"
    ],
    "responses": {
      "204": {
        "description": "in caso di successo"
      },
      "default": {
        "description": "in caso di errore",
        "content": {
          "application/json": {
            "schema": {
              "$ref": "#/components/schemas/BadResponse"
            }
          }
        }
      }
    },
    "security": [
      {
        "BearerToken": []
      }
    ],
    "parameters": [
      {
        "name": "id",
        "in": "path",
        "description": "l'id del gruppo",
        "required": true,
        "schema": {
          "type": "integer",
          "minimum": -9007199254740991,
          "maximum": 9007199254740991
        },
        "example": 1
      },
      {
        "name": "idUtente",
        "in": "query",
        "description": "L'id dell'utente da invitare",
        "schema": {
          "type": "integer",
          "minimum": -9007199254740991,
          "maximum": 9007199254740991
        },
        "required": true,
        "example": 1
      }
    ]
  }
}
```

Come si può leggere da questo esempio, vengono descritti tutti i parametri, quali sono opzionali, di che tipo sono, il nome della route, l'autenticazione usata, il tipo di risposta, ecc.


Per imparare a leggere questo tipo di documenti è necessario consultare la documentazione ufficiale di OpenAPI, anche se non è strettamente necessario per il nostro lavoro.

## Come lo usiamo

Essendo un documento scritto in YAML o JSON, è facilmente analizzabile dai programmi e permette quindi di essere usato da alcuni tool per facilitare la creazione delle API. Il modo in cui lo usiamo è il seguente:

- **Generazione richieste Bruno:** Lo usiamo per creare automaticamente tutte le richieste in Bruno. Visto che il documento descrive tutte le route e i parametri, abbiamo tutto ciò che ci serve.
- **Generazione codice Flutter:** Lo usiamo per generare il codice in Flutter per chiamare le API. Similmente a Bruno, avendo le route, i parametri e i valori di ritorno, possiamo creare il codice che effettua la chiamata e analizza i risultati.
- **Generazione backend (route, controller, interfacce):** Lo usiamo per generare le route, i controller e le interfacce dei service nel server. I controller si occupano di verificare i parametri e i tipi passati dalla richiesta HTTP e, dato che queste informazioni sono nel documento OpenAPI, possiamo generarli automaticamente.
- **Raggruppamento logico delle funzionalità:** Utilizziamo i tag definiti nella specifica per raggruppare le funzionalità in gruppi logici, come menzionato nei capitoli precedenti. Ad esempio, tutte le richieste relative a `Food` hanno il tag `Food`, in modo da segnalare che fanno parte dello stesso gruppo.

## Vantaggi

Questo approccio è molto comodo perché abbiamo un solo punto in cui modificare la definizione dell'API. Di conseguenza, tutti i servizi che la utilizzano vengono aggiornati e creati automaticamente.

Prima di usare questa specifica, bisognava creare a mano le chiamate su Bruno, la classe per la chiamata in Flutter e il controller per la verifica dei parametri sul server. Molto spesso ci si dimenticava di aggiornare uno di questi elementi, e in generale era un processo molto ripetitivo e noioso che adesso non è più necessario.

## Posizione del file

Nel nostro progetto, il file OpenAPI si trova in `ApiDocumentation\OpenApi\OpenApi\HapGreeOpenApi.json`. Questa cartella è parte di un submodule Git del progetto.
