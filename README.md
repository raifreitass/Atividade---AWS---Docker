# Atividade---AWS---Docker
Este projeto foi desenvolvido como parte de uma atividade prática de DevSecOps, integrando Docker e serviços da AWS.  A atividade envolve o deploy de uma aplicação WordPress com os seguintes requisitos:

- Container para a aplicação.
- Banco de dados MySQL no serviço RDS.
- Configuração do serviço EFS da AWS para armazenar arquivos estáticos do container WordPress.
- Configuração de um Load Balancer AWS para a aplicação WordPress.

![image](https://github.com/user-attachments/assets/19ce08ac-4f0a-47cf-8baa-c4242a28b1b3)


## Etapa 1: Criar VPC
Configurações essenciais:
- CIDR Block: Escolha um bloco de IP adequado, como 10.0.0.0/16.
- Sub-redes públicas e privadas: Defina sub-redes para segmentar o tráfego.
- Internet Gateway: Configure também um gateway de internet.
- NAT Gateway: Ative um gateway NAT em cada zona.

![image](https://github.com/user-attachments/assets/479a8f66-ef9d-4d89-accf-333130a1304e)

 
Finalize a criação da VPC.


## Etapa 2: Configurando Regras de Acesso (Grupos de Segurança)
Grupo Público, entradas permitidas:

> HTTP (porta 80) de qualquer origem (0.0.0.0/0).
> 
> HTTPS (porta 443) de qualquer origem (0.0.0.0/0).
>
> SSH (porta 22) de qualquer origem (0.0.0.0/0).


Saídas permitidas:
>Todo o tráfego, sem restrição de portas ou protocolos.




Grupo Privado, entradas permitidas:
  
> MySQL (porta 3306) de qualquer origem.
>
> HTTP (porta 80) e HTTPS (porta 443) apenas do grupo público.
>
>  SSH (porta 22) de qualquer origem.
>
> NFS (porta 2049) de qualquer origem.

Saídas permitidas:
> Todo o tráfego liberado.


## Etapa 3: Configuração do Sistema de Arquivos (EFS)
1. No console AWS, acesse o serviço EFS (Elastic File System) e inicie a criação do sistema:
- Dê um nome a EFS.
- Selecione as sub-redes privadas configuradas na VPC para integrar o EFS.
  
2. Finalize a criação e configure o ponto de montagem:
- Escolha a opção recomendada para sistemas Linux.
- Anote as instruções de montagem, pois serão usadas no script de inicialização.
- Certifique-se de que o EFS esteja associado ao grupo de segurança privado para garantir segurança e compatibilidade com os outros serviços da arquitetura.


## Etapa 4: Criar uma Sub-Rede para o RDS
É necessário configurar uma sub-rede específica para o banco de dados. Essa sub-rede privada isola o RDS de acessos externos, garantindo que ele seja acessado apenas por recursos internos, como as instâncias EC2.

1. Vá em "Grupo de Sub-redes" e inicie a criação.
- Dê um nome para a sua sub-rede.
- Selecione sua vpc criada anteriormente.
  
Agora, escolha a zona de disponibiliade e as sub-redes privadas da seguinte maneira:
![Captura de tela 2024-12-23 163553](https://github.com/user-attachments/assets/c85e54f2-aff7-404f-a007-2f5554903b9d)


## Etapa 5: Criar RDS para o WordPress
1. No console AWS, procure pelo serviço RDS e clique em "Criar banco de dados".
2. Escolha as seguintes configurações básicas:
   
- Tipo de mecanismo: MySQL.
- Modelo: Camada gratuita (Free Tier).
- Tipo de instância: db.t3.micro.
- Armazenamento: Configuração padrão.
  
3. Configure as credenciais:
- Crie um nome de usuário e senha para o banco. Anote essas informações, pois serão usadas mais tarde no container do WordPress.
  
4. Em Conectividade:
- Grupo de segurança: Vincule o grupo privado que foi configurado anteriormente.
- Zona de disponibilidade (AZ): Escolha a mesma AZ onde está localizada uma de suas instâncias EC2.
- Selecione a sub-rede criada para o RDS.
- Acesso público: Desative esta opção para manter o banco de dados protegido contra acessos externos diretos.

5. Em Configuração adicional:
- Insira o nome inicial do banco (por exemplo, wordpress_db), que será usado na configuração do container.
- Adicione tags, se necessário, para identificar o recurso.
- Confirme clicando em "Criar banco de dados".


## Etapa 6: Instâncias EC2
1. Crie duas instâncias EC2, uma em cada zona de disponibilidade:
- Utilize a AMI "Amazon Linux 2023".
- Escolha o tipo de instância t2.micro.
- Gere e salve uma chave SSH.
  
2. Configure a rede:
- Selecione a VPC e uma sub-rede privada.
- Desative o IP público automático.
- Vincule ao grupo de segurança privado.

3. Adicione o seguinte script de inicialização no user_data:

``` #!/bin/bash 
 
sudo yum update -y 
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
newgrp docker
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo mkdir /home/ec2-user/wordpress
cat <<EOF > /home/ec2-user/wordpress/docker-compose.yml
services:
 
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: <aqui coloque o endpoint do seu RDS>:3306
      WORDPRESS_DB_USER: <seu user>
      WORDPRESS_DB_PASSWORD: <sua senha>
      WORDPRESS_DB_NAME: <aqui coloque aquele nome que colocamos nos detalhes adicionais do RDS(revise o passo 4, tópico 11)>
    volumes:
      - /mnt/efs:/var/www/html
EOF
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-082b0295d71cc0635.efs.us-east-1.amazonaws.com:/ /mnt/efs
docker-compose -f /home/ec2-user/wordpress/docker-compose.yml up -d

``````


## Etapa 7: Configurando o Load Balancer
Escolha o tipo "Classic Load Balancer".

1. Configure:
- Utilize a VPC e as sub-redes públicas.
- Selecione o grupo de segurança público.
- Adicione as instâncias EC2 criadas.

  
2. Configure o monitoramento de saúde:
- Protocolo: HTTP.
- Porta: 80.
- Caminho: /wp-admin/install.php.

![image](https://github.com/user-attachments/assets/aa482105-6dcc-4ea9-86f1-6e5c132789bd)


  
## Etapa 8: Criando um Grupo de Auto Scaling

1. Configure o Auto Scaling para replicar automaticamente suas instâncias EC2:
- Use a mesma configuração das EC2 criadas anteriormente, exceto pelas sub-redes (selecione sub-redes privadas).
- Vincule ao Load Balancer configurado.

2. Finalize a criação do grupo.


## Etapa 9: Acessando o WordPress

- Copie o DNS do Load Balancer e cole no navegador para acessar o painel de configuração do WordPress.

![Captura de tela 2024-12-23 035851](https://github.com/user-attachments/assets/961fff8f-4db5-4eea-9a87-8f5ae67e3bf4)







