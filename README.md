# Atividade---AWS---Docker
Este projeto foi desenvolvido como parte de uma atividade prática de DevSecOps, integrando Docker e serviços da AWS.  A atividade envolve o deploy de uma aplicação WordPress com os seguintes requisitos:

- Container para a aplicação.
- Banco de dados MySQL no serviço RDS.
- Configuração do serviço EFS da AWS para armazenar arquivos estáticos do container WordPress.
- Configuração de um Load Balancer AWS para a aplicação WordPress.

# Etapa 1: Criar VPC
Configurações essenciais:
- CIDR Block: Escolha um bloco de IP adequado, como 10.0.0.0/16.
- Sub-redes públicas e privadas: Defina sub-redes para segmentar o tráfego.
- Internet Gateway: Associe um Internet Gateway à VPC para acesso externo.
- NAT Gateway: Configure se precisar de acesso à internet para sub-redes privadas.
- Grupos de segurança: Defina regras de segurança para portas específicas como HTTP e SSH.

  # Etapa 2: Criar RDS para o WordPress
Passo a passo para criar o RDS:

1. Acesse o console AWS e procure pelo serviço RDS.
2. Clique em "Create database".
3. Escolha o banco de dados (Engine type).
4. No campo Templates, selecione Free Tier.

Configure a instância:
- Tipo de instância: Escolha um modelo leve como db.t2.micro ou db.t3.micro (2 vCPUs, 1 GiB RAM).
- Armazenamento: Selecione gp2 com 20 GiB alocados.
- Defina um nome para sua instância RDS.
- Crie as credenciais do banco de dados (username e senha). Guarde essas informações, pois serão necessárias ao configurar o container do WordPress.

Em "Connectivity", configure:
- Security Group: Crie um grupo de segurança configurado para o RDS.
- Availability Zone: Selecione a mesma zona de disponibilidade (AZ) onde está localizada a sua instância EC2.
- Public Access: Ative o acesso público para facilitar a comunicação.
- Na seção "Additional configuration", insira o nome inicial do banco de dados (Initial database name), que também será usado na criação do container do WordPress.

Confirme as configurações clicando em "Create database". Detalhes Importantes:

- Versão do MySQL: Certifique-se de escolher uma compatível com o WordPress.
- Porta do banco de dados: Use o padrão 3306.


# Etapa 3: Criar a EC2
1. Instalação e Configuração do Docker
Criar uma Instância EC2 na AWS:

- Inicie uma instância EC2 no Amazon Web Services.
- Escolha uma imagem (AMI) Linux, como Ubuntu ou Amazon Linux 2.

 Security Group da EC2: 
- SSH (22): Permite acesso remoto de qualquer lugar (0.0.0.0/0).
- HTTP (80): Habilita o acesso público à aplicação (0.0.0.0/0).
 

Crie um arquivo user_data.sh no console da AWS com o seguinte conteúdo:

````
bash
Copiar código
#!/bin/bash
apt-get update
apt-get install -y docker.io
systemctl enable docker
systemctl start docker

``````

Durante a inicialização da instância, o AWS irá executar este script automaticamente para instalar o Docker. Depois, acesse a instância via SSH.





