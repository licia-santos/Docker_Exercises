# Nível Fácil

## 1. Rodando um Container Básico


### Adicione os Arquivos da Landing Page

1. Acesse: [Tailwind CSS Landing Page Template](https://tailwindcss.com/templates/landing-page)  
2. Clique em **"View source"**  
3. Baixe ou copie o conteúdo HTML da página  
4. Crie a estrutura de diretórios do projeto:

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


## 3.	Listando e removendo containers
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


# Nível Médio

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
