# Local development toolkit

## Units

- mkcert
- traefik (as Loadbalancer & proxy
- dnsmasq (as DNS resolver
- mysql
- redis
- memcached
- minio ( as AWS S3 service
- rethinkdb (optional

## Quick start

### Configure local trust CA certification


```bash
# install mkcert and trust the self-signed CA Root

brew install mkcert nss
mkcert -instal
mkcert "*.local.merik.dev" local.merik.dev localhost 127.0.0.1 ::1

# copy and rename the generated SSL & key to certs folder

cp _wildcard.local.merik.dev+4-key.pem certs/rootCA-key.pem
cp _wildcard.local.merik.dev+4.pem certs/rootCA.pem
```

### Create traefik network

```bash
docker network create traefik
```

### DNS resolver

```bash
cd dnsmasq
docker compose up -d

# visiting https://dns.local.merik.dev with user `root` and `passwd` for password
```

### LoadBalancer

It will occupy the 53, 80, 443, 28015 ports locally.
Visiting https://traefik.local.merik.dev with user `merik` and `Merik@2020!` for password 

```bash
cd traefik
docker compose up -d
```

### MySQL

In docker, you can simply use hostname `mysql` to connect to MySQL server, 3306 port were also exposed to the host machine

Default root password: `JNDn23fVpEyt_6Dd7v`

```bash
cd mysql

# (optional) you can change your mysql root password in the .env file
vim .env

docker compose up -d
```

### Redis

In docker, you can simply use hostname `redis` to connect to Redis server, 6379 post were also exposed to the host machine

Default root password: `JNDn23fVpEyt_6Dd7v`

```bash
cd redis

# You can configure the redis server by editing redis.conf
vim redis.conf

docker compose up -d
```

### Memcached

In docker, you can simply use hostname `memcached` to connect to Memcached server.

```bash
cd memcached

docker compose up -d
```

### MINIO

In docker, you can simply use hostname `minio` to connect to Memcached server.
You can access the console by https://console-minio.local.merik.dev/ with user `minioadmin` same as password.

```bash
cd minio

docker compose up -d
```


## Configure your DNS resolver setting

Change your DNS settings before you visit your local projects.

By visit this guide https://www.techsolutions.support.com/how-to/how-to-change-dns-settings-on-a-mac-10189 and change your DNS server to `127.0.0.1`.

## Proxy services

Traefik will hook the docker process and dynamically create the proxy for new docker instances created.

```yaml
version: '3.8'
services:
  service-not-proxy-by-traefik:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./:/var/www/html/
    labels:
      - "traefik.enable=false" # not proxy by traefik
  service-proxy-by-traefik:
    image: nginx:mainline
    volumes:
      - ./:/var/www/html/
      - .build/nginx/nginx.conf:/etc/nginx/nginx.conf
      - .build/nginx/security.conf:/etc/nginx/security.conf
      - .build/nginx/default.conf:/etc/nginx/conf.d/default.conf
    labels: # in here, you can configure the traefik settings for proxy this service
      - "traefik.http.routers.proxy-service-name.service=proxy-service-name"
      - "traefik.http.routers.proxy-service-name.entrypoints=http,https"
      - "traefik.http.routers.proxy-service-name.rule=Host(`proxy-service-name.local.merik.dev`)"
      - "traefik.http.services.proxy-service-name.loadbalancer.server.port=80"
networks:
  default:
    external:
      name: traefik

```