+++ title = "Nginx SSL with Certbot in Docker without the agonizing pain" date = 2025-02-27

[taxonomies] categories = ["DevOps"] tags = ["nginx","ssl","network","docker"] +++

Certbot makes securing websites easier, offering a straightforward solution to obtain SSL/TLS certificates from Let's Encrypt. This has become the go-to method for many to make their websites more secure and trustworthy by encrypting HTTP traffic. However, when working in a Dockerized environment, the process becomes a bit more complicated.

In a standalone environment, Certbot handles entire process automatically. However, in a containerized setup, the issue is that Docker containers are isolated from each other and running [Certbot](https://hub.docker.com/r/certbot/certbot) image won't configure everything automatically. While [EFF's Certbot docs](https://eff-certbot.readthedocs.io/en/latest/install.html) suggest to generate certificates manually and place them into Nginx volume, another option is a two-phase Docker Compose deployment.


### First phase - obtaining the certificate

In the first phase we will create a temporary Nginx server to handle acme-challenges during certificate generation. We can 

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

TBC

The file tree of the entire deployment with Node.js server 

```bash
├── backend
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── docker-compose-gencert.yml
├── docker-compose.yml
└── etc
    └── nginx
        ├── templates
        │   └── default.conf.template
        └── templates-gencert
            └── default.conf.template
```
