
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
