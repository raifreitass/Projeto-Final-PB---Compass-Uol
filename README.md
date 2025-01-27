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
- Ferramentas AWS para Migração "Lift-and-Shift" ou "As-Is":

Para realizar migrações nesses modelos, a AWS oferece serviços gerenciados que simplificam e tornam o processo mais eficiente, garantindo a continuidade dos sistemas e a segurança dos dados.  O AWS Application Migration Service (MGN) e o AWS Database Migration Service (DMS) são ferramentas-chave nesse cenário, permitindo migrar aplicações e bancos de dados com o mínimo de impacto e sem a necessidade de alterações significativas.

## Visão Geral da Arquitetura
1. Serviços Utilizados:

- AWS Application Migration Service (MGN): Migra os servidores Frontend (React) e Backend (Nginx + APIs) para instâncias EC2 na AWS.
- AWS Database Migration Service (DMS): Migra o banco de dados MySQL para o Amazon RDS, garantindo a integridade e continuidade dos dados.
- Amazon Elastic Block Store (EBS): Armazena os volumes persistentes das instâncias EC2.
- Amazon S3: Armazena objetos estáticos (imagens, vídeos, arquivos).
- Amazon Route 53: Gerencia DNS e direciona tráfego para recursos na AWS.
- Amazon Virtual Private Cloud (VPC): Isola a infraestrutura e gerencia subnets públicas e privadas.
- AWS Backup: Garante backups automáticos dos recursos.

2. Processo de Migração:

- Banco de Dados: A migração do MySQL é feita primeiro para o RDS usando o DMS, garantindo que os endpoints já estejam configurados para a nova infraestrutura.
- Servidores Frontend e Backend: O AWS MGN utiliza agentes instalados nos servidores on-premises para replicar dados para uma VPC de Staging. Após os testes, os dados são movidos para a VPC Final.


### Passo a Passo
1. Planejamento e Preparação.
   
1.1 Catalogar os Recursos On-Premises:

- Frontend: Servidor com 2 GB RAM, 1 Core CPU, 5 GB de armazenamento.
- Backend: Servidor com 4 GB RAM, 2 Core CPU, 5 GB de armazenamento, utilizando Nginx.
- Banco de Dados: Servidor MySQL com 10 GB RAM, 3 Core CPU, 500 GB de dados.

1.2 Definir a Janela de Manutenção:

- Determine o tempo de inatividade aceitável.
- Planeje rollback para restaurar os servidores originais em caso de falhas.

1.3 Configurar o Ambiente AWS:

Crie uma VPC de Staging e uma VPC Final:
- VPC Staging: Para testes antes de ativar os recursos na VPC final.
- VPC Final: Configurada com subnets públicas (Frontend e Load Balancer) e subnets privadas (Backend e RDS).

2. Migração do Banco de Dados
2.1 Provisionar Amazon RDS:

- Configure um banco de dados MySQL Multi-AZ para garantir alta disponibilidade.
- Utilize backup automático para recuperação de dados.

2.2 Configurar AWS DMS:

*Crie endpoints:*

- Source Endpoint: Banco MySQL on-premises.
- Target Endpoint: RDS MySQL.
  
*Migre:*
- Full Load: Migração inicial completa dos dados.
- CDC (Change Data Capture): Replica as alterações em tempo real.

3. Migração dos Servidores Frontend e Backend
3.1 Instalar AWS Replication Agent:

- Instale o agente nos servidores on-premises.
- Configure para enviar dados para o Replication Server na VPC Staging.

3.2 Configurar o AWS MGN:

- O Replication Server converte os dados em volumes EBS na AWS.
- Gere AMIs e instâncias EC2 correspondentes:
- Frontend: 1 instância EC2.
- Backend: 1 instância EC2 com acesso ao RDS e S3.


4. Configuração de Recursos Complementares
4.1 DNS e Balanceamento de Carga:
   
- Configure o Amazon Route 53 para direcionar o tráfego para o Load Balancer (caso utilizado) ou diretamente para as instâncias EC2.

4.2 Segurança:

Configure Grupos de Segurança:
- Porta 443: Permita acesso HTTPS ao Frontend.
- Porta 3306: Restrinja o acesso ao RDS para o Backend.
- Utilize AWS WAF para proteger o Frontend contra ataques comuns.

4.3 Backup:

- Configure políticas no AWS Backup para EC2, RDS e volumes EBS.

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
- WAF: Utilizar o AWS WAF para proteger a aplicação contra ataques web.
- KMS: Utilizar o KMS para criptografar dados sensíveis.
- Secrets Manager: Utilizar o Secrets Manager para armazenar credenciais de forma segura.

8. Monitoramento e Observabilidade:

CloudWatch: Utilizar o CloudWatch para monitorar o desempenho da aplicação e gerar alertas.
Amazon ECS Anywhere: Integrar o cluster EKS com o Amazon ECS Anywhere para obter visibilidade e gerenciamento centralizados.

