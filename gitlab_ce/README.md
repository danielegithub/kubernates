# Installazione di GitLab su Minikube

Questa guida spiega come installare GitLab su Minikube utilizzando un file di configurazione personalizzato.

## Configurazione

1. **Modifica del file `values-minikube-minimum.yaml`**:
   - Ho cambiato l'indirizzo IP con l'output di `minikube ip`.
   - Ho sostituito `nip.io` con `gitlab.local` per una configurazione più semplice.

2. **Apertura del file hosts**:
   - Modifica il file `hosts` situato in `C:\Windows\System32\drivers\etc` aggiungendo la riga:
     ```
     <ip-minikube> gitlab.gitlab.local
     ```
   - Per verificare gli ingressi, usa:
     ```bash
     kubectl get ingress
     ```

3. **Installazione di GitLab**:
   - Esegui il comando:
     ```bash
     helm upgrade --install gitlab gitlab/gitlab -f values-minikube-minimum.yaml --timeout 600s
     ```

4. **Ottenere la password di accesso**:
   - Utilizza il seguente comando in PowerShell per ottenere la password iniziale dell'utente root di GitLab:
     ```powershell
     kubectl get secret gitlab-gitlab-initial-root-password -o jsonpath='{.data.password}' | Out-String | %{ [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_.Trim())) }
     ```

# Progetto Node.js su GitLab

## Passaggi per la Configurazione

1. **Creare il file `.gitignore`**:
   Aggiungi `node_modules/` per escludere le dipendenze dal repository.

2. **Inizializzare Git**:
   ```bash
   git init
   git add .
   git commit -m "Primo commit"
   ```

3. **Bypassare il controllo SSL**:
   Poiché non hai un certificato valido, esegui:
   ```bash
   git config --global http.sslVerify false
   git remote set-url origin http://gitlab.gitlab.local/root/nodesito.git
   ```

4. **Creare un Repository su GitLab**:
   Aggiungi il repository con:
   ```bash
   git remote set-url origin https://gitlab.gitlab.local/root/nodesito.git
   ```
   Ora puoi fare push:
   ```bash
   git push -u origin master
   ```

