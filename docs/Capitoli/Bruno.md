# Bruno

Per testare velocemente le API implementate dal nostro server, facciamo uso di [*Bruno*](https://www.usebruno.com/), un programma molto simile a Postman che permette di definire ed eseguire una serie di richieste API.

È dotato di una semplice interfaccia grafica che consente di specificare facilmente i parametri e la rotta della richiesta, eseguirla e visualizzarne la risposta. Per approfondire l'utilizzo di Bruno, si consiglia di consultare la documentazione ufficiale.

## Archiviazione basata su file

La particolarità di Bruno rispetto a Postman è che tutte le richieste, e in generale la configurazione del progetto, vengono trasformate in file di testo scritti in un linguaggio specifico di Bruno. Questo approccio basato su file rende possibile versionare il progetto Bruno all'interno della repository Git. Di conseguenza, ogni cambiamento viene tracciato e può essere committato, semplificando la collaborazione.

Questo rappresenta un grande vantaggio rispetto a Postman, che salva i dati sul proprio cloud e richiede un abbonamento a pagamento per collaborare in team su più progetti.

## Ambienti (Environments)

Un'altra funzionalità di Bruno che utilizziamo sono gli **ambienti** (environments). Questi sono insiemi di variabili che possono essere utilizzate all'interno della definizione delle richieste. La loro particolarità è che è possibile cambiare l'ambiente corrente per modificare il valore di queste variabili.

Nel nostro progetto utilizziamo tre ambienti:
- **Locale**
- **Test**
- **Produzione**

Le variabili definite in questi ambienti sono principalmente due: una per l'**endpoint del server** e un'altra per il **token di autenticazione**. Grazie a questa configurazione, possiamo testare facilmente le API sui tre diversi ambienti di sviluppo semplicemente selezionando l'ambiente desiderato.

## Posizione del Progetto

Il progetto Bruno del nostro server si trova nella seguente directory: `ApiDocumentation\GeneratedBruno`.
