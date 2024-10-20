# Arquivo Original
Este código provisiona uma infraestrutura básica na AWS, incluindo uma rede privada (VPC), sub-rede, gateway de internet, segurança, e uma instância EC2 executando Debian 12, acessível via SSH.

##### 1. Provider
O provider "aws" define que o Terraform usará a AWS para criar os recursos. A região especificada é us-east-1.
##### 2. Variáveis
Variável "projeto": Define o nome do projeto.
Variável "candidato": Define o nome do candidato.
Essas variáveis são usadas para nomear recursos, o que facilita sua identificação na AWS.
##### 3. Chave Privada (RSA)
Recurso "tls_private_key": Gera uma chave privada RSA com 2048 bits, que será usada para criar um par de chaves SSH para acessar a instância EC2.
Recurso "aws_key_pair": Cria um par de chaves AWS EC2 utilizando a chave pública gerada a partir da chave privada. O nome da chave é composto pelas variáveis projeto e candidato.
##### 4. VPC (Virtual Private Cloud)
Recurso "aws_vpc": Cria uma VPC com o CIDR 10.0.0.0/16, ativando suporte a DNS e resoluções de nomes DNS. Todos os recursos são nomeados de acordo com as variáveis projeto e candidato.
##### 5. Subnet
Recurso "aws_subnet": Cria uma sub-rede dentro da VPC, com o CIDR 10.0.1.0/24, localizada na zona de disponibilidade us-east-1a.
##### 6. Gateway de Internet
Recurso "aws_internet_gateway": Cria um gateway de internet que permitirá que os recursos na VPC tenham acesso à internet.
##### 7. Tabela de Rotas
Recurso "aws_route_table": Cria uma tabela de rotas que define uma rota para a internet, redirecionando o tráfego 0.0.0.0/0 (qualquer destino) para o gateway de internet.
Recurso "aws_route_table_association": Associa a tabela de rotas criada à sub-rede.
##### 8. Grupo de Segurança (Security Group)
Recurso "aws_security_group": Cria um grupo de segurança para a instância EC2. Ele define:
Regras de entrada: Permite acesso via SSH (porta 22) de qualquer IP (0.0.0.0/0 para IPv4 e ::/0 para IPv6).
Regras de saída: Permite todo o tráfego de saída.
##### 9. AMI (Amazon Machine Image)
Data "aws_ami": Filtra a imagem AMI mais recente da Debian 12 com a arquitetura amd64 e tipo de virtualização hvm, sendo esta AMI fornecida pelo ID do proprietário 679593333241.
##### 10. Instância EC2
Recurso "aws_instance": Cria uma instância EC2 utilizando:
AMI da Debian 12.
Tipo de instância t2.micro.
Sub-rede criada anteriormente.
O par de chaves SSH gerado.
O grupo de segurança configurado.
Um endereço IP público associado.
Disco raiz de 20 GB, do tipo GP2.
Um script de user data para atualizar e fazer upgrade do sistema no primeiro boot.
##### 11. Outputs
Output "private_key": Exibe a chave privada para acesso à instância EC2, sendo sensível.
Output "ec2_public_ip": Exibe o endereço IP público da instância EC2 provisionada.

# Arquivo Modificado
##### 1. Melhorias de Segurança
SSH limitado a um IP específico: No grupo de segurança, a regra de entrada para SSH foi alterada para permitir acesso apenas de um endereço IP específico (1.2.3.4/32)

Permissão HTTP/HTTPS: Adicionei permissões para tráfego HTTP (porta 80) e HTTPS (porta 443) de qualquer endereço IP, uma vez que o objetivo é permitir o acesso ao servidor web Nginx.

##### 2. Automação da Instalação do Nginx
Script de User Data: O código foi modificado para alterar o script de User Data que é executado assim que a instância EC2 é provisionada. Esse script realiza as seguintes ações:
Atualiza a lista de pacotes e faz upgrade dos pacotes do sistema.
- Instala o Nginx (um servidor web popular).
- Inicia o serviço do Nginx automaticamente.
- Configura o Nginx para iniciar automaticamente em cada boot da instância.