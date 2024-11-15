# Volumes, como o kubernetes tem diversos tipos de volumes... utilizei 2 tipos de volumes, emptyDir e hostPath

# Volume emptyDir, esse volume é criado vazio em um container, toda vez que o pod for reiniciado, os dados serão perdidos

# Entrando no pod com o volume emptyDir. 
```bash
 kubectl exec -it redis-pod -- bash
 ```
 
# Dentro do contianer do redis, entre no volume /my-volume e crie um arquivo
```bash
 cd /my-volume 
 echo "Deu certo!" > test.txt
 cat test.txt
 

# Atualize o repositorio 
 apt update
 apt install procps -y

# verifica os processos
ps aux

# mate o processo do redis e depois verifique se o arquvio criado anteriormente ainda persiste, se persistir, funcionou! Ou seja, o emptyDir só vai ficar vazio novamente quando reiniciar ou refazer o pod. Para matar o processo do redis, execute: 
kill redis PID-DO-REDIS

# Note que o kill vai derruba-lo do pod, basta entrar novamente e verifificar o diretório criado.
 kubectl exec -it redis-pod -- bash
  cd /my-volume 

# Delete pod e aplica o arquivo novamente e verifica o volume, caso estiver vázio a explicação do emptyDir está correta!
kubectl get pod
kubectl delete pod redis-pod
kubectl apply -f volumes.yml
kubectl exec -it redis-pod -- bash
cd /my-volume

# Usando volumes persistente, hostPath
### O hostPath acredito que deve ser o mais utilizado para um único node, pois ele é específico de um unico node e não é recomendado utiliza-lo em outros nodes

# Testando o volume
 kubectl exec -it redis-pod -- bash
cd ../my-data
echo "deu certo o volume hostpath" > test.txt
cat test.txt

#### Mate o processo do container da forma que foi feito anteriormente, terá que atualizar o repositório e instalar o procps e verifica se deu tudo certo, caso o arquivo test.txt estiver dentro do container assim que ele for inciado novamente, deu tudo certo! Delete o pod também e verifique.
kubectl get pod
kubectl delete pod redis-pod
kubectl apply -f volumes-hostpath.yml
kubectl exec -it redis-pod -- bash
cd ../my-data
ls
cat test.txt

# Com o teste feito, você pode verificar se o arquvio test.txt esta no seu worker node, basta ir neste diretório abaixo que a pasta 2-persist criada estará lá:
sudo su
cd /var/lib/docker/volumes/minikube/_data/lib/
ls
```
### Todo tipo de arquivos, dados que serão criados ou gravados dentro do diretório /my-data eles aparecerão também no worker node neste caminho: *cd /var/lib/docker/volumes/minikube/_data/lib/2-persist/*


# Mais sobre os volumes, está na pasta *StatefulSets* que aborda mais sobre PV, PVC, SC