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

**TO BE CONTINUED...**