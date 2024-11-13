# Os cronjobs cria os jobs e os jobs cria os pods: CronJob > Job > Pod > Container

###  Formato cron. 
```bash
* * * * * * 
| | | | | | 
| | | | | +-- Ano            (intervalo: 1900-3000)
| | | | +---- Dia da semana  (intervalo: 0-7, sendo domingo 0 e segunda 1)
| | | +------ Mês            (intervalo: 1-12)
| | +-------- Dia            (intervalo: 1-31)
| +---------- Hora           (intervalo: 0-23)
+------------ Minuto         (intervalo: 0-59)

# 1) Exemplo: Tarefa ao sábados ás 00:00, e todo dia 20: 0 0 20 * 6
# 2) Backup diário ás 5h: 0 5 * * *
# 3) Tarefa a cada minuto: * * * * *

# Para facilitar o uso do Cron, basta acessar o site: crontab.guru

# AMBAS SINTAXES FUNCIONAM CORRETAMENTE, SENDO [ * * * * * ] ou [ ? ? ? ? ?]

? ? ? ? ? ?
| | | | | | 
| | | | | +-- Ano            (intervalo: 1900-3000)
| | | | +---- Dia da semana  (intervalo: 1-7, 1 sendo domingo, 2 segunda, etc)
| | | +------ Mês            (intervalo: 1-12)
| | +-------- Dia            (intervalo: 1-31)
| +---------- Hora           (intervalo: 0-23)
+------------ Minuto         (intervalo: 0-59)


```


# Identificações: *CronJobs > Jobs > Pods*

#### Com o comando abaixo, você consegue ver o nome do CronJob:
```bash
kubect get cj

# Saída esperada: 
NAME    SCHEDULE    TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
my-cj   * * * * *   <none>     False     0        <none>          3s

# Note que ele tem o nome de *my-cj*
```



#### Com o comando abaixo você consegue ver os jobs
```bash
kubectl get jobs

#Saída esperada: 
NAME             STATUS     COMPLETIONS   DURATION   AGE
my-cj-28853507   Complete   1/1           5s         62s
my-cj-28853508   Running    0/1           2s         2s

# Note que ele possui uma uma identificação de que esse job foi criado por um CronJob: my-cj-*28853507*

```

#### Com o comando abaixo você consegue ver os pods
```bash
kubectl get pods

#Saída esperada: 
NAME                   READY   STATUS      RESTARTS   AGE
my-cj-28853507-q84gw   0/1     Completed   0          2m46s
my-cj-28853508-sq856   0/1     Completed   0          106s
my-cj-28853509-pdsjr   0/1     Completed   0          46s

# Note que ele possui uma uma identificação de que esse pod foi criado por um job que por sua vez foi criado por um cronjob, ele possu uma identificação a mais: my-cj-28853507-*q84gw*

```

# Colocando o cronjob em modo de suspensção
kubectl patch cronjob my-cj -p '{"spec" : {"suspend":true}}'

# Colocando o cronjob para continuar da suspen
kubectl patch cronjob my-cj -p '{"spec" : {"suspend":false}}'

#### Deletando o CronJob
```bash
kubectl delete cj my-cj

```