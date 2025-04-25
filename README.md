# 🟢 Nível Fácil

## 1. Rodando um Container Básico


### Adicione os Arquivos da Landing Page

1. Acesse: [Tailwind CSS Landing Page Template](https://tailwindcss.com/templates/landing-page)  
2. Clique em **"View source"**  
3. Baixe ou copie o conteúdo HTML da página  
4. Crie a estrutura de diretórios do projeto em um editor de código (Exemplo: Visual Studio Code)

meu-projeto/ 
|── Dockerfile
│── site/ 
    └── index.html 

### Escreva um Dockerfile para usar o Nginx

Crie um arquivo chamado Dockerfile na raiz do projeto com este conteúdo:
````
# Usando imagem oficial do Nginx
FROM nginx:alpine

# Remove a página padrão
RUN rm -rf /usr/share/nginx/html/*

# Copia os arquivos do seu site para dentro do container
COPY site/ /usr/share/nginx/html/

# Expondo a porta 81
EXPOSE 81
````

### Construa a imagem Docker
```
docker build -t meu-nginx .
```

### Execute o container
```
docker run -d -p 8081:80 --name nginx-tailwind meu-nginx
```

Agora você pode abrir no navegador:

`http://localhost:8081`

![Captura de tela 2025-04-19 173315](https://github.com/user-attachments/assets/d0e91c05-0ddb-4c00-b41a-30d2f3830fcf)


## 2. Criando e rodando um container interativo
```
docker run -dti --name Ubuntu-A ubuntu

docker exec -ti Ubuntu-A bash

apt update && apt install nano

nano logs.sh
```

Dentro de logs.sh: 
```
#!/bin/bash

echo "=== Logs do Kernel ==="
dmesg | tail -50
```

Depois no terminal: 

```
chmod +x logs.sh

./logs.sh
```
![interativo](https://github.com/user-attachments/assets/5dc1ce47-6dcc-4c02-89d9-291b8e0bde32)


## 3. Listando e removendo containers
Para listar apenas os containers em execução:

```
docker ps
```

Para listar todos os containers, inclusive os parados:

```
docker ps -a
```

Parar um container em execução:

```
docker stop <container_id>
````

![container apagado](https://github.com/user-attachments/assets/5170c0a6-efc4-491f-91da-56549b9f4941)

## 4. Criando um Dockerfile para uma aplicação simples em Python

1. Baixe o seguinte projeto: `https://github.com/docker/awesome-compose/tree/master/flask`
2. Abra esse arquivo em um editor de código (Exemplo: Visual Studio Code)
3. Confira se os seguintes arquivos estão na porta `5000`:
   * app.py
   * Dockerfile
   * compose.yaml
     
Depois de conferir esses arquivos, você pode construir e rodar seu container assim:

```
docker build -t flask-app .
docker run -p 5000:5000 flask-app
```

![Captura de tela 2025-04-23 092253](https://github.com/user-attachments/assets/16ac17b4-90e1-41ca-90d7-deef6a56c71b)

Acesse em: `http://localhost:5000`

Você verá a resposta: `"Hello World!"`.

![Captura de tela 2025-04-23 092200](https://github.com/user-attachments/assets/5230a978-056a-44ff-8464-c5d8a7a8ebf7)

# 🟡 Nível Médio

## 5. Criando e utilizando volumes para persistência de dados

1. Baixe o seguinte projeto: `https://github.com/docker/awesome-compose/tree/master/react-express-mysql`
2. A pasta react-express-mysql contém:
   * frontend/: código React
   * backend/: código Express
   * mysql/: configuração do MySQL (como init.sql)
   * docker-compose.yaml: orquestra os serviços
3. Abra esse arquivo em um editor de código (Exemplo: Visual Studio Code)
4. No `docker-compose.yaml`, vamos garantir que o serviço db (MySQL) utilize um volume nomeado, para que os dados persistam.
5. Abra o `docker-compose.yaml` e localize a parte do serviço `db`
   
````
services:
  backend:
    build:
      args:
      - NODE_ENV=development
      context: backend
      target: development
    command: npm run start-watch
    environment:
      - DATABASE_DB=example
      - DATABASE_USER=root
      - DATABASE_PASSWORD=/run/secrets/db-password
      - DATABASE_HOST=db
      - NODE_ENV=development
    ports:
      - 80:80
      - 9229:9229
      - 9230:9230
    secrets:
      - db-password
    volumes:
      - ./backend/src:/code/src:ro
      - ./backend/package.json:/code/package.json
      - ./backend/package-lock.json:/code/package-lock.json
      - back-notused:/opt/app/node_modules
    networks:
      - public
      - private
    depends_on:
      - db
  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal
    # If you really want to use MySQL, uncomment the following line
    #image: mysql:8.0.27
    command: '--default-authentication-plugin=mysql_native_password'
    restart: always
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - private
    environment:
      - MYSQL_DATABASE=example
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db-password
  frontend:
    build:
      context: frontend
      target: development
    ports:
      - 3000:3000
    volumes:
      - ./frontend/src:/code/src
      - /code/node_modules
    networks:
      - public
    depends_on:
      - backend
networks:
  public:
  private:
volumes:
  back-notused:
  db-data:
secrets:
  db-password:
    file: db/password.txt
````

6. Na seção de volumes no final do arquivo certifique-se que essa parte existe no final do `docker-compose.yaml`:
   
```
volumes:
  back-notused:
  db-data:
```

Isso declara o volume nomeado `db-data`.

### Execute o projeto
```
docker-compose up -d
```
Esse comando:

1. Sobe os containers (React, Express, MySQL)

2. Cria o volume db-data

3. Aplica o script de inicialização init.sql (se houver)

### Testando a persistência

1. Insira algum dado no banco via API (Express) ou manualmente.

2. Remova o container do MySQL

````
docker-compose stop db
docker-compose rm -f db
````

### Suba o container novamente:

```
docker-compose up -d db
```

### Verificando volumes:

Veja os volumes criados:
```
docker volume ls
```

Inspecione o conteúdo:
```
docker volume inspect react-express-mysql_db-data
```

![Captura de tela 2025-04-22 110043](https://github.com/user-attachments/assets/ac532818-1a0e-4819-b504-0962b5da06b2)

## 6. Criando e rodando um container multi-stage

1. Baixe o seguinte projeto: `https://github.com/docker/docker-gs-ping`
2. Abra esse arquivo em um editor de código (Exemplo: Visual Studio Code)
3. Crie um `Dockerfile` com o seguinte código:

```
# Etapa 1: build
FROM golang:1.21 AS builder

WORKDIR /app

COPY go.mod ./
COPY go.sum ./
RUN go mod download

COPY . .

# Corrigido: compila o main.go da raiz
RUN go build -o gs-ping main.go

# Etapa 2: imagem final
FROM debian:bookworm-slim

WORKDIR /app

COPY --from=builder /app/gs-ping .

EXPOSE 8080

CMD ["./gs-ping"]

```

Para construir a imagem:

```
docker build -t gs-ping .
```

Para rodar o container:
```
docker run -p 8080:8080 gs-ping
```
Agora é só acessar: `http://localhost:8080/ping`

Você verá a resposta: `"OK"`.

![Captura de tela 2025-04-22 103755](https://github.com/user-attachments/assets/e7e0c835-fbd8-4880-863c-a5949b71b77d)

Verifique o tamanho da imagem final:

```
docker images gs-ping
```

![Captura de tela 2025-04-22 111038](https://github.com/user-attachments/assets/26c7309a-ffa0-4f44-9f04-c2e07454806f)

## 7. Construindo uma rede Docker para comunicação entre containers

1. Baixe o seguinte projeto: `https://github.com/docker/awesome-compose/tree/master/react-express-mongodb`

No terminal, execute:

```
docker-compose up --build
```

![Captura de tela 2025-04-23 095138](https://github.com/user-attachments/assets/aa18dbd3-52e0-4fb8-a389-16b5ac57baee)

Acesse a API em: `http://localhost:3000`

![Captura de tela 2025-04-23 095657](https://github.com/user-attachments/assets/6ae2a8eb-3dd3-4a71-878b-f8e6cbc4ff56)

## 8. Criando um compose file para rodar uma aplicação com banco de dados

1. Baixe o seguinte projeto: `https://github.com/docker/awesome-compose/tree/master/postgresql-pgadmin`

No terminal, execute:

```
docker-compose up -d
```

![Captura de tela 2025-04-23 162149](https://github.com/user-attachments/assets/157a9e04-bf78-498f-a056-d684cfe8313d)

Abre o navegador e acesse: `http://localhost:5050`

![Captura de tela 2025-04-23 162157](https://github.com/user-attachments/assets/6c3a57ee-af38-482d-9043-4a4c05f230a0)

Para acessar coloque nos campos:

* Email: `your@email.com`
* Senha: `changeit`

![Captura de tela 2025-04-23 170157](https://github.com/user-attachments/assets/b524f648-4d16-4b3f-8c12-578d6b792d78)


# 🔴 Nível Difícil

## 9. Criando uma imagem personalizada com um servidor web e arquivos estáticos

1. Baixe o seguinte projeto: `https://github.com/creativetimofficial/material-kit`
2. Agora você só precisa copiar os arquivos que ficam dentro da pasta `material-kit` para uma pasta no seu projeto (por exemplo, `site`).
3. Crie um Dockerfile com as seguintes informações:

```
# Usa a imagem oficial do Nginx
FROM nginx:alpine

# Apaga os arquivos default do Nginx
RUN rm -rf /usr/share/nginx/html/*

# Copia os arquivos do site para o diretório padrão do Nginx
COPY . /usr/share/nginx/html/

# Expõe a porta 80
EXPOSE 80

# Inicia o Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Comandos para build e run

No terminal:

```
docker build -t meu-site-nginx .
```

Depois:
```
docker run -d -p 8080:80 meu-site-nginx
```

![Captura de tela 2025-04-24 093717](https://github.com/user-attachments/assets/6d57b5e6-9155-4a0a-aa3e-33116463ef25)


Acesse em: `http://localhost:8080`

![Captura de tela 2025-04-24 093616](https://github.com/user-attachments/assets/860edae0-9555-4041-bd3b-a8c82b88bc2b)

## 10. Evitar execução como root

### Crie uma pasta chamada `site-teste`

1. Crie os arquivos dentro da sua pasta `site-teste`

a. `app.js`
```
const http = require('http');

const server = http.createServer((req, res) => {
  res.end('Ola, Docker sem root!');
});

server.listen(3000, () => {
  console.log('Servidor rodando na porta 3000');
});
```

b. `package.json`

```
{
  "name": "site-teste",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
}
```

2. Agora crie o `Dockerfile`:
```
# Imagem base
FROM node:18

# Criar diretório da aplicação
WORKDIR /app

# Copiar arquivos para dentro do container
COPY package*.json ./
RUN npm install
COPY . .

# Criar um usuário não-root
RUN adduser --disabled-password --gecos '' appuser

# Usar o novo usuário
USER appuser

# Expor porta e iniciar app
EXPOSE 3000
CMD ["npm", "start"]
```

3. Abra o terminal na pasta `site-teste` e rode:
```
docker build -t site-teste .
docker run -d -p 3000:3000 --name site-container site-teste
```

Acesse em: `http://localhost:3000`

![Captura de tela 2025-04-24 155338](https://github.com/user-attachments/assets/a51d8857-fae8-4d3f-971d-dcf2487cc6a8)


4. Verificar usuário em execução:
```
docker exec site-container whoami
```
![Captura de tela 2025-04-24 154304](https://github.com/user-attachments/assets/60c58df6-f2d7-4cbf-a9e3-258e93c83f8b)

## 11. Analisar imagem Docker com Trivy

1. Instalação do Trivy na sua máquina
```
# Baixar o Trivy com o comando curl
sudo apt install wget
wget https://github.com/aquasecurity/trivy/releases/download/v0.39.0/trivy_0.39.0_Linux-64bit.deb


# Instalar o pacote
sudo dpkg -i trivy_0.39.0_Linux-64bit.deb
```

2. Rode o comando trivy image `<nome-da-imagem>`

```
trivy image `<nome-da-imagem>
```

3. Identificar vulnerabilidades com severidade HIGH ou CRITICAL
O Trivy classificará as vulnerabilidades encontradas por severidade (Low, Medium, High, Critical). Fique atenta às vulnerabilidades de severidade HIGH e CRITICAL, pois essas são as mais críticas e podem representar riscos à segurança.

As vulnerabilidades serão listadas com informações detalhadas, como:
* Nome do pacote afetado.
* Descrição da vulnerabilidade.
* CVE (se aplicável).
* Severidade.

4. Exemplo de saída
```
Total: 3 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 2, CRITICAL: 0)
```

## 12. Corrigir vulnerabilidades encontradas

### Dockerfile com más praticas:
```
Dockerfile
Dockerfile vulnerável
FROM python:3.9
WORKDIR /app COPY requirements.txt . RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

Problemas do Dockerfile original:
* Imagem base genérica (`python:3.9`) – Pode ter muitas libs desnecessárias.
* Usuário root por padrão – Pode representar riscos de segurança.
* Dependências desatualizadas – As versões antigas no `requirements.txt` podem ter vulnerabilidades conhecidas.
* Camadas não otimizadas – Instalação de pacotes sem limpeza ou cache pode deixar a imagem maior.
* Sem especificar explicitamente um ambiente de produção – Não é ideal para segurança e performance.

### Dockerfile atualizado:
```
# Usar uma imagem slim para reduzir tamanho e superfície de ataque
FROM python:3.9-slim

# Define variáveis de ambiente para evitar criação de arquivos .pyc e definir path não interativo
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Cria e usa um usuário não-root
RUN adduser --disabled-password --no-create-home appuser

# Diretório de trabalho
WORKDIR /app

# Copia apenas o requirements.txt primeiro (melhora cache)
COPY requirements.txt .

# Atualiza o apt e instala dependências mínimas necessárias (por ex. gcc, libpq-dev), depois remove o cache
RUN apt-get update \
    && apt-get install -y --no-install-recommends gcc \
    && pip install --upgrade pip \
    && pip install -r requirements.txt \
    && apt-get purge -y --auto-remove gcc \
    && rm -rf /var/lib/apt/lists/*

# Copia o restante do código
COPY . .

# Altera para o usuário não-root
USER appuser

# Comando padrão
CMD ["python", "app.py"]
```

### Atualização das libs no `requirements.txt`:

De: 
```
flask==1.1.1
requests==2.22.0
```

Para:
```
flask==2.2.5
requests==2.31.0
```
Usando versões mais recentes, com correções de segurança.
