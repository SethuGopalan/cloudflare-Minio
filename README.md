# Local Enterprise Ingress Infrastructure with Cloudflare Tunnel

This repository documents a private ingress architecture for a self-hosted AI and data platform running on Fedora Linux.

The goal of this setup is to provide secure access to development, storage, database, and machine learning services without exposing inbound ports directly to the public Internet.

This project demonstrates security principles commonly used in enterprise environments, including encrypted communication, private service routing, least-privilege access, and centralized ingress control.

---

![Cloudflare Dashboard](cloudflare/cloudflare.jpg)

## Architecture Overview

```text
                Internet
                    │
                    ▼
             Cloudflare DNS
                    │
                    ▼
         Cloudflare Tunnel over TLS
                    │
                    ▼
          Fedora Infrastructure Server
        ┌──────────────────────────────┐
        │ Coder                        │
        │ MinIO                        │
        │ MLflow                       │
        │ PostgreSQL                   │
        └──────────────────────────────┘
```

---

## What This Project Shows

This project shows how a local infrastructure server can securely expose selected services through Cloudflare Tunnel.

Instead of opening inbound firewall ports, the Fedora server creates an outbound encrypted tunnel to Cloudflare. Requests are routed through Cloudflare and then forwarded to local services running on the server.

This approach helps keep the server private while still allowing controlled access to development and data services.

---

## Design Principles

- No public inbound firewall ports
- Encrypted communication through TLS
- Private service routing with Cloudflare Tunnel
- Separation between public DNS and internal services
- Dedicated service accounts where possible
- Least-privilege Linux permissions
- Central ingress layer for multiple infrastructure services

---

## Example Services

| Service       | Example Hostname          | Local Port | Purpose                          |
| ------------- | ------------------------- | ---------: | -------------------------------- |
| Coder         | coder.example.com         |       3000 | Cloud development workspace      |
| MinIO API     | minio.example.com         |       9000 | S3-compatible object storage API |
| MinIO Console | console-minio.example.com |       9001 | MinIO web console                |
| MLflow        | mlflow.example.com        |       5000 | ML experiment tracking           |
| PostgreSQL    | sql.example.com           |       5432 | Database access                  |

Replace `example.com` with your own domain.

Do not publish real domains, tunnel IDs, access tokens, credentials, or private IP addresses in a public repository.

---

## Install Cloudflared on Fedora

Download and install Cloudflare Tunnel:

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm

sudo dnf install -y ./cloudflared-linux-x86_64.rpm
```

Login to Cloudflare:

```bash
cloudflared tunnel login
```

This opens a browser window and allows Cloudflare to authorize tunnel creation for your domain.

---

## Create a Tunnel

```bash
cloudflared tunnel create enterprise-edge
```

After the tunnel is created, Cloudflare generates a Tunnel UUID.

Keep the Tunnel UUID and credentials file private.

---

## Example Cloudflared Configuration

Create or edit the Cloudflare Tunnel configuration file:

```bash
sudo nano /etc/cloudflared/config.yml
```

Example configuration:

```yaml
tunnel: <YOUR-TUNNEL-UUID>
credentials-file: /etc/cloudflared/<YOUR-TUNNEL-UUID>.json

ingress:
  - hostname: coder.example.com
    service: http://localhost:3000

  - hostname: minio.example.com
    service: http://localhost:9000

  - hostname: console-minio.example.com
    service: http://localhost:9001

  - hostname: mlflow.example.com
    service: http://localhost:5000

  - hostname: sql.example.com
    service: tcp://localhost:5432

  - service: http_status:404
```

---

## Register Cloudflared as a System Service

Install Cloudflare Tunnel as a Linux service:

```bash
sudo cloudflared service install
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable the service:

```bash
sudo systemctl enable cloudflared
```

Start the service:

```bash
sudo systemctl start cloudflared
```

Check service status:

```bash
sudo systemctl status cloudflared
```

---

## Create DNS Routes

Create DNS records for each hostname:

```bash
cloudflared tunnel route dns enterprise-edge coder.example.com

cloudflared tunnel route dns enterprise-edge minio.example.com

cloudflared tunnel route dns enterprise-edge console-minio.example.com

cloudflared tunnel route dns enterprise-edge mlflow.example.com

cloudflared tunnel route dns enterprise-edge sql.example.com
```

---

## Verification

After deployment, verify the public service URLs:

```text
https://coder.example.com
https://console-minio.example.com
https://mlflow.example.com
```

For PostgreSQL, use Cloudflare Access TCP routing or a secure SSH tunnel depending on your environment.

---

## Security Practices

This implementation demonstrates several security practices commonly used in enterprise environments:

- Cloudflare Tunnel instead of router port forwarding
- HTTPS/TLS encrypted communication
- Private service routing
- Dedicated service accounts
- Least-privilege Linux permissions
- Centralized ingress control
- Reduced exposure of the infrastructure server

This repository is intended as a learning and reference implementation rather than a production deployment guide.

---

## Do Not Commit

Do not commit the following files or values to GitHub:

```text
/etc/cloudflared/cert.pem
/etc/cloudflared/<YOUR-TUNNEL-UUID>.json
Cloudflare API tokens
Tunnel UUIDs
MinIO access keys
PostgreSQL passwords
SSH private keys
Real domain names
Private IP addresses
```

Use placeholders in public documentation.

---

## Related Projects

- Private MinIO Data Lake
- Python Data Lake SDK
- MLflow Tracking Server
- Enterprise AI Lab
