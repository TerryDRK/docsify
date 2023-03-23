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
*Reference: [HTTPS using Nginx and Let's encrypt in Docker](https://mindsers.blog/post/https-using-nginx-certbot-docker/)*

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

### Nginx config and HTTPS setup
Once Nginx and Certbot is ready to use, let's try to enable HTTPS for `example.com`.

#### Configure port 80
We set up port 80 (i.e. `http://example.com`) first. It should do 2 things:

- Serve `/.well-known/acme-challenge/` for Certbot verification usage,
- Redirect all other traffic to HTTPS (port 443).

In order to do so, add the following configuration file in `./nginx/conf/`:

```nginx
# example.com.conf
server {
    listen 80;
    listen [::]:80;

    server_name example.com www.example.com;
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://example.com$request_uri;
    }
}
```

Every time you change Nginx configurations, remember to **restart container**, or **reload configuration**.

- Restart container: `sudo docker-compose restart`,
- Reload configuration (**recommended**): `sudo docker-compose exec [CONTAINER_NAME] nginx -s reload`.

The second way is more recommended, as it avoids service interruptions. `[CONTAINER_NAME]` can be obtained by `sudo docker-compose ps`, or you can set it in `docker-compose.yml`.

#### Generate certificates with Certbot
Once we have that configuration loaded, we can start generating certificates for `example.com`.

First, try to simulate the generation to check if everything's working:

```bash
sudo docker-compose run --rm certbot certonly --webroot --webroot-path /var/www/certbot/ --dry-run -d example.com
```

If that goes through without errors, you may remove the `--dry-run` option to generate the real certificates:

```bash
sudo docker-compose run --rm certbot certonly --webroot --webroot-path /var/www/certbot/ -d example.com
```

#### Configure port 443 for HTTPS
Since we now have the certificates for `example.com`, we can set up port 443, which is the HTTPS port. Edit the previous `./nginx/conf/example.com.conf` file":

```nginx
# example.com.conf
server {
    listen 80;
    listen [::]:80;

    server_name example.com www.example.com;
    server_tokens off;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://example.com$request_uri;
    }
}

server {
    listen 443 default_server ssl http2;
    listen [::]:443 ssl http2;

    server_name example.org;

    ssl_certificate /etc/nginx/ssl/live/example.org/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/live/example.org/privkey.pem;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

Reload Nginx, and visit `https://example.com`, you shall see Nginx home page.

*If you are using Cloudflare proxy, you can go to your domain's `SSL/TLS` section, and change the encryption mode to `Full (strict)`.*

### Other service setup (used later)
When configuring other services later, we have 2 ways: either to run it on `https://example.com/new-service`, or on `https://new-service.example.com`. We are going to cover both here.

#### Method 1: `https://example.com/new-service`
Say we have a service running on port 8080.

**TO BE CONTINUED...**