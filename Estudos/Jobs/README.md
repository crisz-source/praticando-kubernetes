## As jobs serão responsáveis para executar algum tipo de ação com curta tarefa.

# Aplique o arquivo para criação do pod
```bash
kubectl apply -f my-pod.yml

# verifique o job
kubectl get job
```
# e veja que a criação dele foi com sucesso e job criado, verifique as logs do pod e veja os número aleatorio que foi gerado
```bash
kubectl get pod
kubectl logs <nome-do-pod>
```

# As demais explicações está no arquivo yml

# Saídas de erros de um container que esta dentro de um pod que esta dentro de um job
