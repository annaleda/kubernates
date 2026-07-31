- [ Home ](../../readme.md) | [ Teoria ](../../arguments.md) | [ Info Exam ](../../doc/ckad_exam_strategy.md) | [ Home Other Exercises ](../o_exercises.md) | [ Teory EP ](../../doc/tep.md)

---

### Service senza Selector e EndpointSlice (11 esercizi)

---

## ES-1 — Service senza selector base

- Service: `external-api`
- Configurazione
- Creare un Service senza `selector`
- Porta Service: `8080`
- `targetPort`: `8080`
- Validazione
- Il Service esiste
- Il manifest non contiene `selector`

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc external-api --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=external-api --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
```

```sh
k apply -f external-api-svc.yaml
k get svc external-api
k get svc external-api -o yaml
```

</details>

<details>
<summary>Verifica</summary>

```sh
k get svc external-api
k get endpointslice -l kubernetes.io/service-name=external-api
```

```sh
k get svc external-api -o yaml
```

</details>

---

## ES-2 — EndpointSlice per Service senza selector

- Service esistente: `external-api`
- EndpointSlice: `external-api-1`
- Configurazione
- Collegare il Service all'IP `10.0.0.50`
- Porta: `8080`
- `addressType`: `IPv4`
- Validazione
- L'EndpointSlice esiste
- La label collega l'EndpointSlice al Service
- La porta non è nominata, come quella del Service
 
<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc external-api --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=external-api --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

```
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-api-1
  labels:
    kubernetes.io/service-name: external-api
addressType: IPv4
ports:
  - protocol: TCP
    port: 8080
endpoints:
  - addresses:
      - "10.0.0.50"
```

</details>

<details>
<summary>Verifica</summary>

```sh
k get svc external-api
k get endpointslice -l kubernetes.io/service-name=external-api
```

```sh
k run temp --rm -it --image=busybox:1.36 --restart=Never -- wget -T 3 -qO- http://external-api:8080
```

</details>

---

## ES-3 — Service database esterno

- Service: `external-db`
- EndpointSlice: `external-db-1`
- Configurazione
- Creare un Service senza selector
- Porta Service: `5432`
- Endpoint esterno: `192.168.1.100`
- Porta endpoint: `5432`
<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc external-db --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=external-db --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

```
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  ports:
  - port: 5432
    targetPort: 5432
    protocol: TCP
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-db-1
  labels:
    kubernetes.io/service-name: external-db
addressType: IPv4
ports:
- port: 5432
  protocol: TCP
endpoints:
- addresses:
  - "192.168.1.100"
  conditions:
    ready: true
```
</details>

<details>
<summary>Verifica</summary>

```sh
k get svc external-db
k get endpointslice -l kubernetes.io/service-name=external-db
```

```sh
k run temp --rm -it --image=busybox:1.36 --restart=Never -- nc -vz external-db 5432
```

</details>

---

## ES-4 — EndpointSlice con due backend

- Service: `web-external`
- EndpointSlice: `web-external-1`
- Backend:
  - `192.168.1.10:80`
  - `192.168.1.11:80`

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc web-external --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=web-external --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

```
apiVersion: v1
kind: Service
metadata:
  name: web-external
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: web-external-1
  labels:
    kubernetes.io/service-name: web-external
addressType: IPv4
ports:
- protocol: TCP
  port: 80
endpoints:
- addresses:
  - "192.168.1.10"
  conditions:
    ready: true
- addresses:
  - "192.168.1.11"
  conditions:
    ready: true
```

</details>

<details>
<summary>Verifica</summary>

```sh
k get svc web-external
k get endpointslice -l kubernetes.io/service-name=web-external
```

```sh
k run temp --rm -it --image=busybox:1.36 --restart=Never -- wget -S -O- http://web-external
```

</details>

---

## ES-5 — Namespace dedicato

- Namespace: `external-services`
- Service: `external-app`
- EndpointSlice: `external-app-1`
  
<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc external-app --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=external-app --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

```
apiVersion: v1
kind: Namespace
metadata:
  name: external-services
