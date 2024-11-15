# Projeto: Jupyter Notebook Application
### Este projeto é uma aplicação Jupyter Notebook implantada em um cluster Kubernetes, criada com o objetivo de aprender e praticar conceitos fundamentais do Kubernetes. A aplicação Jupyter Notebook é executada em um cluster Kubernetes local configurado com o Minikube. Com esse ambiente, foi possível explorar o gerenciamento de recursos, a definição de configurações sensíveis com secrets, a criação de volumes persistentes e o uso de namespaces para organizar os recursos do cluster.

## Após a instalação do minikube, para realizar a criação desse projeto eu separei por 10 fases:
- **Projeto Fase 1** - Manifesto Namespace & Annotations
- **Projeto Fase 2** - Manifesto Deployment
- **Projeto Fase 3** - Implantando Aplicação
- **Projeto Fase 4** - Verificando Aplicação
- **Projeto Fase 5** - Testando Aplicação (teste de funcionalidade)
- **Projeto Fase 6** - Migrando Aplicação para Service
- **Projeto Fase 7** - Migrando Porta da Aplicação
- **Projeto Fase 8** - Escalando Aplicação  
- **Projeto Fase 9** - Atualizando Aplicação
- **Projeto Fase 10** - Analise Final e Remoção da aplicação

## **Entre no diretório Projeto para aplicação dos arquivos manifesto.**
# Projeto fase 1:
```bash
# Inicie o minikube:
minikube start


# Criação da namespace, na raiz da pasta Projeto aplique o manifest
kubectl apply -f ./namespace/jupyter-ns.yml
```

# Projeto fase 2 & 3: 
```bash
# Criação do deployment, na raiz da pasta Projeto e aplique o manifest
kubectl apply -f ./deployment/jupyter-deploy.yml

# Verificando o deployment, aguarde a criação do container pois a imagem possui do Jupyter é de 2GB
kubectl get pods -n jupyter -w
```

