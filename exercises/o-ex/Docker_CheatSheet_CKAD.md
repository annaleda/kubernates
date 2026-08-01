- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md) |  [ Exercises ](./oci.md)
--- 

## Docker Cheat Sheet - CKAD

> Cheatsheet pratica per l'esame CKAD, con particolare attenzione ai
> comandi Docker più richiesti.

# Docker Commands Cheat Sheet

## Informazioni

| Comando              | Cosa fa                                                                                                               |
| -------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `docker version`     | Mostra la versione del client Docker e del Docker Engine.                                                             |
| `docker info`        | Mostra informazioni generali sull'installazione Docker: container, immagini, storage driver, rete, runtime e risorse. |
| `docker system info` | **Comando non valido.** Usare `docker info`.                                                                          |
| `docker system df`   | Mostra lo spazio occupato da immagini, container, volumi e cache di build.                                            |

---

## Images

| Comando                  | Cosa fa                                                               |
| ------------------------ | --------------------------------------------------------------------- |
| `docker search nginx`    | Cerca immagini chiamate `nginx` su Docker Hub.                        |
| `docker pull nginx`      | Scarica l'immagine `nginx` con il tag predefinito `latest`.           |
| `docker pull nginx:1.27` | Scarica la versione `1.27` dell'immagine nginx.                       |
| `docker images`          | Mostra le immagini presenti localmente.                               |
| `docker image ls`        | Equivalente a `docker images`.                                        |
| `docker inspect nginx`   | Mostra tutti i dettagli JSON dell'immagine `nginx`.                   |
| `docker rmi nginx`       | Rimuove localmente l'immagine `nginx`.                                |
| `docker image rm nginx`  | Equivalente a `docker rmi nginx`.                                     |
| `docker image prune`     | Elimina le immagini dangling, cioè immagini senza tag non utilizzate. |
| `docker image prune -a`  | Elimina tutte le immagini non utilizzate da alcun container.          |

---

## Build

| Comando                                      | Cosa fa                                                                                    |
| -------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `docker build -t myapp .`                    | Costruisce un'immagine dal Dockerfile nella directory corrente e la chiama `myapp:latest`. |
| `docker build -t myapp:v1 .`                 | Costruisce l'immagine assegnando nome `myapp` e tag `v1`.                                  |
| `docker build -f Dockerfile.prod -t myapp .` | Costruisce l'immagine utilizzando il file `Dockerfile.prod`.                               |

Il punto finale:

```bash
.
```

indica che la directory corrente è il build context.

---

## Tag

| Comando                            | Cosa fa                                                                                  |
| ---------------------------------- | ---------------------------------------------------------------------------------------- |
| `docker tag myapp myrepo/myapp:v1` | Crea un nuovo tag per l'immagine `myapp`, generalmente prima del push verso un registry. |
| `docker tag nginx nginx:test`      | Crea il tag locale `nginx:test` partendo dall'immagine nginx esistente.                  |

Il comando `docker tag` non duplica realmente i dati dell'immagine: crea un nuovo riferimento alla stessa immagine.

---

## Push / Pull

| Comando                       | Cosa fa                                                            |
| ----------------------------- | ------------------------------------------------------------------ |
| `docker login`                | Effettua l'autenticazione a Docker Hub o a un registry.            |
| `docker push myrepo/myapp:v1` | Carica l'immagine nel registry configurato nel nome dell'immagine. |
| `docker pull myrepo/myapp:v1` | Scarica l'immagine dal registry.                                   |
| `docker logout`               | Rimuove le credenziali della sessione Docker.                      |

Per fare il push, il nome deve normalmente contenere il repository:

```bash
docker tag myapp:v1 annaleda/myapp:v1
docker push annaleda/myapp:v1
```

---

## Run

| Comando                                | Cosa fa                                                                              |
| -------------------------------------- | ------------------------------------------------------------------------------------ |
| `docker run nginx`                     | Crea e avvia un container dall'immagine nginx in foreground.                         |
| `docker run -it ubuntu bash`           | Avvia Ubuntu in modalità interattiva con una shell Bash.                             |
| `docker run -d nginx`                  | Avvia nginx in background, in modalità detached.                                     |
| `docker run --name web nginx`          | Crea un container chiamato `web`.                                                    |
| `docker run -p 8080:80 nginx`          | Pubblica la porta `80` del container sulla porta `8080` dell'host.                   |
| `docker run -e ENV=prod nginx`         | Imposta la variabile d'ambiente `ENV=prod` nel container.                            |
| `docker run -v /host:/container nginx` | Monta la directory `/host` dell'host nella directory `/container` del container.     |
| `docker run --restart always nginx`    | Riavvia automaticamente il container quando termina o quando Docker viene riavviato. |

