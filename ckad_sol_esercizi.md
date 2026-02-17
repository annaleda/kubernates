# CKAD Soluzioni Esercizi

*(Questa pagina contiene tutte le soluzioni agli esercizi sopra elencati, esempi YAML e comandi pronti da eseguire.)*

Esempi:

## Core Concepts

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
kubectl scale deployment nginx --replicas=5
kubectl create ns dev
kubectl run nginx-dev --image=nginx -n dev
```

## Configuration

```bash
kubectl create configmap my-config --from-literal=key=value
kubectl create secret generic my-secret --from-literal=password=abc123
```

## Multi-Container Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  initContainers:
  - name: init-myfile
    image: busybox
    command: ['sh', '-c', 'echo hello > /data/message']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'tail -f /dev/null']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

## Services & Networking

```bash
kubectl expose deployment nginx --type=ClusterIP --port=80
kubectl expose deployment nginx --type=NodePort --port=80
```

