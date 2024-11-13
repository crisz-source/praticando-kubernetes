# StatefulSets é muito parecido com os deployment e replicaset
### Para criar um statefulset é necessário ter um PersistentVolume e um service, o PV seria mais para armazanemanto dos dados pois um PV é independente do pod, se um pod for deletado por exemplo, os volumes vão persistir e manter vivo. Já o service, é para gerenciar a questao de redes, se um pod for trocado graças aos services, poderá acessar esse novo pod pelo memso DNS

### Realizando teste de DNS dos pods

## Aplique o arquivo manifesto e siga as instruções
```bash
kubectl apply -f svc-sts.yml
kubectl get pod

# Saída esperada: 

NAME       READY   STATUS    RESTARTS   AGE
my-sts-0   1/1     Running   0          30s
my-sts-1   1/1     Running   0          8s
my-sts-2   1/1     Running   0          7s

# Crie um novo pod com um nome diferente, e dentro do pod será necessário de 2 ferramentas para fazer o teste, curl e o dns-utils
kubectl run -it --image debian network-id-test

# Já dentro do container, digite: 
apt update && apt install curl dnsutils -y 

# utilizando o lookup
nslookup svc-sts

# Saída esperada dentro do container: 

Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   svc-sts.default.svc.cluster.local
Address: 10.244.0.17
Name:   svc-sts.default.svc.cluster.local
Address: 10.244.0.16
Name:   svc-sts.default.svc.cluster.local
Address: 10.244.0.15
;; Got recursion not available from 10.96.0.10

# teste o curl com o nome do dns
curl svc-sts.default.svc.cluster.local


# Saída esperada:
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.23.1</center>
</body>
</html>

```

#### Qualquer deleção dos pods, eles vão ser recriados novamente com o mesmo nome, porém com ip diferentes e com o mesmo DNS. Se tentar utilizar o curl pelo ip, não vai conseguir porque a cada recriação de pods, eles são criados com IP's diferentes porém, mantém o mesmo nome de DNS. 


### Vericando os persistenVolume
```bash
kubectl get pv

# saída esperada
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-15b39343-f86a-4e21-9152-efb2516eab35   1Gi        RWO            Delete           Bound    default/my-pv-my-sts-2   standard       <unset>                          3h31m
pvc-2e3eb299-56e1-4f70-a5d5-bbb3fc415846   1Gi        RWO            Delete           Bound    default/my-pv-my-sts-3   standard       <unset>                          3h30m
pvc-68bf5c11-676a-4901-a342-a6419ccd96bb   1Gi        RWO            Delete           Bound    default/my-pv-my-sts-4   standard       <unset>                          3h30m
pvc-94b8e652-8cb0-4846-9251-ed44c89bc393   1Gi        RWO            Delete           Bound    default/my-pv-my-sts-1   standard       <unset>                          3h31m
pvc-a09f16c6-8c41-499a-8a88-d69e431263c4   1Gi        RWO            Delete           Bound    default/my-pv-my-sts-0   standard       <unset>                          3h31m

## CAPACITY = 1Gi foi definido no manifesto
## ACCESS MODES = RWO Foi definido no manifesto
## RECLAIM POLICY = Delete, esse Reclaim Policy delete significa que esse PV será deletado se o PersistentVolumeClaim for deletado. 
## STATUS = Bound, indica que esta vinculado a um determinado pod
## CLAIM = default/my-pv-my-sts-2, basicamente esta informando que o pv esta vinculado a um pod da namespace default. <namespace>/ <vol. name> - <Pode name>
## STORAGECLASS = Standard, definindo o tipo de armazenamento que a aplicação vai utilizar. O kubernetes permite utilizar diversos tipos de armazenamento, ex: Google cloud storage. 

```

### Verificando os persistenVolumeClaim
```bash
kubectl get pvc

# saída esperada: 

NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
my-pv-my-sts-0   Bound    pvc-a09f16c6-8c41-499a-8a88-d69e431263c4   1Gi        RWO            standard       <unset>                 3h40m
my-pv-my-sts-1   Bound    pvc-94b8e652-8cb0-4846-9251-ed44c89bc393   1Gi        RWO            standard       <unset>                 3h40m
my-pv-my-sts-2   Bound    pvc-15b39343-f86a-4e21-9152-efb2516eab35   1Gi        RWO            standard       <unset>                 3h40m
my-pv-my-sts-3   Bound    pvc-2e3eb299-56e1-4f70-a5d5-bbb3fc415846   1Gi        RWO            standard       <unset>                 3h39m
my-pv-my-sts-4   Bound    pvc-68bf5c11-676a-4901-a342-a6419ccd96bb   1Gi        RWO            standard       <unset>                 3h39m


## Os volumes estão sendo armazenados nesse seguinte diretório:
sudo su
cd /var/lib/docker/volumes/minikube/_data/hostpath-provisioner/default
ls

# saída esperada:
drwxrwxrwx 2 root root 4096 Nov 10 18:00 my-pv-my-sts-0
drwxrwxrwx 2 root root 4096 Nov 10 18:00 my-pv-my-sts-1
drwxrwxrwx 2 root root 4096 Nov 10 18:01 my-pv-my-sts-2
drwxrwxrwx 2 root root 4096 Nov 10 18:01 my-pv-my-sts-3
drwxrwxrwx 2 root root 4096 Nov 10 18:01 my-pv-my-sts-4

```

### Deletando PV e PVC, para deletar algum pv precisa deletar primeiramente um pod e depois um pvc. Pois o kubernetes tem o *Storage Objetct in Use Protection*, siginifica que enquanto um pod estiver utilizando um pvc ele não vai ser deletado

