# Caddy-reverse-proxy

## What is Caddy?
Caddy is an open source web server written in Go that supports HTTPS automatically. I used this since it is more convenient than handling all of the SSL certificates manually.

Caddy uses its own file format called Caddyfile for configuration which I have written for my site. It uses a * marker to signify to Caddy to obtain a wildcard certificate for the subdomains and the root domain can still use its own certificate. Everything is reverse proxied to several Nginx instances running in LXC containers in Proxmox. A single Nginx instance couldve been used but I prefer to keep different parts of my site separate in the case that should something happen to one part of my site, the rest wouldn't be affected. Grafana is used to monitor the proxmox server and can be accessed at the status subdomain. This config works with Cloudflare SSL set to Full(Strict) aswell in the event that cloudflare is no longer used or is used in DNS only, then valid SSL certificates from LetsEncrypt are stil used removing any errors that might occur. Since this method uses ACME-DNS to verify, access is required to your DNS records by Caddy to allow for proper verification.

## Creating a Cloudflare API token

To get a Cloudflare API token, you must have a cloudflare account with the domain to be configured. You will need to head to your settings which can be found in the top right. Then go to API token on the left sidebar. From there you 'create token' with 2 permisions:

- ZONE DNS READ
- ZONE DNS EDIT

If you are lucky to have a static public IP where your Caddy instance is located then you can add it to the Client IP Address Filtering:

- IS IN {STATIC IP}

otherwise you can leave it blank.

Once you have created the token, you will have been given a string... **DON'T LOSE THE STRING!** otherwise you'll need to create a new token.

## Creating your own Caddyfile

The below format is an example configuration file that you can adapt to your own site:

- Use your email to use with LetsEncrypt in place of {EMAIL}
- Use your generated Cloudflare API token in place of {API TOKEN}
- Use your destination ip (where you want the subdomain to take you) in place of {DESTINATION IP}
- Add as many matchers as you need, make sure they all have unique names
- The handle Abort line at the bottom of the wildcard domain is to close any unknown subdomains

```
{
  {EMAIL}
}
*.domain.com {
  tls {
    dns cloudflare {API TOKEN}
  }
  
  @matcher host SUBDOMAIN.domain.com
  handle @matcher {
    reverse_proxy {DESTINATION IP}
  }
  
  handle {
    abort
  }
}

domain.com {
  reverse_proxy {DESTINATION IP}
}
```
