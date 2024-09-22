**** Semplicissimo a questo punto installare tipo mongo db

kubectl create deployment hello-mongo --image=mongo --port=27017

kubectl expose deployment hello-mongo --type=NodePort

minikube service hello-mongo --url
