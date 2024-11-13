# configmaps, serve para gerenciar toda a parte que configura a aplicação como um todo
### Para criar um configmap e configurar um pod para utilizar um configmap, é necessário criar um configmap PRIMEIRO e depois o pod

# aplicando o arquivo yml 
```bash
kubectl apply -f my-cm-test-env.yml 

# Verficando os configmap
kubectl get cm

# verificando a descrição do configmap 
kubectl describe cm my-cm

# Verificando dentro do container se as variáveis de ambiente defininda no arquivo de manifesto estão la
```bash
kubectl get pod
kubectl exec -it pod-cm-env -- bash
env

# saída esperada:

font.title=Arial
theme.1=clean
1theme.2=dark
background-color=red
KUBERNETES_SERVICE_PORT_HTTPS=443
database_uri=mongodb://localhost:27017
KUBERNETES_SERVICE_PORT=443
HOSTNAME=pod-cm-env
PWD=/
PKG_RELEASE=1~bookworm
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
database=mongodb
DYNPKG_RELEASE=1~bookworm
NJS_VERSION=0.8.6
TERM=xterm
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NGINX_VERSION=1.27.2
NJS_RELEASE=1~bookworm
_=/usr/bin/env

```

### Qualquer atualização que for feito em um configmap, será necessário deletar um pod que esta atrelado a ele pois o pod não atualiza em tempo real. 

### Caso deseje realizar uma atualização do configmap sem a necessidade de deletar um pod, pode-se usar um recurso de ReadOnly Volume. Pois, cada atualização feita em um ConfigMap o pod será atualizado em " tempo real". Diferente de um configMap padrão, ele requer que você crie um confimap primeior e depois um pod, utilizando o ReadOnly Volume, ele permite que você crie um pod primeiro e depois um ConfigMap. Assim que o pod esteja criado, ele fica aguardando um configmap a ser criado.

### Aplique o arquivo pod-env.yml e verifique o que esta acontencedo
```bash
kubectl apply -f pod-cm-env.yml
kubectl get pod 

# Saída esperada com o erro:
NAME         READY   STATUS                       RESTARTS   AGE
pod-cm-env   0/1     CreateContainerConfigError   0          16s

# Note que o status do container esta com erro, indica que ele esta esperando uma configuração do configmap

kubectl describe pod pod-cm-env

# Saída esperada: 

Normal   Pulled     2m12s                kubelet            Successfully pulled image "nginx" in 7.778s (7.779s including waiting). Image size: 191670474 bytes.
Normal   Pulled     2m10s                kubelet            Successfully pulled image "nginx" in 1.399s (1.399s including waiting). Image size: 191670474 bytes.
....

Warning  Failed     45s (x8 over 2m12s)  kubelet            Error: configmap "my-cm" not found

# Note que o pod tem um warning informando que não encontrou o configmap, criei dois arquivos: pod-cm-vol.yml e my-cm.yml para que entre na ideia de, assim que o pod for criado ele aguarde o configmap a ser criado depois e que não ocasione o erro acima
kubectl apply -f pod-cm-vol.yml
kubectl get pod

# saída esperada
NAME         READY   STATUS              RESTARTS   AGE
pod-cm-vol   0/1     ContainerCreating   0          31s

# Note que o contaniner esta no estatos ContainerCreating, quando você verifica a descricação do pod acontece isso:
kubectl describe pod pod-cm-vol

# saída esperada:

Events:
  Type     Reason       Age                From               Message
  ----     ------       ----               ----               -------
  Normal   Scheduled    88s                default-scheduler  Successfully assigned default/pod-cm-vol to minikube
  Warning  FailedMount  25s (x8 over 89s)  kubelet            MountVolume.SetUp failed for volume "my-vol" : configmap "my-cm" not found

  # Note que o tipo do evento está Com um warning que o Reason esta FailedMount, e a mensagem informando que falhou em criar o volume my-vol e não encontrou o ConfigMap my-cm, ele basicamente esta esperando a criação do ConfigMap para que o Volume "my-vol" seja associado a ele e por fim, criar o container corretamente

  kubectl apply -f my-cm.yml
  kubectl get pod --watch # use o --watch e aguarde a criação

# saída esperada:
NAME         READY   STATUS              RESTARTS   AGE
pod-cm-vol   0/1     ContainerCreating   0          5m52s
pod-cm-vol   1/1     Running             0          6m15s


