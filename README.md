# Kubernetes - Projeto de Estudos Pessoais
#### Este Estudo tem como objetivo apresentar o meu aprendizado inicial em Kubernetes, focando em conceitos e práticas fundamentais como volumes, deployments, namespaces, secrets e outros tópicos essenciais para o uso de Kubernetes em um ambiente de desenvolvimento.

#### Como parte do estudo, utilizei o Minikube para criar e gerenciar um cluster local e ubuntu 22.04. Abaixo, apresento um passo a passo para a instalação do Docker, Minikube, Kubectl e a configuração de um cluster com 4GB de RAM.

# Requisitos

- Sistema operacional compatível: Linux ubuntu 22.04
- Docker instalado para gerenciar containers
- Minikube instalado para criar um cluster Kubernetes Local
- Kubectl instalado para interagir com o cluster Kubernetes.


### Passo 1: Instalar o docker

```bash 
# Atualize o gerenciador de pacotes
sudo apt update

# Instale pacotes necessários
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

#instalando o docker
sudo apt install docker.io docker-compose docker-compose-v2 docker-doc


# Inicie o Docker e adicione-o ao inicializar do sistema
sudo systemctl enable docker
sudo systemctl start docker

# adicionando docker a um grupo sem precisar ficar digitando "sudo" toda vez" 
sudo usermod -aG docker $USER
# Depois de disparar o comando acima, reinicie sua máquina ou saia da sessão e volte novamente para que o usermod funcione corretamente

#testando sem o sudo
docker ps
```

### Passo 2: Instalar o Kubectl
```bash 
sudo apt install kubectl --classic
```

### Passo 3: Instalar o Minikube
```bash
# opcional
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Configurando o Minikube
# Iniciar o Cluster com 4GB de RAM
# Para iniciar o Minikube com 4GB de RAM e 2 CPUs, execute o seguinte comando:
minikube start --memory=4096 --cpus=2

# Verificar o Status do Cluster
minikube status
``` 

### 

