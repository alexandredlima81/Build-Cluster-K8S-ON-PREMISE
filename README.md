

# Cluster K8S ON PREMISE
<p align="justify">

Este repositório contém a configuração e os arquivos necessários para a criação de um **Cluster Kubernetes On-Premise** utilizando SO UBUNTU-24.04 Server. A configuração ainda não utiliza o conceito de infraestrutura como código (IaC) para garantir que o ambiente Kubernetes seja provisionado e gerenciado de forma automatizada, permitindo maior controle e customização em ambientes locais.

A abordagem utilizada permite a instalação de componentes essenciais do Kubernetes, como o etcd, apiserver, controller-manager, scheduler, e os workers, além de recursos opcionais, como ingress controllers e monitoramento.

## Pré-requisitos

Antes de começar a instalação, certifique-se de que seu ambiente atenda aos seguintes pré-requisitos:

- Ambiente para deploy, no meu caso este ambiente foi criado em um Cluster de Hypervisor PROXMOX.
- ISO SO Ubuntu 24.04 LTS.
- Acesso privilegiado ao sistema (usuário root ou sudo).
- Conexão ativa com a internet.
- Mínimo de 2 GB de RAM ou mais.
- Mínimo de 2 núcleos de CPU (ou 2 vCPUs).
- 20 GB de espaço livre em disco em /var (ou mais).

## Estrutura do Cluster
O cluster Kubernetes on-premise é composto pelos seguintes componentes e servidores:

**Master - Control Plane:** \
Nó responsável por controlar o estado do cluster e gerenciar a comunicação entre os componentes.

**etcd:** \
Armazenamento distribuído chave-valor, utilizado para manter o estado de todos os componentes do cluster.

**kube-apiserver:** \
Interface de comunicação com o cluster, responsável por expor a API do Kubernetes.

**kube-controller-manager:** \
Controlador que garante o estado desejado dos objetos no cluster (replication controller, endpoints, etc.).

**kube-scheduler:** \
Responsável por distribuir as cargas de trabalho (pods) entre os nós (workers) com base nos recursos disponíveis.

**Workers:** \
Nós de trabalho.

**kubelet:** \
Agente de cada nó que aplica as instruções do control plane, garantindo que os contêineres rodem conforme especificado.

**kube-proxy:** \ 
Gerencia as regras de rede que permitem a comunicação entre os pods.

**Container Runtime:** \
Responsável pela execução dos contêineres no nó (e.g., Docker ou containerd).

# Rede e Comunicação:

**CNI (Container Network Interface):** \
Plugin de rede para permitir a comunicação entre os pods (exemplos: Flannel, Calico). \
Aqui estaremos utilizando o Calico.

**Load Balancer:** \
Balanceador de carga para distribuir o tráfego de entrada para os serviços expostos no cluster.

# Componentes Opcionais:

**Ingress Controller:** \
Controlador para gerenciar o acesso externo a serviços no cluster.

**Ferramentas de Monitoramento:** \
Como o Prometheus para monitorar métricas do cluster e o Grafana para visualizações.

**Ferramentas de Logging:** \
Como ELK Stack (Elasticsearch, Logstash, Kibana) ou EFK Stack (Elasticsearch, Fluentd, Kibana) para centralizar os logs.

Essa estrutura define os principais componentes do cluster Kubernetes, detalhando o papel de cada um e as funções essenciais para a operação do ambiente on-premise.

# CONFIGURANDO CLUSTER K8S

## 1: CONFIGURAR O HOSTNAME 
## (REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)

Atribua o hostname do nó em questão. Neste exemplo, estamos atribuindo o novo hostname ao nó Master.

**Sintaxe:**
```bash
$ sudo hostnamectl set-hostname “k8s-master01” && exec bash
```

## 2: CONFIGURAR INTERFACE DE REDE USANDO O NETPLAN (VALIDAR O NOME DA INTERFACE) 
## (REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)

Aqui estamos utilizando o CIDR 192.168.18.0/24. 
Ajuste o CIDR conforme o range de endereço IP necessário para o seu cenário.
Ajuste o IP, através do netplan.