# Projeto fase 4:
```bash
# 1) - Verificando a aplicação
kubectl describe ns jupyter

# Saída esperada da descricação do namespace jupyter:
Name:         jupyter
Labels:       kubernetes.io/metadata.name=jupyter
Annotations:  desc: Namespace for Jupyter Notebook Application
              owner: Cristhian Ramos
              phone: 123-456-789
              proj: Final project K8s Studies
Status:       Active

# -------------------------------

# 2) - Verificando o deployment
kubectl get deploy -n jupyter

# Saída esperada do deployment:
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
jupyter-deploy   1/1     1            1           6m24s

# OPCIONAL! Caso deseje mudar o namespace padrão para o jupyter sem a necessidade de ficar digitando -n <nome-da-namespace>, faça o seguinte:
kubectl config set-context --current --namespace=jupyter
kubectl get deploy # testando sem o -n jupyter

# -------------------------------

# 3) - Verificando as namespaces presentes no cluster
kubectl get ns

# Saída esperada de todas as namespaces do cluster:
NAME              STATUS   AGE
default           Active   16m
jupyter           Active   14m
kube-node-lease   Active   16m
kube-public       Active   16m
kube-system       Active   16m

# -------------------------------

# 4) - Verificando as replicaset
kubectl get rs -n jupyter

# Saída esperada das replicasets
NAME                        DESIRED   CURRENT   READY   AGE
jupyter-deploy-6cbfc49c6c   1         1         1       12m

# -------------------------------

# 5) - Verificando os pods
kubectl get pod -n jupyter

# saída esperada dos pods:
NAME                              READY   STATUS    RESTARTS   AGE
jupyter-deploy-6cbfc49c6c-gwqfc   1/1     Running   0          15m

# Verifcando as logs do pod
kubectl logs jupyter-deploy-6cbfc49c6c-gwqfc -n jupyter

# saída esperada das logs do container:
Entered start.sh with args: jupyter lab --NotebookApp.token=''
Executing the command: jupyter lab --NotebookApp.token=''
[W 2024-11-11 03:43:12.681 LabApp] 'token' has moved from NotebookApp to ServerApp. This config will be passed to ServerApp. Be sure to update your config before our next release.
/opt/conda/lib/python3.10/site-packages/traitlets/traitlets.py:2412: FutureWarning: Supporting extra quotes around strings is deprecated in traitlets 5.0. You can use '' instead of "''" if you require traitlets >=5.
  warn(
[I 2024-11-11 03:43:12.689 ServerApp] jupyterlab | extension was successfully linked.
[W 2024-11-11 03:43:12.693 NotebookApp] 'ip' has moved from NotebookApp to ServerApp. This config will be passed to ServerApp. Be sure to update your config before our next release.
[W 2024-11-11 03:43:12.693 NotebookApp] 'port' has moved from NotebookApp to ServerApp. This config will be passed to ServerApp. Be sure to update your config before our next release.
[W 2024-11-11 03:43:12.693 NotebookApp] 'port' has moved from NotebookApp to ServerApp. This config will be passed to ServerApp. Be sure to update your config before our next release.
[I 2024-11-11 03:43:12.699 ServerApp] nbclassic | extension was successfully linked.
[I 2024-11-11 03:43:12.703 ServerApp] Writing Jupyter server cookie secret to /home/jovyan/.local/share/jupyter/runtime/jupyter_cookie_secret
[I 2024-11-11 03:43:13.023 ServerApp] notebook_shim | extension was successfully linked.
/opt/conda/lib/python3.10/site-packages/traitlets/traitlets.py:2412: FutureWarning: Supporting extra quotes around strings is deprecated in traitlets 5.0. You can use '' instead of "''" if you require traitlets >=5.
  warn(
[W 2024-11-11 03:43:13.041 ServerApp] All authentication is disabled.  Anyone who can connect to this server will be able to run code.
[I 2024-11-11 03:43:13.043 ServerApp] notebook_shim | extension was successfully loaded.
[I 2024-11-11 03:43:13.044 LabApp] JupyterLab extension loaded from /opt/conda/lib/python3.10/site-packages/jupyterlab
[I 2024-11-11 03:43:13.044 LabApp] JupyterLab application directory is /opt/conda/share/jupyter/lab
[I 2024-11-11 03:43:13.046 ServerApp] jupyterlab | extension was successfully loaded.
[I 2024-11-11 03:43:13.049 ServerApp] nbclassic | extension was successfully loaded.
[I 2024-11-11 03:43:13.050 ServerApp] Serving notebooks from local directory: /home/jovyan
[I 2024-11-11 03:43:13.050 ServerApp] Jupyter Server 1.23.3 is running at:
[I 2024-11-11 03:43:13.050 ServerApp] http://jupyter-deploy-6cbfc49c6c-gwqfc:8888/lab
[I 2024-11-11 03:43:13.050 ServerApp]  or http://127.0.0.1:8888/lab
[I 2024-11-11 03:43:13.050 ServerApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
```
# Projeto fase 5: 
```bash
# Testando Aplicação, fazendo um forward para a porta do container que é 8888
# Acessando o container sem o service
kubectl get pod -n jupyter

# Mapeando a porta local para a porta do container
kubectl port-forward  jupyter-deploy-6cbfc49c6c-gwqfc 8080:8888 -n jupyter

# Saída esperada após o mapeamento:
Forwarding from 127.0.0.1:8080 -> 8888
Forwarding from [::1]:8080 -> 8888

# Entre no seu navegador na seguinte url:
http://localhost:8888/lab


# ?????????

```

