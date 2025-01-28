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

 **Criar VPC de Staging** 
 - A VPC temporária será utilizada para a fase inicial de migração, onde o Replication Server receberá dados dos servidores on-premises (via AWS Replication Agent na porta TCP 1500).

  
 **Criar VPC Final**
- Subnets Públicas: Configuradas para hospedar o Frontend EC2 e o Load Balancer.
- Subnets Privadas: Para o Backend EC2 e o RDS.
- Gateway de Internet (IGW): Para as subnets públicas, permitindo acesso externo, enquanto o NAT Gateway será utilizado para instâncias privadas que precisem acessar a internet.


**Grupos de Segurança**
- Banco de Dados (Porta 3306): O acesso ao RDS será restrito apenas para o Backend EC2, garantindo que somente esse serviço tenha permissão para se conectar.
- Tráfego HTTP/HTTPS (Portas 80/443): As instâncias Frontend e Backend terão acesso liberado para essas portas, permitindo o tráfego web, especialmente se um Load Balancer estiver em uso para gerenciar as requisições.

**Migração do Banco de Dados**
- Provisionar Amazon RDS (MySQL): Criar uma instância RDS para o banco de dados MySQL, escolhendo a versão compatível e configurando o backup automático para alta disponibilidade.


**Configurar AWS DMS; Criar endpoints para a migração dos dados:**
- Source Endpoint: Banco de dados MySQL on-premises.
- Target Endpoint: Banco de dados MySQL na instância RDS.
- Realizar a migração de dados com a abordagem de Full Load para a carga inicial e CDC (Change Data Capture) para replicação contínua.

**Migração do Frontend e Backend**

- Instalar AWS Replication Agent on-premises: Configurar o agente para enviar dados e virtualizações para o Replication Server na VPC de Staging.
- Configuração do AWS MGN: Usar o Replication Server para criar AMIs e gerar instâncias EC2 correspondentes para o Frontend e o Backend na VPC Final.
- Volumes EBS: Associar os volumes de dados a cada instância EC2 migrada para garantir a persistência dos dados.


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

1. Criação do Ambiente Kubernetes:
Criação do Cluster EKS: Utilizar o eksctl ou o Console da AWS para criar um cluster EKS com os seguintes recursos:

- Nós: Escolher o tipo de instância adequado à carga de trabalho (e.g., m5.large) e configurar o auto-scaling para escalar automaticamente o cluster.

- Subnets: Criar subnets privadas para os nós e subnets públicas para o load balancer.
- IAM Roles: Criar roles para os nós do EKS com as permissões necessárias para acessar outros serviços da AWS (e.g., S3, RDS).
- Configuração do Network Ingress Controller: Instalar um ingress controller (e.g., NGINX Ingress Controller) para expor os serviços do Kubernetes para o mundo externo.

2. Containerização das Aplicações:

- Criação de Dockerfiles: Criar Dockerfiles para cada componente da aplicação, incluindo o frontend, backend e qualquer outro serviço.
- Construção das Imagens: Utilizar o AWS CodeBuild para construir as imagens Docker e enviá-las para o Amazon ECR.
- Configuração de Variáveis de Ambiente: Definir as variáveis de ambiente necessárias para cada container (e.g., conexões com o banco de dados, chaves de API).
  
3. Criação dos Manifests Kubernetes:

- Deployment: Definir os deployments para cada serviço, especificando o número de réplicas, as portas, as imagens Docker e os recursos de CPU e memória.
- Service: Criar serviços para expor os deployments internamente ao cluster.
-Ingress: Criar ingresses para expor os serviços externamente, utilizando o ingress controller.

4. Migração do Banco de Dados:
  
- AWS DMS: Utilizar o AWS DMS para migrar o banco de dados MySQL para o RDS.
- Configuração do RDS: Criar um instance RDS MySQL em uma configuração Multi-AZ para alta disponibilidade.
- Atualização das Conexões: Atualizar as conexões da aplicação para apontarem para o novo endpoint do RDS.

5. Implementação da CI/CD:

- AWS CodePipeline: Configurar um pipeline de CI/CD para automatizar o processo de build, teste e deploy das aplicações.
- AWS CodeBuild: Utilizar o CodeBuild para construir as imagens Docker e executar testes.
- AWS CodeDeploy: Utilizar o CodeDeploy para realizar os deployments no Kubernetes.

6. Segurança:

- IAM: Utilizar o IAM para controlar o acesso aos recursos da AWS.
- Security Groups: Configurar os security groups para permitir apenas o tráfego necessário entre os componentes.


