# Utilizando o configmap é possivel criar arquivos como já foi criado, cada configuração que é colocado dentro de data no arquivo manifesto, é criado um arquivo dentro do volume informad igual que já possui, database e database_uri

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: my-cm # <-----
data: 
  # cria um arquivo no /etc/my-vol
  database: mariadb

  # cria um arquivo no /etc/my-vol
  database_uri: mariadb://localhost:3306
```

# Se adicionar 10 configurações, chaves valores dentro de *data:* será criado 10 arquivos de configuração. Porém, é possivel criar um único arquivo e dentro dele é contido diversas informações da seguinte maneira:

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: my-cm # <-----
data: 
  my.confing.db: |  
      database: mariadb
      database_uri: mariadb://localhost:3306
```

# Verificando se criou um único arquivo com duas informações dentro do arquivo, no seguinte diretório, /etc/my-vol
```bash
kubectl apply -f my-cm.yml
kubectl apply -f pod-cm-vol.yml

kubectl get pod
kubectl exec -it pod-cm-vol -- bash
cd /etc/my-vol
ls -al 

# saída esperada 
drwxrwxrwx 3 root root 4096 Nov 10 16:12 .
drwxr-xr-x 1 root root 4096 Nov 10 16:03 ..
drwxr-xr-x 2 root root 4096 Nov 10 16:12 ..2024_11_10_16_12_42.351940405
lrwxrwxrwx 1 root root   31 Nov 10 16:12 ..data -> ..2024_11_10_16_12_42.351940405
lrwxrwxrwx 1 root root   20 Nov 10 16:03 my.confing.db -> ..data/my.confing.db

cat my.confing.db

# saída esperada: 
database: mariadb
database_uri: mariadb://localhost:3306
```
# É possível adicionar mais chave e valor que não seja um arquivo de propriedade, basta apenas adicionar um campo dessa maneira: 

```bash

my.confing.db: |  
      database: mariadb
      database_uri: mariadb://localhost:3306
test.field: eaii!!

# precisa esta aninhado junto com o my.confing.db se não o kubernetes vai entender que o test.field também é uma propriedade do arquivo my.confing.db. Basta aplicar novamente o arquivo my-cm.yml e aguardar alguns segundos para que atualize e entre no diretório novmante.

kubectl apply -f my-cm.yml
kubectl get pod
kubectl exec -it pod-cm-vol -- bash
cd /etc/my-vol
ls -al 

#saída esperada
drwxrwxrwx 3 root root 4096 Nov 10 16:12 .
drwxr-xr-x 1 root root 4096 Nov 10 16:03 ..
drwxr-xr-x 2 root root 4096 Nov 10 16:12 ..2024_11_10_16_12_42.351940405
lrwxrwxrwx 1 root root   31 Nov 10 16:12 ..data -> ..2024_11_10_16_12_42.351940405
lrwxrwxrwx 1 root root   20 Nov 10 16:03 my.confing.db -> ..data/my.confing.db
lrwxrwxrwx 1 root root   17 Nov 10 16:12 test.field -> ..data/test.field

cat my.confing.db && printf "\n" && cat test.field && printf "\n"

# saída esperada: 
database: mariadb
database_uri: mariadb://localhost:3306

eaii!!
```

# Deletando tudo
```bash
kubectl get cm 
kubectl delete cm my-cm
kubectl get pod
kubectl delete pod pod-cm-vol
``` 
