# Installs
## UBI (RHEL-like)
FROM ubi9
RUN dnf install httpd && dnf clean all

## Ubuntu
FROM ubuntu
RUN apt-get update && apt-get install -y apache2

## Alpine
FROM alpine
RUN apk add --no-cache apache2

## Arch
pacman -Syu package

## Python
pip install requests

## OpenSUSE
zypper install package

## Node.js
npm install

# Starting web in the foreground
##  httpd
httpd -D FOREGROUND
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]

## nginx
nginx -g "daemon off;"
CMD ["nginx", "-g", "daemon off;"]


# nginx config

events {}
 
http {
server {
listen 80;
 
location / {
proxy_pass http://app:80;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
}
}
}

# dashboard setup
chmod +x dashboard.sh

# advanced compose
services:
  nginx:
    image: nginx
    ports:
      - "8080:80"
    volumes: 
      - ./nginx.conf:/etc/nginx/nginx.conf:ro,Z
    depends_on:
      - app
 
  adminer:
    image: adminer
    ports:
      - "8081:8080"
    depends_on:
      - db
 
  app:
    image: nginxdemos/hello
    container_name: app
    mem_limit: 512m
    depends_on:
      db:
        condition: service_healthy
 
  db:
    image: postgres:16
    container_name: db
    mem_limit: 512m
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d testdb"]
      interval: 5s
      timeout: 30s
      retries: 10
      start_period: 5s
    environment:
      POSTGRES_DB: testdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: sec
    volumes:
      - task3:/var/lib/postgresql/data
 
volumes:
  task3:



---------------------
# Examples

# multi stage
FROM alpine as builder
run echo "artefact" > /file
 
from alpine
copy --from=builder /file /file
cmd ["/bin/sh"]

# healthcheck
from nginx
 
copy . .
 
healthcheck --interval=5s --timeout=5s cmd curl http://localhost:80/ || exit 1

# non root user
from alpine
 
copy /python /app
 
run adduser -S myuser && mkdir -p /app && chown -R myuser /app
user myuser 

workdir /app
 
cmd whoami && ls -la

# pinned version
from alpine
 
run apk add --no-cache curl=8.20.0-r1 && rm -rf /var/cache/apk/*

# python static server
from python
 
expose 8080
 
copy /python .
 
cmd ["python", "-m", "http.server", "8080"]
 
services:
  app:
    image: nginx
    container_name: app
    ports:
      - "8080:80"
    depends_on:
      - db
 
  db:
    image: postgres
    container_name: db
    enviornment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: sec
      POSTGRES_DB: testdb
    volumes:
      - simple:/var/lib/postgresql/data
 
volumes:
  simple: