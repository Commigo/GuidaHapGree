# Analisi del Dockerfile di Progetto

In questo capitolo analizziamo il `Dockerfile` presente nella root del progetto, che sfrutta la funzionalità di multi-stage build di Docker per ottimizzare la creazione dell'immagine del container.

## Cos'è un Multi-Stage Build?

Un multi-stage build è una tecnica che permette di utilizzare più immagini intermedie (chiamate "stage") durante il processo di build di un'immagine Docker finale. Ogni stage può avere un proprio `Dockerfile`, dipendenze e comandi specifici.

I vantaggi principali di questo approccio sono:
- **Riduzione delle dimensioni dell'immagine finale**: È possibile copiare solo gli artefatti necessari (es. un eseguibile compilato) da uno stage intermedio all'immagine finale, escludendo dipendenze di build, file sorgenti e tool non necessari a runtime.
- **Migliore organizzazione**: Ogni fase del processo di build è isolata nel proprio stage, rendendo il `Dockerfile` più pulito e manutenibile.

Nel nostro progetto, utilizziamo i multi-stage build non solo per ridurre le dimensioni dell'immagine, ma soprattutto per automatizzare la generazione della specifica OpenAPI e del codice del server derivato, come descritto nei capitoli [GeneratoriOpenapi](./GeneratoriOpenapi.md) e [GenerareLaSpecificaOpenapi](./GenerareLaSpecificaOpenapi.md).

---

## Dettaglio degli Stage

Vediamo nel dettaglio come è strutturato il nostro `Dockerfile`.

### Stage 1: `openapi-generator`

Questo primo stage è dedicato alla generazione del file `openapi-document.json`, che rappresenta la specifica OpenAPI della nostra API.

```dockerfile
FROM ghcr.io/commigo/typescript-to-openapi-generator:latest as openapi-generator
ARG NPM_TOKEN

WORKDIR /gen/openApiGenerator
COPY ./ApiDocumentation/OpenApi/OpenApi/src ./src
COPY ./ApiDocumentation/OpenApi/OpenApi/configuration.json ./
COPY ./ApiDocumentation/OpenApi/OpenApi/package.json ./
COPY ./ApiDocumentation/OpenApi/OpenApi/tsconfig.json ./
COPY ./ApiDocumentation/OpenApi/OpenApi/package-lock.json ./
COPY ./ApiDocumentation/OpenApi/OpenApi/.npmrc ./
RUN npm ci

WORKDIR /app
RUN /app/dockerEntrypoint.sh /gen/openApiGenerator/configuration.json ./openapi-document.json
```
**Operazioni eseguite:**
1.  Utilizza un'immagine pre-configurata per la generazione di specifiche OpenAPI da codice TypeScript.
2.  Copia i file sorgenti e di configurazione necessari dalla cartella `ApiDocumentation`.
3.  Installa le dipendenze con `npm ci`.
4.  Esegue lo script `dockerEntrypoint.sh` per generare il file `openapi-document.json`.

### Stage 2: `generator`

Il secondo stage utilizza il file `openapi-document.json` generato in precedenza per creare il codice sorgente del server.

```dockerfile
FROM ghcr.io/commigo/openapi-generator:latest as generator
WORKDIR /usr/generateNode

COPY --from=openapi-generator /app/openapi-document.json ./openapi-document.json
RUN /usr/local/bin/docker-entrypoint.sh generate -g typescript-node-server -i ./openapi-document.json -o ./GeneratedServer
```
**Operazioni eseguite:**
1.  Utilizza un'immagine del generatore OpenAPI.
2.  Copia il file `openapi-document.json` dallo stage precedente (`openapi-generator`).
3.  Esegue il comando `generate` per creare i file del server nella cartella `GeneratedServer`.

### Stage 3: `builder`

Questo stage assembla l'applicazione finale, combinando il codice sorgente del server con i file generati nello stage precedente.

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /usr/src/app

COPY GeneratedServer/package*.json ./
RUN npm ci
COPY ./GeneratedServer ./
COPY --from=generator /usr/generateNode/GeneratedServer/generated ./generated
RUN npx prisma generate
RUN npm run build
```
**Operazioni eseguite:**
1.  Parte da un'immagine Node.js.
2.  Installa le dipendenze del server.
3.  Copia il codice del server e i file generati dallo stage `generator`.
4.  Esegue `npx prisma generate` per creare il client Prisma.
5.  Compila il progetto con `npm run build`.

### Stage 4: Immagine Finale

L'ultimo stage crea l'immagine finale, leggera e ottimizzata per l'esecuzione.

```dockerfile
FROM node:20-slim
WORKDIR /app

# Copia solo gli artefatti compilati e necessari a runtime
COPY --from=builder /usr/src/app/build ./build
COPY --from=builder /usr/src/app/platform ./platform
COPY --from=builder /usr/src/app/prisma ./prisma
COPY --from=builder /usr/src/app/Scripts ./Scripts

RUN chmod +x ./Scripts/RunServer.sh ./Scripts/SetDbEnv.sh
EXPOSE 3000
CMD ["./Scripts/RunServer.sh"]
```
**Caratteristiche:**
- **Base leggera**: Utilizza `node:20-slim`, che contiene solo il minimo necessario per eseguire un'applicazione Node.js.
- **Artefatti essenziali**: Copia solo le cartelle `build`, `platform`, `prisma` e `Scripts` dallo stage `builder`, escludendo sorgenti e dipendenze non necessarie.
- **Esecuzione**: Imposta lo script `RunServer.sh` come comando di avvio del container.
In realtà questo stage è diverso se si va a vedere nel progetto, le cose in più riguardano delle cose non ancora finite
e che quindi ho omesso dalla spiegazione.
---

## Script di Avvio: `RunServer.sh`

Questo script, situato in `GeneratedServer/Scripts/RunServer.sh`, gestisce l'avvio del server all'interno del container.

```bash
#!/bin/bash

# Script per avviare il server HapGree con migrazioni Prisma
# Questo script:
# 1. Imposta le variabili di ambiente del database
# 2. Esegue le migrazioni Prisma in produzione
# 3. Avvia il server Node.js

set -e  # Esce immediatamente se un comando fallisce

echo "🚀 Avvio del server HapGree..."

echo "📋 Step 1: Impostazione variabili di ambiente database..."
# Esegui lo script per impostare DATABASE_URL
. ./Scripts/SetDbEnv.sh

if [ -z "$DATABASE_URL" ]; then
    echo "❌ Errore: DATABASE_URL non è stata impostata correttamente"
    exit 1
fi

echo "✅ DATABASE_URL configurata correttamente"

echo "📋 Step 2: Esecuzione migrazioni Prisma..."
# Applica le migrazioni esistenti come spiegato nel capitolo [Migrazioni](./Migrazioni.md)
npx prisma migrate deploy

echo "📋 Step 3: Avvio del server Node.js..."
# Avvia il server
node build/generated/index.js
```
Lo script si assicura che le migrazioni del database siano applicate prima di avviare l'applicazione.

---

## Conclusione

L'utilizzo di un `Dockerfile` multi-stage offre notevoli vantaggi:
- **Automazione**: Il processo di generazione del codice è completamente automatizzato e integrato nella build dell'immagine Docker.
- **Affidabilità**: Si evitano problemi legati a versioni differenti dei generatori o a configurazioni locali non allineate, garantendo che la build sia riproducibile su qualsiasi macchina.
- **Efficienza**: Si produce un'immagine finale ottimizzata e di dimensioni ridotte, contenente solo ciò che è strettamente necessario per l'esecuzione.
