# Simulazione Kubernetes con Minikube

## Introduzione
Questo documento descrive una simulazione di un ambiente Kubernetes utilizzando Minikube. Verranno illustrati i passaggi per creare e gestire deployment, esporre servizi e interagire con il cluster.

## Prerequisiti
* **kubectl:** installato e configurato correttamente.
* **Minikube:** installato e avviato.

## Avvio della Dashboard
Per una visualizzazione grafica dei tuoi pod e risorse, avvia la dashboard Minikube:
```bash
minikube dashboard
```
scarico il classico HelloMinikube ( * **deprecato)
```bash
kubectl run hello-minikube-2 --image=kicbase/echo-server:1.0 --port=8080
```

ora si usa
```bash
kubectl create deployment hello-minikube-test --image=registry.k8s.io/e2e-test-images/agnhost:2.39 --port=8080
```

## Lezione 1: Creare un Deployment di Tomcat

### File `deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat
  template:
    metadata:
      labels:
        app:   
 tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:latest
        ports:
        - containerPort: 8080 1  
