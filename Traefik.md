

# Notes for Self-Hosting Traefik

## 1. Introduction to Traefik

Traefik is an open-source reverse proxy and load balancer designed for microservices and containerized applications. It offers several key advantages:

- Automatic service discovery and configuration
- Dynamic routing without restarts
- Built-in support for SSL/TLS encryption
- Integration with Docker and other container orchestration platforms
- Load balancing capabilities
- Middleware support for request/response modifications

## 2. VPS Requirements

To self-host Traefik on a VPS, ensure your system meets these minimum requirements:

- At least 2 vCPUs
- 2 GB of RAM
- 30 GB of storage
- Docker and Docker Compose installed
- A registered domain name
- Open ports 80 (HTTP) and 443 (HTTPS)

## 3. Best Practices for Self-Hosting

When setting up Traefik on your VPS, consider the following best practices:

1. Use Docker networks for container communication
2. Implement automated SSL certificate management with Let's Encrypt
3. Store configuration files in version control (e.g., Git)
4. Set resource limits for Docker containers
5. Regularly update Traefik and Docker images
6. Implement monitoring and logging
7. Use a firewall to restrict access to your VPS

## 4. Traefik Configuration Overview

Traefik configuration consists of two main parts:

1. Static Configuration: Defines Traefik's behavior and is set at startup
2. Dynamic Configuration: Defines the routing rules and can be updated at runtime

## 5. Security Considerations

Ensure the security of your Traefik setup by:

- Using HTTPS for all services
- Implementing rate limiting
- Configuring authentication for sensitive endpoints
- Regularly updating Traefik and all dependencies
- Using Docker secrets for sensitive information

# Codebase for Self-Hosting Traefik on VPS

Based on the analyzed repositories, here's a codebase structure for self-hosting Traefik on a VPS:

```
traefik-proxy/
├── docker-compose.yml
├── traefik.yml
├── config/
│   ├── dynamic_conf.yml
│   └── acme.json
├── .env
└── README.md
```

## docker-compose.yml

This file defines the Traefik service and any additional services you want to proxy:

```yaml
version: '3'

services:
  traefik:
    image: traefik:v2.8
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/traefik.yml:ro
      - ./config/dynamic_conf.yml:/config/dynamic_conf.yml:ro
      - ./config/acme.json:/acme.json
    environment:
      - CF_API_EMAIL=${CF_API_EMAIL}
      - CF_API_KEY=${CF_API_KEY}
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=https"
      - "traefik.http.routers.traefik.rule=Host(`traefik.yourdomain.com`)"
      - "traefik.http.routers.traefik.tls=true"
      - "traefik.http.routers.traefik.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik.service=api@internal"

networks:
  proxy:
    external: true
```

## traefik.yml

This file contains the static configuration for Traefik:

```yaml
api:
  dashboard: true
  debug: true

entryPoints:
  http:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: https
          scheme: https
  https:
    address: ":443"

serversTransport:
  insecureSkipVerify: true

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
  file:
    filename: /config/dynamic_conf.yml
    watch: true

certificatesResolvers:
  cloudflare:
    acme:
      email: your-email@example.com
      storage: acme.json
      dnsChallenge:
        provider: cloudflare
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
```

## config/dynamic_conf.yml

This file contains dynamic configuration for Traefik:

```yaml
http:
  middlewares:
    secureHeaders:
      headers:
        sslRedirect: true
        forceSTSHeader: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 31536000
```

## .env

Store sensitive information in this file:

```
CF_API_EMAIL=your-cloudflare-email@example.com
CF_API_KEY=your-cloudflare-global-api-key
```

## README.md

Create a README file with instructions on how to set up and use your Traefik proxy:

```markdown
# Traefik Proxy Setup

This repository contains the configuration files for setting up Traefik as a reverse proxy on a VPS.

## Prerequisites

- Docker and Docker Compose installed on your VPS
- A domain name pointed to your VPS IP address
- Cloudflare account (for DNS challenge and SSL certificates)

## Setup Instructions

1. Clone this repository to your VPS.
2. Create a `proxy` network in Docker:
   ```
   docker network create proxy
   ```
3. Copy `.env.example` to `.env` and fill in your Cloudflare credentials.
4. Create an empty `acme.json` file and set proper permissions:
   ```
   touch config/acme.json
   chmod 600 config/acme.json
   ```
5. Modify the `traefik.yml` file to include your email address for Let's Encrypt.
6. Start Traefik:
   ```
   docker-compose up -d
   ```

## Adding Services

To add a new service to be proxied by Traefik, add it to the `docker-compose.yml` file and include the necessary Traefik labels.

## Accessing the Dashboard

The Traefik dashboard will be available at `https://traefik.yourdomain.com`. Make sure to secure this endpoint with authentication in a production environment.
```

This codebase provides a solid foundation for self-hosting Traefik on a VPS. It includes automatic HTTPS using Let's Encrypt with Cloudflare DNS challenge, secure headers, and a basic setup for the Traefik dashboard. Remember to replace placeholder values like `yourdomain.com` and Cloudflare credentials with your actual information.

To use this setup, you'll need to:

1. Set up a VPS with Docker and Docker Compose installed
2. Configure your domain's DNS to point to your VPS IP address
3. Clone this repository to your VPS
4. Follow the setup instructions in the README.md file

This configuration provides a secure and scalable foundation for hosting multiple services behind Traefik, with automatic SSL certificate management and modern security practices. As you add more services, you can easily extend this setup by adding new service definitions to the `docker-compose.yml` file and using Traefik labels to configure routing and SSL.