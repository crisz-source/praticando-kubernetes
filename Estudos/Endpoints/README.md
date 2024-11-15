# Endpoint conceitualmente falando, é um destino de um serviço;  No K8s os endpoints é um recurso usado para estabelecer acesso quando nao tem um selector, somente IP

# dentro do diretório Estudos, aplique o manifesto yml
```bash
kubectl apply -f ./Endpoints/my-pod-endpoints.yml

# Abra o terminal, Verifique se estao rodando normalmente
kubectl get pods 

# Verifique o endereço ip, pois para fazer o acesso a esses pods através de endpoint, vou utilizar o ip ao invez de selectors
kubectl get pods -o wide 

# saída esperada
# obs: os IPs do pod, não são fixos, se derrubar esses pods eles irão ter ips diferentes quando subirem novamente
NAME         READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
pod-apache   1/1     Running   0          2m23s   10.244.0.6   minikube   <none>           <none>
pod-nginx    1/1     Running   0          2m23s   10.244.0.7   minikube   <none>           <none>
```
 
# É possível fixar esses endereços IPs dos pods
```bash
# Copiei o endereço ip do pod apache e nginx e aidicionei na frente como comentário de cada pod no manifesto, criei um outro arquivo manifesto e fiz a criação do service e endpoint; Aplique-os
kubectl apply -f ./Endpoints/my-endpoints-test.yml

# Saída esperada:
service/my-endpoints-service unchanged
endpoints/my-endpoints-service created

# Verifique se os pods estão ok
kubectl get pods -o wide

NAME         READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
pod-apache   1/1     Running   0          78m   10.244.0.6   minikube   <none>           <none>
pod-nginx    1/1     Running   0          78m   10.244.0.7   minikube   <none>           <none>

# Verificando se os recursos foram criados corretamente
kubectl get endpoints

# Node que na saída, os endereços ip são os mesmos do arquivo yml my-endpoints-test.yml
NAME                   ENDPOINTS                     AGE
kubernetes             192.168.49.2:8443             94m
my-endpoints-service   10.244.0.6:80,10.244.0.7:80   2m9s

# Verifique o service
kubectl get service

# Note que na saída o service esta "ligado" com o endpoint atravez do mesmo nome
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP   96m
my-endpoints-service   ClusterIP   10.108.169.98   <none>        80/TCP    5m54s
```


# Testando o endpoint
```bash
# Inicie um pod qualquer
kubectl run -it --image debian test-eps

# Dentro do container, instale algumas ferramentas para fazer os teste
apt update && apt install dnsutils -y && apt install curl -y

# Depois de atualizar o repositorio, e instalar as ferramentas. Fiz um nslookup em cima desse serviço
# Ainda dentro do container
nslookup my-endpoints-service

# Note que na Saída esperada, no campo Address é o mesmo ip do service que esta no campo CLUSTER-IP acima, e o kubernetes resolvou o DNS no campo Name para chegar ao serviço:
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   my-endpoints-service.default.svc.cluster.local
Address: 10.108.169.98

# teste do endpoint usando o DNS com nsloockup e curl, realize esse teste pelo menos 3x e verá o resultado do nginx e apache
curl my-endpoints-service.default.svc.cluster.local

# Deletando tudo
kubectl get endpoints
kubectl delete endpoints my-endpoints-service
kubectl get svc
kubectl delete svc my-endpoints-service
kubectl get pod
kubectl delete pod --all

```