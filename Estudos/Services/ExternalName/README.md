# O uso de um externalName necessita de uma assinatura cloud para poder utilizar, aqui deixei um exemplo e uma breve explicação do que seria um ExternalName no meu ponto de vista

# Esse serviço seria para acessar serviços externos que esta armazendo na internet, na nuvem por exemplo, seja google gloud, azure ou aws. Para acessar esses serviços externos geralmente são atravpes de DNS Cname, A Record, serviços sem selectors. Basicamente, seria acessar de uma aplicação interna do cluster para fora

# exemplo de um externalname:

```bash
apiVersion: v1
kind: Service
metadata:
  name: my-external-database-service
  namespace: production
spec: 
  type: ExternalName 
  externalName: mydatabase.dbcompany.com
```

# Explicação de mydatabase.dbcompany.com

#### uma empresa ficitica tem o banco de dados chamado mydatabase e este banco esta populado com dados da empresa, como cliente... etc. A empresa se chama db.company.com e o caminho do banco pra essa empresa seria mydatabase.dbcompany.com 

#### Como o kubernetes enxerga isso internamente?  Da seguinte forma, quando a aplicação do cluster pesquisar, ou seja, fizer um DNS Looking up no serviço, *my-external-database-service.produciton.svc.cluster.local* O serviço do proprio cluster ele retorna  um  registro CNAME Recorde para = mydatabase.dbcompany.com e dessa forma a aplicação tem o local onde ela vai utilziar o bnanco de dados

```bash
Service name                    Namespace        Local Cluster Service 
my-external-database-service | produciton.svc | cluster.local
```