---
apiVersion: v1
kind: Service
metadata:
  name: external-app
  namespace: external-services
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-app-1
  namespace: external-services
  labels:
    kubernetes.io/service-name: external-app
addressType: IPv4
ports:
- port: 8080
  protocol: TCP
endpoints:
- addresses:
  - "10.0.0.60"
  conditions:
    ready: true
```
    
</details>

<details>
<summary>Verifica</summary>

```sh
k get svc external-app
k get endpointslice -l kubernetes.io/service-name=external-app
```

```sh
k run temp -n external-services --rm -it --image=busybox:1.36 --restart=Never -- wget -T 3 -qO- http://external-app:8080
```

</details>

---

## ES-6 — Verifica EndpointSlice

- Service: `external-db`
- Verificare l'associazione tra Service ed EndpointSlice


<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc external-db --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=external-db --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

```sh
k get svc external-db
k get endpointslice -l kubernetes.io/service-name=external-db
```

</details>

<details>
<summary>Verifica</summary>

```sh
k get svc external-db
k get endpointslice -l kubernetes.io/service-name=external-db
```

```sh
k describe svc external-db
k describe endpointslice external-db-1
```

</details>

---

## ES-7 — Test DNS dal Pod

- Pod: `dns-test`
- Image: `busybox`
- Verificare il DNS del Service `external-api`

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc external-api --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=external-api --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

```
k run dns-test \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- nslookup external-api

k run dns-test \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- nslookup external-api.default.svc.cluster.local
```
</details>

<details>
<summary>Verifica</summary>

```sh
k get svc external-api
k get endpointslice -l kubernetes.io/service-name=external-api
```

```sh
k run dns-test --rm -it --image=busybox:1.36 --restart=Never -- nslookup external-api
```

</details>

---

## ES-8 — Correggere label errata

* Service: `payment-api`
* EndpointSlice: `payment-api-1`
* Problema: label errata nell'EndpointSlice

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc payment-api --ignore-not-found
k delete endpointslice payment-api-1 --ignore-not-found
```

Creare il Service senza selector:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment-api
spec:
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
```

Creare l'EndpointSlice con una label intenzionalmente errata:

```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: payment-api-1
  labels:
    kubernetes.io/service-name: payment-service
addressType: IPv4
ports:
- name: http
  protocol: TCP
  port: 8080
endpoints:
- addresses:
  - "10.0.0.70"
  conditions:
    ready: true
```

Applicare le risorse:

```sh
k apply -f payment-api-svc.yaml
k apply -f payment-api-eps.yaml
```

Verificare lo stato iniziale:

```sh
k get svc payment-api
k get endpointslice payment-api-1 --show-labels
```

La label errata deve risultare:

```text
kubernetes.io/service-name=payment-service
```

Verificare inoltre che il filtro basato sul nome corretto del Service non restituisca alcuna EndpointSlice:

```sh
k get endpointslice \
  -l kubernetes.io/service-name=payment-api
```

</details>

<details>
<summary>Soluzione</summary>

Individuare la label errata:

```sh
k get endpointslice payment-api-1 --show-labels
```

Correggerla:

```sh
k patch endpointslice payment-api-1 \
  --type=merge \
  -p '{"metadata":{"labels":{"kubernetes.io/service-name":"payment-api"}}}'
```

In alternativa, modificare direttamente la risorsa:

```sh
k edit endpointslice payment-api-1
```

La sezione corretta deve essere:

```yaml
metadata:
  labels:
    kubernetes.io/service-name: payment-api
```

</details>

<details>
<summary>Verifica</summary>

Controllare la nuova label:

```sh
k get endpointslice payment-api-1 --show-labels
```

Output atteso:

```text
kubernetes.io/service-name=payment-api
```

Verificare che la EndpointSlice venga trovata tramite il nome del Service:

```sh
k get endpointslice \
  -l kubernetes.io/service-name=payment-api
