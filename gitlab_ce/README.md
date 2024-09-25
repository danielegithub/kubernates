# Installazione di GitLab su Kubernetes

Questa guida descrive i passaggi per installare GitLab Community Edition su un cluster Kubernetes, inclusa la configurazione dei volumi, dei servizi e la creazione di un secret per la password dell'utente root.

## Prerequisiti

Assicurati di avere i seguenti strumenti installati:

- Kubernetes in esecuzione (Minikube o un cluster Kubernetes reale)
- `kubectl` installato e configurato
- per installare minikube in VBox e non Hyper v è possibile
  - minikube config set driver virtualbox
  - minikube start --driver=virtualbox

## Passaggi per l'Installazione di GitLab

### 1. Creare un Persistent Volume e un Persistent Volume Claim

Crea un file chiamato `gitlab-pvc.yaml` per definire il volume e il claim:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: gitlab-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/gitlab
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: gitlab-volume-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Applica il file:

```bash
kubectl apply -f gitlab-pvc.yaml
```

### 2. Creare un Secret per la Password dell'Utente Root

Crea un secret per memorizzare la password dell'utente root di GitLab:

```bash
kubectl create secret generic gitlab-ce-initial-root-password --from-literal=password='your_secure_password' -n gitlab-namespace
```


### 3. Creare il Deployment di GitLab

Crea un file chiamato `gitlab-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitlab
  namespace: gitlab-namespace
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitlab
  template:
    metadata:
      labels:
        app: gitlab
    spec:
      containers:
        - name: gitlab
          image: gitlab/gitlab-ce:latest
          ports:
            - containerPort: 80
            - containerPort: 443
          env:
            - name: GITLAB_OMNIBUS_CONFIG
              value: "external_url 'http://gitlab.example.com'; gitlab_rails['gitlab_shell_ssh_port'] = 22;"
            - name: GITLAB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: gitlab-ce-initial-root-password
                  key: password
          volumeMounts:
            - name: gitlab-storage
              mountPath: /var/opt/gitlab
      volumes:
        - name: gitlab-storage
          persistentVolumeClaim:
            claimName: gitlab-volume-claim
```

Applica il file:

```bash
kubectl apply -f gitlab-deployment.yaml
```

### 4. Creare il Service per GitLab

Crea un file chiamato `gitlab-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: gitlab-service
  namespace: gitlab-namespace
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
  selector:
    app: gitlab
```

Applica il file:

```bash
kubectl apply -f gitlab-service.yaml
```

### 5. Controllare lo Stato di GitLab

Verifica che i pod siano in esecuzione:

```bash
kubectl get pods -n gitlab-namespace
```

Controlla i log del pod per eventuali errori:

```bash
kubectl logs <nome-del-pod> -n gitlab-namespace
```

### 6. Ottenere la Password dell'Utente Root di GitLab

Per ottenere la password dell'utente root, usa il comando:

```bash
kubectl get secret gitlab-ce-initial-root-password -n gitlab-namespace -o jsonpath="{.data.password}" | base64 --decode ; echo
```

### 7. Accedere a GitLab

Una volta che GitLab è in esecuzione, puoi accedere all'interfaccia web tramite l'IP del tuo nodo. Usa il comando seguente per ottenere l'IP:

```bash
kubectl get svc -n gitlab-namespace
```

Puoi accedere a GitLab usando l'URL `http://<IP-del-nodo>:<porta-nodeport>`.

### Esempi e Considerazioni

- **Persistent Volume**: Un volume persistente è un'unità di archiviazione nel cluster Kubernetes che mantiene i dati anche se i pod vengono ricreati. In questo caso, abbiamo creato un PV di 10Gi.
- **Secret**: I segreti in Kubernetes vengono utilizzati per memorizzare informazioni sensibili come password, token e chiavi SSH. In questo esempio, abbiamo utilizzato un secret per memorizzare la password dell'utente root di GitLab.
- **Service**: Un servizio in Kubernetes consente di accedere ai pod in esecuzione. In questo esempio, abbiamo creato un servizio di tipo NodePort per esporre GitLab all'esterno.

## Pulizia delle Risorse

Se desideri rimuovere tutte le risorse create, puoi eseguire i seguenti comandi:

```bash
kubectl delete deployment gitlab -n gitlab-namespace
kubectl delete service gitlab-service -n gitlab-namespace
kubectl delete persistentvolumeclaim gitlab-volume-claim -n gitlab-namespace
kubectl delete secret gitlab-ce-initial-root-password -n gitlab-namespace
```
