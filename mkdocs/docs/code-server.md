## Setting Up Code-Server with Docker Compose¶

## Introduction to Code-Server¶

The Visual Studio Code Server is a service you can run on a remote development machine, like your desktop PC or a virtual machine (VM). It allows you to securely connect to that remote machine from anywhere through a local VS Code client, without the requirement of SSH.

## Docker Compose Configuration for Code-Server¶

This Docker Compose setup deploys code-server in a Docker container, ensuring a secure and isolated environment as an IDE.

Docker Compose File (docker-compose.yml)¶
```yaml
version: "2.1"

services:

  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
      - PASSWORD=akosua123
      - SUDO_PASSWORD=akosua123 #optional
      #- SUDO_PASSWORD_HASH= #optional
      - PROXY_DOMAIN=vscode.example.com #optional
      - DEFAULT_WORKSPACE=/config/workspace #optional
    volumes:
      - ${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/vscode/config:/config
    ports:
      - 8443:8443
    restart: unless-stopped
    networks:
      - web
    labels:
      - traefik.enable=true
      - traefik.docker.network=web
      - traefik.http.routers.codeserver.rule=Host(`code.nanaoware.online`)
      - traefik.http.routers.codeserver.tls=true
      - traefik.http.routers.codeserver.tls.certResolver=lets-encrypt
      - traefik.http.services.codeserver.loadbalancer.server.port=8443
    #  # Optional part for file upload max sizes
      - traefik.http.middlewares.limit.buffering.maxRequestBodyBytes=50000000
      - traefik.http.middlewares.limit.buffering.maxResponseBodyBytes=50000000
      - traefik.http.middlewares.limit.buffering.memRequestBodyBytes=50000000
      - traefik.http.middlewares.limit.buffering.memResponseBodyBytes=50000000
    #  # Optional part for traefik middlewares
      - traefik.http.routers.codeserver.middlewares=simpleAuth@file

networks:
  web:
    external: true
```


## Key Components of the Configuration¶

### Environment Variables¶

* PUID=1000 and PGID=1000: Sets user and group IDs for file permissions.
TZ=Etc/UTC: Sets the container's timezone.

* Volumes¶
${DOCKER_VOLUME_STORAGE:-/mnt/docker-volumes}/vscode/config:/config Maps the local config directory to the container's config directory.

* Ports¶
8443:8443: Web GUI.

* Restart Policy¶

unless-stopped: Ensures the container restarts automatically unless explicitly stopped. 

You can decide to run this container through traefik or disable it. You'll need to change the ```SUDO_PASSWORD``` in the environment variable to your liking.