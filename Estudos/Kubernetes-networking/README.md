# testando a comunicação de pods.
```bash
# Aplicando os pods
kubectl apply -f redis.yml
kubectl apply -f tomcat-pod.yml

# pegando o ip do pod redis para testar a comunicação
kubectl describe pod redis-pod | grep IP
ip: 10.244.0.21

# entrando dentro do container que esta dentro do pod tomcat
 kubectl exec -it tomcat-pod -- bash

# dentro do container, atualize o repositório
apt update

# instale o ping
apt install iputils-ping -y

# teste o ping que pegou no comando acima do redis-pod
ping 10.244.0.21

# deletando os pod
kubectl delete pod redis-pod tomcat-pod
``` 