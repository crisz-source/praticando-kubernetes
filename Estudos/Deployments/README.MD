# Modo imperativo de auto scale com 10 replicas, pela linha de comando:
```bash
kubectl apply -f my-deployment.yml
kubectl scale deploy/frontend-deploy --replicas=11


# Deletando o deployment
kubectl delete deploy frontend-deploykk
 
 
para fazer os testes de rollback, modifique a imagem do nginx para qualquer versão de sua escolha e execute o comando de apply novamente para que funcione corretamente o comando rollout

# rollout:
# ativando o status:
kubectl rollout status deploy/frontend-deploy

# verificando o historico:
kubectl rollout history deploy/frontend-deploy

# voltando para uma versão anterior, rollback
kubectl rollout undo deploy/frontend-deploy

# voltando para uma versão especifica
kubectl rollout undo deploy/frontend-deploy --to-revision=4

# Pausa o rollout
kubectl rollout pause deploy/frontend-deploy

# Retoma o rollout, aplicando todas as mudanças de uma vez
kubectl rollout resume deploy/frontend-deploy
```