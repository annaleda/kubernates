- [ Home ](../../readme.md) | [ Teoria ](../../arguments.md) | [ Info Exam ](../../doc/ckad_exam_strategy.md) | [ Home Other Exercises ](../o_exercises.md)

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

```sh
k delete svc external-api --ignore-not-found
k get svc external-api
```

Il Service non deve esistere. Il candidato deve crearlo senza selector.

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

```sh
k delete endpointslice external-api-1 --ignore-not-found
k delete svc external-api --ignore-not-found

cat <<'MANIFEST' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
MANIFEST

k get svc external-api
k get endpointslice -l kubernetes.io/service-name=external-api
```

Il Service deve esistere senza EndpointSlice associati. `10.0.0.50` è un indirizzo dimostrativo.

</details>

<details>
<summary>Soluzione</summary>

```yaml
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
  - 10.0.0.50
```

```sh
k apply -f external-api-eps.yaml
k get endpointslice external-api-1
k get endpointslice -l kubernetes.io/service-name=external-api
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

```sh
k delete endpointslice external-db-1 --ignore-not-found
k delete svc external-db --ignore-not-found
k get svc external-db
k get endpointslice external-db-1
```

Le risorse non devono esistere. Il candidato deve creare Service ed EndpointSlice con porta nominata `postgres`.

</details>

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  ports:
  - name: postgres
    port: 5432
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
- name: postgres
  protocol: TCP
  port: 5432
endpoints:
- addresses:
  - 192.168.1.100
```

```sh
k apply -f external-db.yaml
k get svc external-db
k get endpointslice -l kubernetes.io/service-name=external-db
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

```sh
k delete endpointslice web-external-1 --ignore-not-found
k delete svc web-external --ignore-not-found
k get svc web-external
k get endpointslice web-external-1
```

Le risorse non devono esistere. Gli indirizzi dei backend sono dimostrativi.

</details>

<details>
<summary>Soluzione</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-external
spec:
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: web-external-1
  labels:
    kubernetes.io/service-name: web-external
addressType: IPv4
ports:
- name: http
  protocol: TCP
  port: 80
endpoints:
- addresses:
  - 192.168.1.10
- addresses:
  - 192.168.1.11
```

```sh
k apply -f web-external.yaml
k describe endpointslice web-external-1
```

</details>

---

## ES-5 — Namespace dedicato

- Namespace: `external-services`
- Service: `external-app`
- EndpointSlice: `external-app-1`

<details>
<summary>Preparazione ambiente</summary>

```sh
k delete ns external-services --ignore-not-found
k wait --for=delete namespace/external-services --timeout=60s 2>/dev/null || true
k get ns external-services
```

Il namespace non deve esistere. Usare Service port `8080`, backend `10.0.0.60:8080`.

</details>

<details>
<summary>Soluzione</summary>

```sh
k create ns external-services
```

```yaml
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
- protocol: TCP
  port: 8080
endpoints:
- addresses:
  - 10.0.0.60
```

```sh
k apply -f external-app.yaml
k get svc,endpointslice -n external-services
```

</details>

---

## ES-6 — Verifica EndpointSlice

- Service: `external-db`
- Verificare l'associazione tra Service ed EndpointSlice

<details>
<summary>Preparazione ambiente</summary>

```sh
k delete endpointslice external-db-1 --ignore-not-found
k delete svc external-db --ignore-not-found

cat <<'MANIFEST' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  ports:
  - name: postgres
    port: 5432
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
- name: postgres
  protocol: TCP
  port: 5432
endpoints:
- addresses:
  - 192.168.1.100
  conditions:
    ready: true
MANIFEST
```

</details>

<details>
<summary>Soluzione</summary>

```sh
k get endpointslice -l kubernetes.io/service-name=external-db
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

```sh
k delete pod dns-test --ignore-not-found
k delete endpointslice external-api-1 --ignore-not-found
k delete svc external-api --ignore-not-found

cat <<'MANIFEST' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
MANIFEST

k get svc external-api
```

Non serve un EndpointSlice: l'esercizio verifica solo la risoluzione DNS.

</details>

<details>
<summary>Soluzione</summary>

```sh
k run dns-test --image=busybox --restart=Never -it --rm -- nslookup external-api
```

</details>

---

## ES-8 — Correggere label errata

- Service: `payment-api`
- EndpointSlice: `payment-api-1`
- Problema: label errata nell'EndpointSlice

<details>
<summary>Preparazione ambiente</summary>

```sh
k delete endpointslice payment-api-1 --ignore-not-found
k delete svc payment-api --ignore-not-found

cat <<'MANIFEST' | k apply -f -
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
---
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
  - 10.0.0.70
  conditions:
    ready: true
MANIFEST

k get endpointslice payment-api-1 --show-labels
```

La label errata iniziale è `kubernetes.io/service-name=payment-service`.

</details>

<details>
<summary>Soluzione</summary>

```sh
k patch endpointslice payment-api-1 \
  --type merge \
  -p '{"metadata":{"labels":{"kubernetes.io/service-name":"payment-api"}}}'

