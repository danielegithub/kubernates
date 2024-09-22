
# Attribuzione di Label e Selector ai Nodi
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  selector:
    matchLabels:
      app: tomcat
  replicas: 4
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:9.0
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 3
      nodeSelector:
        storageType: ssd



```

## Concetti base

- **Label**: Le label sono delle etichette chiave-valore che possono essere assegnate a risorse Kubernetes, come nodi, pod, e deployment. Aiutano a organizzare e selezionare risorse all'interno del cluster.
  
- **Selector**: Un selector è un criterio che viene utilizzato per selezionare le risorse basandosi sulle label. Un esempio comune è quando vogliamo che un deployment sia eseguito solo su nodi con una determinata label.

## Health Check dei Pod

Kubernetes offre due tipi di probe per monitorare la salute e la disponibilità dei servizi:

- **Liveness Probe**: Verifica se il pod è ancora attivo. Se fallisce, Kubernetes riavvierà il pod.
  
- **Readiness Probe**: Controlla se il pod è pronto per ricevere traffico. Se fallisce, il servizio non invierà richieste a quel pod.

Queste funzionalità si impostano tramite i parametri `livenessProbe` e `readinessProbe` nel file di deployment.

## Applicazione delle configurazioni

Per applicare un file di configurazione aggiornato (come ad esempio un file di deployment che contiene liveness e readiness probe), si utilizza il comando:

```bash
kubectl apply -f ./deployment.yaml
```

## Attribuire una Label a un Nodo

In un ambiente **Minikube**, è importante notare che si ha a disposizione un solo nodo. Possiamo comunque attribuire delle label a quel nodo per scopi organizzativi o per influenzare i deployment.

### Esempio: Attribuire una Label al Nodo Minikube

Vogliamo attribuire la label `storageType=ssd` al nodo Minikube per indicare che i deployment su quel nodo fanno riferimento a un volume SSD:

```bash
kubectl label node minikube storageType=ssd
```

### Verifica della Label

Per verificare che la label sia stata correttamente assegnata, eseguiamo il comando:

```bash
kubectl describe node minikube
```

Questo mostrerà le informazioni dettagliate sul nodo, inclusa la nuova label.

## Utilizzo del Node Selector

Ora possiamo modificare il file di deployment per fare in modo che solo i nodi etichettati con `ssd` vengano presi in considerazione per il deploy. Questo si fa aggiungendo il parametro `nodeSelector` nel file di deployment.

Il `nodeSelector` garantisce che il deployment avvenga solo sui nodi che soddisfano determinati criteri, come le label che abbiamo assegnato in precedenza.

Esempio di `nodeSelector` nel file di deployment:

```yaml
spec:
  nodeSelector:
    storageType: ssd
```

In questo modo, i pod saranno deployati solo sui nodi che hanno la label `storageType=ssd`.

```

Questo file include le spiegazioni sui concetti di label e selector, le funzioni di health check con i probe, e i comandi Kubernetes con la formattazione corretta. Puoi copiarlo direttamente per il tuo repository Git.
