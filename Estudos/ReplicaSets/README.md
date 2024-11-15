```bash
# Modo imperativo de auto scale com 10 replicas, pela linha de comando:
kubectl apply -f my-replicaset.yml
kubectl scale replicasets frontend-rs --replicaset=10 && watch kubectl get pods

# Deletando o replicaset
kubectl delete rs frontend-rs
``` 