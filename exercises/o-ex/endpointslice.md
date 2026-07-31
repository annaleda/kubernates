- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)    | [ Home Other Exercises ](../o_exercises.md)   
--- 
## ES-1 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse create in precedenza:

```sh
k delete svc external-api --ignore-not-found
```

Verificare che il Service non esista:

```sh
k get svc external-api
```

Il comando deve restituire un errore `NotFound`.

L'esercizio parte senza risorse preesistenti: il candidato deve creare il Service `external-api` senza selector.

</details>

---

## ES-2 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse precedenti:

```sh
k delete endpointslice external-api-1 --ignore-not-found
k delete svc external-api --ignore-not-found
```

Creare il Service senza selector:

```sh
cat <<'EOF' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
EOF
```

Verificare:

```sh
k get svc external-api
k get endpointslice \
  -l kubernetes.io/service-name=external-api
```

Il Service deve esistere, mentre non deve essere presente alcun EndpointSlice associato.

Nota: l'indirizzo `10.0.0.50` è utilizzato per esercitarsi nella creazione del manifest. La connettività funzionerà solamente se esiste realmente un backend raggiungibile su `10.0.0.50:8080`.

</details>

---

## ES-3 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse precedenti:

```sh
k delete endpointslice external-db-1 --ignore-not-found
k delete svc external-db --ignore-not-found
```

Verificare che le risorse non esistano:

```sh
k get svc external-db
k get endpointslice external-db-1
```

Entrambi i comandi devono restituire `NotFound`.

Il candidato deve creare:

* il Service `external-db`;
* l'EndpointSlice `external-db-1`;
* la porta nominata `postgres`;
* il collegamento al backend `192.168.1.100:5432`.

Nota: l'indirizzo `192.168.1.100` è dimostrativo. Non è necessario che il database sia realmente raggiungibile per verificare la struttura delle risorse.

</details>

---

## ES-4 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse precedenti:

```sh
k delete endpointslice web-external-1 --ignore-not-found
k delete svc web-external --ignore-not-found
```

Verificare che non esistano:

```sh
k get svc web-external
k get endpointslice web-external-1
```

Il candidato deve creare un Service senza selector e un EndpointSlice contenente entrambi gli indirizzi:

```text
192.168.1.10
192.168.1.11
```

Gli indirizzi sono dimostrativi. L'obiettivo dell'esercizio è verificare la presenza di più backend nello stesso EndpointSlice.

</details>

---

## ES-5 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere l'eventuale namespace creato in precedenza:

```sh
k delete ns external-services --ignore-not-found
```

Attendere che venga eliminato:

```sh
k wait \
  --for=delete namespace/external-services \
  --timeout=60s 2>/dev/null || true
```

Verificare:

```sh
k get ns external-services
```

Il comando deve restituire `NotFound`.

Il candidato deve:

1. creare il namespace `external-services`;
2. creare al suo interno un Service senza selector;
3. creare nello stesso namespace il relativo EndpointSlice.

Per rendere l'esercizio completo, utilizzare queste caratteristiche:

```text
Service:       external-app
EndpointSlice: external-app-1
Service port:  8080
Backend IP:    10.0.0.60
Backend port:  8080
```

Le due risorse devono trovarsi entrambe nel namespace `external-services`.

</details>

---

## ES-6 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse precedenti:

```sh
k delete endpointslice external-db-1 --ignore-not-found
k delete svc external-db --ignore-not-found
```

Creare il Service e l'EndpointSlice da analizzare:

```sh
cat <<'EOF' | k apply -f -
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
EOF
```

Verificare che entrambe le risorse esistano:

```sh
k get svc external-db
k get endpointslice external-db-1
```

Il candidato deve individuare l'associazione usando la label:

```text
kubernetes.io/service-name=external-db
```

</details>

---

## ES-7 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse precedenti:

```sh
k delete pod dns-test --ignore-not-found
k delete endpointslice external-api-1 --ignore-not-found
k delete svc external-api --ignore-not-found
```

Creare il Service senza selector:

```sh
cat <<'EOF' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
EOF
```

Verificare che il Service disponga di un ClusterIP:

```sh
k get svc external-api
```

Non è necessario creare un EndpointSlice, perché l'esercizio verifica solamente la risoluzione DNS del nome del Service.

Il candidato deve verificare dal Pod `dns-test` che il nome:

```text
external-api
```

venga risolto nel ClusterIP del Service.

</details>

---

## ES-8 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse precedenti:

```sh
k delete endpointslice payment-api-1 --ignore-not-found
k delete svc payment-api --ignore-not-found
```

Creare il Service:

```sh
cat <<'EOF' | k apply -f -
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
EOF
```

Creare un EndpointSlice con una label intenzionalmente errata:

```sh
cat <<'EOF' | k apply -f -
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
EOF
```

Controllare la label errata:

