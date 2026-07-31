- [ Home ](../../readme.md) | [ Teoria ](../../arguments.md) | [ Info Exam ](../../doc/ckad_exam_strategy.md) | [ Home Other Exercises ](../o_exercises.md)

---

### Service senza Selector e EndpointSlice (11 esercizi)

---

 ## ES-1 — Service senza selector base

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

  > Inserire qui il manifest della soluzione dell'esercizio ES-1.

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

  > Inserire qui il manifest della soluzione dell'esercizio ES-2.

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

  > Inserire qui il manifest della soluzione dell'esercizio ES-3.

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

  > Inserire qui il manifest della soluzione dell'esercizio ES-4.

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

  > Inserire qui il manifest della soluzione dell'esercizio ES-5.

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
