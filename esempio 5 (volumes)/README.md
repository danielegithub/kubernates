
# Deploy di MySQL e WordPress con Volumi Persistenti su Kubernetes

In questo progetto, eseguiremo il deploy di MySQL e WordPress su un cluster Kubernetes utilizzando Minikube. Aggiungeremo anche **Persistent Volumes** per garantire che i dati non vengano persi quando i Pod vengono ricreati.

## Aggiunta di Volumi Persistenti

I volumi possono essere di diversi tipi, tra cui:
- **Local**: Memoria locale sul nodo.
- **Network**: Storage di rete, come NFS.
- **Cloud Block Storage**: Servizi di storage forniti da cloud provider.
- **Directory on the Host**: Directory locale sul filesystem del nodo.

In questo esempio, utilizziamo volumi locali.

### Definizione di Persistent Volume

Definiamo due **Persistent Volume** da 20 GB ciascuno, montati nelle directory `/tmp/data/pv-1` e `/tmp/data/pv-2`.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-1
  labels:
    type: local
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/data/pv-1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-2
  labels:
    type: local
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/data/pv-2
```

Questi volumi verranno creati in locale sulla macchina host, utilizzando `/tmp/data/pv-1` e `/tmp/data/pv-2` come path.

### Creazione dei Volumi

Per creare i volumi, eseguire il seguente comando:

```bash
kubectl create -f local-volumes.yaml
```

Per ottenere informazioni sui volumi creati, puoi utilizzare il comando:

```bash
kubectl get persistentvolumes
```

## Aggiunta dei Volumi al Deployment di MySQL

Nel file di deploy di MySQL, aggiungeremo un **Persistent Volume Claim** che richiede accesso ai volumi persistenti definiti in precedenza. Aggiungeremo inoltre la configurazione per montare il volume nei container.

### YAML del deploy di MySQL con Volume Persistente:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: PASSWORDS_IN_PLAIN_TEXT_ARE_BAD_WE_WILL_SHOW_SOMETHING_MORE_SECURE_LATER
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

### Spiegazione:
- **PersistentVolumeClaim**: Richiede l'accesso a uno dei volumi definiti in precedenza.
- **volumeMounts**: Monta il volume persistente nella directory `/var/lib/mysql` all'interno del container MySQL, dove vengono memorizzati i dati del database.
- **volumes**: Definisce che il volume montato proviene dal Persistent Volume Claim.

## Aggiunta dei Volumi al Deployment di WordPress

Allo stesso modo, aggiungeremo un **Persistent Volume Claim** e monteremo un volume per WordPress.

### YAML del deploy di WordPress con Volume Persistente:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          value: PASSWORDS_IN_PLAIN_TEXT_ARE_BAD_WE_WILL_SHOW_SOMETHING_MORE_SECURE_LATER
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

### Spiegazione:
- Il **PersistentVolumeClaim** per WordPress richiede 20 GB di spazio e monta il volume persistente in `/var/www/html`, la directory dove vengono memorizzati i file di WordPress.

## Creazione dei Deployment

Per creare i deployment, esegui i seguenti comandi:

1. **Deploy di MySQL**:
   ```bash
   kubectl create -f mysql-deployment.yaml
   ```

2. **Deploy di WordPress**:
   ```bash
   kubectl create -f wordpress-deployment.yaml
   ```

3. **Verifica del servizio WordPress**:
   ```bash
   kubectl get service wordpress
   ```

4. **Ottieni l'URL del servizio WordPress tramite Minikube**:
   ```bash
   minikube service wordpress --url
   ```

Visitando l'URL ottenuto, potrai visualizzare il sito WordPress connesso al database MySQL, entrambi con volumi persistenti per garantire che i dati siano conservati.

## Conclusioni

In questo progetto, abbiamo aggiunto volumi persistenti per MySQL e WordPress su Kubernetes. Questo garantisce che i dati non vengano persi anche quando i Pod vengono ricreati, permettendo una gestione più sicura e affidabile delle applicazioni su Kubernetes.

