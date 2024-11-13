# Foi criado um pod com 2 containers, um container do tomcat e do apache
```bash
kubectl apply -f clusterip.yml
```
# Foi criado um service, e execute os dois
```bash
kubectl apply -f clusterIp.yml -f service.yml


# verificando se deu tudo certo
kubectl get svc
kubectl get all

# Testando a comunicação do serviço com os pods, para isso, criei um pod debian pelo comando imperativo
kubectl run -it debian-pod --image=debian -- bash
apt update
apt install curl -y

# depois de atualizar o repositório, e instalar o curl, teste a comunicação com o ip do serviço na porta desejada
curl ip-service:80

# se retornar its works dentro do container, esta configurado corretamente!
<html><body><h1>It works!</h1></body></html>

# Para testar a parta 8080 que seria do outro container, basta modificar no arquivo service.yml para a porta 8080 e aplicar novamente
ports:
    - name: httpd 
      port: 80 
      targetPort: 8080

kubectl apply -f service.yml 

# ainda dentro do pod execute o comando curl novamente na mesma porta 80, pois o service vai fazer redirecionamento da porta 80 para a porta 8080 do container
curl ip-service:80


### Caso se pergunte, porque preciso utilizar a porta 80 e não 8080? a pporta 80 é do service, caso queira executar o comando curl ip-serivce:8080 vai ser necessário modificar a porta do service que esta no arquivo service.yml e modificar para a porta 8080 e se for necessário, modificar a porta do container para 8081

# Deletando as configurações
kubectl delete svc frontend-service
kubectl delete pod debian-pod web-pod
```