# Projeto fase 6:
```bash
# Criação do service, na raiz da pasta Projeto e aplique o manifest, e tente entrar na url localhost:30090
kubectl apply -f ./service/jupyter-svc.yml

# Caso não consiga entrar na url localhost30090, execute o comando abaixo
minikube service jupyter-service -n jupyter # expondo o service pelo minikube

# ----------------------------------------

# Criação de service pela linha de comando, verifque o nome do deployment
kubectl get deploy -n jupyter

# Criando service
kubectl expose deploy/jupyter-deploy -n jupyter --type=NodePort --name=jupyter-svc

# Verificando o arquivo yaml e json criado do service:
kubectl get svc jupyter-svc -n jupyter -o yaml
kubectl get svc jupyter-svc -n jupyter -o json

# Obtendo a url do serviço para testar a aplicação e serviço
minikube service jupyter-svc -n jupyter

# Saída esperada:
|-----------|-------------|-------------|---------------------------|
| NAMESPACE |    NAME     | TARGET PORT |            URL            |
|-----------|-------------|-------------|---------------------------|
| jupyter   | jupyter-svc |        8888 | http://192.168.49.2:31440 |
|-----------|-------------|-------------|---------------------------|
🏃  Starting tunnel for service jupyter-svc.
|-----------|-------------|-------------|------------------------|
| NAMESPACE |    NAME     | TARGET PORT |          URL           |
|-----------|-------------|-------------|------------------------|
| jupyter   | jupyter-svc |             | http://127.0.0.1:34707 |
|-----------|-------------|-------------|------------------------|
🎉  Opening service jupyter/jupyter-svc in default browser...
👉  http://127.0.0.1:34707
❗  Because you are using a Docker driver on linux, the terminal needs to be open to run it.

# Entre na url que for exposta e verifique se a aplicação deu tudo certo
```
## Até a fase 6, o projeto está rodando perfeitamente! 

# Projeto fase 7:

```bash
# Migrando porta da aplicação, usando uma porta específica.
kubectl get svc jupyter-svc -n jupyter -o yaml | grep nodePort

# Saída esperada:
 - nodePort: 31708

# Edite o campo  node-Port do arquivo gerado 
KUBE_EDITOR="nano" kubectl edit service jupyter-svc -n jupyter
- nodePort: 30000

# Confirme se mudou algo
kubectl get svc jupyter-svc -n jupyter -o yaml | grep nodePort

# Saída esperada: 
- nodePort: 30000

# Depois de editar, entre na url: localhost:30000

```

# Projeto fase 8:

```bash
# Escalando a aplicação, adicionei 3 replicas ao meu deployment. Antes estava apenas 1 replica
kubectl get deploy -n jupyter

#  saída esperada: 
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
jupyter-deploy   1/1     1            1           41m


# Modifiquei para 3 replicas, você pode modifcar para 5 se desejar e vai ser necessário aplicar o arquivo novamente
kubectl apply -f ./deployment/jupyter-deploy.yml

# Saída esperada depois de aplicar o arquivo manifesto após adicionar mais replicas
deployment.apps/jupyter-deploy configured


# Saída esperada do deploy
kubectl get deploy -n jupyter

NAME             READY   UP-TO-DATE   AVAILABLE   AGE
jupyter-deploy   3/3     3            3           44m

# Saída esperada dos pods
kubectl get pod -n jupyter

NAME                             READY   STATUS    RESTARTS   AGE
jupyter-deploy-54c6877f4-2tgc2   1/1     Running   0          55s
jupyter-deploy-54c6877f4-prktt   1/1     Running   0          55s
jupyter-deploy-54c6877f4-vdzcw   1/1     Running   0          44m
```

