# 📘 CKAD – Esercizi e Soluzioni con Alias / Shortcut

*(contiene esercizi CKAD principali e comandi shortcut.)*

---

## 🔹 1. Core Concepts

### 1.1 Pod Singolo
**Esercizio:** Crea un Pod `nginx-pod` con immagine nginx.

<details>
<summary>Soluzione</summary>

```bash
# Normale
kubectl run nginx-pod --image=nginx

# Shortcut
k run nginx-pod --image=nginx
```
