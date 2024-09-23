# Cluster K8S ON PREMISE

Este repositório contém a configuração e os arquivos necessários para a criação de um **Cluster Kubernetes On-Premise**. A configuração ainda não utiliza o conceito de infraestrutura como código (IaC) para garantir que o ambiente Kubernetes seja provisionado e gerenciado de forma automatizada, permitindo maior controle e customização em ambientes locais.

A abordagem utilizada permite a instalação de componentes essenciais do Kubernetes, como o etcd, apiserver, controller-manager, scheduler, e os workers, além de recursos opcionais, como ingress controllers e monitoramento.

## Pré-requisitos

Antes de começar a instalação, certifique-se de que seu ambiente atenda aos seguintes pré-requisitos:

1. Um sistema Ubuntu 24.04 LTS.
2. Acesso privilegiado ao sistema (usuário root ou sudo).
3. Conexão ativa com a internet.
4. Mínimo de 2 GB de RAM ou mais.
5. Mínimo de 2 núcleos de CPU (ou 2 vCPUs).
6. 20 GB de espaço livre em disco em /var (ou mais).

## Estrutura do Cluster
O cluster Kubernetes on-premise é composto pelos seguintes componentes e servidores:

# Control Plane (Plano de Controle):

```
etcd: Armazenamento distribuído chave-valor, utilizado para manter o estado de todos os componentes do cluster.
kube-apiserver: Interface de comunicação com o cluster, responsável por expor a API do Kubernetes.
kube-controller-manager: Controlador que garante o estado desejado dos objetos no cluster (replication controller, endpoints, etc.).
kube-scheduler: Responsável por distribuir as cargas de trabalho (pods) entre os nós (workers) com base nos recursos disponíveis.
Nós de Trabalho (Workers):
kubelet: Agente de cada nó que aplica as instruções do control plane, garantindo que os contêineres rodem conforme especificado.
kube-proxy: Gerencia as regras de rede que permitem a comunicação entre os pods.
Container Runtime (e.g., Docker ou containerd): Executa os contêineres no nó.
Rede e Comunicação:

CNI (Container Network Interface): Plugin de rede para permitir a comunicação entre os pods (exemplos: Flannel, Calico).
Load Balancer: Balanceador de carga para distribuir o tráfego de entrada para os serviços expostos no cluster.
Componentes Opcionais:

Ingress Controller: Controlador para gerenciar o acesso externo a serviços no cluster.
Ferramentas de Monitoramento: Como o Prometheus para monitorar métricas do cluster e o Grafana para visualizações.
Ferramentas de Logging: Como ELK Stack (Elasticsearch, Logstash, Kibana) ou EFK Stack (Elasticsearch, Fluentd, Kibana) para centralizar os logs.
Essa estrutura define os principais componentes do cluster Kubernetes, detalhando o papel de cada um e as funções essenciais para a operação do ambiente on-premise.
```




### Stage 1: NGINX Build

No primeiro estágio, utilizamos a imagem oficial do NGINX em sua versão **alpine** para extrair os binários necessários do NGINX:

```dockerfile
# Stage 1: Use an official NGINX image to extract binaries
FROM nginx:alpine as nginx-build
```

### Stage 2: Distroless Final Image
No segundo estágio, utilizamos a imagem Distroless baseada no Debian para garantir que a imagem final seja o mais enxuta e segura possível:

```dockerfile
# Stage 2: Use Distroless as the base for the final image
FROM gcr.io/distroless/base-debian12
```
Copiamos os binários do NGINX e os arquivos de configuração do estágio anterior:

```dockerfile
# Copy NGINX binaries from the first stage
COPY --from=nginx-build /usr/sbin/nginx /usr/sbin/nginx
COPY --from=nginx-build /etc/nginx /etc/nginx
```
Expomos a porta 80 para servir as requisições HTTP e iniciamos o NGINX em modo daemon off:

```dockerfile
# Expose the port NGINX will serve on
EXPOSE 80

# Start NGINX
CMD ["nginx", "-g", "daemon off;"]
```
## Como Construir a Imagem

Siga os passos abaixo para construir a imagem Docker:

Clone o repositório:

```bash

git clone https://github.com/seu-usuario/distroless-nginx.git
cd distroless-nginx
```
Construa a imagem Docker:

```bash
docker build -t seu-usuario/nginx-distroless:latest .
```
Execute a imagem:

```bash
docker run -d -p 8080:80 seu-usuario/nginx-distroless:latest
```
Agora o NGINX estará rodando no seu container e estará acessível em http://localhost:8080.

## Pipeline de CI com GitHub Actions

Este repositório inclui uma pipeline de CI com o GitHub Actions que automaticamente:

1. Realiza o build da imagem Docker.
2. Realiza o push da imagem para o Docker Hub.
   
Para configurar o pipeline, certifique-se de adicionar as seguintes variáveis de GitHub Secrets ao seu repositório:
>
> **DOCKER_USERNAME:** Seu nome de usuário no DockerHub.
>
> **DOCKER_PASSWORD:** Sua senha ou token de acesso ao DockerHub.
>

Exemplo de Pipeline

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/nginx-distroless:latest .

      - name: Push Docker image
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/nginx-distroless:latest
```

## Vantagens do Uso de Distroless

**Segurança:** A imagem Distroless é minimalista e não contém pacotes ou bibliotecas desnecessárias, reduzindo a superfície de ataque.

**Tamanho da Imagem:** A imagem resultante é muito menor do que uma imagem de NGINX tradicional, economizando espaço e melhorando os tempos de download e deploy.

## Links úteis

https://markdown.net.br/sintaxe-basica/

https://portal.revendadesoftware.com.br/manuais/base-de-conhecimento/sintaxe-markdown

https://console.cloud.google.com/artifacts/docker/distroless/us/gcr.io?pli=1

https://edu.chainguard.dev/chainguard/chainguard-images/getting-started-distroless/

https://www.redhat.com/en/blog/why-distroless-containers-arent-security-solution-you-think-they-are

https://www.docker.com/blog/is-your-container-image-really-distroless/

https://juniorjbn.medium.com/o-que-%C3%A9-esse-tal-de-distroless-d1cc5dcd070e

https://hub.docker.com/

https://docs.github.com/en/actions/about-github-actions/understanding-github-actions
