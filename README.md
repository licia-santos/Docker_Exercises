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

# Nível Médio

## 5. Criando e utilizando volumes para persistência de dados

1. Acesse: https://github.com/docker/awesome-compose/tree/master/react-express-mysql

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