### Opzioni principali di `docker run`

```text
-d            background
-it           modalità interattiva con terminale
--name        nome del container
-p            pubblicazione porta
-e            variabile d'ambiente
-v            volume o bind mount
--rm          elimina il container quando termina
--restart     politica di riavvio
```

---

## Container

| Comando                    | Cosa fa                                             |
| -------------------------- | --------------------------------------------------- |
| `docker ps`                | Mostra i container attualmente in esecuzione.       |
| `docker ps -a`             | Mostra tutti i container, inclusi quelli arrestati. |
| `docker start container`   | Avvia un container già esistente e fermo.           |
| `docker stop container`    | Arresta il container in modo controllato.           |
| `docker restart container` | Riavvia il container.                               |
| `docker kill container`    | Termina immediatamente il container.                |
| `docker rm container`      | Elimina un container fermo.                         |
| `docker rm -f container`   | Forza l'arresto e la rimozione del container.       |

`docker stop` invia prima un segnale di terminazione controllata.

`docker kill` interrompe il container immediatamente.

---

## Logs

| Comando                            | Cosa fa                               |
| ---------------------------------- | ------------------------------------- |
| `docker logs container`            | Mostra i log prodotti dal container.  |
| `docker logs -f container`         | Segue i log in tempo reale.           |
| `docker logs --tail 100 container` | Mostra solamente le ultime 100 righe. |

Esempio combinato:

```bash
docker logs -f --tail 50 container
```

Mostra le ultime 50 righe e continua a seguire i nuovi log.

---

## Exec

| Comando                          | Cosa fa                                                               |
| -------------------------------- | --------------------------------------------------------------------- |
| `docker exec -it container bash` | Apre una shell Bash interattiva dentro un container in esecuzione.    |
| `docker exec -it container sh`   | Apre una shell `sh`, utile per immagini Alpine o minimali.            |
| `docker exec container ls /`     | Esegue `ls /` dentro il container senza aprire una shell interattiva. |

Il container deve essere in esecuzione.

Esempio:

```bash
docker exec container printenv APP_ENV
```

Mostra la variabile `APP_ENV` presente nel container.

---

## Inspect

| Comando                                                          | Cosa fa                                                   |
| ---------------------------------------------------------------- | --------------------------------------------------------- |
| `docker inspect container`                                       | Mostra i dettagli completi del container in formato JSON. |
| `docker inspect image`                                           | Mostra i dettagli completi dell'immagine.                 |
| `docker inspect -f '{{ .NetworkSettings.IPAddress }}' container` | Estrae solamente l'indirizzo IP del container.            |

Altri esempi:

```bash
docker inspect -f '{{ .State.Status }}' container
```

Mostra lo stato del container.

```bash
docker inspect -f '{{ .Config.Image }}' container
```

Mostra l'immagine utilizzata dal container.

---

## Copy

| Comando                             | Cosa fa                                                          |
| ----------------------------------- | ---------------------------------------------------------------- |
| `docker cp file.txt container:/tmp` | Copia `file.txt` dall'host nella directory `/tmp` del container. |
| `docker cp container:/tmp/file .`   | Copia un file dal container nella directory corrente dell'host.  |

Il container può essere attivo oppure fermo.

---

## Stats / Top / Events

| Comando                  | Cosa fa                                                                       |
| ------------------------ | ----------------------------------------------------------------------------- |
| `docker stats`           | Mostra in tempo reale CPU, memoria, rete e I/O di tutti i container attivi.   |
| `docker stats container` | Mostra le statistiche di un solo container.                                   |
| `docker top container`   | Mostra i processi in esecuzione dentro il container.                          |
| `docker events`          | Mostra in tempo reale gli eventi Docker: start, stop, create, delete e altri. |

Uscire da `docker stats` o `docker events` con:

```text
Ctrl+C
```

---

## Networks

