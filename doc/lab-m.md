# CKAD Volume Lab — Esercizio completo con soluzione

## Obiettivo

Creare un laboratorio Kubernetes che utilizzi:

- `emptyDir`
- `PersistentVolumeClaim`
- `ConfigMap`
- `Secret`
- `ServiceAccount`
- token esplicito tramite `projected`
- `DownwardAPI`
- `subPath`
- mount in sola lettura
- `items`
- `defaultMode`
- `optional`
- `initContainer`
- sidecar
- volume condiviso tra container
- dati persistenti

Tutte le risorse devono essere create nel namespace:

```text
volume-lab
```

---

# PARTE 1 — ESERCIZIO

## Scenario

Devi creare un Pod chiamato:

```text
volume-app
```

Il Pod deve contenere:

- un `initContainer` chiamato `prepare-content`
- un container `nginx`
- un container sidecar chiamato `sidecar`

Il Pod deve usare il ServiceAccount:

```text
volume-reader
```

Il token del ServiceAccount non deve essere montato automaticamente.

---

## 1. Namespace

Crea il namespace:

```text
volume-lab
```

---

## 2. ConfigMap

Crea una ConfigMap chiamata:

```text
web-config
```

Deve contenere:

```text
APP_MODE=production
APP_COLOR=blue
```

e un file chiamato:

```text
nginx-extra.conf
```

con contenuto:

```nginx
location /health {
    return 200 "healthy\n";
}
```

---

## 3. Secret

Crea un Secret generico chiamato:

```text
app-secret
```

Deve contenere:

```text
username=admin
password=ckad-pass
api-key=123456789
```

---

## 4. ServiceAccount

Crea un ServiceAccount chiamato:

```text
volume-reader
```

Il Pod deve usare:

```yaml
serviceAccountName: volume-reader
automountServiceAccountToken: false
```

---

## 5. PVC

Crea un PVC chiamato:

```text
app-data
```

Requisiti:

```text
storage: 100Mi
access mode: ReadWriteOnce
storageClassName: standard
```

Se nel cluster non esiste provisioning dinamico, crea anche un PV manuale compatibile.

---

## 6. emptyDir

Crea un volume `emptyDir` chiamato:

```text
runtime
```

Mount richiesti:

| Container | Percorso | Modalità |
|---|---|---|
| initContainer | `/work` | read-write |
| nginx | `/usr/share/nginx/html/runtime` | read-only |
| sidecar | `/runtime` | read-write |

Il sidecar deve scrivere ogni 10 secondi la data in:

```text
/runtime/status.log
```

---

## 7. PVC condiviso

Monta il PVC `app-data` tramite un volume chiamato:

```text
persistent-data
```

Mount richiesti:

| Container | Percorso | Modalità |
|---|---|---|
| initContainer | `/data` | read-write |
| nginx | `/usr/share/nginx/html/data` | read-only |
| sidecar | `/data` | read-write |

Il sidecar deve aggiungere la data al file:

```text
/data/persistent.log
```

---

## 8. ConfigMap come directory

Monta tutta la ConfigMap `web-config` nel container nginx su:

```text
/etc/app-config
```

Il mount deve essere read-only.

---

## 9. ConfigMap con items

Crea un secondo volume dalla ConfigMap `web-config`.

Deve contenere soltanto:

```text
APP_MODE
```

rinominato nel volume come:

```text
mode.txt
```

Montalo nel sidecar su:

```text
/config-selected
```

---

## 10. ConfigMap con subPath

Monta soltanto:

```text
nginx-extra.conf
```

nel container nginx come:

```text
/etc/nginx/conf.d/extra.conf
```

Usa `subPath` e `readOnly: true`.

---

## 11. Secret come directory

Monta tutto il Secret `app-secret` nel container nginx su:

```text
/etc/credentials
```

Usa permessi predefiniti:

```text
0400
```

Il mount deve essere read-only.

---

## 12. Secret con subPath

Monta solo la chiave:

```text
api-key
```

nel sidecar come:

```text
/var/run/app/api-key.txt
```

Usa `subPath` e `readOnly: true`.

