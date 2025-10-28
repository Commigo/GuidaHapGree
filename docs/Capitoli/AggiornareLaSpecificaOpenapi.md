Come abbiamo detto nel capitolo [**OpenApi**](/Capitoli/OpenApi.md) il nostro progetto utilizza la specifica Openapi per 
definire l'interfaccia delle nostre api, questo capitolo affronta come aggiornare la specifica per modificare o aggiungere nuove richieste

inizialmente la specifica veniva aggiornata a mano modificando il file `ApiDocumentation\OpenApi\OpenApi\HapGreeOpenApi.json`, questo 
era un processo molto scomodo poichè essendo un file json l'editor non aveva acesso a tutte le funzioni di autocompletamento e di revisione di quello che scrivevi
in oltre la specifica è molto verbosa e bisognava scrivere molto codice boilerplate e ripetitivo per ogni nuova operazione
stesso Openapi consiglia di non scrivere la specifica a mano per questi motivi, per migliorare questo processo è stato creato
un generatore della specifica che genera la specifica a partire da dei file typescript


