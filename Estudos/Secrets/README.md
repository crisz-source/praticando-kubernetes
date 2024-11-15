# As secrets é a mesma coisa que um ConfigMap. Todas as explicações, uso do configmap esta no diretório de ConfigMaps, pois os dois objetos são semelhantes. 

### Para criação de secrets, os valores e chaves precisam ter condificações em base64, se não tiver nenhuma codificação na base64, verá um erro parecido:

```bash
# Criei um arquivo para demostrar o erro:

apiVersion: v1
kind: Secret 
metadata:
  name: my-secret
data: # as configurações de chaves e valors a partir dessa linha, precisam e devem ser codificados em base64
  user: admin
  pass: My-pass-123

# Ao aplicar esse manifest, me identifiquei com este erro abaixo:
Error from server (BadRequest): error when creating "my-secret.yml": Secret in version "v1" cannot be handled as a Secret: illegal base64 data at input byte 2
```
# Gerando codificação em base64 e decodificando
```bash
echo -n 'admin' | base64 
echo -n 'My-pass-123' | base64

# Saída esperdada
YWRtaW4=
TXktcGFzcy0xMjM=

# Decodificar a base64
echo -n 'YWRtaW4=' | base64 --decode && printf "\n" && echo -n 'TXktcGFzcy0xMjM=' | base64 --decode && printf "\n"

#Saída esperada:
admin
My-pass-123

```