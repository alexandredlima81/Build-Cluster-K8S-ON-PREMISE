

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

## 1: CONFIGURAR O HOSTNAME (REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)

Atribua o hostname do nó em questão. Neste exemplo, estamos atribuindo o novo hostname ao nó Master.

```bash
$ sudo hostnamectl set-hostname “k8s-master01” && exec bash
```

## 2: CONFIGURAR INTERFACE DE REDE USANDO O NETPLAN (VALIDAR O NOME DA INTERFACE) (REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)

Aqui estamos utilizando o CIDR 192.168.18.0/24. 
Ajuste o CIDR conforme o range de endereço IP necessário para o seu cenário.
Ajuste o IP, através do netplan.

Para editar o arquivos use a sintaxe a seguir:

```bash
$ nano /etc/netplan/50-cloud-init.yaml
```
ou se preferir

```bash
$ vim /etc/hosts/50-cloud-init.yaml
```

>
>network: \
>  version: 2 \
>  renderer: networkd \
>  ethernets: \
>    enp0s3: \
>      dhcp4: no \
>      addresses: [192.168.18.201/24] \
>      nameservers: \
>          addresses: [8.8.8.8, 8.8.2.2, 192.168.18.1, 192.168.18.2] \
>      routes: \
>        - to: default \
>          via: 192.168.18.1 \
>

Ao salvar o arquivo, Valide a configuração antes de aplicar para garantir que não há erros.

Sintaxe:
```bash
$ sudo netplan try
```

Aplicar a configuração permanentemente após testar:

Sintaxe:
```bash
$ sudo netplan apply
```
Feito isso valide se o ip foi atribuido.

Sintaxe:
```bash
$ sudo ip address
```

## 2: CONFIGURAR O ARQUIVO /etc/hosts (REALIZAR CONFIGURAÇÃO EM NÓS MASTERS E WORKERS)

Neste projeto, estamos propondo dois nós Master e três nós Worker. 
Para editar o arquivos use a sintaxe a seguir:

```bash
$ nano /etc/hosts
```
ou se preferir

```bash
$ vim /etc/hosts
```

Adicione o conteúdo conforme o exemplo a seguir. Como neste caso não temos um DNS, é necessário adicionar os endereços IP com seus respectivos nomes para permitir a resolução de nomes dentro do cluster Kubernetes.

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



## Links de referência

https://markdown.net.br/sintaxe-basica/

https://docs.docker.com/engine/install/ubuntu/

https://kubernetes.io/docs/setup/



</p>


