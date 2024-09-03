# **Bootcamp Cloud: Aula 08: Criando um Código Python com Schedule e Requests para Consultar APIs e Salvar em um RDS**

**Objetivo**: Nesta aula, você aprenderá a desenvolver um código Python que usa as bibliotecas `schedule` e `requests` para bater em uma API a cada 5 minutos e salvar os dados em um banco de dados RDS PostgreSQL hospedado na AWS. O projeto será implementado em um ambiente Docker, utilizando GitHub para versionamento e controle de código. Você também aprenderá a configurar toda a infraestrutura necessária na AWS, incluindo a criação de uma VPC, sub-redes, e grupos de segurança.

### **1. Preparando o Ambiente de Desenvolvimento**

1. **Configuração do Projeto no GitHub**:
   - Crie um repositório no GitHub para armazenar o código do projeto.
   - Clone o repositório em sua máquina local.

2. **Criar o Código Python**:
   - Instale as bibliotecas necessárias (`requests`, `schedule`, `psycopg2`) no seu ambiente Python.
   - Implemente um código que consulta uma API a cada 5 minutos e insere os dados no RDS.

**Exemplo de Código Python**:

```python
import requests
import schedule
import time
import psycopg2
from datetime import datetime

# Configuração do banco de dados RDS
DB_HOST = "seu-rds-endpoint.amazonaws.com"
DB_NAME = "seu_banco"
DB_USER = "admin"
DB_PASS = "sua_senha"

# Função para consultar a API
def consultar_api():
    try:
        response = requests.get('https://api.example.com/data')
        if response.status_code == 200:
            data = response.json()
            salvar_no_rds(data)
        else:
            print(f"Erro ao consultar API: {response.status_code}")
    except Exception as e:
        print(f"Erro ao consultar API: {e}")

# Função para salvar dados no RDS
def salvar_no_rds(data):
    try:
        conn = psycopg2.connect(
            host=DB_HOST,
            database=DB_NAME,
            user=DB_USER,
            password=DB_PASS
        )
        cursor = conn.cursor()
        # Exemplo de inserção de dados
        insert_query = "INSERT INTO dados_api (data, valor) VALUES (%s, %s)"
        cursor.execute(insert_query, (datetime.now(), data['valor']))
        conn.commit()
        cursor.close()
        conn.close()
        print("Dados salvos com sucesso!")
    except Exception as e:
        print(f"Erro ao salvar dados no RDS: {e}")

# Agendar a função para rodar a cada 5 minutos
schedule.every(5).minutes.do(consultar_api)

if __name__ == "__main__":
    while True:
        schedule.run_pending()
        time.sleep(1)
```

3. **Configuração do Docker**:
   - Crie um arquivo `Dockerfile` para empacotar o aplicativo em um container.

**Exemplo de Dockerfile**:

```Dockerfile
# Usar uma imagem base do Python
FROM python:3.12

# Definir o diretório de trabalho
WORKDIR /app

# Copiar arquivos do projeto para o container
COPY . .

# Instalar dependências
RUN pip install requests schedule psycopg2

# Comando para rodar o script
CMD ["python", "main.py"]
```

### **2. Configuração da Infraestrutura na AWS**

#### **1. Criando a VPC e Configurando a Rede**

1. **Criar a VPC**:
   - Acesse o console da AWS e navegue até **VPC** > **Create VPC**.
   - Nome: `MinhaVPC-API`.
   - CIDR Block: `10.0.0.0/16`.

2. **Criar a Sub-rede Pública**:
   - Em **Subnets**, clique em **Create Subnet**.
   - Nome: `MinhaSubnetPublica`.
   - Escolha a zona de disponibilidade (ex: `us-east-1a`).
   - CIDR Block: `10.0.1.0/24`.

3. **Configurar o Internet Gateway**:
   - Navegue até **Internet Gateways** > **Create Internet Gateway**.
   - Nome: `MeuInternetGateway`.
   - Anexe o gateway à VPC criada.

4. **Configurar a Tabela de Roteamento**:
   - Em **Route Tables**, edite a tabela associada à sua VPC.
   - Adicione uma rota para `0.0.0.0/0` apontando para o Internet Gateway.

#### **2. Criando a Instância do RDS na Rede Pública**

1. **Configuração do Banco de Dados no RDS**:
   - Navegue até **RDS** > **Create Database**.
   - Escolha **PostgreSQL** como engine.
   - Utilize o template **Free Tier** (se disponível).
   - Nome da instância: `APIDatabase`.
   - Tipo de instância: `db.t4g.micro`.
   - Habilite o acesso público para ambientes de desenvolvimento.
   - Crie um Security Group para permitir acesso na porta `5432`.

#### **3. Configuração da Instância EC2 e Deploy do Projeto**

1. **Criar uma Instância EC2**:
   - Acesse **EC2** > **Launch Instance**.
   - Escolha uma instância Free Tier (`t2.micro`).
   - Selecione a VPC e sub-rede criadas anteriormente.
   - Habilite o Security Group para permitir acesso na porta `22` (SSH).

2. **Conectar à Instância EC2 e Configurar o Ambiente**:
   - Conecte-se à instância via SSH usando AWS Systems Manager ou o terminal.

**Comandos para Instalar Dependências e Rodar o Docker**:

```bash
# Atualizar pacotes
sudo apt update

# Instalar Git, Docker e Docker Compose
sudo apt install -y git docker.io docker-compose

# Clonar o repositório do GitHub
git clone https://github.com/seuusuario/seurepositorio.git

# Navegar para o diretório do projeto
cd seurepositorio

# Construir a imagem Docker
sudo docker build -t api-schedule-app .

# Rodar o container
sudo docker run -d --name api-schedule-app-container api-schedule-app
```

### **3. Testando e Monitorando o Projeto**

1. **Verifique se o container está rodando**:
   - Utilize o comando `sudo docker ps` para verificar se o container está ativo.

2. **Monitoramento de Logs**:
   - Para visualizar os logs do aplicativo, use `sudo docker logs api-schedule-app-container`.

3. **Testando a Conectividade com o RDS**:
   - Teste a conectividade com o RDS diretamente pelo container para garantir que os dados estão sendo salvos corretamente.

### **Conclusão**

Essa aula mostra como configurar um ambiente completo de desenvolvimento com Python, Docker, GitHub, e AWS, criando uma aplicação que interage com APIs e armazena dados em um RDS PostgreSQL. Esse conhecimento é essencial para automatizar tarefas e integrar serviços de forma eficiente em ambientes de nuvem.