Para editar o arquivos use a sintaxe a seguir:

**Sintaxe:**
```bash
$ sudo nano /etc/netplan/50-cloud-init.yaml
```
ou se preferir

**Sintaxe:**
```bash
$ sudo vim /etc/hosts/50-cloud-init.yaml
```

>
>Neste repositorio temos um arquivo **50-cloud-init.yaml** para referência. \
>Se atente ao nome da nome da interface a ser ajustado no arquivo (***eth0, eth1, enp0s1,*** etc...)
>

Ao salvar o arquivo, valide a configuração antes de aplicar para garantir que não há erros.

**Sintaxe:**
```bash
$ sudo netplan try
```

Aplicar a configuração permanentemente após testar:

**Sintaxe:**
```bash
$ sudo netplan apply
```
Feito isso valide se o ip foi atribuido.

**Sintaxe:**
```bash
$ sudo ip address
```

## 3: CONFIGURAR O ARQUIVO /etc/hosts 
## (REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)

Neste projeto, estamos propondo dois nós Master e três nós Worker. 
Para editar o arquivos use a sintaxe a seguir:

**Sintaxe:**
```bash
$ sudo nano /etc/hosts
```
ou se preferir

**Sintaxe:**
```bash
$ sudo vim /etc/hosts
```

Adicione o conteúdo conforme o exemplo a seguir. Como neste caso não temos um DNS, é necessário adicionar os endereços IP com seus respectivos nomes para permitir a resolução de nomes dentro do cluster Kubernetes.

>
>**OBSERVAÇÂO:** Aqui não estaremos usando um servidor de DNS, por este motivo a necessidade de editar o arquivo **hosts**.
>

Exemplo de adição ao arquivo **/etc/hosts**:

>
>127.0.0.1 localhost \
>127.0.1.1 k8smaster01 \
>192.168.18.201   k8smaster01 \
>192.168.18.202   k8smaster02 \
>192.168.18.203   k8sworker01 \
>192.168.18.204   k8sworker02 \
>192.168.18.205   k8sworker03
>

## 4: DESABILITAR O SWAP 
## (REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)

É essencial para clusters Kubernetes desativar a **swap**, pois o K8s exige que o swap esteja desabilitado para garantir a correta alocação de recursos e evitar problemas de desempenho.

**Sintaxe:**
```bash
$ sudo swapoff -a
```

Em seguida comentamos a entrada do **swap** no arquivo **/etc/fstab** para evitar que ele seja ativado após reinicializações.

**Sintaxe:**
```bash
$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
## 5: ATUALIZAR O SISTEMA 
## (REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)
Atualize a lista de pacotes disponíveis no repositório.

**Sintaxe:**
```bash
$ sudo apt update
```

Instale as atualizações de pacotes no sistema para garantir que o ambiente esteja com as versões mais recentes e seguras dos softwares.

**Sintaxe:**
```bash
$ sudo apt upgrade -y
```

## 6: ADICIONAR KERNEL MODULES E PARÂMETROS 
# <span style="color:red;">(REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)</span>


Essas alterações são necessárias para habilitar o roteamento e a filtragem de pacotes 
entre redes no Kubernetes. Os módulos do kernel overlay e br_netfilter permitem a comunicação 
entre contêineres, enquanto os parâmetros de rede ajustam o comportamento do kernel para 
gerenciar tráfego de rede e encaminhamento de pacotes, essenciais para o funcionamento 
correto do cluster.

Carregue os módulos necessários para o Kubernetes:

**Sintaxe:**
```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```
Configure os parâmetros do kernel:

**Sintaxe:**
```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
Aplique as mudanças:

**Sintaxe:**
```bash
sudo sysctl --system
```

## Links de referência

https://markdown.net.br/sintaxe-basica/

https://netplan.io/

https://netplan.io/examples

https://docs.docker.com/engine/install/ubuntu/

https://kubernetes.io/docs/setup/



</p>


$ sudo apt update