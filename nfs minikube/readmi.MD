minikube ssh  # entra nella VM di Minikube
sudo apt-get update
sudo apt-get install nfs-kernel-server -y
sudo mkdir -p /mnt/data
sudo chmod 777 /mnt/data
echo "/mnt/data *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
sudo exportfs -rav
