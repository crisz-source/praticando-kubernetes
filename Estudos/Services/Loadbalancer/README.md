# o Loadbalancer segue a mesma regra de nodePort e ClusterIP, precisei apenas trocar o tipo de serviço e manter as portas que serão utilizadas
```bash
kubectl apply -f loadbalancer.yml -f service.yml

# acessando o serviço
minikube service --url frontend-service-loadbalancer

# entre com o ip gerado no browser
``` 