```bash
kubectl get pvc
kubectl delete pvc my-pv-my-sts-0  my-pv-my-sts-1 my-pv-my-sts-2 my-pv-my-sts-3 my-pv-my-sts-4

# Ou pode se utilizar
kubectl delete pvc --all # o recomendado é deletar um por vez!

# saída esperada:
persistentvolumeclaim "my-pv-my-sts-0" deleted
persistentvolumeclaim "my-pv-my-sts-1" deleted
persistentvolumeclaim "my-pv-my-sts-2" deleted
persistentvolumeclaim "my-pv-my-sts-3" deleted
persistentvolumeclaim "my-pv-my-sts-4" deleted

# Verificando que não tem mais pv e nem pvc
kubectl get pv
kubectl get pvc
```


# Conceitos básicos sobre PV e PVC:
#### Basicamente, os pv (PersistentVolume) são volumes físicos ou lógicos persistentes independente dos pods, então tanto faz se um pod ou container falhar, reiniciar... que os volumes vão se manter no cluster. Já os PVC (PersistentVolumeClaim) são volumes "pedido" que os pods pedem para o kubernetes, esses pvc define como será um pv vinculado a um pvc. As definições são tipos de permissões que um pv terá e tamanho dos volumes, quando um pvc se "conecta" com um pv, eles ficam vinculados um ao outro com o status de (boundou ou bind). Os pvc também são independentes dos pods, o bom que se um pod falhar ou ser deletado, os dados vão permanecer no cluster e quando levantar mais um pod, o pvc vai garantir que esse novo pod se conecte aos volumes. 

### Resumo:
- O PV é o armazenamento físico ou lógico persistente que existe no cluster.
- O PVC é a forma como os Pods solicitam o armazenamento (especificando requisitos como tamanho e tipo de acesso).
- Se um Pod for deletado, os dados continuam no PV, pois o PVC mantém o "vínculo" com o volume.
- O PV é independente dos Pods, mas o PVC é a chave que vincula um Pod a um PV específico.

# Exemplo prático de PV e PVC
```bash
# Exemplo de um PV 
#O PV é o recurso que representa o armazenamento físico ou lógico que será utilizado por um Pod. O Kubernetes vai provisionar esse volume e ele ficará disponível para uso por qualquer PVC que faça uma solicitação compatível.
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi  # Tamanho do volume
  accessModes:
    - ReadWriteOnce  # Modo de acesso, por exemplo, leitura e escrita por um único nó
  persistentVolumeReclaimPolicy: Retain  # O que acontece quando o PVC for deletado (opções: Retain, Delete, Recycle) O Retain significa que o volume não será apagado automaticamente.
  storageClassName: standard  # A StorageClass que será usada (pode ser customizada ou padrão)
  hostPath:
    path: /mnt/data  # Local no nó para o armazenamento físico (exemplo usando armazenamento local)
---
# Exemplo de um PVC
# O PVC é o pedido que o Pod faz para alocar um volume persistente, com base nas suas necessidades (tamanho e tipo de acesso). O Kubernetes vai buscar um PV que atenda a esses requisitos e fazer o "binding" entre o PVC e o PV.
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce  # Acesso de leitura e escrita por um único nó
  resources:
    requests:
      storage: 5Gi  # Tamanho solicitado para o volume
  storageClassName: standard  # A StorageClass que deve ser utilizada
---
# Exemplo deum pod que usa o pvc
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  containers:
  - name: my-app
    image: nginx  # Exemplo com a imagem do Nginx, mas pode ser qualquer imagem
    volumeMounts:
    - name: my-storage
      mountPath: /usr/share/nginx/html  # O local onde o volume será montado no container
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc  # O PVC que será usado pelo Pod
```

# Como o PV e o PVC se vinculam?
### O Kubernetes não exige que o PV saiba qual PVC irá usá-lo. O processo de vinculação entre o PVC e o PV acontece automaticamente através de um processo chamado Binding, que é gerenciado pelo próprio Kubernetes.
- Quando o PVC é criado, o Kubernetes verifica se há um PV disponível que atenda aos requisitos definidos no PVC (como storage, accessModes, e storageClassName).

- O Kubernetes então faz o binding entre o PVC e o PV. Isso significa que o PVC se vincula ao PV adequado e esse volume será "reservado" para o PVC. Assim, qualquer Pod que utilizar esse PVC conseguirá acessar o armazenamento.

- Não é necessário especificar o PVC no PV. O PV fica disponível no cluster para que o Kubernetes faça o binding automaticamente com o PVC que atender aos requisitos.

# Conceito básico sobre StorageClass
#### Define o tipo e as características do volume que o Kubernetes criará automaticamente quando um PVC for solicitado. Ele é usado para controlar o tipo de armazenamento (SSD, HDD, cloud storage, etc.), além de parâmetros de provisionamento como desempenho, replicação e local.

# Exemplo prático de StorageClass
```bash
# StorageClass para EBS da AWS (volume ssd)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-test-aws
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2  # Tipo de volume SSD da AWS
  fsType: ext4
reclaimPolicy: Retain  # Não apaga o volume quando o PVC é deletado
volumeBindingMode: WaitForFirstConsumer  # Cria o volume apenas quando o Pod for agendado

# PVC que usa o StorageClass storage-test-aws
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi  # Tamanho solicitado
  storageClassName: storage-test-aws  # Refere-se ao StorageClass "fast-storage"

```