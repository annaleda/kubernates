# 📘 CKAD – Strategia Esame + Cheatsheet

---

## 1️⃣ Informazioni sull’esame

La certificazione **CKAD (Certified Kubernetes Application Developer)** è rilasciata dalla Cloud Native Computing Foundation (CNCF).

Caratteristiche principali:

* ⏱️ Durata: 2 ore
* 💻 100% pratico (hands-on)
* 🌍 Online e proctored
* 📊 ~15–20 task
* ✅ Passing score: 66%
* 📅 Validità: 2 anni

L’esame si svolge su cluster Kubernetes reali.

---

## 2️⃣ Setup iniziale consigliato

Appena inizi l’esame:

```bash
alias k=kubectl
source <(kubectl completion bash)
```

---

# 📘 CKAD – Cheatsheet Rapido (Tabellare)

| Risorsa            | Comando rapido                                                                 | Uso / Note                                |
|-------------------|-------------------------------------------------------------------------------|------------------------------------------|
| **Cluster/Context**| `kubectl config get-contexts` <br> `kubectl config current-context` <br> `kubectl config use-context <ctx>` | Verificare e cambiare contesto           |
| **Namespace**      | `kubectl get ns` <br> `kubectl config set-context --current --namespace=<ns>` <br> `kubectl get pods -n <ns>` | Impostare namespace corrente o usare `-n`|
| **Pod**            | `kubectl run nginx --image=nginx` <br> `kubectl describe pod <pod>` <br> `kubectl logs <pod>` <br> `kubectl exec -it <pod> -- sh` | Creazione e debug rapido                 |
| **Deployment**     | `kubectl create deployment web --image=nginx --replicas=3` <br> `kubectl edit deployment web` <br> `kubectl scale deployment web --replicas=5` <br> `kubectl set image deployment/web nginx=nginx:1.25` <br> `kubectl rollout status deployment/web` <br> `kubectl rollout undo deployment/web` | Gestione cicli di vita deployment        |
| **Service**        | `kubectl expose deployment web --port=80 --type=ClusterIP` <br> `kubectl expose deployment web --port=80 --type=NodePort` <br> `kubectl create service clusterip my-cs --tcp=5678:8080 --dry-run=client -o yaml` | Esposizione e test servizi               |
| **ConfigMap**      | `kubectl create configmap my-config --from-literal=key=value` <br> `kubectl create configmap my-config --from-env-file=file.env` | Configurazioni chiave/valore             |
| **Secret**         | `kubectl create secret generic my-secret --from-literal=key=value` <br> `kubectl create secret generic my-secret --from-env-file=file.env` | Gestione segreti                          |
| **Job / CronJob**  | `kubectl create job my-job --image=busybox -- date` <br> `kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *"` | Job singolo o pianificato                 |
| **ServiceAccount** | `kubectl create sa my-app-sa`                                                 | Account per risorse                       |
| **Role / ClusterRole** | `kubectl create role pod-reader --verb=get,list,watch --resource=pods` <br> `kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods` | Gestione permessi                         |
| **RoleBinding / ClusterRoleBinding** | `kubectl create rolebinding admin-binding --clusterrole=admin --user=user1 --user=user2 --group=group1` <br> `kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=user1` | Associa permessi utenti/gruppi           |
| **Debug / Info**   | `kubectl get all` <br> `kubectl describe <resource>` <br> `kubectl logs <pod>` <br> `kubectl exec -it <pod> -- sh` <br> `kubectl top pod` <br> `kubectl top node` <br> `kubectl explain <resource>` | Stato cluster, risorse, metriche         |
| **Dry-Run / YAML** | `kubectl create <resource> ... --dry-run=client -o yaml > file.yaml` <br> `kubectl apply -f file.yaml` | Genera e applica YAML rapidamente        |

---

## ✅ Regola d’oro CKAD

> Se puoi farlo in un comando, fallo in un comando.  
> Se serve YAML, genera con `--dry-run=client -o yaml` e applica.