# Projeto fase 9: 
```bash
# Atualizando a aplicação pela linha de comando, mudando a imagem de um container específico
kubectl set image deploy/jupyter-deploy jupyter-container=jupyter/minimal-notebook:python-3.10 -n jupyter

# saída esperada após setar a imagem:
deployment.apps/jupyter-deploy image updated

# Verifique se esta tudo correto com os pods
kubectl get pod -n jupyter

# Saida esperda:
NAME                             READY   STATUS              RESTARTS   AGE
jupyter-deploy-54c6877f4-2tgc2   1/1     Running             0          6m58s
jupyter-deploy-54c6877f4-prktt   1/1     Running             0          6m58s
jupyter-deploy-54c6877f4-vdzcw   1/1     Running             0          50m
jupyter-deploy-894584bc5-ssrlx   0/1     ContainerCreating   0          73s

# Note que tem um pod com o status de ContainerCreating, assim que este status mudar para Running, o proximo pod será atualizado
jupyter-deploy-894584bc5-cf7qp   1/1     Running   0          53s
jupyter-deploy-894584bc5-pbfp4   1/1     Running   0          51s
jupyter-deploy-894584bc5-ssrlx   1/1     Running   0          2m34s

# Verifique se esses novos pods estão com as imagens que acabou de atualizar: *jupyter/minimal-notebook:python-3.10*
kubectl get pod -n jupyter
kubectl describe pod jupyter-deploy-894584bc5-pbfp4  -n jupyter

# saída esperada
 Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m36s  default-scheduler  Successfully assigned jupyter/jupyter-deploy-894584bc5-pbfp4 to minikube
  Normal  Pulled     2m35s  kubelet            Container image "jupyter/minimal-notebook:python-3.10" already present on machine
  Normal  Created    2m34s  kubelet            Created container jupyter-container
  Normal  Started    2m34s  kubelet            Started container jupyter-container

  # note que o pod ficou com a nova imagem com sucesso!

  ## Verifique se status do deployment esta tudo certo com o rollout status
  kubectl rollout status deploy/jupyter-deploy -n jupyter

  # saída esperada:
  deployment "jupyter-deploy" successfully rolled out

  # Voltando para a imamge anterior
  kubectl rollout undo deploy/jupyter-deploy -n jupyter

  # Saída esperada inforamdno que deu um rollback na imagem:
  deployment.apps/jupyter-deploy rolled back

  # verifique qualquer um dos novos pods se a imagem voltou o que era antes
  kubectl describe pod jupyter-deploy-894584bc5-pbfp4  -n jupyter

  # saída esperada do novo pod
  Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  35s   default-scheduler  Successfully assigned jupyter/jupyter-deploy-54c6877f4-7hjvf to minikube
  Normal  Pulled     35s   kubelet            Container image "jupyter/minimal-notebook:2022-12-05" already present on machine
  Normal  Created    35s   kubelet            Created container jupyter-container
  Normal  Started    35s   kubelet            Started container jupyter-container

  # Note que agora a imagem deu um rollback com sucesso!
```

# Projeto fase 10:

```bash
# Analisando a aplicação e fazendo a remoção

# 1) - Verificando os deploy da aplicação:
kubectl get deploy -n jupyter

# saída esperada:
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
jupyter-deploy   3/3     3            3           61m

# 2) - Verificando as replicaset da aplicação:
kubectl get rs -n jupyter

# saída esperada 
NAME                       DESIRED   CURRENT   READY   AGE
jupyter-deploy-54c6877f4   3         3         3       62m
jupyter-deploy-894584bc5   0         0         0       13m

# 3) - Verificando os endpoints e endpointslices da aplicação:
kubectl get endpoints -n jupyter

# saída esperada dos endpoints:
NAME          ENDPOINTS                                           AGE
jupyter-svc   10.244.0.10:8888,10.244.0.11:8888,10.244.0.9:8888   30m

#endpointslices
kubectl get endpointslices -n jupyter

# saída esperada dos endpointslices:
NAME                ADDRESSTYPE   PORTS   ENDPOINTS                            AGE
jupyter-svc-snzgc   IPv4          8888    10.244.0.9,10.244.0.10,10.244.0.11   31m

# 4) - deletando a aplicação:
kubectl delete deploy jupyter-deploy -n jupyter

# 5) - deletando o service
kubectl delete svc jupyter-svc -n jupyter

# 6) - deletando tudo e deixando o cluster zerado
minikube delete && minikube start
```