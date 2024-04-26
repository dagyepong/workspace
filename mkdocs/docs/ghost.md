## Setting Up Ghost Blog with Docker Compose¶

### Introduction to Ghost¶

Ghost is a powerful app for professional publishers to create, share, and grow a business around their content. It comes with modern tools to build a website, publish content, send newsletters & offer paid subscriptions to members.

#### Docker Compose Configuration for Ghost Blog¶

This Docker Compose setup deploys Ghost application in a Docker container, allowing you to have a personalized blog to share ideas to people.

Docker Compose File (docker-compose.yml)¶

```yaml
version: '3.3'

services:

  blog:
    image: ghost:5-alpine
    container_name: ghost
    restart: always
    depends_on:
      database:
        condition: service_healthy
    healthcheck:
      test: "/usr/bin/nc localhost 2368 || exit 1"
      interval: 30s
      timeout: 10s
      retries: 5        
    expose:
      - 2368
    ports:
      - 2368:2368
    volumes:
      - ${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/ghost/content:/var/lib/ghost/content
    environment:
      # see https://ghost.org/docs/config/#configuration-options
      database__client: ${DB_CLIENT:-mysql}
      database__connection__host: ${DB_HOST:-database}
      database__connection__user: ${DB_USER:-ghost}
      database__connection__password: ${DB_USER_PASS:-DatabasePassword1234}
      database__connection__database: ${DB_NAME:-ghost}
      url: https://blog.nanaoware.online # adjust to your domain and correct protocol handler + port
      #NODE_ENV: developmnent # default is production already
      #mail__transport: SMTP
      #mail__options__host: ${SMTP_HOST:-smtp.google.com}
      #mail__options__port: ${SMTP_PORT:-587}
      #mail__options__auth__user: ${SMTP_USER:-ghost@example.com}
      #mail__options__auth__pass: ${SMTP_PASS:-SMTPPassword}
      #mail__from: ${SMTP_MAIL_FROM:-Ghost}
    networks:
      - web
    labels:
      - traefik.enable=true
      - traefik.http.routers.ghost.rule=Host(`blog.nanaoware.online`)
      - traefik.http.services.ghost.loadbalancer.server.port=2368
      - traefik.http.routers.ghost.tls=true
      - traefik.http.routers.ghost.tls.certResolver=lets-encrypt
      - traefik.docker.network=web
    #   Part for local lan services only
      - traefik.http.routers.ghost.middlewares=simpleAuth@file

  database:
    image: mysql:8
    container_name: ghost_db
    restart: always
    healthcheck:
      test: ["CMD", 'mysqladmin', 'ping', '-h', 'localhost', '-u', 'root', '-p$$DB_ROOT_PASS' ]
      timeout: 20s
      retries: 10
    volumes:
      - ${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/ghost/mysql:/var/lib/mysql
    expose:
      - 3306
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASS:-DatabaseRootPassword54321}
      MYSQL_DATABASE: ${DB_NAME:-ghost}
      MYSQL_USER: ${DB_USER:-ghost}
      MYSQL_PASSWORD: ${DB_USER_PASS:-DatabasePassword1234}
    networks:
      - web
    
networks:
  web:
    external: true
```

 Key Components of the Configuration¶

Service: Ghost¶

Image: ghcr.io/benphelps/homepage:latest is the Docker image used for Homepage.
Ports:
3000:3000 maps port 3000 on the host to port 3000 in the container, where the Homepage web interface is accessible.
Volumes:
./config:/app/config: Maps a local configuration directory to the container's configuration directory.
/var/run/docker.sock:/var/run/docker.sock:ro: (Optional) Allows Homepage to integrate with Docker, provided read-only.
Deploying Homepage¶

Ensure that the local ./config directory exists and contains your configuration files.
Save the Docker Compose configuration in a docker-compose.yml file.
Run docker-compose up -d to start Homepage in detached mode.
Access Homepage by navigating to http://<host-ip>:3000.
Configuring and Using Homepage¶

After deployment, you can customize Homepage through the configuration files in your local ./config directory. This allows you to personalize the start page with your preferred links and services.

