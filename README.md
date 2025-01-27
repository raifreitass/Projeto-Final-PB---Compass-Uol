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
Seguir as melhores práticas arquitetura em Cloud AWS, a nova arquitetura deve seguir as seguintes
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


#### Passo a Passo
1. Planejamento e Preparação
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
