# Helm

## Installazione

1. Per installare Helm su Debian, prima di tutto devi aggiungere il repository ufficiale.
   ```bash
   curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
   ```
   Questo comando scarica e aggiunge la chiave GPG per verificare i pacchetti di Helm. Questo è importante per motivi di sicurezza: assicura che i pacchetti che stai installando provengano effettivamente dal team di Helm.
   ```bash
   sudo apt-get install apt-transport-https --yes
   echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
   sudo apt-get update
   sudo apt-get install helm
   helm version
   ```
2. 