---

## 13. Secret come variabile d'ambiente

Esponi nel sidecar:

```text
APP_USERNAME
```

usando la chiave `username` del Secret `app-secret`.

---

## 14. Downward API

Crea un volume chiamato:

```text
pod-metadata
```

Deve produrre i file:

| File | Origine |
|---|---|
| `pod-name` | `metadata.name` |
| `pod-namespace` | `metadata.namespace` |
| `pod-labels` | tutte le label |
| `cpu-request` | CPU request di nginx |
| `memory-limit` | memory limit di nginx |

Montalo nel sidecar su:

```text
/etc/podinfo
```

---

## 15. Projected volume

Crea un volume chiamato:

```text
projected-info
```

Deve combinare:

- ConfigMap
- Secret
- Downward API
- token del ServiceAccount

Struttura desiderata:

```text
/var/run/projected/config/app-mode
/var/run/projected/secret/api-key
/var/run/projected/metadata/name
/var/run/projected/token
```

Il token deve avere:

```text
expirationSeconds: 3600
audience: https://kubernetes.default.svc
```

Monta il volume nel sidecar come read-only.

---

## 16. ConfigMap opzionale

Crea un volume che faccia riferimento a una ConfigMap inesistente:

```text
future-config
```

Usa:

```yaml
optional: true
```

Montalo nel sidecar su:

```text
/optional
```

Il Pod deve partire anche se la ConfigMap non esiste.

---

## 17. InitContainer

L'initContainer deve usare:

```text
busybox:1.36
```

Deve leggere:

```text
/config/mode.txt
/secret/username
```

e creare:

```text
/work/index.html
```

con contenuto equivalente a:

```html
<h1>Volume Lab</h1>
<p>Mode: production</p>
<p>User: admin</p>
```

I valori non devono essere scritti direttamente nel comando.

---

## 18. nginx

Il container nginx deve:

- usare immagine `nginx`
- esporre porta `80`
- avere request CPU `50m`
- avere limit CPU `100m`
- avere request memoria `32Mi`
- avere limit memoria `64Mi`

---

## 19. Sidecar

Il sidecar deve:

- usare `busybox:1.36`
- scrivere ogni 10 secondi su `runtime/status.log`
- scrivere ogni 10 secondi su `persistent.log`
- restare sempre attivo

---

## 20. Service

Crea un Service chiamato:

```text
volume-app-service
```

Tipo:

```text
ClusterIP
```

Porta:

```text
80
```

Selector:

```yaml
app: volume-app
```

---

# PARTE 2 — SOLUZIONE

## File completo

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: volume-lab
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-config
  namespace: volume-lab
data:
  APP_MODE: production
  APP_COLOR: blue
  nginx-extra.conf: |
    location /health {
        return 200 "healthy\n";
    }
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: volume-lab
type: Opaque
stringData:
  username: admin
  password: ckad-pass
  api-key: "123456789"
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: volume-reader
  namespace: volume-lab
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
  namespace: volume-lab
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 100Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: volume-app
  namespace: volume-lab
  labels:
    app: volume-app
    environment: training
