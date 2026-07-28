```sh
# 1. Shell BusyBox (Linux / CKAD)
kubectl run temp --rm -it --restart=Never --image=busybox -- sh

# 2. Shell BusyBox (Git Bash)
winpty kubectl run temp --rm -it --restart=Never --image=busybox -- sh

# 3. Test DNS
kubectl run temp -i --rm --restart=Never --image=busybox -- nslookup nginx

# 4. Test HTTP
kubectl run temp -i --rm --restart=Never --image=busybox -- wget -qO- http://nginx

# 5. HTTP e salva in un file locale
kubectl run temp -i --rm --restart=Never --image=busybox -- wget -qO- http://nginx > nginx.txt

# 6. HTTP e salva in un file nel container
kubectl run temp -i --rm --restart=Never --image=busybox -- sh -c 'wget -qO- http://nginx > /tmp/nginx.txt'

# 7. Visualizza la configurazione DNS del Pod
kubectl run temp -i --rm --restart=Never --image=busybox -- cat /etc/resolv.conf

# 8. Test di una porta TCP
kubectl run temp -i --rm --restart=Never --image=nicolaka/netshoot -- nc -zv nginx 80

# 9. Genera il manifest YAML
kubectl run temp --image=busybox --restart=Never --dry-run=client -o yaml -- sleep 3600

# 10. Pod persistente per il debug
kubectl run debug --restart=Never --image=busybox -- sleep 3600

# Entrare successivamente nel Pod
kubectl exec -it debug -- sh

# Git Bash
winpty kubectl exec -it debug -- sh

```
