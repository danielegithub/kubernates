
I volumi possono essere di diversi tipi, tra cui:
- **Local**: Memoria locale sul nodo.
- **Network**: Storage di rete, come NFS.
- **Cloud Block Storage**: Servizi di storage forniti da cloud provider.
- **Directory on the Host**: Directory locale sul filesystem del nodo.

In questo esempio, utilizziamo volumi locali.

### Configurare un server NFS su Minikube
Per usare NFS come storage condiviso, dovrai configurare un server NFS. Puoi installarlo direttamente su Minikube o usare un pod separato.
```bash
minikube ssh  # entra nella VM di Minikube
sudo apt-get update
sudo apt-get install nfs-kernel-server -y
sudo mkdir -p /mnt/data
sudo chmod 777 /mnt/data
echo "/mnt/data *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
sudo exportfs -rav
```
Questi comandi installano e configurano un server NFS locale che esporta la directory /mnt/data.

### Configurare un server NFS su Minikube
Ora, crea un PersistentVolume che punti al server NFS che hai configurato.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.49.2  # L'IP della VM Minikube (verifica l'IP con `minikube ip`)
    path: "/mnt/data"      # Percorso esportato dal server NFS
```
Ora crea un PersistentVolumeClaim che richiede il volume con accesso ReadWriteMany.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```
Creare più pod che usano lo stesso PVC
Ora puoi creare due o più pod che montano lo stesso PersistentVolumeClaim. Questi pod condivideranno lo stesso volume NFS.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo 'Hello from pod1' >> /mnt/data/hello.txt; sleep 5; done"]
    volumeMounts:
    - name: nfs-storage
      mountPath: /mnt/data
  volumes:
  - name: nfs-storage
    persistentVolumeClaim:
      claimName: nfs-pvc
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo 'Hello from pod2' >> /mnt/data/hello.txt; sleep 5; done"]
    volumeMounts:
    - name: nfs-storage
      mountPath: /mnt/data
  volumes:
  - name: nfs-storage
    persistentVolumeClaim:
      claimName: nfs-pvc

```
posso navigare tra i file e trovo il file su hostpath (di solito crea una cartella sda1) oppure fare direttamente il cat
```bash
kubectl exec -it pod1 -- cat /mnt/data/hello.txt
```
