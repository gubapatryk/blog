+++
title = "Nginx SSL with Certbot in Docker without the agonizing pain" 
date = 2025-02-27

[taxonomies]
categories = ["DevOps"] 
tags = ["nginx","ssl","network","docker"]
+++

Certbot makes securing websites easier, offering a straightforward solution to obtain SSL/TLS certificates from Let's Encrypt. This has become the go-to method for many to make their websites more secure and trustworthy by encrypting HTTP traffic. However, when working in a Dockerized environment, the process becomes a bit more complicated.

{{ responsive(src="docker-nginx.png", width=690, height=400, alt="Dockerized Nginx with SSL") }}

In a standalone environment, Certbot handles entire process automatically. However, in a containerized setup, the issue is that Docker containers are isolated from each other and running [Certbot](https://hub.docker.com/r/certbot/certbot) image won't configure everything automatically. While [EFF's Certbot docs](https://eff-certbot.readthedocs.io/en/latest/install.html) suggest to generate certificates manually and place them into Nginx volume, another option is a two-phase Docker Compose deployment.


### First phase - obtaining the certificate

In the first phase we will create a temporary Nginx server to handle acme-challenges during certificate generation. 

*./nginx/templates-gencert/default.conf.template*
```bash

server {
    listen [::]:80;
    listen 80;
    server_name $DOMAIN;
    location ~/.well-known/acme-challenge {
        allow all;
        root /var/www/certbot;
    }
}
```

*./docker-compose-gencert.yml*
```yml

services:
  nginx:
    container_name: nginx
    image: nginx:latest
    environment:
      - DOMAIN_NAME
    ports:
      - 80:80
    volumes:
      - ./nginx/templates-gencert:/etc/nginx/templates:ro
      - ./certbot/letsencrypt:/etc/letsencrypt:ro
      - ./certbot/data:/var/www/certbot:ro
  certbot:
    container_name: certbot
    image: certbot/certbot:latest
    depends_on:
      - nginx
    command: >- 
             certonly --reinstall --webroot --webroot-path=/var/www/certbot
             --email ${EMAIL} --agree-tos --no-eff-email
             -d ${DOMAIN_NAME}
    volumes:
      - ./certbot/letsencrypt:/etc/letsencrypt:rw
      - ./certbot/data:/var/www/certbot:rw
```

By running *docker compose up* we can generate a certificate for a domain specified as *DOMAIN_NAME* in *.env* file. After successful certificate generation, we can proceed to the normal deployment.

*/nginx/templates-gencert/default.conf.template*
```bash

server {
    listen [::]:80;
    listen 80;
    server_name $DOMAIN_NAME;
    return 301 https://$host$request_uri;
}
 
server {
    listen [::]:443 ssl;
    listen 443 ssl;
    server_name $DOMAIN_NAME; 
 
    ssl_certificate /etc/letsencrypt/live/$DOMAIN_NAME/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/$DOMAIN_NAME/privkey.pem;
 
    location ~ /.well-known/acme-challenge {
        allow all;
        root /var/www/certbot;
    }
 
    location / {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-Host $host;
      proxy_set_header X-Forwarded-Proto https;
      proxy_pass http://backend:3000;
  }
}
```

*./docker-compose.yml*
```yml

services:
  
  nginx:
    container_name: nginx
    image: nginx:latest
    restart: always
    environment:
      - DOMAIN_NAME
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./nginx/templates:/etc/nginx/templates:ro
      - ./certbot/letsencrypt:/etc/letsencrypt:ro
      - ./certbot/data:/var/www/certbot:ro
      
  backend:
    build: ./backend
    restart: always
    container_name: backend
    env_file: .env

  certbot:
    container_name: certbot
    image: certbot/certbot:latest
    depends_on:
      - nginx
    command: >-
             certonly --reinstall --webroot --webroot-path=/var/www/certbot
             --email ${EMAIL} --agree-tos --no-eff-email
             -d ${DOMAIN_NAME}
    volumes:
      - ./certbot/letsencrypt:/etc/letsencrypt:rw
      - ./certbot/data:/var/www/certbot:rw
```


The file tree of the entire deployment code, both for certificate generation and normal deployment, with sample Node.js app. 

```bash

├── backend
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── docker-compose-gencert.yml
├── docker-compose.yml
└── nginx
    ├── templates
    │   └── default.conf.template
    └── templates-gencert
        └── default.conf.template
```

To renew certificates you can use the commands below. They can be placed at crontab to periodically check for renewal:
```bash

docker-compose -f docker-compose.yml run --rm certbot
docker-compose -f docker-compose.yml exec nginx nginx -s reload
```