```

Controllare i dettagli:

```sh
k describe svc payment-api
k describe endpointslice payment-api-1
```

Testare il DNS del Service:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- nslookup payment-api
```

Testare la connessione HTTP:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- wget -T 3 -qO- http://payment-api
```

Il test HTTP funzionerà solo se esiste davvero un backend raggiungibile su:

```text
10.0.0.70:8080
```

Se il backend è dimostrativo, la validazione principale consiste nel verificare che la label sia corretta e che la EndpointSlice venga associata al Service.

</details>

---


## ES-9 — Convertire Endpoints deprecated

Convertire una risorsa `Endpoints` in `EndpointSlice`.

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc cache-external --ignore-not-found
k delete endpoints cache-external --ignore-not-found
k delete endpointslice cache-external-1 --ignore-not-found
```

Creare il Service senza selector:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cache-external
spec:
  ports:
  - port: 6379
    targetPort: 6379
    protocol: TCP
```

Creare la vecchia risorsa `Endpoints`:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: cache-external
subsets:
- addresses:
  - ip: 172.16.0.10
  ports:
  - port: 6379
    protocol: TCP
```

Applicare entrambe le risorse:

```sh
k apply -f cache-external-svc.yaml
k apply -f cache-external-endpoints.yaml
```

Verificare lo stato iniziale:

```sh
k get svc cache-external
k get endpoints cache-external
k get endpointslice cache-external-1
```

Il Service e la risorsa `Endpoints` devono esistere.

L'EndpointSlice `cache-external-1` non deve esistere.

Visualizzare il contenuto da convertire:

```sh
k get endpoints cache-external -o yaml
```

> Su Kubernetes 1.33+ il comando può mostrare un warning di deprecazione. È il comportamento previsto dall'esercizio.

</details>

<details>
<summary>Soluzione</summary>

Creare un nuovo manifest `EndpointSlice` usando le informazioni contenute nella vecchia risorsa `Endpoints`:

```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: cache-external-1
  labels:
    kubernetes.io/service-name: cache-external
addressType: IPv4
ports:
- port: 6379
  protocol: TCP
endpoints:
- addresses:
  - "172.16.0.10"
  conditions:
    ready: true
```

Applicare:

```sh
k apply -f cache-external-eps.yaml
```

Verificare che la nuova EndpointSlice esista:

```sh
k get endpointslice cache-external-1
```

Dopo aver verificato la conversione, eliminare la vecchia risorsa:

```sh
k delete endpoints cache-external
```

</details>

<details>
<summary>Verifica</summary>

Controllare che la vecchia risorsa non esista più:

```sh
k get endpoints cache-external
```

Output atteso:

```text
Error from server (NotFound)
```

Controllare la nuova EndpointSlice:

```sh
k get endpointslice cache-external-1
```

Verificare la label di associazione:

```sh
k get endpointslice cache-external-1 --show-labels
```

Output atteso:

```text
kubernetes.io/service-name=cache-external
```

Verificare IP e porta:

```sh
k get endpointslice cache-external-1 \
  -o custom-columns='NAME:.metadata.name,IP:.endpoints[0].addresses[0],PORT:.ports[0].port,READY:.endpoints[0].conditions.ready'
```

Output atteso:

```text
NAME                 IP            PORT   READY
cache-external-1     172.16.0.10   6379   true
```

Verificare il DNS del Service:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- nslookup cache-external
```

Verificare la connessione TCP:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- nc -vz -w 3 cache-external 6379
```

Il test TCP funzionerà solo se esiste davvero un backend raggiungibile su:

```text
172.16.0.10:6379
```

Se l'indirizzo è dimostrativo, la verifica principale consiste nella corretta conversione della struttura e nell'associazione tra Service ed EndpointSlice.

</details>


---

## ES-10 — Porta nominata

