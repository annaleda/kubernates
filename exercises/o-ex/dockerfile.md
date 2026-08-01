### Dockerfile Cheat Sheet 

## Dockerfile completo di esempio

```dockerfile
FROM alpine:3.21

ENV APP_ENV=production
ENV APP_PORT=8080

WORKDIR /app

COPY app.sh .

RUN chmod +x app.sh

EXPOSE 8080

CMD ["./app.sh"]
```

Build:

```bash
docker build -t myapp:1.0 .
```

Run:

```bash
docker run --rm myapp:1.0
```

---

## `FROM`

Definisce l'immagine di partenza.

```dockerfile
FROM alpine:3.21
```

Esempi comuni:

```dockerfile
FROM nginx:1.27-alpine
FROM ubuntu:24.04
FROM python:3.13-slim
```

---

## `ENV`

Imposta variabili d'ambiente disponibili nel container.

```dockerfile
ENV APP_ENV=production
ENV APP_PORT=8080
```

Verifica:

```bash
docker run --rm myapp:1.0 printenv APP_ENV
```

---

## `WORKDIR`

Imposta la directory di lavoro.

```dockerfile
WORKDIR /app
```

Le istruzioni successive useranno `/app` come directory corrente.

---

## `COPY`

Copia file dal computer nell'immagine.

```dockerfile
COPY app.sh .
```

Con `WORKDIR /app`, il file viene copiato in:

```text
/app/app.sh
```

Altri esempi:

```dockerfile
COPY index.html /usr/share/nginx/html/
COPY config/ /app/config/
COPY . .
```

---

## `RUN`

Esegue comandi durante la build.

```dockerfile
RUN chmod +x app.sh
```

Installazione pacchetti su Alpine:

```dockerfile
RUN apk add --no-cache curl
```

Su Ubuntu:

```dockerfile
RUN apt-get update && apt-get install -y curl
```

`RUN` viene eseguito durante `docker build`, non quando il container parte.

---

## `EXPOSE`

Documenta la porta utilizzata dall'applicazione.

```dockerfile
EXPOSE 8080
```

Non pubblica realmente la porta.

Per pubblicarla:

```bash
docker run -p 8080:8080 myapp:1.0
```

---

## `CMD`

Definisce il comando predefinito eseguito all'avvio del container.

```dockerfile
CMD ["./app.sh"]
```

Altro esempio:

```dockerfile
CMD ["sleep", "3600"]
```

Se nel Dockerfile sono presenti più istruzioni `CMD`, viene usata solo l'ultima.

---

## `ENTRYPOINT`

Definisce il programma principale del container.

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

Il comando finale sarà:

```text
python app.py
```

Differenza pratica:

```dockerfile
CMD ["sleep", "3600"]
```

può essere completamente sostituito:

```bash
docker run myimage echo hello
```

Con:

```dockerfile
ENTRYPOINT ["sleep"]
CMD ["3600"]
```

questo comando:

```bash
docker run myimage 10
```

esegue:

```text
sleep 10
```

---

## `ARG`

Definisce una variabile disponibile durante la build.

```dockerfile
ARG APP_VERSION=1.0
```

Build:

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  -t myapp:2.0 .
```

Differenza:

```text
ARG = build
ENV = container
```

---

## `USER`

Imposta l'utente che esegue il processo nel container.

```dockerfile
USER 1000
```

Oppure:

```dockerfile
USER appuser
```

È preferibile evitare di eseguire applicazioni come `root`.

---

# Esempio nginx

```dockerfile
FROM nginx:1.27-alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Build:

```bash
docker build -t custom-nginx:1.0 .
```

Run:

```bash
docker run \
  -d \
  --name custom-nginx \
  -p 8080:80 \
  custom-nginx:1.0
```

Verifica:

```bash
curl http://localhost:8080
```

---

# Esempio Alpine

File `app.sh`:

```sh
#!/bin/sh

echo "APP_ENV=$APP_ENV"
sleep 3600
```

Dockerfile:

```dockerfile
FROM alpine:3.21

ENV APP_ENV=production

WORKDIR /app

COPY app.sh .

RUN chmod +x app.sh

CMD ["./app.sh"]
```

Build:

```bash
docker build -t alpine-app:1.0 .
```

Run:

```bash
docker run \
  -d \
  --name alpine-app \
  alpine-app:1.0
```

Verifica:

```bash
docker logs alpine-app
docker exec alpine-app printenv APP_ENV
```

---

# Comandi principali

Build:

```bash
docker build -t myapp:1.0 .
```

Elenco immagini:

```bash
docker images
```

Avvio container:

```bash
docker run -d --name myapp myapp:1.0
```

Porta:

```bash
docker run -d -p 8080:80 myapp:1.0
```

Variabile:

```bash
docker run -d -e APP_ENV=test myapp:1.0
```

Container attivi:

```bash
docker ps
```

Tutti i container:

```bash
docker ps -a
```

Log:

```bash
docker logs myapp
```

Comando nel container:

```bash
docker exec myapp printenv APP_ENV
```

Shell:

```bash
docker exec -it myapp sh
```

Stop e rimozione:

```bash
docker stop myapp
docker rm myapp
```

---

# Ordine consigliato

```dockerfile
FROM
ARG
ENV
WORKDIR
COPY
RUN
EXPOSE
USER
ENTRYPOINT
CMD
```

---

# Errori comuni

* Dimenticare `FROM`.
* Usare un percorso errato in `COPY`.
* Confondere `RUN` con `CMD`.
* Dimenticare i permessi di esecuzione dello script.
* Esporre una porta diversa da quella usata dall'applicazione.
* Usare più di un `CMD`.
* Avviare un processo che termina subito.
* Dimenticare il punto finale nel comando:

```bash
docker build -t myapp:1.0 .
```

---

# Checklist rapida

```text
□ FROM corretto
□ WORKDIR impostata
□ COPY usa il percorso giusto
□ RUN installa o prepara i file
□ ENV contiene le variabili richieste
□ EXPOSE indica la porta corretta
□ CMD o ENTRYPOINT avviano il processo
□ docker build termina senza errori
□ docker run mantiene il container attivo
□ docker logs mostra l'output atteso
```
