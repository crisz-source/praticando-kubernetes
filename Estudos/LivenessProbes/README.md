# LivenessProbe verifica se a aplicação está com algum tipo de problema, e crie verificação de sanidade da aplicação para verificar se esta tudo certo. Quando aplicação der algum tipo de problema, o livenessprobes vai agir; 

#### Então livenesse probe sao verificações de sanidade que são executado pelo agent kubelet, que faz testes contínuos do funcionamento da aplicação.


# Assim que o container for inciado, ele vai ser executado a seguinte linha de comando:
```bash
touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
```

# Basicamente vai ser criado o arquivo o healthy, e depois que criado, ele vai aguardar 30 segundos, e nesses 30 segundos, o livenessProbe vai ser iniciado com as configurações especificado como comentário no arquivo livenessprob.yml


# Testando o funcionamento do LivenessProb (copie tudo e cole no terminal os comados):
```bash
kubectl apply -f livenessprob.yml && \ 
sleep 5 && \ 
kubectl get pods && \  
sleep 30 && \ 
kubectl describe pod liveness-pod && \ 
sleep 35 && \ 
kubectl describe pod liveness-pod && \
sleep 30 && \ 
kubectl describe pods liveness-pod && \ 
kubectl get pods liveness-pod
```