spec:
  serviceAccountName: volume-reader
  automountServiceAccountToken: false

  initContainers:
    - name: prepare-content
      image: busybox:1.36
      command:
        - sh
        - -c
      args:
        - |
          MODE="$(cat /config/mode.txt)"
          USERNAME="$(cat /secret/username)"

          cat > /work/index.html <<EOF
          <h1>Volume Lab</h1>
          <p>Mode: ${MODE}</p>
          <p>User: ${USERNAME}</p>
          EOF
      volumeMounts:
        - name: runtime
          mountPath: /work
        - name: persistent-data
          mountPath: /data
        - name: selected-config
          mountPath: /config
          readOnly: true
        - name: credentials
          mountPath: /secret
          readOnly: true

  containers:
    - name: nginx
      image: nginx
      ports:
        - name: http
          containerPort: 80
      resources:
        requests:
          cpu: 50m
          memory: 32Mi
        limits:
          cpu: 100m
          memory: 64Mi
      volumeMounts:
        - name: runtime
          mountPath: /usr/share/nginx/html/runtime
          readOnly: true

        - name: persistent-data
          mountPath: /usr/share/nginx/html/data
          readOnly: true

        - name: all-config
          mountPath: /etc/app-config
          readOnly: true

        - name: all-config
          mountPath: /etc/nginx/conf.d/extra.conf
          subPath: nginx-extra.conf
          readOnly: true

        - name: credentials
          mountPath: /etc/credentials
          readOnly: true

    - name: sidecar
      image: busybox:1.36
      command:
        - sh
        - -c
      args:
        - |
          while true; do
            date >> /runtime/status.log
            date >> /data/persistent.log
            sleep 10
          done
      env:
        - name: APP_USERNAME
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: username
      volumeMounts:
        - name: runtime
          mountPath: /runtime

        - name: persistent-data
          mountPath: /data

        - name: selected-config
          mountPath: /config-selected
          readOnly: true

        - name: credentials
          mountPath: /var/run/app/api-key.txt
          subPath: api-key
          readOnly: true

        - name: pod-metadata
          mountPath: /etc/podinfo
          readOnly: true

        - name: projected-info
          mountPath: /var/run/projected
          readOnly: true

        - name: optional-config
          mountPath: /optional
          readOnly: true

  volumes:
    - name: runtime
      emptyDir: {}

    - name: persistent-data
      persistentVolumeClaim:
        claimName: app-data

    - name: all-config
      configMap:
        name: web-config

    - name: selected-config
      configMap:
        name: web-config
        items:
          - key: APP_MODE
            path: mode.txt

    - name: credentials
      secret:
        secretName: app-secret
        defaultMode: 0400

    - name: pod-metadata
      downwardAPI:
        items:
          - path: pod-name
            fieldRef:
              fieldPath: metadata.name

          - path: pod-namespace
            fieldRef:
              fieldPath: metadata.namespace

          - path: pod-labels
            fieldRef:
              fieldPath: metadata.labels

          - path: cpu-request
            resourceFieldRef:
              containerName: nginx
              resource: requests.cpu

          - path: memory-limit
            resourceFieldRef:
              containerName: nginx
              resource: limits.memory

    - name: projected-info
      projected:
        defaultMode: 0400
        sources:
          - configMap:
              name: web-config
              items:
                - key: APP_MODE
                  path: config/app-mode

          - secret:
              name: app-secret
              items:
                - key: api-key
                  path: secret/api-key

          - downwardAPI:
              items:
                - path: metadata/name
                  fieldRef:
                    fieldPath: metadata.name

          - serviceAccountToken:
              path: token
              expirationSeconds: 3600
              audience: https://kubernetes.default.svc

    - name: optional-config
      configMap:
        name: future-config
        optional: true
---
apiVersion: v1
kind: Service
metadata:
  name: volume-app-service
  namespace: volume-lab
spec:
  type: ClusterIP
  selector:
    app: volume-app
  ports:
    - name: http
      port: 80
      targetPort: 80
```

---

# PV manuale opzionale

Usa questa sezione solo se il cluster non dispone di provisioning dinamico.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume-lab
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data/volume-lab
```

> `hostPath` è adatto a un laboratorio locale, non a un normale cluster di produzione.

---

# Applicazione

Salva la soluzione in:

```text
volume-lab.yaml
```

Poi applica:

```bash
kubectl apply -f volume-lab.yaml
```

Controlla:

```bash
kubectl get pod,pvc,cm,secret,sa,svc -n volume-lab
```

---

# Validazione

## PVC

```bash
kubectl get pvc -n volume-lab
```

Deve risultare:

```text
Bound
```

---

## ConfigMap completa

```bash
kubectl exec volume-app -n volume-lab -c nginx -- \
  ls -l /etc/app-config
```

```bash
kubectl exec volume-app -n volume-lab -c nginx -- \
  cat /etc/app-config/APP_MODE
```

---

## ConfigMap con subPath

```bash
kubectl exec volume-app -n volume-lab -c nginx -- \
  cat /etc/nginx/conf.d/extra.conf
```

---

