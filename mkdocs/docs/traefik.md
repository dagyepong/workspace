## Traefik Setup
Docker can be an efficient way to run web applications in production, but you may want to run multiple applications on the same Docker host. In this situation, you’ll need to set up a reverse proxy. This is because you only want to expose ports 80 and 443 to the rest of the world.

Traefik is a Docker-aware reverse proxy that includes a monitoring dashboard. Below is a docker compose file

```yaml
version: "3.3"
services:
  traefik:
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - $PWD/traefik.toml:/traefik.toml
      - $PWD/traefik_dynamic.toml:/traefik_dynamic.toml
      - $PWD/acme.json:/acme.json
    ports:
      - 80:80
      - 443:443
    networks:
      - web
    container_name: traefik
    image: traefik:v2.2
networks:
  web:
    external: true
```


You can also run traefik with docker run command as:

```yaml
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $PWD/traefik.toml:/traefik.toml \
  -v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml \
  -v $PWD/acme.json:/acme.json \
  -p 80:80 \
  -p 443:443 \
  --network web \
  --name traefik \
  traefik:v2.2
  ```

## Running Traefik Container
You'll have to create two configuration files as stated in the compose file as traefik.toml and traefik_dynamic.toml.
Adjust both files with the following details as follows:
* traefik.toml
```yaml
[entryPoints]
  [entryPoints.web]
    address = ":80"
    [entryPoints.web.http.redirections.entryPoint]
      to = "websecure"
      scheme = "https"

  [entryPoints.websecure]
    address = ":443"

[api]
  dashboard = true

[certificatesResolvers.lets-encrypt.acme]
  email = "owarenana06@gmail.com"
  storage = "acme.json"
  [certificatesResolvers.lets-encrypt.acme.tlsChallenge]

[providers.docker]
  watch = true
  network = "web"

[providers.file]
  filename = "traefik_dynamic.toml"
```

* traefik_dynamic.toml
```yaml
[http.middlewares.simpleAuth.basicAuth]
  users = [
    "nana:$apr1$10jy6hpm$gx2gI5Epl7/PaPSUnYBPq/"
  ]

[http.routers.api]
  rule = "Host(`monitor.nanaoware.online`)"
  entrypoints = ["websecure"]
  middlewares = ["simpleAuth"]
  service = "api@internal"
  [http.routers.api.tls]
    certResolver = "lets-encrypt"
```
# Traefik.toml
First, you want to specify the ports that Traefik should listen on using the entryPoints section of your config file. You want two because you want to listen on port 80 and 443. Let’s call these web (port 80) and websecure (port 443).

Note that you are also automatically redirecting traffic to be handled over TLS.

Next, configure the Traefik api, which gives you access to both the API and your dashboard interface. The heading of [api] is all that you need because the dashboard is then enabled by default, but you’ll be explicit for the time being.
To finish securing your web requests you want to use Let’s Encrypt to generate valid TLS certificates. Traefik v2 supports Let’s Encrypt out of the box and you can configure it by creating a certificates resolver of the type acme.

The acme.tlsChallenge section allows us to specify how Let’s Encrypt can verify that the certificate. You’re configuring it to serve a file as part of the challenge over port 443.

# Traefik_dynamic.toml
Our final configuration uses the file provider. With Traefik v2, static and dynamic configurations can’t be mixed and matched. To get around this, you will use traefik.toml to define your static configurations and then keep your dynamic configurations in another file, which you will call traefik_dynamic.toml. Here you are using the file provider to tell Traefik that it should read in dynamic configurations from a different file.


The dynamic configuration values that you need to keep in their own file are the middlewares and the routers. To put your dashboard behind a password you need to customize the API’s router and configure a middleware to handle HTTP basic authentication. Let’s start by setting up the middleware. You can use this online [tool](https://wtools.io/generate-htpasswd-online) to create an encrypted password

The middleware is configured on a per-protocol basis and since you’re working with HTTP you’ll specify it as a section chained off of http.middlewares. Next comes the name of your middleware so that you can reference it later, followed by the type of middleware that it is, which will be basicAuth in this case. Let’s call your middleware simpleAuth.

To configure the router for the api you’ll once again be chaining off of the protocol name, but instead of using http.middlewares, you’ll use http.routers followed by the name of the router. In this case, the api provides its own named router that you can configure by using the [http.routers.api] section. You’ll configure the domain that you plan on using with your dashboard also by setting the rule key using a host match, the entrypoint to use websecure, and the middlewares to include simpleAuth.

## Running traefik
* First, create a network as ```docker network create web``
* Next, create an empty file that will hold your Let’s Encrypt information. You’ll share this into the container so Traefik can use it: ```touch acme.json``
* Traefik will only be able to use this file if the root user inside of the container has unique read and write access to it. To do this, lock down the permissions on acme.json so that only the owner of the file has read and write permission. ```chmod 600 acme.json``

Finally, create the Traefik container with the compose file or docker command as stated above as:
```yaml
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $PWD/traefik.toml:/traefik.toml \
  -v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml \
  -v $PWD/acme.json:/acme.json \
  -p 80:80 \
  -p 443:443 \
  --network web \
  --name traefik \
  traefik:v2.2
```

## Traefik Dashboard
With the container started, you now have a dashboard you can access to see the health of your containers. You can also use this dashboard to visualize the routers, services, and middlewares that Traefik has registered. You can try to access the monitoring dashboard by pointing your browser to https://monitor.your_domain/dashboard/ (the trailing / is required).

You will be prompted for your username and password, which are admin and the password you configured above. Incase you run this locally with a domain, te webui is http://ip:8080/dashboard/
