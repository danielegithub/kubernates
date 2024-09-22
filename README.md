# SIMULAZIONE

## Passaggi:

1. Dopo aver installato **kubectl** ed averlo avviato:
   ```bash
   kubectl start
   ```

2. Avvio della dashboard di Minikube:
   ```bash
   minikube dashboard
   ```
   In questo modo, nella sezione **Pods** troverai tutte le immagini scaricate.

3. Scarico il classico **HelloMinikube** (deprecato):
   ```bash
   kubectl run hello-minikube-2 --image=kicbase/echo-server:1.0 --port=8080
   ```

4. Ora si usa:
   ```bash
   kubectl create deployment hello-minikube-test --image=registry.k8s.io/e2e-test-images/agnhost:2.39 --port=8080
   ```

5. Esporre il servizio all'esterno (renderlo accessibile fuori dal cluster):
   ```bash
   kubectl expose deployment hello-minikube-test --type=NodePort
   ```

6. Ottenere l'URL del servizio esposto:
   ```bash
   minikube service hello-minikube-test --url
   ```

## ALCUNI COMANDI UTILI:

- Visualizzare i servizi:
  ```bash
  kubectl get services
  ```

- Verificare il deployment:
  ```bash
  kubectl get deployment hello-minikube-test
  ```

- Controllare i log di Minikube:
  ```bash
  minikube logs
  ```

## Lezione 1 - Come fare un proprio deployment di Tomcat da eseguire fuori

1. Applicare il file di deployment:
   ```bash
   kubectl apply -f ./deployment.yaml
   ```

2. Esporre il deployment di Tomcat:
   ```bash
   kubectl expose deployment tomcat-deployment --type=NodePort
   ```
```

Puoi copiare e incollare tutto questo testo per utilizzarlo come file `README.md`.
