# Resources

### Requests, define um limite minimo que o container pode usar

### Limits, define um limite máximo que o container pode usar

### Testando o funcionamento
```bash
kubectl apply -f resources.yml

## Verificando  o limite mínimo e máximo
kubectl describe pod resources-pod
```