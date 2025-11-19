<!-- Nota: File riscritto e riformattato per maggiore chiarezza. -->

# Build e Avvio con Docker

In questo capitolo viene descritto il processo di build e avvio del server utilizzando Docker, che garantisce un ambiente di esecuzione consistente e automatizzato.

## Build dell'Immagine Docker

Per creare l'immagine Docker del server, si utilizza il comando `docker build`.

```bash
docker build -t hapgree/server -f ./Dockerfile .
```

-   `-t hapgree/server`: Assegna un nome (`tag`) all'immagine creata.
-   `-f ./Dockerfile`: Specifica il percorso del `Dockerfile` da utilizzare per la build.
-   `.`: Indica il contesto della build (la directory corrente).

### Passare Argomenti alla Build

Durante la build, è possibile passare degli argomenti utilizzando il flag `--build-arg`. Questo è utile per fornire valori segreti, come il token di NPM, senza includerli direttamente nel `Dockerfile`.

```bash
docker build --build-arg NPM_TOKEN=IL_TUO_TOKEN ...
```

Questo token è necessario nello stage `openapi-generator` per autenticarsi e scaricare le librerie necessarie alla generazione delle specifiche OpenAPI, come descritto nel capitolo [GenerareLaSpecificaOpenapi](./GenerareLaSpecificaOpenapi.md).

## Sviluppo in Locale: Estrarre File dalla Build

Durante lo sviluppo, è fondamentale che l'IDE (es. VSCode, WebStorm) abbia accesso ai file sorgenti TypeScript generati dalla specifica OpenAPI per fornire funzionalità come l'autocompletamento e l'analisi del codice. Questi file vengono creati *all'interno* del container durante la build, quindi non sarebbero normalmente disponibili sul filesystem locale.

Per risolvere questo problema, il `Dockerfile` include uno stage speciale chiamato `exporter` che, combinato con un comando specifico, permette di copiare i file generati dal container al progetto locale.

### Lo Stage `exporter`

```Dockerfile
FROM scratch as exporter
WORKDIR /
COPY --from=generator /usr/generateNode/GeneratedServer/generated ./
```

Questo stage copia i file generati dallo stage `generator` in una posizione temporanea.

### Comando per Estrarre i File

Per estrarre i file e salvarli nella cartella `.\GeneratedServer\generated`, si usa il seguente comando:

```bash
docker buildx build --target exporter --output type=local,dest=.\GeneratedServer\generated -f ./Dockerfile . --build-arg NPM_TOKEN=%NPM_TOKEN%
```

Questo processo può essere lanciato manualmente tramite lo script:

-   `Scripts\BuildServerGenerated\GenerateServerFile.bat`

## Avvio dei Container con Docker Compose

Per avviare i container in modo orchestrato, utilizziamo `docker compose`. Questo strumento legge un file di configurazione (`docker-compose.yaml`) per avviare e collegare più servizi, come il server applicativo e il database.

### File di Configurazione `docker-compose.yaml`

Il file di configurazione definisce i servizi, le reti e i volumi:

```yaml
services:
  mysql:
    image: mysql
    container_name: my-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"
      - "33060:33060"
    networks:
      - appnet
    volumes:
      - mysql:/var/lib/mysql

  hapgree-server:
    image: hapgree/server
    container_name: HapGreeServer
    restart: always
    env_file:
      - ./GeneratedServer/.env
    ports:
      - "3000:3000"
    volumes:
      - ./GeneratedServer/firebase:/var/firebase:ro
    depends_on:
      - mysql
    networks:
      - appnet

networks:
  appnet:
    driver: bridge

volumes:
  mysql:
    external: true
```

### Avvio dei Servizi

Per avviare tutti i servizi definiti nel file, si lancia il comando:

```bash
docker compose up -d
```

Il flag `-d` avvia i container in background (detached mode).

## Script di Automazione

L'intero processo di build, estrazione dei file e avvio dei container è automatizzato nello script:

-   `Scripts\BuildServerGenerated\BuildAndStartServer.bat`

Questo è il metodo consigliato per avviare il progetto in locale.

## Ambienti di Test e Produzione (CI/CD)

Il processo di Continuous Integration/Continuous Deployment (CI/CD) si basa sulla stessa immagine Docker creata dal `Dockerfile`.

-   **Commit**: Un commit su un branch specifico avvia uno script che builda l'immagine e la rilascia sul server di **test**.
-   **Release**: La creazione di una nuova release avvia uno script che builda l'immagine e la rilascia sul server di **produzione**.

La configurazione degli ambienti di produzione e test (variabili d'ambiente, volumi, ecc.) è gestita da un altro team. Il nostro compito è fornire un'immagine Docker stabile e funzionante.
