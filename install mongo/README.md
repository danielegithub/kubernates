# Da fare dopo esercizio 1-2-3
## installazione di un database mongodb (replica impostata a 4)
### Direttamente da bash
```bash

kubectl create deployment hello-mongo --image=mongo --port=27017

kubectl expose deployment hello-mongo --type=NodePort

minikube service hello-mongo --url
```

### oppure tramite deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  selector:
    matchLabels:
      app: mongo
  replicas: 4
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:latest
        ports:
        - containerPort: 27017
  ```      
