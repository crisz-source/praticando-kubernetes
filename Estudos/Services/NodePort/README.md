## Utilizei os mesmos arquivo do ClusterIP para implementar o NodePort, troquei algumas coisas, comentei o container do tomcat e modifiquei a porta
## Caso não defina uma porta para o nodePort, o kubernetes vai acrescentar uma porta aleatória em um range de 30000 - 32767
```bash
# Testando as configurações
kubectl apply -f nodeport.yml -f service.yml
kubectl get pod
kubectl get svc

# para ver as informações do serivce
kubectl describe svc frontend-service-nodeport

# Acessando o serviço pelo minikube
minikube service --url frontend-service-nodeport -n=default

# Caso o minikube gere uma porta diferente do que 30008, não tem problema. Basta pegar a url que o minikube gerou e colocar no seu browser que vai da tudo certo! 

# Identificando o ip do node
kubectl get nodes -o yaml | grep address

# Deletando o nodePort service e o pod
kubectl get svc
kubectl delete svc frontend-service-nodeport

kubectl get pod 
kubectl delete pod web-pod 
```