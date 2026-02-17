
# CKAD Multi-Container Pods

## Teoria

* **Sidecar pattern**: container secondario che estende il principale.
* **Init containers**: eseguiti prima dei container principali.
* **Adapter pattern**: trasforma output per compatibilità.

## Esercizi

1. Creare un pod con nginx + sidecar busybox.
2. Aggiungere un initContainer che crea file prima dell’avvio.
3. Condividere volume emptyDir tra container.

---