* Service: `metrics-external`
* EndpointSlice: `metrics-external-1`
* Porta nominata: `metrics`
* Porta: `9090`
* Backend: `10.99.0.15`

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc metrics-external --ignore-not-found
k delete endpointslice metrics-external-1 --ignore-not-found
```

Verificare che le risorse non esistano:

```sh
k get svc metrics-external
k get endpointslice metrics-external-1
```

Entrambi i comandi devono restituire `NotFound`.

Il candidato deve creare:

```text
Service:          metrics-external
EndpointSlice:    metrics-external-1
Nome porta:       metrics
Porta Service:    9090
Porta backend:    9090
Backend IP:       10.99.0.15
```

</details>

<details>
<summary>Soluzione</summary>

Creare Service ed EndpointSlice:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: metrics-external
spec:
  ports:
  - name: metrics
    port: 9090
    targetPort: 9090
    protocol: TCP
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: metrics-external-1
  labels:
    kubernetes.io/service-name: metrics-external
addressType: IPv4
ports:
- name: metrics
  port: 9090
  protocol: TCP
endpoints:
- addresses:
  - "10.99.0.15"
  conditions:
    ready: true
```

Applicare:

```sh
k apply -f metrics-external.yaml
```

</details>

<details>
<summary>Verifica</summary>

Verificare le risorse:

```sh
k get svc metrics-external
k get endpointslice \
  -l kubernetes.io/service-name=metrics-external
```

Controllare i manifest:

```sh
k get svc metrics-external -o yaml
k get endpointslice metrics-external-1 -o yaml
```

Verificare che il nome della porta coincida:

```sh
k get svc metrics-external \
  -o jsonpath='{.spec.ports[0].name}{"\n"}'

k get endpointslice metrics-external-1 \
  -o jsonpath='{.ports[0].name}{"\n"}'
```

Output atteso per entrambi:

```text
metrics
```

Verificare IP, porta e stato:

```sh
k get endpointslice metrics-external-1 \
  -o custom-columns='NAME:.metadata.name,PORT-NAME:.ports[0].name,PORT:.ports[0].port,IP:.endpoints[0].addresses[0],READY:.endpoints[0].conditions.ready'
```

Output atteso:

```text
NAME                   PORT-NAME   PORT   IP           READY
metrics-external-1     metrics     9090   10.99.0.15   true
```

Verificare il DNS del Service:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- nslookup metrics-external
```

Verificare la connessione TCP:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- nc -vz -w 3 metrics-external 9090
```

Il test TCP funzionerà solo se esiste realmente un backend raggiungibile su:

```text
10.99.0.15:9090
```

Se l'indirizzo è dimostrativo, la validazione principale consiste nel verificare:

* la label `kubernetes.io/service-name`;
* la corrispondenza del nome porta `metrics`;
* la porta `9090`;
* l'indirizzo backend corretto.

</details>

---

## ES-11 — Backend esterno individuato con `hostname -I`

* Service: `external-nginx`
* EndpointSlice: `external-nginx-1`
* nginx gira sullo `student-node` sulla porta `9999`
* L'IP dello `student-node` non è indicato
* Individuare l'IP con `hostname -I`

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse Kubernetes precedenti:

```sh
k delete pod temp --ignore-not-found
k delete svc external-nginx --ignore-not-found
k delete endpointslice external-nginx-1 --ignore-not-found
```

Verificare che nginx sia installato sullo `student-node`:

```sh
nginx -v
```

Se nginx non è presente:

```sh
sudo apt-get update
sudo apt-get install -y nginx
```

Creare una configurazione nginx in ascolto sulla porta `9999`:

```sh
sudo tee /etc/nginx/sites-available/ckad-external-nginx \
  >/dev/null <<'EOF'
server {
    listen 9999;
    listen [::]:9999;

    server_name _;

    location / {
        default_type text/html;
        return 200 '<!DOCTYPE html>
<html>
<head>
    <title>CKAD External nginx</title>
</head>
<body>
    <h1>Welcome to nginx!</h1>
    <p>External backend for the EndpointSlice exercise.</p>
</body>
</html>';
    }
}
EOF
```

