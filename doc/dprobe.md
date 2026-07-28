### CKAD Probe Troubleshooting Exercises

These exercises simulate CKAD troubleshooting tasks. The environment is
already prepared: your goal is to identify the problem and apply the
minimum required change.

------------------------------------------------------------------------

## Exercise 1 - Add a Startup Probe

## Environment preparation

``` bash
kubectl create namespace probe-lab

kubectl apply -n probe-lab -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: slow-app
spec:
  containers:
  - name: nginx
    image: nginx
    command: ["sh","-c","sleep 45 && nginx -g 'daemon off;'"]
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      periodSeconds: 5
      failureThreshold: 3
EOF
```

## Task

The container is repeatedly restarted because the existing Liveness Probe fails before the application has finished starting.

Without removing the existing Liveness Probe, modify the Pod so that the application can complete its startup and the restart count stops increasing.


<details>


<summary>

Solution

</summary>

``` yaml
startupProbe:
  httpGet:
    path: /
    port: 80
  periodSeconds: 5
  failureThreshold: 10
```


</details>


------------------------------------------------------------------------

## Exercise 2 - Fix the Wrong Probe

## Environment preparation

``` bash
kubectl apply -n probe-lab -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  - name: web
    image: nginx
    readinessProbe:
      httpGet:
        path: /does-not-exist
        port: 80
EOF
```

## Task

The Pod remains `0/1 Ready`.

The application is working correctly.

Modify the Pod so the Service can send traffic to it.


<details>


<summary>

Solution

</summary>

Change:

``` yaml
path: /does-not-exist
```

to

``` yaml
path: /
```


</details>


------------------------------------------------------------------------

## Exercise 3 - Service Does Not Route Traffic

## Environment preparation

``` bash
kubectl create deployment web --image=nginx -n probe-lab

kubectl expose deployment web --port=80 --target-port=80 --name=web-service -n probe-lab

kubectl patch deployment web -n probe-lab --type=json -p='[
{"op":"add","path":"/spec/template/spec/containers/0/readinessProbe","value":{"httpGet":{"path":"/wrong","port":80},"periodSeconds":5}}
]'
```

## Task

Users report:

-   the Pod is Running
-   the Service has no available endpoints

Fix the problem.


<details>

<summary>

Solution

</summary>

``` yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
```


</details>


------------------------------------------------------------------------

## Exercise 4 - Startup Probe Timeout

## Environment preparation

``` bash
kubectl apply -n probe-lab -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: startup-app
spec:
  containers:
  - name: app
    image: nginx
    command: ["sh","-c","sleep 30 && nginx -g 'daemon off;'"]
    startupProbe:
      httpGet:
        path: /
        port: 80
      periodSeconds: 5
      failureThreshold: 3
EOF
```

## Task

The Pod never becomes Ready.

Modify the Startup Probe using the minimum number of changes.


<details>


<summary>

Solution

</summary>

Increase:

``` yaml
failureThreshold: 7
```

or higher.


</details>

---
