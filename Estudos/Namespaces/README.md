# Namespaces

# Organização e isolamento de pods, cada pod precisa ser único em seu namespace

# criando namespace de forma imperativa
```bash
kubectl create namespace frontend --save-config
kubectl get namespace
```

# aplicando o pod do tomcat do namespace frontend, -n é abreviação de --namespace=frontend
```bash
kubectl apply -f tomcat-pod-ns.yml -n=frontend
```

# verificando se deu tudo certo o pod tomcat dentro do namespace frontend
```bash
kubectl get pod -n=frontend
``` 


# opcional, removendo a namespace default do kubernetes e definindo uma nova namespace como default. Todos os recursos que irão ser criados, sendo forma imperativa ou utilizando arquivos .yml, os recursos por padrão serão criados no namespace frontend. Deixei o comando para voltar ao normal caso prefira
```bash
kubectl config set-context --current --namespace=frontend
kubectl config set-context --current --namespace=default
```

# deletando um pod que esta dentro de uma namespace
```bash
kubectl get pod -n=frontend
kubectl delete pods tomcat-pod -n=frontend
```


##---

# criando uma namespace pelo arquivo .yml forma declarativa
```bash
kubectl apply -f backend-namespace.yml
```

# Depois de criar a namespace de forma declarativa, adicione agora um pod neste namespace criado
```bash
cd Kubernetes-networking
kubectl apply -f redis.yml -n=backend-ns
```


# deletando a namespace, se deletar uma namespace, TODOS os pods que estão nesse namespace que vai ser deletado também serão deletados junto 
```bash
kubectl delete ns frontend
```