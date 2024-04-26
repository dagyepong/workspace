## Setting Up Ghost Blog with Docker Compose¶

### Introduction to Ghost¶

Ghost is a powerful app for professional publishers to create, share, and grow a business around their content. It comes with modern tools to build a website, publish content, send newsletters & offer paid subscriptions to members.

#### Docker Compose Configuration for Ghost Blog¶

This Docker Compose setup deploys Ghost application in a Docker container, allowing you to have a personalized blog to share ideas with like minds.

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

##  Key Components of the Configuration¶

Service: Ghost¶

Image: ghost:5-alpine is the Docker image used for Ghost.
Ports:
2368:2368 maps port 2368 on the host to port 2368 in the container, where Ghost Blog web interface is accessible. You access the admin page with http://ip:2368/ghost to set it up.
Volumes:
${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/ghost/content:/var/lib/ghost/content Maps a local configuration directory to the container's configuration directory.
This alos allows Ghost to integrate with Docker, provided read-only.

### Deploying Ghost¶

Create a directory named ghost on your system to store your configuration files.
Save the Docker Compose configuration in a docker-compose.yml file.
Run docker-compose up -d to start Ghost in detached mode.
Access Homepage by navigating to http://<host-ip>:2368. This compose usesa traefik as the reverse proxy and, if you plan to expose this container to the web, adjust the url to your doamin name. 

### Configuring and Using Homepage¶

After deployment, you can customize your blog through the admin page by visiting http://ip:2368/admin or if you expose ghost to the web, https://ghost.yourdoamin/ghost.

That's it!!