Abilitare la configurazione:

```sh
sudo ln -sf \
  /etc/nginx/sites-available/ckad-external-nginx \
  /etc/nginx/sites-enabled/ckad-external-nginx
```

Verificare e riavviare nginx:

```sh
sudo nginx -t
sudo systemctl restart nginx
```

Se `systemctl` non è disponibile:

```sh
sudo nginx -s reload 2>/dev/null || sudo nginx
```

Controllare che nginx ascolti sulla porta `9999`:

```sh
ss -lntp | grep 9999
```

Verificare localmente il backend:

```sh
curl -v http://localhost:9999
```

Creare il Service senza selector:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-nginx
spec:
  ports:
  - port: 80
    targetPort: 9999
    protocol: TCP
```

Applicare:

```sh
k apply -f external-nginx-svc.yaml
```

Verificare lo stato iniziale:

```sh
k get svc external-nginx

k get endpointslice \
  -l kubernetes.io/service-name=external-nginx
```

Il Service deve esistere, ma non deve essere presente alcun EndpointSlice associato.

Il seguente test deve inizialmente fallire:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- wget -T 3 -qO- http://external-nginx
```

</details>

<details>
<summary>Soluzione</summary>

Individuare l'indirizzo IP dello `student-node`:

```sh
hostname -I
```

Output di esempio:

```text
10.244.151.201
```

Se vengono visualizzati più indirizzi, scegliere quello appartenente alla rete raggiungibile dai nodi Kubernetes.

Verificare direttamente che nginx risponda sull'indirizzo individuato:

```sh
curl -v http://10.244.151.201:9999
```

Il risultato deve contenere:

```text
HTTP/1.1 200 OK
Server: nginx
```

Creare l'EndpointSlice utilizzando l'IP trovato:

```yaml
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: external-nginx-1
  labels:
    kubernetes.io/service-name: external-nginx
addressType: IPv4
ports:
- protocol: TCP
  port: 9999
endpoints:
- addresses:
  - "10.244.151.201"
  conditions:
    ready: true
```

Applicare:

```sh
k apply -f external-nginx-eps.yaml
```

> Sostituire `10.244.151.201` con l'indirizzo realmente restituito da `hostname -I`.

La porta del Service non è nominata; di conseguenza anche la porta dell'EndpointSlice deve essere priva del campo `name`.

</details>

<details>
<summary>Verifica</summary>

Verificare il Service:

```sh
k get svc external-nginx
```

Verificare l'EndpointSlice associato:

```sh
k get endpointslice \
  -l kubernetes.io/service-name=external-nginx
```

Controllare label, IP, porta e stato:

```sh
k get endpointslice external-nginx-1 \
  -o custom-columns='NAME:.metadata.name,SERVICE:.metadata.labels.kubernetes\.io/service-name,IP:.endpoints[0].addresses[0],PORT:.ports[0].port,READY:.endpoints[0].conditions.ready'
```

Output atteso:

```text
NAME                 SERVICE          IP               PORT   READY
external-nginx-1     external-nginx   10.244.151.201   9999   true
```

Verificare nuovamente il backend diretto:

```sh
hostname -I
curl http://<STUDENT-NODE-IP>:9999
```

Verificare il DNS del Service da un Pod temporaneo:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- nslookup external-nginx
```

Verificare la connessione attraverso il Service:

```sh
k run temp \
  --rm -i \
  --image=busybox:1.36 \
  --restart=Never \
  -- wget -qO- http://external-nginx
```

Output atteso:

```html
<h1>Welcome to nginx!</h1>
```

Il flusso del traffico è:

```text
Pod temporaneo
      |
      v
external-nginx:80
      |
      v
EndpointSlice external-nginx-1
      |
      v
<STUDENT-NODE-IP>:9999
      |
      v
nginx
```

</details>


---