```sh
k get endpointslice payment-api-1 --show-labels
```

L'output deve mostrare:

```text
kubernetes.io/service-name=payment-service
```

Il candidato deve correggerla in:

```text
kubernetes.io/service-name=payment-api
```

</details>

---

## ES-9 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse precedenti:

```sh
k delete endpointslice cache-external-1 --ignore-not-found
k delete endpoints cache-external --ignore-not-found
k delete svc cache-external --ignore-not-found
```

Creare il Service senza selector:

```sh
cat <<'EOF' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: cache-external
spec:
  ports:
  - port: 6379
    targetPort: 6379
    protocol: TCP
EOF
```

Creare la vecchia risorsa `Endpoints` da convertire:

```sh
cat <<'EOF' | k apply -f -
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
EOF
```

Verificare:

```sh
k get svc cache-external
k get endpoints cache-external -o yaml
k get endpointslice cache-external-1
```

Il Service e la risorsa `Endpoints` devono esistere, mentre l'EndpointSlice `cache-external-1` non deve essere presente.

Il candidato deve convertire manualmente le informazioni in una risorsa:

```text
discovery.k8s.io/v1
kind: EndpointSlice
```

Dopo la conversione può rimuovere la vecchia risorsa:

```sh
k delete endpoints cache-external
```

</details>

---

## ES-10 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Rimuovere eventuali risorse precedenti:

```sh
k delete endpointslice metrics-external-1 --ignore-not-found
k delete svc metrics-external --ignore-not-found
```

Verificare che le risorse non esistano:

```sh
k get svc metrics-external
k get endpointslice metrics-external-1
```

Il candidato deve creare:

```text
Service:          metrics-external
EndpointSlice:    metrics-external-1
Nome porta:       metrics
Porta Service:    9090
Porta backend:    9090
Backend IP:       10.99.0.15
```

Il nome `metrics` deve essere identico nel Service e nell'EndpointSlice.

Nota: `10.99.0.15` è un indirizzo dimostrativo. La validazione principale riguarda il manifest e la corrispondenza del nome della porta.

</details>

---

## ES-11 — Preparazione ambiente

<details>
<summary>Preparazione ambiente</summary>

Questo esercizio richiede un server nginx in esecuzione sullo `student-node` e raggiungibile sulla porta `9999`.

### 1. Pulizia delle risorse Kubernetes

```sh
k delete pod temp --ignore-not-found
k delete endpointslice external-nginx-1 --ignore-not-found
k delete svc external-nginx --ignore-not-found
```

### 2. Installazione di nginx

Controllare se nginx è già installato:

```sh
nginx -v
```

Se non è presente:

```sh
sudo apt-get update
sudo apt-get install -y nginx
```

### 3. Configurazione della porta `9999`

Creare una configurazione dedicata:

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

Abilitare il sito:

```sh
sudo ln -sf \
  /etc/nginx/sites-available/ckad-external-nginx \
  /etc/nginx/sites-enabled/ckad-external-nginx
```

Verificare e ricaricare nginx:

```sh
sudo nginx -t
sudo systemctl restart nginx
```

Se il sistema non utilizza `systemd`:

```sh
sudo nginx -s reload 2>/dev/null || sudo nginx
```

### 4. Verifica della porta

```sh
ss -lntp | grep 9999
```

L'output deve indicare che nginx ascolta sulla porta `9999`, per esempio:

```text
LISTEN 0 511 0.0.0.0:9999
LISTEN 0 511 [::]:9999
```

Verificare localmente:

```sh
curl -v http://localhost:9999
```

### 5. Creazione del Service

Creare solamente il Service, senza selector e senza EndpointSlice:

```sh
cat <<'EOF' | k apply -f -
apiVersion: v1
kind: Service
metadata:
  name: external-nginx
spec:
  ports:
  - port: 80
    targetPort: 9999
    protocol: TCP
EOF
```

Verificare:

```sh
k get svc external-nginx
k get endpointslice \
  -l kubernetes.io/service-name=external-nginx
```

Il Service deve esistere, ma non deve essere associato ad alcun EndpointSlice.

### 6. Stato iniziale atteso

Il seguente test deve fallire:

```sh
k run temp \
  --rm -i \
  --image=busybox \
  --restart=Never \
  -- wget -T 3 -qO- http://external-nginx
```

Il candidato deve:

1. trovare l'indirizzo IP dello `student-node` con:

   ```sh
   hostname -I
   ```

2. verificare il backend:

   ```sh
   curl http://<STUDENT-NODE-IP>:9999
   ```

3. creare l'EndpointSlice `external-nginx-1`;

4. collegarlo al Service mediante la label:

   ```text
   kubernetes.io/service-name=external-nginx
   ```

5. utilizzare la porta backend `9999`;

6. verificare il collegamento dal Pod.

</details>

---
