# Compass UOL - Projeto Final
## AWS e DevSecOps (Turma: PB – Out 2024)

### 1. Contexto

Nós somos da empresa "Fast Engineering S/A" e gostaríamos de uma solução dos
senhores(as), que fazem parte da empresa terceira "TI SOLUÇÕES INCRÍVEIS".
Nosso eCommerce está crescendo e a solução atual não está atendendo mais a
alta demanda de acessos e compras que estamos tendo.

Atualmente usamos:
- 01 servidor para Banco de Dados Mysql (500GB de dados, 10Gb de RAM, 3
Core CPU);
- 01 servidor para a aplicação utilizando REACT – frontend (5GB de dados,
2Gb de RAM, 1 Core CPU);
- 01 servidor de backend com 3 APIs, com o Nginx servindo de balanceador de
carga e que armazena estáticos como fotos e links. (5GB de dados, 4Gb de
RAM, 2 Core CPU);


![image](https://github.com/user-attachments/assets/de6faf1a-8d69-4d7d-9ff5-ef9f9a667466)


## Desafio 
Seguir as melhores práticas para modernizar esse sistema para a AWS, a nova arquitetura deve seguir as seguintes
diretrizes:

- Ambiente Kubernetes;
- Banco de dados gerenciado (PaaS e Multi AZ);
- Backup de dados;
- Sistema para persistência de objetos (imagens, vídeos etc.);
- Segurança;

  
Primeiro, faremos uma migração "lift-and-shift"; a adaptação para Kubernetes virá depois.

# Passo 1: Completar a migração
### Ferramentas AWS para Migração "Lift-and-Shift" ou "As-Is":

Para realizar migrações nesses modelos, a AWS oferece serviços gerenciados que simplificam e tornam o processo mais eficiente, garantindo a continuidade dos sistemas e a segurança dos dados.  O AWS Application Migration Service (MGN) e o AWS Database Migration Service (DMS) são ferramentas-chave nesse cenário, permitindo migrar aplicações e bancos de dados com o mínimo de impacto e sem a necessidade de alterações significativas.

## Visão Geral da Arquitetura
### 1. Serviços Utilizados:

- **AWS Application Migration Service (MGN):**

Migra os servidores Frontend (React) e Backend (Nginx + APIs) para instâncias EC2 na AWS, utilizando o AWS Replication Agent para replicar dados do ambiente on-premises para a VPC de Staging antes de movê-los para a VPC Final.

- **AWS Database Migration Service (DMS):**

Migra o banco de dados MySQL para o Amazon RDS, garantindo a integridade e continuidade dos dados.

- **Amazon Elastic Block Store (EBS):**

Armazena os volumes persistentes das instâncias EC2.

- **Amazon S3:**

Armazena objetos estáticos (imagens, vídeos, arquivos).

- **Amazon Route 53:**

Gerencia DNS e direciona tráfego para recursos na AWS.

- **Amazon Virtual Private Cloud (VPC):**

Isola a infraestrutura e gerencia subnets públicas e privadas.

- **AWS Backup:**

Garante backups automáticos dos recursos.

### 2. Processo de Migração:

- **Banco de Dados:**

A migração do MySQL é feita primeiro para o RDS usando o DMS, garantindo que os endpoints já estejam configurados para a nova infraestrutura.

- **Servidores Frontend e Backend:**

O AWS MGN utiliza agentes instalados nos servidores on-premises para replicar dados para uma VPC de Staging. Após os testes, os dados são movidos para a VPC Final.


## Passo a Passo de Migração para AWS

**Configurações do ambiente on-premises, incluindo:**
- **Frontend:** Aplicação React
  
Porta TCP 443: Comunicação HTTPS com os usuários.

- **Backend:** Servidor Nginx com APIs
  
Porta TCP 443: Comunicação segura com o frontend.

Porta TCP 1500: Para migração de dados via AWS Replication Agent.

- **Banco de Dados:** MySQL
  
Porta TCP 3306: Comunicação com o banco de dados (RDS).

### Provisionamento de Infraestrutura na AWS

**VPC de Staging:**
Utilizada para a migração inicial, hospedando o Replication Server, responsável por receber dados dos servidores on-premises através do AWS Replication Agent (porta TCP 1500).

**VPC Final:**
- Subnets Públicas: Onde serão alocados o Frontend EC2 e o Load Balancer.
- Subnets Privadas: Para o Backend EC2 e o RDS.

As subnets públicas terão acesso direto à internet através de um **Internet Gateway (IGW)**, enquanto as instâncias privadas que precisarem de acesso externo serão conectadas via **NAT Gateway**.

**Grupos de Segurança**
- O acesso ao RDS MySQL será exclusivo para o Backend EC2, garantindo que apenas esse serviço consiga interagir com o banco de dados.

- Para o Frontend e Backend, a comunicação na porta 443 (HTTP/HTTPS) será viabilizada, para garantir que a comunicação com a web ocorra sem problemas. Se um Load Balancer estiver em uso, ele gerenciará o tráfego entre essas instâncias, permitindo a distribuição eficiente de requisições.


**Migração do Banco de Dados**
- Crie a instância RDS MySQL e ative backups automáticos. Use o AWS DMS para migrar os dados, com Full Load na carga inicial e CDC para a cópia em tempo real.

**Migração do Frontend e Backend**

- Instale o AWS Replication Agent nos servidores on-premises para enviar dados ao Replication Server. Com o AWS MGN, converta os dados em AMIs e gere instâncias EC2 para o Frontend e Backend na VPC Final. Associe volumes EBS às instâncias para garantir a persistência dos dados.


#### Resultados
Após a conclusão da Fase 1, a infraestrutura terá sido replicada na AWS com o seguinte estado:

- Banco de dados MySQL funcionando no Amazon RDS.
- Frontend e Backend funcionando em instâncias EC2.
- Objetos estáticos armazenados no Amazon S3.
- Segurança e backups configurados.

# Diagrama de Migração

![image](https://github.com/user-attachments/assets/3c9617f7-d3b3-4e59-8375-fca8f6373e01)

# Passo 2: Modernização/Kubernetes 

Nesta etapa, a infraestrutura migrada na Fase 1 será modernizada com foco em práticas cloud-native, utilizando o Amazon EKS para orquestração de contêineres e otimizando a arquitetura para escalabilidade, confiabilidade, eficiência e segurança.

## 1. Criação do Ambiente Kubernetes
**Criação do Cluster EKS:**
Utilize o console AWS ou ferramentas como eksctl para provisionar o cluster com nós configurados adequadamente para atender à carga de trabalho. Será feito uso do auto-scaling para garantir a escalabilidade do cluster, com as instâncias EC2 sendo configuradas conforme as necessidades da aplicação.

- Configure o número de nodes necessários para o frontend e backend (recomenda-se instâncias m5.large).

**Configure as subnets públicas e privadas na VPC:**
- Subnets públicas para o Load Balancer.
- Subnets privadas para os pods e banco de dados.

**IAM Roles:**
- Criar roles para os nós do EKS com as permissões necessárias para acessar outros serviços da AWS (e.g., S3, RDS).

  
**2. Containerização das Aplicações:**

- Criação de Dockerfiles: Criar Dockerfiles para os componente da aplicação, incluindo o frontend, backend e qualquer outro serviço.
- Construção das Imagens: Utilizar o AWS CodeBuild para construir as imagens Docker e enviá-las para o Amazon ECR.
  
**3. Criação dos Manifests Kubernetes:**

- Deployment: Definir os deployments para cada serviço, especificando o número de réplicas, as portas, as imagens Docker e os recursos de CPU e memória.
- Service: Crie serviços para permitir a comunicação interna entre os componentes no cluster.
- Ingress: Utilize o Ingress Controller para expor os serviços externamente, configurando rotas e associando um Load Balancer para gerenciar o tráfego.

**Implementação da CI/CD:**

- AWS CodePipeline: Configurar um pipeline de CI/CD para automatizar o processo de build, teste e deploy das aplicações.
- AWS CodeBuild: Utilizar o CodeBuild para construir as imagens Docker e executar testes.
- AWS CodeDeploy: Utilizar o CodeDeploy para realizar os deployments no Kubernetes.

**4. Banco de Dados e Armazenamento:**
- O banco de dados MySQL, que foi migrado para o Amazon RDS, continua a ser utilizado, com configurações Multi-AZ para garantir alta disponibilidade. O armazenamento de arquivos estáticos é feito no Amazon S3, utilizando políticas de IAM para restringir o acesso aos pods que precisam interagir com os dados.


**5. Segurança e Monitoramento:**

A segurança da aplicação é garantida pelo AWS WAF e CloudFront, que ajudam a proteger contra ameaças e melhorar a performance com cache de conteúdo estático. O monitoramento de métricas, logs e performance é feito pelo Amazon CloudWatch, e o gerenciamento de permissões é realizado com o AWS IAM, garantindo o acesso controlado aos recursos. As regras de segurança são configuradas para garantir que apenas os serviços necessários possam se comunicar entre si.

#### Resultados
Após a conclusão da etapa de modernização, a infraestrutura estará no seguinte estado:

- Serviços orquestrados no Amazon EKS com ambientes isolados por namespaces.
- CI/CD implementado, automatizando build, teste e deploy.
- Banco de dados MySQL funcionando no Amazon RDS com alta disponibilidade.
- Arquivos estáticos armazenados no Amazon S3 com acesso seguro.
- Logs e métricas monitorados no Amazon CloudWatch.
- Segurança reforçada com AWS WAF e CloudFront.



![image](https://github.com/user-attachments/assets/33d03f3a-ea2b-4ac4-ba99-b4a72bc02450)



# Estimativa de custos com AWS Calculator 


- Etapa 1:

![Captura de tela 2025-01-28 142723](https://github.com/user-attachments/assets/2f9a781a-a5e6-4253-89ec-1bc25848734d)




- Etapa 2:

![image](https://github.com/user-attachments/assets/3fe5f648-efd6-469d-865f-a72ee8116428)





