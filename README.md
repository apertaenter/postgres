# Sumário
- Introdução
- Ambientes
- Instalação
- Configuração
- Conexão remota



## Introdução

Este documento demonstra a criação de um serviço de banco de dados usando o PostgreSQL em um ambiente virtual ubuntu.



## Ambientes

- Servidor
  - máquina virtual (virtualbox)
  - Ubuntu Server 24.04 LTS
  - 1 vCPU
  - 2 GB RAM
  - 40 GB armazenamento
  - ip: 192.168.56.11
  - hostname: pgserver
  - username: dba
  - language: english
- Cliente
  - máquina virtual (virtualbox)
  - Ubuntu Desktop 24.04 LTS
  - 1 vCPU
  - 4 GB RAM
  - 25 GB armazenamento
  - ip: 192.168.56.9
  - hostname: pgclient
  - username: ubuntu
  - language: português



## Instalação

No linux Ubuntu Server, seguir com as seguintes operações usando um usuário administrativo.

```bash
# Import the repository signing key:
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

# Create the repository configuration file:
. /etc/os-release
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"

# Update the package lists:
sudo apt update

# Install the latest version of PostgreSQL:
# If you want a specific version, use 'postgresql-17' or similar instead of 'postgresql'
sudo apt -y install postgresql-17
```
Referência: https://www.postgresql.org/download/linux/ubuntu/



## Configuração

### Mudar a senha do usuário postgres
Entrar no prompt do postgres usando o comando administrativo (sudo)
```bash
sudo su -c 'psql' postgres
```

Definir a senha do usuário postgres
```postgres
postgres=# \password
Enter new password for user "postgres": 
Enter it again: 
postgres=# 
```

### Editar arquivo postgres.conf
Abrir o arquivo para edição
```bash
sudo vi /etc/postgresql/17/main/postgresql.conf
```
Localizar a linha com o começo:

```conf
#listen_addresses = 'localhost'
```
Retirar o comentário (#) e substituir localhost por *, ficando assim:
```conf
listen_addresses = '*'
```
### Editar arquivo pg_hba.conf
Abrir o arquivo para edição
```bash
sudo vi /etc/postgresql/17/main/pg_hba.conf
```
Localizar a seguinte seção:

```conf
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
```
Adicionar uma nova linha logo abaixo, ficando assim:
```conf
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
host    postgres        postgres        192.168.56.0/24         scram-sha-256
```
Com essa configuração, o usuário postgres, usando sua senha, tem permissão para conectar no banco de dados postgres a partir de qualquer host da subrede informada.

### Reiniciar o serviço postgresql

```bash
sudo systemctl restart postgresql
```



## Conexão remota

No linux Ubuntu Desktop, seguir com as seguintes operações usando um usuário administrativo.
```bash
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc
. /etc/os-release
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"
sudo apt update
sudo apt install postgresql-client-common postgresql-client-17
```

Após a instalação, iniciar a sessão remota
```psql
psql -h 192.168.56.11 -U postgres
Senha para o usuário postgres: 
psql (17.6 (Ubuntu 17.6-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: desativado, ALPN: postgresql)
Digite "help" para obter ajuda.

postgres=# 
```
Verifique as informações da conexão
```psql
postgres=# \conninfo
Você está conectado ao banco de dados "postgres" como usuário "postgres" no hospedeiro "192.168.56.11" na porta "5432".
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: desativado, ALPN: postgresql)
postgres=# 
```