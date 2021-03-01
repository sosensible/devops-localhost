# Docker DevOps for Developers Simplified
This guide is for Windows running Linux baseed Docker.

## 0. Prerequisites

- [Chocolatey](https://chocolatey.org/install)
- [Docker](https://docs.docker.com/docker-for-windows/install/)
- [Git](https://git-scm.com/)
- [MkCert](https://chocolatey.org/packages/mkcert) (install with Chocolatey)
- [VSCode](https://code.visualstudio.com/)


## 1. Setup a local Root CA

```sh
> choco install mkcert

# Setup the local Root CA
> mkcert -install

# Local Root CA files are located under ~/Library/Application\ Support/mkcert
# Look at https://github.com/FiloSottile/mkcert is you need instructions to install them on another device
```

## 2. Setup a Traefik container w/ https

```sh
# Clone this repository
mkdir devops-localhost
cd devops-localhost
git clone https://github.com/sosensible/devops-localhost.git .

# Create an external network for shared services
docker network create inbound

# Create a local development TLS certificate
mkcert -cert-file certs/ssl.crt -key-file certs/ssl.key "docker.localhost" "*.docker.localhost"

# Start DevOps Container Stack
docker-compose pull
docker-compose up -d

# Go on https://traefik.docker.localhost you should have the traefik web dashboard serve over https
```

## 3. Setup your dev containers

```sh
# On your docker-compose.yaml file

# Add the external network web at the end of the file
networks:
  default:
    external:
      name: inbound

# Add these labels on the container
    labels:
      - traefik.enable=true
      - traefik.http.routers.mydevproxy.entrypoints=http,https
      - traefik.http.routers.mydevproxy.rule=Host(`traefik.docker.localhost`)
      - traefik.http.routers.mydevproxy.tls=true
      - traefik.http.routers.mydevproxy.service=api@internal

# Protip: For web applications, use the same origin domain for your frontend and backend to avoid cookies sharing issues.
# By example: https://this.test (frontend) and https://api.this.test (backend)
```