## Secret

```bash
kubectl exec volume-app -n volume-lab -c nginx -- \
  ls -l /etc/credentials
```

```bash
kubectl exec volume-app -n volume-lab -c nginx -- \
  cat /etc/credentials/username
```

---

## Secret come env

```bash
kubectl exec volume-app -n volume-lab -c sidecar -- \
  printenv APP_USERNAME
```

Output atteso:

```text
admin
```

---

## Secret con subPath

```bash
kubectl exec volume-app -n volume-lab -c sidecar -- \
  cat /var/run/app/api-key.txt
```

---

## Downward API

```bash
kubectl exec volume-app -n volume-lab -c sidecar -- \
  cat /etc/podinfo/pod-name
```

```bash
kubectl exec volume-app -n volume-lab -c sidecar -- \
  cat /etc/podinfo/pod-labels
```

```bash
kubectl exec volume-app -n volume-lab -c sidecar -- \
  cat /etc/podinfo/cpu-request
```

---

## Projected volume

```bash
kubectl exec volume-app -n volume-lab -c sidecar -- \
  find /var/run/projected -type f
```

Output atteso:

```text
/var/run/projected/config/app-mode
/var/run/projected/secret/api-key
/var/run/projected/metadata/name
/var/run/projected/token
```

---

## emptyDir condiviso

```bash
kubectl exec volume-app -n volume-lab -c sidecar -- \
  cat /runtime/status.log
```

```bash
kubectl exec volume-app -n volume-lab -c nginx -- \
  cat /usr/share/nginx/html/runtime/status.log
```

I due container devono vedere lo stesso file.

---

## PVC condiviso

```bash
kubectl exec volume-app -n volume-lab -c sidecar -- \
  cat /data/persistent.log
```

```bash
kubectl exec volume-app -n volume-lab -c nginx -- \
  cat /usr/share/nginx/html/data/persistent.log
```

---

## File creato dall'initContainer

Il file viene scritto nel volume `runtime`.

```bash
kubectl exec volume-app -n volume-lab -c nginx -- \
  cat /usr/share/nginx/html/runtime/index.html
```

---

# Particolarità da ricordare

## readOnly

```yaml
volumeMounts:
  - name: persistent-data
    mountPath: /data
    readOnly: true
```

`readOnly` vale soltanto per quel mount nel container.

Lo stesso volume può essere:

```text
nginx   → read-only
sidecar → read-write
```

---

## subPath

```yaml
volumeMounts:
  - name: all-config
    mountPath: /etc/nginx/conf.d/extra.conf
    subPath: nginx-extra.conf
```

Permette di montare un singolo file senza sostituire tutta la directory.

Un file ConfigMap o Secret montato tramite `subPath` non riceve normalmente aggiornamenti automatici mentre il Pod è in esecuzione.

---

## items

```yaml
configMap:
  name: web-config
  items:
    - key: APP_MODE
      path: mode.txt
```

Permette di:

- scegliere solo alcune chiavi
- rinominare i file risultanti

---

## optional

```yaml
configMap:
  name: future-config
  optional: true
```

Il Pod può partire anche se la ConfigMap non esiste.

---

## defaultMode

```yaml
secret:
  secretName: app-secret
  defaultMode: 0400
```

Imposta i permessi di default dei file generati dal volume Secret.

---

## ServiceAccount token

Con:

```yaml
automountServiceAccountToken: false
```

il token non viene montato automaticamente.

Può però essere richiesto esplicitamente:

```yaml
projected:
  sources:
    - serviceAccountToken:
        path: token
        expirationSeconds: 3600
```

---

## emptyDir e PVC

```text
emptyDir
→ sopravvive al riavvio del container
→ viene eliminato con il Pod

PVC
→ sopravvive all'eliminazione del Pod
→ dipende dal PV e dalla reclaim policy
```

---

# Pulizia

```bash
kubectl delete namespace volume-lab
```

Se hai creato manualmente il PV:

```bash
kubectl delete pv pv-volume-lab
```

Se hai creato manualmente la StorageClass:

```bash
kubectl delete storageclass standard
```
