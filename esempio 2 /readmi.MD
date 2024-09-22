
**Esempio di come replicare l'istanza tomcat:**

### 1. **File README.md:**
* **Vantaggi:** Uno dei vantaggi è proprio quello di avere più repliche del servizio
  * Scalabilità
  * Ridondanza e Failover
  * etc

### 2. **Wiki:**
* **Funzionalità:** Con kubectl posso facilmente, senza modificare il file di deployment creare più repliche 
* **Utilizzo:** Il comando sotto crea più repliche del servizio tomcat

Per avviare il progetto:
```bash
kubectl scale --replicas=4 deployment/tomcat-deployment
```
essendo che ho più repliche da esporre, non va più bene NodePort perchè un servizio può avere un solo NodePort associato, l'alternativa sarebeb un port-forward
Ma fortunatamente possiamo usare LoadBalancer e quindi si può riaggiornare il servizio con LoadBalancer al posto di NodePort

```bash
kubectl expose deployment tomcat-deployment --type=LoadBalancer --port=8080 --target-port=8080 --name=tomcat-load-balancerservice/tomcat-load-balancer exposed

```
Adesso se vedo l'output di con describe trovo un ip unico al quale accedere per uno dei quattro servizi
```bash
root@192:~/Scrivania/kubernetes-demo-master$ kubectl describe services tomcat-load-balancer
Name:                     tomcat-load-balancer
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=tomcat
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.106.188.121
IPs:                      10.106.188.121
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31609/TCP
Endpoints:                10.244.0.7:8080,10.244.0.9:8080,10.244.0.10:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:                   <none>
```
