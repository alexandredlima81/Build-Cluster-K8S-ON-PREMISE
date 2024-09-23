# Cluster K8S ON PREMISE
<p align="justify">
Este repositório contém a configuração e os arquivos necessários para a criação de um **Cluster Kubernetes On-Premise**. A configuração ainda não utiliza o conceito de infraestrutura como código (IaC) para garantir que o ambiente Kubernetes seja provisionado e gerenciado de forma automatizada, permitindo maior controle e customização em ambientes locais.

A abordagem utilizada permite a instalação de componentes essenciais do Kubernetes, como o etcd, apiserver, controller-manager, scheduler, e os workers, além de recursos opcionais, como ingress controllers e monitoramento.
</p>
## Pré-requisitos

Antes de começar a instalação, certifique-se de que seu ambiente atenda aos seguintes pré-requisitos:

- Um sistema Ubuntu 24.04 LTS.
- Acesso privilegiado ao sistema (usuário root ou sudo).
- Conexão ativa com a internet.
- Mínimo de 2 GB de RAM ou mais.
- Mínimo de 2 núcleos de CPU (ou 2 vCPUs).
- 20 GB de espaço livre em disco em /var (ou mais).

## Estrutura do Cluster
O cluster Kubernetes on-premise é composto pelos seguintes componentes e servidores:

**Master - Control Plane:**  
- Nó responsável por controlar o estado do cluster e gerenciar a comunicação entre os componentes.

**etcd:** 
- Armazenamento distribuído chave-valor, utilizado para manter o estado de todos os componentes do cluster.

**kube-apiserver:** 
- Interface de comunicação com o cluster, responsável por expor a API do Kubernetes.

**kube-controller-manager:** 
- Controlador que garante o estado desejado dos objetos no cluster (replication controller, endpoints, etc.).

**kube-scheduler:** 
- Responsável por distribuir as cargas de trabalho (pods) entre os nós (workers) com base nos recursos disponíveis.

**Workers:** 
- Nós de trabalho.

**kubelet:** 
- Agente de cada nó que aplica as instruções do control plane, garantindo que os contêineres rodem conforme especificado.

**kube-proxy:** 
- Gerencia as regras de rede que permitem a comunicação entre os pods.

**Container Runtime:**  
- Responsável pela execução dos contêineres no nó (e.g., Docker ou containerd).

# Rede e Comunicação:

**CNI (Container Network Interface):** \
Plugin de rede para permitir a comunicação entre os pods (exemplos: Flannel, Calico).
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