k get endpointslice -l kubernetes.io/service-name=payment-api
```

</details>

---

## ES-9 — Convertire Endpoints deprecato

Convertire una risorsa `Endpoints` in `EndpointSlice`.

<details>
<summary>Preparazione ambiente</summary>

```sh
k delete endpointslice cache-external-1 --ignore-not-found
k delete endpoints cache-external --ignore-not-found
k delete svc cache-external --ignore-not-found

cat <<'MANIFEST' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: cache-external
spec:
  ports:
  - port: 6379
    targetPort: 6379
    protocol: TCP
---
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
MANIFEST

k get svc cache-external
k get endpoints cache-external -o yaml
```

</details>

<details>
<summary>Soluzione</summary>

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
  - 172.16.0.10
```

```sh
k apply -f cache-external-eps.yaml
k get endpointslice cache-external-1
k delete endpoints cache-external
```

</details>

---

## ES-10 — Porta nominata

- Service: `metrics-external`
- EndpointSlice: `metrics-external-1`
- Porta nominata: `metrics`
- Porta: `9090`
- Backend: `10.99.0.15`

<details>
<summary>Preparazione ambiente</summary>

```sh
k delete endpointslice metrics-external-1 --ignore-not-found
k delete svc metrics-external --ignore-not-found
k get svc metrics-external
k get endpointslice metrics-external-1
```

Le risorse non devono esistere. Il nome `metrics` deve coincidere nel Service e nell'EndpointSlice.

</details>

<details>
<summary>Soluzione</summary>

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
  - 10.99.0.15
```

```sh
k apply -f metrics-external.yaml
k get svc metrics-external
k get endpointslice metrics-external-1
```

</details>

---

## ES-11 — Backend esterno individuato con `hostname -I`

- Service: `external-nginx`
- EndpointSlice: `external-nginx-1`
- nginx gira sullo `student-node` sulla porta `9999`
- L'IP dello `student-node` non è indicato
- Individuare l'IP con `hostname -I`

<details>
<summary>Preparazione ambiente</summary>

Pulire le risorse Kubernetes:

```sh
k delete pod temp --ignore-not-found
k delete endpointslice external-nginx-1 --ignore-not-found
k delete svc external-nginx --ignore-not-found
```

Installare nginx, se necessario:

```sh
nginx -v || {
  sudo apt-get update
  sudo apt-get install -y nginx
}
```

Configurare nginx sulla porta `9999`:

```sh
sudo tee /etc/nginx/sites-available/ckad-external-nginx >/dev/null <<'NGINX'
server {
    listen 9999;
    listen [::]:9999;
    server_name _;

    location / {
        default_type text/html;
        return 200 '<!DOCTYPE html><html><body><h1>Welcome to nginx!</h1><p>External backend for the EndpointSlice exercise.</p></body></html>';
    }
}
NGINX

sudo ln -sf \
  /etc/nginx/sites-available/ckad-external-nginx \
  /etc/nginx/sites-enabled/ckad-external-nginx

sudo nginx -t
sudo systemctl restart nginx 2>/dev/null || sudo nginx -s reload 2>/dev/null || sudo nginx
```

Verificare il backend:

```sh
ss -lntp | grep 9999
curl -v http://localhost:9999
```

Creare soltanto il Service:

```sh
cat <<'MANIFEST' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-nginx
spec:
  ports:
  - port: 80
    targetPort: 9999
    protocol: TCP
MANIFEST
```

Verificare che non esista ancora alcun EndpointSlice associato:

```sh
k get svc external-nginx
k get endpointslice -l kubernetes.io/service-name=external-nginx
```

Il test iniziale deve fallire:

```sh
k run temp --rm -i --image=busybox --restart=Never -- \
  wget -T 3 -qO- http://external-nginx
```

</details>

<details>
<summary>Soluzione</summary>

Trovare l'IP dello `student-node`:

```sh
hostname -I
```

Verificare quale indirizzo risponde sulla porta `9999`:

```sh
curl -v http://<STUDENT-NODE-IP>:9999
```

Creare l'EndpointSlice sostituendo il placeholder:

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
  - <STUDENT-NODE-IP>
  conditions:
    ready: true
```

```sh
k apply -f external-nginx-eps.yaml
```

La porta del Service non ha un nome; anche la porta dell'EndpointSlice deve essere senza `name`.

Verificare:

```sh
k get endpointslice -l kubernetes.io/service-name=external-nginx

k get endpointslice external-nginx-1 \
  -o custom-columns='NAME:.metadata.name,IP:.endpoints[0].addresses[0],PORT:.ports[0].port,READY:.endpoints[0].conditions.ready'
```

Testare il Service da un Pod:

```sh
k run temp --rm -i --image=busybox --restart=Never -- \
  wget -qO- http://external-nginx
```

Flusso:

```text
Pod -> external-nginx:80 -> EndpointSlice -> <STUDENT-NODE-IP>:9999 -> nginx
```

</details>

---
