
# Deploy di MySQL e WordPress su Kubernetes con Minikube

In questo progetto, eseguiremo il deploy di MySQL e WordPress su un cluster Kubernetes utilizzando Minikube.

## Struttura del deploy

Il deploy è diviso in due parti principali:
1. **Service** - Definisce come i Pod sono esposti in rete.
2. **Deployment** - Definisce la creazione dei Pod che eseguono MySQL e WordPress.

Entrambi i file YAML sono divisi dai `---`, che separano più risorse in un singolo file. In alternativa, potresti separare queste risorse in file distinti.

## Creazione di MySQL

### YAML per il deploy di MySQL:

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
```

### Descrizione:
- **Service**: Espone il database MySQL sulla porta `3306`. Con `clusterIP: None`, si crea un **headless service**, utile per la scoperta dei Pod.
- **Deployment**: Crea un Pod con MySQL, specificando la variabile d'ambiente `MYSQL_ROOT_PASSWORD` (in questo caso, una password di esempio).

## Creazione di WordPress

### YAML per il deploy di WordPress:

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
```

### Descrizione:
- **Service**: Espone WordPress sulla porta `80` come un **LoadBalancer**, utile per far sì che il servizio sia accessibile dall'esterno del cluster.
- **Deployment**: Crea un Pod con WordPress, specificando le variabili d'ambiente necessarie per collegarsi al database MySQL (`WORDPRESS_DB_HOST` e `WORDPRESS_DB_PASSWORD`).

## Comandi per il deploy

1. Crea il deployment di MySQL:
   ```bash
   kubectl create -f mysql-deployment.yaml
   ```

2. Crea il deployment di WordPress:
   ```bash
   kubectl create -f wordpress-deployment.yaml
   ```

3. Controlla che il servizio WordPress sia stato creato correttamente:
   ```bash
   kubectl get service wordpress
   ```

4. Usa Minikube per ottenere l'URL del servizio WordPress:
   ```bash
   minikube service wordpress --url
   ```

Visitando l'URL ottenuto, potrai visualizzare il sito WordPress appena creato.

## Conclusioni

Questo semplice esempio mostra come eseguire il deploy di MySQL e WordPress su Kubernetes. Puoi espandere questo progetto aggiungendo volumi persistenti per mantenere i dati anche dopo la terminazione dei Pod o gestire la configurazione delle password in modo più sicuro utilizzando Kubernetes Secrets.

