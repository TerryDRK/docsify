# Domain, HTTPS and reverse proxy
After setting up the server and its environment, we are going to work on networking. The purpose is to allow access to our server with a **domain**, instead of IP address.

For example, I want to access my server by `https://example.com` or `https://www.example.com`, and maybe access my services by `https://service-name.example.com` (or `https://example.com/service-name`, we are going to use the first one though).

## Domain
First things first, we need a domain, obviously. There are many places where you can buy a domain, like [Cloudflare](https://www.cloudflare.com), or [GoDaddy](https://www.godaddy.com). We are using Cloudflare today.

### Purchase a domain
The purchase process is rather simple: you choose a domain you like, buy it, and it's yours. I purchased this domain for $9.33/yr, which is quite a budget. Some domains may cost you more. It's totally up to you as to which domain to choose.

### Manage DNS configuration
Here comes the tricky part. Once we have the domain (we are going to use `example.com` for the rest of this note), how to bind the domain with our server?

In order to do so, we can go to DNS configurations on Cloudflare, and add the following records:

| Type  | Name | Content       |
|-------|------|---------------|
| A     | @    | `1.2.3.4`     |
| CNAME | www  | `example.com` |

A few explanations to these two records:
- The "A" record means "`example.com` points to the IP address `1.2.3.4`",
- The "CNAME" record means "`www.example.com` is an alias of `example.com`".

With these two records, the domain is now pointed to our server.

Also, it is worth mentioning that Cloudflare proxy is enabled for my server. This will actually point the domain to a proxy server, instead of our server. This protects the IP address of our server from being exposed. This is fully optional, so you can disable proxy if you like.

## HTTPS and reverse proxy
The next step is to configure our server so that it can support HTTPS connections. We are using [Nginx](https://nginx.org/en/) with [Certbot](https://certbot.eff.org) to generate certificates and do reverse proxy.

### Nginx and Certbot
We start Nginx and Certbot with Docker that we just installed in the [previous chapter](/build_a_personal_website_server_and_env_setup). The `docker-compose.yml` file for Nginx with Certbot is:

```yaml
# docker-compose.yml

version: '3'

services:
  nginx:
    image: nginx:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf/:/etc/nginx/conf.d/:ro
      - ./certbot/www:/var/www/certbot/:ro
      - ./certbot/conf/:/etc/nginx/ssl/:ro
  certbot:
    image: certbot/certbot:latest
    volumes:
      - ./certbot/www/:/var/www/certbot/:rw
      - ./certbot/conf/:/etc/letsencrypt/:rw
    network_mode: "host"
```

Use command `sudo docker-compose up -d` to start Nginx and Certbot. Run `sudo docker-compose ps` and you should see container `nginx` up running.

*Reference: [HTTPS using Nginx and Let's encrypt in Docker](https://mindsers.blog/post/https-using-nginx-certbot-docker/)*

### Nginx config and HTTPS setup
Once Nginx and Certbot is ready to use, let's try to enable HTTPS for `example.com`.

In order to do so, add the following configuration file in `./nginx/conf/`:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name [YOUR_DOMAIN_NAME] www.[YOUR_DOMAIN_NAME];
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://[YOUR_DOMAIN_NAME]$request_uri;
    }
}
```

**TO BE CONTINUED...**