# DaemonSets esta vinculado aos nodes e não na aplicação. Servem para monitorar, pegar logs e etc

# Adicione mais um node no minikube com o seguinte comando
```bash
minikube node add


# Caso queira entrar dentro do node criado, execute:
kubectl get nodes
minikube ssh --node=nome-do-node

# Testando o daemonset
kubectl apply -f daemonset.yml 

# Verifique se ocorreu tudo certo.
kubectl get ds

# Saida esperada
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
my-daemonset   2         2         2       2            2           <none>          22s

#### Verifique os pods, caso tenha 2 pod esta correto! Pois o daemonset vai criar quantos pod for preciso dependendo do seu ambiente cluster, se tiver 5 node, o daemonset vai criar 5 pod, 1 pod em cada node. Para verificar como e onde esses pods foram criados, segue
kubectl get pods
kubectl get ds
kubectl get pods -o wide --field-selector spec.nodeName=nome-do-node

#### Daemonset orphan Pods, quando deleta um daemonset todos os pods que estão em cada worker node também são removidos, com o oprhan pods isso é modificado e assim que deleta um daemonset os pods permanecem nos nodes
#### O próprio kubernetes permite que outro daemonset "adote" esses orphan pods, porém o novo daemonset precisa ter o mesmo seletor do daemonset anterior

# Deletando o DaemonSet e deixando os pod em cada node
kubectl delete ds my-daemonset --cascade=orphan

# Verificando os node
kubectl get pods
kubectl get pods -o wide --field-selector spec.nodeName=minikube
kubectl get pods -o wide --field-selector spec.nodeName=minikube-m2
kubectl get pods -o wide --field-selector spec.nodeName=minikube-m3

### Caso os pod continuam nos nodes após a verificação acima, deu tudo certo!

# Adoção de pods, apague um pod de um node
kubectl get pod
kubectl get pods -o wide --field-selector spec.nodeName=minikube-m03
kubectl delete pod my-daemonset-2ctfx
kubectl get pods -o wide --field-selector spec.nodeName=minikube-m03

*saida esperada: No resources found in default namespace.*

kubectl get nodes
# Para adotar um pod orphan, precisa do selector com chave e valor da labels do DaemonSet

```bash
selector:
    matchLabels:
      apps: my-app
```

# Aplique novamente o arquivo.yml e verifique o minikube-m03 novamente que ele vai criar e adotar o restante.
```bash
kubectl apply -f daemonset.yml
kubectl get nodes

# Deletando tudo
kubectl delete ds my-daemonset
```

#
### Por padrão o DaemonSet cria 1 pod em cada node assim que o daemonset é iniciado, mas pode mudar isso e selecionar através de *Node Labels* que seria limitar os daemonset pods a nodes específico do nosso cluster

### Exemplo utilizando o DaemonSet: Uma aplicação rodando em um node que esta o backend e outro node que esta o frontend. Se quiser fazer o monitoramento apenas do node backend, utiliza o daemonset pod com uma configuração de monitoramento e aponte para um node específico. Para a criação de um pod em um node específico, será necessário que um node tenha a label *ssd* e para que isso seja necessário, tem que adicionar um rótulo *diktype: ssd* ao node:
```bash
# no arquivo daemonset adicone o trecho a baixo na ma hierarquia do container
containers:
    - name: my-container-nginx
    image: nginx
nodeSelector: 
    disktype: ssd


# Escolha um node, neste caso vou utilizar o minikube-m03
kubectl get nodes

# adicione a label
kubectl label nodes minikube-m03 disktype=ssd

# saída esperada: 
*node/minikube-m03 labeled*

# use o comando para verificar 
kubectl get node --show-labels | grep disktype

# se retornar a saída abaixo, deu tudo certo:
minikube-m03   Ready    <none>          10h   v1.31.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,*disktype=ssd*,kubernetes.io/arch=amd64,kubernetes.io/hostname=minikube-m03,kubernetes.io/os=linux,minikube.k8s.io/commit=210b148df93a80eb872ecbeb7e35281b3c582c61,minikube.k8s.io/name=minikube,minikube.k8s.io/primary=false,minikube.k8s.io/updated_at=2024_11_07T13_17_18_0700,minikube.k8s.io/version=v1.34.0

# pode aplicar novamente o arquivo daemonset.yml e note que o pod só vai ser criado no node minikube-m03
kubectl apply -f daemonset.yml

# verifique o daemonset e terá essa saída:
kubectl get ds

NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
my-daemonset   1         1         1       1            1           disktype=ssd     3s


```

# É possível direcionar um pod diretamente sem a utilização das labels, que seria a forma de *Direct Attribution* ao invés de ser nodeSelect, *seria nodeName: nome-do-node*

```bash

containers:
     - name: my-container-nginx
    image: nginx
    # nodeSelector: 
    #   disktype: ssd
    nodeName: minikube

# aplique o arquivo do daemonset e a sáida esperada é essa
kubectl get ds
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
my-daemonset   3         3         1       3            1           <none>          16s

# Note que tem apenas 1 READY, sigfica que deu tudo certo, apenas um pod foi adicionado ao node específico através de nodeName: xx

```