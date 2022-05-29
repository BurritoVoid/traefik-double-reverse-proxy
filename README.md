Double Traefik Reverse Proxy Example
===

# Purpose
To test a configuration running 2 Traefik instances which will provide SSL certs for all services, internal and external, while disconnecting all routing between the outside world and the internal services. 

You can get the same effect with a single Traefik instance using auth middlewares. This approach would, in theory, prevent any possible attacks via those middlewares.


# Prereq
- Docker/Docker-compose
- A domain that you own. (This uses example.com)
- A public DNS record for the service(s) you want to expose publicly. (This uses jellyfin.example.com)
- A private DNS record (On your router, PiHole, etc) for your domain (This uses *.example.com)
- Ports 80 and 443 forwarded from your router to your docker host on ports 81 and 444, respectively.

# How it works
2 Traefik containers, one for internal and one for external. 

The internal traefik container will run on the standard 80/443 ports on your docker host. A private dns record on your network will point *.example.com requests directly to this container. The container will be configured to get a LetsEncrypt certificiate for the domain via DNS-01 Challange (thus doesn't need to be accessable from the internet). All of your docker services will be routed through this container using docker label configs.

The external container will be for handling all traffic that comes from the internet. Ports 80 and 443 should be forwarded to this container by your router. This instance of traefik will have routers configured via a `config.yml` file only for services which are to be externally accessible. The services are defined by their internal URLs, (via the internal proxy) which should match their external ones.

Both instances will generate a LetsEncrypt cert for `example.com`, which will be individually managed.

If this is set up correctly, all services will be available internally with a valid SSL cert. All external services should be available at the same url, also with a valid SSL cert. 

If you have a public DNS entry for `*.example.com` (or someone had your IP and set up their own DNS server to point to it), internal service requests that come from the internet will be served with a 404 from the external Traefik instance. (As opposed a 401/405, which indicates explicit denial)

# Why tho?
I just started playing with Traefik to consider replacing my NGINX reverse proxy setup. I didn't know much about it (or networking, really) when I started. I don't know how feasible an attack via the Traefik middlewares would be, but I wanted to see if there was a way to mitigate.