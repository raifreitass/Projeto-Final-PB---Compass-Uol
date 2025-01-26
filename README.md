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

2. Fluxo de Migração:

- Banco de Dados: A migração do MySQL é feita primeiro para o RDS usando o DMS, garantindo que os endpoints já estejam configurados para a nova infraestrutura.
- Servidores Frontend e Backend: O AWS MGN utiliza agentes instalados nos servidores on-premises para replicar dados para uma VPC de Staging. Após os testes, os dados são movidos para a VPC Final.

  