| Comando                                     | Cosa fa                                                |
| ------------------------------------------- | ------------------------------------------------------ |
| `docker network ls`                         | Mostra le reti Docker esistenti.                       |
| `docker network create mynet`               | Crea una nuova rete chiamata `mynet`.                  |
| `docker network inspect mynet`              | Mostra configurazione e container collegati alla rete. |
| `docker network connect mynet container`    | Collega un container esistente alla rete `mynet`.      |
| `docker network disconnect mynet container` | Scollega il container dalla rete.                      |
| `docker network rm mynet`                   | Elimina la rete.                                       |

Esempio:

```bash
docker network create mynet

docker run -d \
  --name web \
  --network mynet \
  nginx
```

I container collegati alla stessa rete Docker possono comunicare usando il nome del container.

---

## Volumes

| Comando                      | Cosa fa                                     |
| ---------------------------- | ------------------------------------------- |
| `docker volume ls`           | Mostra i volumi Docker esistenti.           |
| `docker volume create data`  | Crea un volume chiamato `data`.             |
| `docker volume inspect data` | Mostra i dettagli e il percorso del volume. |
| `docker volume rm data`      | Elimina il volume se non è utilizzato.      |
| `docker volume prune`        | Elimina tutti i volumi non utilizzati.      |

Utilizzo di un volume:

```bash
docker run -d \
  --name web \
  -v data:/usr/share/nginx/html \
  nginx
```

I dati del volume sopravvivono alla rimozione del container.

---

## Save / Load

| Comando                          | Cosa fa                                                               |
| -------------------------------- | --------------------------------------------------------------------- |
| `docker save -o nginx.tar nginx` | Salva un'immagine Docker, inclusi tag e layer, in un archivio `.tar`. |
| `docker load -i nginx.tar`       | Carica nuovamente un'immagine creata con `docker save`.               |

Esempio:

```bash
docker save -o myapp.tar myapp:v1
docker load -i myapp.tar
```

Utilizzare `save` e `load` per trasferire immagini Docker mantenendo la loro struttura originale.

---

## Export / Import

| Comando                                   | Cosa fa                                                     |
| ----------------------------------------- | ----------------------------------------------------------- |
| `docker export container > container.tar` | Esporta il filesystem corrente di un container.             |
| `docker import container.tar myimage`     | Crea una nuova immagine a partire dal filesystem esportato. |

Esempio:

```bash
docker export web > web.tar
docker import web.tar web-image:v1
```

`export` non mantiene la cronologia, i layer, i tag o buona parte della configurazione originale dell'immagine.

---

# Differenza tra `save` ed `export`

| `docker save`                         | `docker export`                             |
| ------------------------------------- | ------------------------------------------- |
| Salva un'immagine                     | Salva il filesystem di un container         |
| Mantiene i layer                      | Perde i layer                               |
| Mantiene tag e metadata dell'immagine | Non mantiene la cronologia dell'immagine    |
| Si usa con `docker load`              | Si usa con `docker import`                  |
| Ideale per trasferire immagini        | Ideale per creare uno snapshot semplificato |

Schema:

```text
IMAGE
  |
  ├── docker save
  |
  └── docker load
```

```text
CONTAINER
  |
  ├── docker export
  |
  └── docker import
```

---

## History / Diff / Commit

``` bash
docker history nginx
docker diff container
docker commit container myimage:v1
```

## Pulizia

``` bash
docker system prune
docker system prune -a
```

## Dockerfile

``` dockerfile
COPY app.py /app
ADD file.tar.gz /app
WORKDIR /app
ENV APP=prod
CMD ["python","app.py"]
ENTRYPOINT ["python"]
EXPOSE 8080
USER app
VOLUME /data
```

## Docker Compose

``` bash
docker compose up
docker compose up -d
docker compose down
docker compose ps
docker compose logs
docker compose restart
```

# I 20 comandi da ricordare

``` bash
docker build -t app:v1 .
docker images
docker ps
docker ps -a
docker run
docker run -d
docker exec -it container sh
docker logs container
docker stop container
docker start container
docker restart container
docker rm container
docker rmi image
docker tag app repo/app:v1
docker pull nginx
docker push repo/app:v1
docker save -o app.tar app:v1
docker load -i app.tar
docker export container > container.tar
docker import container.tar newimage
```

## Esempio CKAD

``` bash
docker build -t myapp:1.2 .
docker save -o /tmp/myapp.tar myapp:1.2
```

Oppure con Podman:

``` bash
podman build -t myapp:1.2 .
podman save -o /tmp/myapp.tar myapp:1.2
```