# Verifique dentro do container se esta tudo correto, se o volume foi criado e se os dados também
kubectl exec -it pod-cm-vol -- bash
env 

# saída esperada: 
KUBERNETES_SERVICE_PORT_HTTPS=443 
database_uri=mysql://localhost:3306 # aqui esta a configuração definida no configmap
KUBERNETES_SERVICE_PORT=443
HOSTNAME=pod-cm-vol
PWD=/
PKG_RELEASE=1~bookworm
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
database=mysql # aqui esta a configuração definida no configmap
DYNPKG_RELEASE=1~bookworm
NJS_VERSION=0.8.6
TERM=xterm
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NGINX_VERSION=1.27.2
NJS_RELEASE=1~bookworm
_=/usr/bin/env

# Agora, verifique se o volume foi criado com sucesso e as variáveis de ambiente dentro do volume
cd /etc/my-vol
ls -al 

# saída esperada

total 16
drwxrwxrwx 3 root root 4096 Nov 10 15:24 .
drwxr-xr-x 1 root root 4096 Nov 10 15:24 ..
drwxr-xr-x 2 root root 4096 Nov 10 15:24 ..2024_11_10_15_24_20.1839200175
lrwxrwxrwx 1 root root   32 Nov 10 15:24 ..data -> ..2024_11_10_15_24_20.1839200175
lrwxrwxrwx 1 root root   15 Nov 10 15:24 database -> ..data/database
lrwxrwxrwx 1 root root   19 Nov 10 15:24 database_uri -> ..data/database_uri

#  Verifique se o conteúdo dessas variáveis de ambiente esteja correto
cat database && printf "\n" && cat database_uri && printf "\n"

# Saída esperada:
mysql
mysql://localhost:3306

```

# Atualização do configmap sem a necessidade de apagar o pod

```bash
# comentei as linhas do mysql para fins didáticos e adicione o mariadb para realizar a atualização

kind: ConfigMap
apiVersion: v1
metadata:
  name: my-cm # <-----
data: # definições de chaves e valor que vai configurar o container
  # Connection database config
  # database: mysql
  # database_uri: mysql://localhost:3306
  database: mariadb
  database_uri: mariadb://localhost:3306

  # aplique o arquivo
  kubectl apply -f my-cm.yml
  # configmap/my-cm configured

  # Agora, verifique se o volume foi criado com sucesso e as variáveis de ambiente dentro do volume
cd /etc/my-vol
ls -al 

# saída esperada

total 16
drwxrwxrwx 3 root root 4096 Nov 10 15:34 .
drwxr-xr-x 1 root root 4096 Nov 10 15:24 ..
drwxr-xr-x 2 root root 4096 Nov 10 15:34 ..2024_11_10_15_34_31.3465711416
lrwxrwxrwx 1 root root   32 Nov 10 15:34 ..data -> ..2024_11_10_15_34_31.3465711416
lrwxrwxrwx 1 root root   15 Nov 10 15:24 database -> ..data/database
lrwxrwxrwx 1 root root   19 Nov 10 15:24 database_uri -> ..data/database_uri

#  Verifique se o conteúdo dessas variáveis de ambiente esteja correto, Pode acontencer demorar um pouco a questão de atualização, a cada 1 minuto dê o comando de printf e veja a mudança
cat database && printf "\n" && cat database_uri && printf "\n"

# Saída esperada:
mariadb
mariadb://localhost:3306

```

#### Caso você queira veriricar as variável de ambiente pelo comando *printenv database && printenv database_uri* vai identificar que as variáveis ainda estão com mysql, para que essas variáveis sejam atualizadas também aí sim teria que reiniciar o pod ou criar um novo. Por isso que utilizei o volume, basta que a aplicação leia o conteúdo desse volume, desses arquivos e a aplicação estará sempre atualizada

# Deletando tudo
```bash
kubectl get cm
kubectl delete cm my-cm
kubectl get pod
kubectl delete pod pod-cm-vol
``` 


### Sobre o arquivo my-cm-test-command.yml, basicamente estava testando acessar o configmap através de um comando dentro do container, basta aplicar o arquivo my-cm-test-command.yml e verfifcar as logs do pod que vai ter o resultado esperado: *My Database = mysql - mysql://localhost:3306*