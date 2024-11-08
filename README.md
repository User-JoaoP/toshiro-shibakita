
# Projeto Docker com NGINX, PHP e MySQL

Este projeto demonstra a configuração de um ambiente com Docker, utilizando NGINX como proxy reverso, PHP para o backend, e MySQL como banco de dados. O projeto foi desenvolvido para consolidar conhecimentos sobre containers, Docker Swarm, volumes, e redes, além de implementar balanceamento de carga com o NGINX.

## Estrutura do Projeto

### 1. Arquivos principais

- **banco.sql**: Script para criação da tabela `dados` no banco de dados.
  - Tabela: `dados`
    - `AlunoID`: Identificador único do aluno (int).
    - `Nome`, `Sobrenome`, `Endereco`, `Cidade`, `Host`: Campos de texto com informações variadas sobre o aluno.
  
- **Dockerfile**: Arquivo Dockerfile usado para criar a imagem do NGINX com uma configuração personalizada.
  ```Dockerfile
  FROM nginx
  COPY nginx.conf /etc/nginx/nginx.conf
  ```

- **index.php**: Script PHP que exibe a versão atual do PHP e insere um registro aleatório na tabela `dados`.
  - Cria uma conexão com o banco de dados usando as credenciais fornecidas.
  - Gera valores aleatórios para o `AlunoID`, `Nome`, `Sobrenome`, `Endereco`, `Cidade`, e o nome do host atual.
  - Insere os dados gerados na tabela `dados` do banco.

- **nginx.conf**: Configuração do NGINX para balanceamento de carga entre três servidores.
  - Define um upstream chamado `all` com três servidores, e redireciona as requisições para a porta 4500.

### 2. Pré-requisitos

- Docker e Docker Compose instalados.
- MySQL configurado e rodando em um container ou em um servidor remoto.

### 3. Configuração do Banco de Dados

Crie a tabela `dados` em seu banco de dados MySQL usando o script `banco.sql`:

```sql
CREATE TABLE dados (
    AlunoID int,
    Nome varchar(50),
    Sobrenome varchar(50),
    Endereco varchar(150),
    Cidade varchar(50),
    Host varchar(50)
);
```

### 4. Configuração do PHP

O arquivo `index.php` é responsável por:
- Exibir a versão do PHP.
- Conectar ao banco de dados.
- Inserir um registro aleatório na tabela `dados`.

Certifique-se de substituir as credenciais de conexão (`$servername`, `$username`, `$password`, `$database`) no `index.php` para apontar para seu banco de dados.

### 5. Configuração do NGINX

No `nginx.conf`, definimos três servidores para balanceamento de carga:

```nginx
http {
    upstream all {
        server 172.31.0.37:80;
        server 172.31.0.151:80;
        server 172.31.0.149:80;
    }
    server {
         listen 4500;
         location / {
              proxy_pass http://all/;
         }
    }
}
```

Esses IPs precisam estar acessíveis e rodando o serviço que você quer balancear (neste caso, provavelmente o serviço PHP).

### 6. Build e Execução

1. Crie a imagem do NGINX com o Dockerfile fornecido:

   ```bash
   docker build -t meu_nginx .
   ```

2. Execute o container do NGINX:

   ```bash
   docker run -d -p 4500:4500 meu_nginx
   ```

3. Execute o script PHP em um ambiente que tenha acesso ao banco de dados.

### 7. Testando o Projeto

Acesse `http://localhost:4500` para ver o proxy em ação. O script PHP inserirá registros no banco de dados MySQL automaticamente em cada acesso.
