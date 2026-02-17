# CKAD Core Concepts

## Teoria

* **Pod**: unità minima deployabile, può contenere uno o più container.
* **ReplicaSet**: garantisce un numero definito di pod in esecuzione.
* **Deployment**: gestisce ReplicaSet e aggiornamenti.
* **Namespace**: isolamento logico delle risorse.
* **Labels & Selectors**: raggruppamento e selezione dei pod.
* **Annotations**: metadati extra.

## Esercizi

1. Creare un pod nginx via YAML.
2. Creare un deployment con 3 repliche.
3. Scalare il deployment a 5 repliche.
4. Creare un namespace "dev" e deployare un pod dentro di esso.

---


