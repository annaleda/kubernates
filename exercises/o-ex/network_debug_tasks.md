- [ Home ](../../readme.md)   | [ Teoria ](../../arguments.md)   | [ Info Exam ](../../doc/ckad_exam_strategy.md)   | [ Home Other Exercises ](../o_exercises.md)

---

### CKAD Network Debug Tasks (20 esercizi)

---

## NET-1 — Pod temporaneo BusyBox

SECTION: APPLICATION OBSERVABILITY AND MAINTENANCE

- Obiettivo
  - Creare un Pod temporaneo BusyBox

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run tmp --rm -it --restart=Never --image=busybox:1.36 -- sh
```
usa winpty se su git bash
```sh
winpty kubectl run po-temp --rm -it --restart=Never --image=busybox -- sh
```
</details>

---

## NET-2 — Pod temporaneo Alpine

- Obiettivo
  - Avviare un Pod Alpine interattivo

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run alpine --rm -it --restart=Never --image=alpine -- sh
```
usa winpty se su git bash
```sh
winpty kubectl run alpine --rm -it --restart=Never --image=alpine -- sh
```
</details>

---

## NET-3 — Verificare DNS con nslookup

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --target-port=80
```

Attendi che il Pod sia `Running`:

```sh
kubectl get pods
```
Verifica che il Service `nginx` sia risolvibile tramite il DNS del cluster utilizzando un Pod temporaneo BusyBox.

- Service: nginx

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run dns-test --rm -it --restart=Never --image=busybox:1.36 -- nslookup nginx
```

</details>

---

## NET-4 — nslookup FQDN

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --target-port=80
```

Attendi che il Pod sia `Running`:

```sh
kubectl get pods
```

---

### Task

Verifica che il Service `nginx` sia raggiungibile utilizzando il suo Fully Qualified Domain Name (FQDN).

---
- Namespace: default
- Service: nginx

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run dns-test --rm -it --restart=Never --image=busybox:1.36 -- nslookup nginx.default.svc.cluster.local
```

</details>

---

## NET-5 — wget verso Service

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx  --port=80 --target-port=80
```

Attendi che il Pod sia `Running`:

```sh
kubectl get pods
```

---

### Task

Verifica che il Service `nginx` risponda a una richiesta HTTP utilizzando il suo nome DNS.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- wget -qO- http://nginx
```

</details>

---

## NET-6 — wget verso ClusterIP

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --target-port=80
```

Attendi che il Pod sia `Running`:

```sh
kubectl get pods
```

Recupera il ClusterIP del Service:

```sh
kubectl get svc nginx
```

---

### Task

Utilizza il ClusterIP del Service `nginx` per verificare che risponda a una richiesta HTTP.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- wget -qO- http://10.96.0.10
```

</details>

---

## NET-7 — wget su NodePort

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort
```

Visualizza la porta assegnata:

```sh
kubectl get svc nginx
```

---

### Task

Verifica che il Service sia raggiungibile utilizzando il Node IP e la NodePort.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- wget -qO- http://NODE_IP:NODEPORT
```
```sh
winpty kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- wget -qO- http://NODE_IP:NODEPORT
```

</details>

---

## NET-8 — Test connessione HTTPS

### Task

Verifica che un Pod possa stabilire una connessione HTTPS verso un sito esterno.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- wget https://kubernetes.io
```
```sh
winpty kubectl run wget --rm -it --restart=Never --image=busybox:1.36 -- wget https://kubernetes.io
```
</details>

---

## NET-9 — Test DNS esterno

### Task

Verifica che un Pod riesca a risolvere un nome DNS esterno.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run dns --rm -it --restart=Never --image=busybox:1.36 -- nslookup google.com
```

</details>

---

## NET-10 — Ping Pod

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl get pods -o wide
```

Annota l'indirizzo IP del Pod.

---

### Task

Verifica che il Pod sia raggiungibile tramite il suo indirizzo IP.

---


<details>
<summary>Soluzione</summary>

```sh
kubectl run test --rm -it --restart=Never --image=busybox:1.36 -- ping POD_IP
```
```sh
winpty kubectl run test --rm -it --restart=Never --image=busybox:1.36 -- ping POD_IP
```

</details>

---

## NET-11 — shell in Pod temporaneo

### Task

Avvia un Pod temporaneo utilizzando l'immagine `nicolaka/netshoot` ed entra nella shell.

---
<details>
<summary>Soluzione</summary>

```sh
kubectl run debug --rm -it --restart=Never --image=nicolaka/netshoot -- bash
```
```sh
winpty kubectl run debug --rm -it --restart=Never --image=nicolaka/netshoot -- bash
```
</details>

---

## NET-12 — Test endpoint HTTP

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80  --target-port=80
```

Attendi che il Pod sia `Running`.

---

### Task

Verifica che il Service `nginx` risponda a una richiesta HTTP utilizzando `curl`.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run curl --rm -it --restart=Never --image=curlimages/curl -- curl http://nginx
```
```sh
winpty kubectl run curl --rm -it --restart=Never --image=curlimages/curl -- curl http://nginx
```
</details>

---
---

## NET-13 — Recuperare la pagina index tramite il Service

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --target-port=80
```

Attendi che il Pod sia pronto:

```sh
kubectl wait --for=condition=Available deployment/nginx  --timeout=90s
```

