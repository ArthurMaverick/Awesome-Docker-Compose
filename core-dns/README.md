
# CoreDNS with Docker Compose

CoreDNS is a highly flexible, extensible, and modular DNS server that can be configured for various DNS-based tasks, from simple domain forwarding to advanced service discovery. Originally designed for cloud-native environments, CoreDNS is often used in Kubernetes clusters to manage DNS requests between services. However, it can also be employed as a standalone DNS server, ideal for managing DNS within local networks or development environments.

This setup demonstrates how to configure CoreDNS with Docker Compose to:
- Resolve custom local domains that aren’t registered in public DNS.
- Forward requests for external domains (e.g., `google.com`) to public DNS servers for resolution.
- Cache DNS responses to improve speed and reduce load on the upstream servers.

CoreDNS’s plugin-based architecture allows you to extend its capabilities. In this configuration, we use plugins like `forward`, `log`, and `cache` to control how CoreDNS handles DNS requests, logs activity, and caches responses. This setup is suitable for development environments or small networks needing customizable DNS configurations.

## Table of Contents
- [CoreDNS with Docker Compose](#coredns-with-docker-compose)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Configuration](#configuration)
    - [Step 1: `Corefile` Configuration](#step-1-corefile-configuration)
    - [Step 2: `docker-compose.yml` Configuration](#step-2-docker-composeyml-configuration)
  - [Usage](#usage)
  - [Testing](#testing)
    - [Example Commands](#example-commands)
  - [Logs](#logs)
  - [Customizing CoreDNS](#customizing-coredns)
  - [Troubleshooting](#troubleshooting)

## Overview
This guide walks you through setting up CoreDNS as a DNS resolver using Docker Compose. The setup includes:
- Custom domains for local resolution.
- Forwarding to public DNS servers for external domains.

## Prerequisites
- Docker and Docker Compose installed.
- Basic knowledge of DNS and networking.

## Installation
1. Clone this repository or create a directory for CoreDNS.
2. Within the directory, create the following files:
   - `docker-compose.yml`: Defines the Docker service for CoreDNS.
   - `Corefile`: Configuration file for CoreDNS.

## Configuration

### Step 1: `Corefile` Configuration
The `Corefile` is the main configuration file for CoreDNS. Below is a sample `Corefile` to enable custom domain resolution and forward external requests:

\`\`\`plaintext
.:53 {
    forward . 8.8.8.8 8.8.4.4  # Forward unknown requests to Google's DNS
    log                         # Log all DNS requests
    errors                      # Log errors
    cache 30                    # Cache responses for 30 seconds
}

example.local:53 {
    hosts {
        127.0.0.1 example.local               # Map example.local to localhost
        192.168.1.10 service1.example.local   # Map custom domain to IP
        192.168.1.11 service2.example.local   # Map another custom domain to IP
        fallthrough
    }
    log
}
\`\`\`

- **`forward`**: Forwards DNS requests to external DNS servers (Google DNS in this case) when CoreDNS doesn’t have a local record.
- **`log`**: Logs all DNS requests.
- **`cache`**: Caches responses for faster performance.
- **`hosts`**: Defines custom DNS entries for local domains.
- **`fallthrough`**: Allows non-matching queries to be forwarded to the global DNS settings.

### Step 2: `docker-compose.yml` Configuration

Create the `docker-compose.yml` file to define the CoreDNS service:

\`\`\`yaml
version: '3.8'

services:
  coredns:
    image: coredns/coredns:latest
    container_name: coredns
    volumes:
      - ./Corefile:/Corefile       # Mount the Corefile
    ports:
      - "53:53/udp"                # Expose UDP 53 for DNS
      - "53:53/tcp"                # Expose TCP 53 for DNS (optional)
    networks:
      - dns-net                    # Network for CoreDNS

networks:
  dns-net:
    driver: bridge
\`\`\`

- **`volumes`**: Mounts the `Corefile` for DNS configuration.
- **`ports`**: Exposes DNS ports (UDP and TCP) on 53.
- **`networks`**: Defines a bridge network for CoreDNS to interact with other services.

## Usage

1. Start the CoreDNS service:
   \`\`\`bash
   docker-compose up -d
   \`\`\`
2. CoreDNS will now resolve domains according to the `Corefile` configuration. Custom domains will resolve as specified, while other requests will be forwarded to the public DNS.

## Testing

You can test the DNS resolution using tools like `nslookup` or `dig`.

### Example Commands

1. **Test Custom Domain**:
   \`\`\`bash
   nslookup service1.example.local 127.0.0.1
   \`\`\`
   Expected output: IP `192.168.1.10`

2. **Test Forwarding to Public DNS**:
   \`\`\`bash
   nslookup google.com 127.0.0.1
   \`\`\`
   Expected output: IP address of `google.com` from Google’s DNS.

## Logs

To view DNS queries and responses, check the logs with:
\`\`\`bash
docker-compose logs -f coredns
\`\`\`

The logs will display details about each DNS request, including the client IP, query type, and response status.

## Customizing CoreDNS

You can customize CoreDNS further by:
- Adding more zones and domains in the `Corefile`.
- Configuring additional plugins, such as **prometheus** for monitoring or **file** for advanced zone configurations.

For more options, refer to the [CoreDNS documentation](https://coredns.io/manual/).

## Troubleshooting

- **Service Not Starting**: Ensure that Docker Compose and Docker are installed and that no other service is using port 53.
- **Domain Not Resolving**: Check the `Corefile` syntax and confirm that the CoreDNS container has been restarted after changes.
- **Logs Not Showing**: Verify that `log` is enabled in the `Corefile`.

---

This setup provides a lightweight DNS server with custom domain resolution, ideal for development and internal networks.
