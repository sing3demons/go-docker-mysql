
# Dockerfile
```
Dockerfile
```
```
FROM golang:alpine

RUN mkdir /app

WORKDIR /app

ADD go.mod .
ADD go.sum .

RUN go mod download
ADD . .

RUN go get github.com/githubnemo/CompileDaemon

EXPOSE ${PORT}

ENTRYPOINT CompileDaemon --build="go build main.go" --command=./main
```
```
docker build -t hello_world:1.0 .
```
```
docker run --rm -d -p 8000:8000 hello_world:1.0 --name go-blog
```
# .env

```
.env
```
```
MYSQL_ROOT_PASSWORD=root
DB_USER=user
DB_PASSWORD=*******
DB_HOST=db
DB_PORT=3306
DB_NAME=golang
SERVER_PORT=8000
ENVIRONMENT=local
PORT=8080
```

# mysql

```
docker-compose.yml
```
```yml 
version: '3.9'
services:
  db:
    image: mysql:5.7.22
    ports:
      - "3305:3306"
    restart: always
    environment:
      - "MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}"
      - "MYSQL_USER=${DB_USER}"
      - "MYSQL_PASSWORD=${DB_PASSWORD}"
      - "MYSQL_DATABASE=${DB_NAME}" 
    volumes:
     - .db_data:/var/lib/mysql
  web:
    build: .
    ports:
      - "8000:${PORT}"
    volumes:
      - ".:/app"
    restart: always
    depends_on:
      - db
    links:
      - "db:database"
```

#  postgres
```
docker-compose.yml
```

```yml 
version: '3.8'
services:
    db:
        image: postgres
        environment:
          - POSTGRES_DB=${DB_NAME}
          - POSTGRES_USER=${DB_USER}
          - POSTGRES_PASSWORD=${DB_PASSWORD}
        ports:
          - 5432:5432
        volumes:
          - ./db_data:/var/lib/postgresql/data
        restart: always
    web:
        build: .
        ports:
          - "8080:${PORT}"
        volumes:
          - ".:/app"
        restart: always
        depends_on:
          - db
        links:
          - "db:database"
```

# mongo
```
version: '3.1'

services:

  mongo:
    image: mongo
    container_name: "mongo"
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: password
    ports: 
        - 27017:27017
    volumes: 
        - .gomongodbdata:/data/db
```

```
"mongodb://root:example@localhost:27017"
```


# redis
```
version: '3.5'

services:
    redis:
    container_name: "redis"
    image: redis:alpine
     # Specify the redis.conf file to use and add a password.
    command: redis-server /usr/local/etc/redis/redis.conf --requirepass passw0rd
    ports:
      - 6379:6379
     # map the volumes to that redis has the custom conf file from this project.
    volumes:
      - $PWD/redis.conf:/usr/local/etc/redis/redis.conf
```
```
redis.conf
```
redis.conf > maxmemory 41943040
```
# Use the official redis.conf file supplied by redis.
# As a test use a simple config for testing.
maxmemory 41943040
```

# nginx
```
nginx:
    image: nginx:latest
    container_name: webserver
    restart: unless-stopped
    ports:
      - 80:80
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - webapi #connect web 
```

# nginx.conf

```
user nginx;
# can handle 1000 concurrent connections
events {
    worker_connections 1024;
}
# forwards http requests
http {
        # http server
        server {
              # listens the requests coming on port 80
              listen 80;
              access_log  off;
              # / means all the requests have to be forwarded to api service
              location / {
                # resolves the IP of api using Docker internal DNS
                proxy_pass http://webapi:8080;
              }
        }
}
```


# .env 

```
.env
```
```
PORT=8080
DB_HOST=db
DB_USER=dnz-dev
DB_PASSWORD=dnz-dev
DB_NAME=dnz-dev
```

run command 
``` 
docker-compose up --build 
```