---

### Task

Utilizza un Pod temporaneo per recuperare e visualizzare la pagina index esposta dal Service `nginx`.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run wget-test --rm -it --restart=Never  --image=busybox:1.36 -- wget -qO- http://nginx
```
```sh
winpty kubectl run wget-test --rm -it --restart=Never  --image=busybox:1.36 -- wget -qO- http://nginx
```

</details>

---

## NET-14 — Verificare la risoluzione DNS da un Pod

### Environment preparation

Crea un Pod BusyBox che rimanga in esecuzione:

```sh
kubectl run dns-client  --image=busybox:1.36  --restart=Never  -- sleep 3600
```
```sh
winpty kubectl run dns-client  --image=busybox:1.36  --restart=Never  -- sleep 3600
```
Attendi che il Pod sia pronto:

```sh
kubectl wait --for=condition=Ready pod/dns-client --timeout=60s
```

Il Service `kubernetes` esiste normalmente nella namespace `default`.

---

### Task

Dal Pod `dns-client`, verifica che il nome DNS `kubernetes.default` venga risolto correttamente.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl exec dns-client -- nslookup kubernetes.default
```

È possibile utilizzare anche il FQDN completo:

```sh
kubectl exec dns-client -- nslookup kubernetes.default.svc.cluster.local
```

</details>

---

## NET-15 — Verificare la configurazione DNS di un Pod

### Environment preparation

Se non esiste già, crea un Pod BusyBox che rimanga in esecuzione:

```sh
kubectl run dns-client  --image=busybox:1.36  --restart=Never  -- sleep 3600
```
```sh
winpty kubectl run dns-client  --image=busybox:1.36  --restart=Never  -- sleep 3600
```

Attendi che il Pod sia pronto:

```sh
kubectl wait --for=condition=Ready  pod/dns-client  --timeout=60s
```

---

### Task

Visualizza la configurazione DNS utilizzata dal Pod `dns-client`.

Identifica:

* il nameserver;
* i domini di ricerca;
* le opzioni del resolver.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl exec dns-client -- cat /etc/resolv.conf
```

Un output tipico può contenere:

```text
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

I valori possono variare in base alla configurazione del cluster.

</details>

---

## NET-16 — Creare un Pod di debug persistente

### Task

Crea un Pod chiamato `debug`:

* con immagine `busybox:1.36`;
* senza controller;
* configurato per rimanere in esecuzione per un'ora.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run debug --image=busybox:1.36 --restart=Never --  sleep 3600
```
```sh
winpty kubectl run debug --image=busybox:1.36 --restart=Never --  sleep 3600
```
Verifica che sia in esecuzione:

```sh
kubectl get pod debug
```

</details>

---

## NET-17 — Eliminare il Pod di debug

### Environment preparation

Crea il Pod, se non esiste già:

```sh
kubectl run debug --image=busybox:1.36 --restart=Never -- sleep 3600
```
```sh
winpty kubectl run debug --image=busybox:1.36 --restart=Never -- sleep 3600
```
---

### Task

Elimina il Pod `debug`.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl delete pod debug
```

</details>

---

## NET-18 — Verificare gli endpoint di un Service

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --target-port=80
```

Attendi che il Deployment sia disponibile:

```sh
kubectl wait --for=condition=Available deployment/nginx --timeout=90s
```

---

### Task

Verifica quali indirizzi IP e porte sono associati al Service `nginx`.

---

<details>
<summary>Soluzione</summary>

Con la risorsa Endpoints:

```sh
kubectl get endpoints nginx
```

Per maggiori dettagli:

```sh
kubectl describe endpoints nginx
```

È possibile controllare anche gli EndpointSlice:

```sh
kubectl get endpointslices -l kubernetes.io/service-name=nginx
```

</details>

---

## NET-19 — Verificare una connessione TCP

### Environment preparation

```sh
kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --target-port=80
```

Attendi che il Deployment sia disponibile:

```sh
kubectl wait --for=condition=Available deployment/nginx --timeout=90s
```

---

### Task

Utilizza un Pod temporaneo con strumenti di rete per verificare che la porta TCP `80` del Service `nginx` sia raggiungibile.

Non è necessario scaricare la pagina HTTP.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run netshoot --rm -it --restart=Never --image=nicolaka/netshoot -- nc -zv nginx 80
```
```sh
winpty kubectl run netshoot --rm -it --restart=Never --image=nicolaka/netshoot -- nc -zv nginx 80
```
Un risultato positivo dovrebbe indicare che la connessione alla porta `80` è riuscita.

</details>

---

## NET-20 — Avviare una shell con comando sovrascritto

### Task

Crea un Pod temporaneo chiamato `toolbox` utilizzando `busybox:1.36`.

Sovrascrivi il comando predefinito dell'immagine e avvia una shell interattiva. Il Pod deve essere eliminato automaticamente al termine della sessione.

---

<details>
<summary>Soluzione</summary>

```sh
kubectl run toolbox --rm -it --restart=Never --image=busybox:1.36 -- sh
```
usa winpty se su git bash
```sh
winpty kubectl run toolbox --rm -it --restart=Never --image=busybox:1.36 -- sh
```
Il doppio trattino:

```text
--
```

separa gli argomenti di `kubectl run` dal comando da eseguire nel container.

</details>

---

