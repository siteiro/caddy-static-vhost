# Multi-site static web server

This Docker image allows you to host multiple static websites, each on a subdomains or custom domain, on a VPS.

It's powered by Caddy Server, Let's Encrypt and DigitalOcean DNS.

## Requirements

* A domain
* DigitalOcean DNS
* DigitalOcean API token with write access
* Marketing website URL

## What it does

Given a parent domain, e.g. `example.net`, this serves multiple static websites
on subdomains, e.g. `foo.example.net`, `bar.example.net`, etc. The parent
domain itself redirects to a marketing URL, e.g. `https://example.com`.

This server automatically obtains a TLS certificate for the parent domain
(including its www subdomain) and a wildcard TLS certificate for its
subdomains. Let's Encrypt requires a successful DNS-01 challenge in order to
issue a wildcard domain. Therefore, you must provide a DigitalOcean API token
with write access.

In addition to subdomain hosting, this server caters for custom domains and
issues TLS certificates for them on demand. To prevent Let's Encrypt rate limit
exhaustion, you must implement an API endpoint that validates new domains. If
the endpoint is `https://api.example.com/dnscheck`, for example, then the
server will issue a `GET` request to
`https://api.example.com/dnscheck?domain={CUSTOM_DOMAIN}`, where
`{CUSTOM_DOMAIN}` is the custom domain to check. The endpoint validates the
domain by responding with a 200 status code.

## Environment variables

| Name                         | Description                              | Required | Example                                                  |
|------------------------------|------------------------------------------|----------|----------------------------------------------------------|
| `HOSTNAME`                   | Parent domain of subdomains              | Yes      | `example.net`                                            |
| `MARKETING_URL`              | Marketing website URL                    | Yes      | `https://example.com`                                    |
| `DNS_CHECK_ENDPOINT`         | Subdomain DNS check endpoint             | Yes      | `https://api.example.com/v1/dns/check`                   |
| `DIGITALOCEAN_DNS_API_TOKEN` | DigitalOcean API token with write access | Yes      |                                                          |
| `ACME_CA`                    | Alternative ACME CA URL                  | No       | `https://acme-staging-v02.api.letsencrypt.org/directory` |

## Let's Encrypt rate limits

Let's Encrypt rate limits are quite tight and you can exhaust them easily. To
avoid exceeding rate limits, do the following:

* Set `ACME_CA` to the Let's Encrypt staging URL during development, i.e.
  `https://acme-staging-v02.api.letsencrypt.org/directory`.
* Persist the Docker container's data between runs, by mapping `/data` to an
  external or bind volume. (See the included `docker-compose.yml` example.)
* Back up the Docker volume mapped to `/data` regularly, as regenerating a
  large number of certificates risks exceeding rate limits.

## Setting up an Ubuntu server with Docker on DigitalOcean

See [setup-ubuntu-22.04.sh](setup-ubuntu-22.04.sh) for commands to run after
provisioning an Ubuntu 22.04 droplet on DigitalOcean.

## Using other DNS providers

You can easily modify the Dockerfile and Caddyfile to use another DNS provider.
See https://github.com/caddy-dns for supported providers.
