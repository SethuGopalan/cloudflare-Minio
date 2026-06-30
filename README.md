# Local Enterprise Ingress Infrastructure (Cloudflare Tunnel)

This repository documents a private ingress architecture for a self-hosted AI and data platform running on Fedora Linux. The goal is to provide secure access to development and AI services without exposing inbound ports to the public Internet.

This setup applies security principles commonly used in enterprise environments, including encrypted communication, least-privilege access, secure routing, and private connectivity.

---

## Architecture Overview

```text
                Internet
                    │
                    ▼
             Cloudflare DNS
                    │
                    ▼
         Cloudflare Tunnel (TLS)
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

## Design Principles

- No inbound firewall ports
- TLS encrypted communication
- Secure routing through Cloudflare Tunnel
- Private access to infrastructure services
- Separation between Internet-facing and internal services
- Least-privilege service accounts

---

## Example Services

| Service | Example Hostname | Local Port |
|---------|------------------|-----------:|
| Coder | coder.example.com | 3000 |
| MinIO API | minio.example.com | 9000 |
| MinIO Console | console-minio.example.com | 9001 |
| MLflow | mlflow.example.com | 5000 |
| PostgreSQL | sql.example.com | 5432 |

Replace `example.com` with your own domain.

---

## Install Cloudflared

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm

sudo dnf install -y ./cloudflared-linux-x86_64.rpm

cloudflared tunnel login
```

---

## Create a Tunnel

```bash
cloudflared tunnel create enterprise-edge
```

Record the generated Tunnel UUID.

---

## Example Configuration

Create or edit:

```bash
sudo nano /etc/cloudflared/config.yml
```

Example:

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

```bash
sudo cloudflared service install

sudo systemctl daemon-reload

sudo systemctl enable cloudflared

sudo systemctl start cloudflared

sudo systemctl status cloudflared
```

---

## Create DNS Routes

```bash
cloudflared tunnel route dns enterprise-edge coder.example.com
cloudflared tunnel route dns enterprise-edge minio.example.com
cloudflared tunnel route dns enterprise-edge console-minio.example.com
cloudflared tunnel route dns enterprise-edge mlflow.example.com
cloudflared tunnel route dns enterprise-edge sql.example.com
```

---

## Security Practices

This implementation demonstrates several security practices commonly used in enterprise environments:

- Cloudflare Tunnel instead of public port forwarding
- HTTPS/TLS encrypted communication
- Private service routing
- Dedicated service accounts
- Least-privilege Linux permissions
- Infrastructure services isolated behind a single ingress layer

This repository is intended as a learning and reference implementation rather than a production deployment guide.

---

## Verification

Open the following URLs after deployment:

- https://coder.example.com
- https://console-minio.example.com
- https://mlflow.example.com

For PostgreSQL, use Cloudflare Access TCP routing or an SSH tunnel depending on the environment.

---

## Related Projects

- Private MinIO Data Lake
- Python Data Lake SDK
- MLflow Tracking Server
- Enterprise AI Lab
