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

> Inserire qui il manifest della soluzione dell'esercizio ES-6.

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

> Inserire qui il manifest della soluzione dell'esercizio ES-7.

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

- Service: `payment-api`
- EndpointSlice: `payment-api-1`
- Problema: label errata nell'EndpointSlice

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc payment-api --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=payment-api --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

> Inserire qui il manifest della soluzione dell'esercizio ES-8.

</details>

<details>
<summary>Verifica</summary>

```sh
k get svc payment-api
k get endpointslice -l kubernetes.io/service-name=payment-api
```

```sh
k get endpointslice payment-api-1 --show-labels
```

</details>

---

## ES-9 — Convertire Endpoints deprecated

Convertire una risorsa `Endpoints` in `EndpointSlice`.

<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc cache-external --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=cache-external --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

> Inserire qui il manifest della soluzione dell'esercizio ES-9.

</details>

<details>
<summary>Verifica</summary>

```sh
k get svc cache-external
k get endpointslice -l kubernetes.io/service-name=cache-external
```

```sh
k get endpoints
k get endpointslice
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

Pulire eventuali risorse precedenti:

```sh
k delete svc metrics-external --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=metrics-external --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

> Inserire qui il manifest della soluzione dell'esercizio ES-10.

</details>

<details>
<summary>Verifica</summary>

```sh
k get svc metrics-external
k get endpointslice -l kubernetes.io/service-name=metrics-external
```

```sh
k get svc metrics-external -o yaml
k get endpointslice metrics-external-1 -o yaml
```

</details>

---

## ES-11 — Backend esterno individuato con hostname -I


- Service: `external-nginx`
- EndpointSlice: `external-nginx-1`
- nginx gira sullo `student-node` sulla porta `9999`
- L'IP dello `student-node` non è indicato
- Individuare l'IP con `hostname -I`

  
<details>
<summary>Preparazione ambiente</summary>

Pulire eventuali risorse precedenti:

```sh
k delete svc external-nginx --ignore-not-found
k delete endpointslice -l kubernetes.io/service-name=external-nginx --ignore-not-found
```

Preparare l'ambiente come descritto nell'esercizio.

</details>

<details>
<summary>Soluzione</summary>

> Inserire qui il manifest della soluzione dell'esercizio ES-11.

</details>

<details>
<summary>Verifica</summary>

```sh
k get svc external-nginx
k get endpointslice -l kubernetes.io/service-name=external-nginx
```

```sh
hostname -I
curl http://<STUDENT-NODE-IP>:9999
k run temp --rm -it --image=busybox:1.36 --restart=Never -- wget -qO- http://external-nginx
```

</details>

---
