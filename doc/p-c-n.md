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
| Opzione            | Significato                                                                     |        Usata nei pattern       |
| ------------------ | ------------------------------------------------------------------------------- | :----------------------------: |
| `--image=IMAGE`    | Specifica l'immagine del container                                              |                ✅               |
| `--restart=Never`  | Crea un Pod (non un Deployment)                                                 |                ✅               |
| `--rm`             | Elimina il Pod al termine dell'esecuzione                                       |                ✅               |
| `-i`               | Collega stdin/stdout al Pod (necessario con `--rm` per comandi non interattivi) |                ✅               |
| `-t`               | Alloca un terminale (TTY)                                                       |   Solo con shell interattive   |
| `-it`              | Equivale a `-i -t`                                                              |      Shell (`sh`, `bash`)      |
| `--`               | Separa le opzioni di `kubectl` dal comando eseguito nel container               |                ✅               |
| `--dry-run=client` | Genera il manifest senza creare il Pod                                          |                ✅               |
| `-o yaml`          | Visualizza il manifest in formato YAML                                          |         Con `--dry-run`        |
| `sh` / `bash`      | Apre una shell nel container                                                    |      Pattern 1, 2 e `exec`     |
| `sh -c`            | Esegue un comando tramite la shell del container (utile per redirect e pipe)    | Salvataggio file nel container |
| `wget -qO-`        | Scarica una pagina e la stampa su stdout                                        |          Pattern HTTP          |
| `nslookup`         | Verifica la risoluzione DNS                                                     |           Pattern DNS          |
| `nc -zv`           | Verifica la raggiungibilità di una porta TCP                                    |           Pattern TCP          |
| `sleep 3600`       | Mantiene il Pod in esecuzione per il debug                                      |         Pod persistente        |

| Opzione | Significato                                                       |
| ------- | ----------------------------------------------------------------- |
| `-q`    | Quiet mode (non mostra il progresso del download)                 |
| `-O-`   | Scrive il contenuto scaricato su **stdout** invece che in un file |

| Opzione | Significato                                                     |
| ------- | --------------------------------------------------------------- |
| `nc`    | **Netcat**, strumento per testare connessioni di rete (TCP/UDP) |
| `-z`    | Verifica solo se la porta è aperta, senza inviare dati          |
| `-v`    | Mostra informazioni dettagliate (verbose)                       |



| Voglio...                                     | Opzioni                                 |
| --------------------------------------------- | --------------------------------------- |
| Aprire una shell (`sh`, `bash`)               | `-it`                                   |
| Eseguire un comando e vedere l'output         | `-i`                                    |
| Salvare l'output in un file                   | `-i` + `> file`                         |
| Usare Git Bash con una shell                  | `winpty` + `-it`                        |
| Usare Git Bash per un comando non interattivo | **Niente `winpty`**, solo `-i` se serve |


