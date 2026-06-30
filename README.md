# Local Enterprise Ingress Infrastructure (Cloudflare Tunnels)

This repository tracks the configuration, deployment, and routing logic for a production-grade local ingress layer on Fedora Linux. The architecture securely exposes localized data engines, IDE workspaces, and MLOps platforms to the internet using Cloudflare Zero Trust Edge Tunnels (`cloudflared`), eliminating the need for risky inbound firewall rules or public port forwarding.

---

## 🏗️ Architecture Overview

The ingress layer establishes a persistent outbound proxy connection to Cloudflare’s global edge network:

```
  ┌────────────────────────────────────────────────────────┐
  │                  Your Fedora Laptop                    │
  │                                                        │
  │  ┌──────────────┐   ┌─────────────────┐   ┌─────────┐  │
  │  │ Coder (3000) │   │  MinIO (9001)   │   │ MLflow  │  │
  │  └──────▲───────┘   └────────▲────────┘   │ (5000)  │  │
  │         │                    │            └────▲────┘  │
  │         └──────────┐         │                 │       │
  │                    │         │                 │       │
  │              ┌─────┴─────────┴─────────────────┴──┐    │
  │              │    cloudflared systemd service    │    │
  │              └─────────────────▲──────────────────┘    │
  │                                │ (Outbound TLS)        │
  └────────────────────────────────┼───────────────────────┘
                                   │
                     ┌─────────────┴─────────────┐
                     │   Cloudflare Edge Network │
                     │  (*.terrafoxai.com Zone)  │
                     └───────────────────────────┘
```

* **Zero-Inbound Security:** Blocks all scanner and malicious brute-force attempts by keeping your home network ports hidden.
* **Edge Routing Engine:** Inspects request subdomains at Cloudflare's perimeter and proxies them safely down to your local Fedora ports over encrypted TLS streams.
* **Protocol Flexing:** Seamlessly handles web-native application wrappers (HTTP/HTTPS) alongside raw developer data pipeline layers (PostgreSQL TCP Streams).

---

## ⚙️ Service Ingress Blueprint

Traffic entering the `terrafoxai.com` zone is parsed and securely targeted according to this blueprint:

| Subdomain Address | Target Service Interface | Service Port | Traffic Protocol |
| :--- | :--- | :--- | :--- |
| `coder.terrafoxai.com` | Coder IDE Cloud Instance | `3000` | HTTP |
| `minio.terrafoxai.com` | MinIO Storage API Endpoint | `9000` | HTTP / S3 API |
| `console-minio.terrafoxai.com` | MinIO Web Storage Browser | `9001` | HTTP |
| `mlflow.terrafoxai.com` | MLflow Central MLOps Dashboard | `5000` | HTTP |
| `sql.terrafoxai.com` | PostgreSQL Database Instance | `5432` | TCP Stream |

---

## 🚀 Step-by-Step Edge Setup (Fedora Linux)

### 1. System Directories & Security Boundary
The daemon architecture runs as an isolated system runtime tool with configuration states centralized under `/etc/cloudflared`.

#### Operational Paths:
* **Binary Link:** `/usr/bin/cloudflared`
* **Configuration Blueprint:** `/etc/cloudflared/config.yml`
* **Identity Token:** `/etc/cloudflared/cert.pem`
* **Systemd Controller:** `/etc/systemd/system/cloudflared.service`

---

### 2. Binary Installation & Authentication
Install the official Cloudflare stable engine binary natively via the command line:

```bash
# Download the native cloudflared RPM release package
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-x86_64.rpm

# Install via DNF package manager
sudo dnf install -y ./cloudflared-linux-x86_64.rpm

# Authorize your local terminal to manage your cloud infrastructure profile
cloudflared tunnel login
```
*(Follow the browser terminal link prompt to grant control permissions for the `terrafoxai.com` zone).*

---

### 3. Creating the Persistent Edge Tunnel
Provision a dedicated secure connection path for your server cluster:

```bash
# Generate the named enterprise tunnel link
cloudflared tunnel create terrafox-edge-tunnel
```
*Note the returned **Tunnel UUID** string and the matching `.json` credential key file generated inside your path.*

---

### 4. Configuration Blueprint Management
Build your centralized multi-service application router map:

```bash
sudo nano /etc/cloudflared/config.yml
```

#### Contents:
```yaml
tunnel: <YOUR-TUNNEL-UUID-HERE>
credentials-file: /etc/cloudflared/<YOUR-TUNNEL-UUID-HERE>.json

ingress:
  - hostname: coder.terrafoxai.com
    service: http://localhost:3000

  - hostname: minio.terrafoxai.com
    service: http://localhost:9000

  - hostname: console-minio.terrafoxai.com
    service: http://localhost:9001

  - hostname: mlflow.terrafoxai.com
    service: http://localhost:5000

  - hostname: sql.terrafoxai.com
    service: tcp://localhost:5432

  # Global fallback catch-all configuration
  - service: http_status:404
```

---

## 5. Systemd Service Registration & Control
Lock `cloudflared` into your Fedora system backend daemon manager to guarantee persistent connection streams across system reboots:

```bash
# Generate the formal background daemon configuration setup profiles
sudo cloudflared service install

# Refresh configuration dependencies
sudo systemctl daemon-reload

# Configure the runtime daemon to activate on host system boot
sudo systemctl enable cloudflared

# Control running processes
sudo systemctl start cloudflared
sudo systemctl restart cloudflared

# Check real-time process integrity status
sudo systemctl status cloudflared
```

---

## 6. Cloudflare Routing Table Mapping
Map your local subdomains to the edge proxy by adding CNAME records to your global zone DNS table:

```bash
# Sync web UI routes to edge networks
cloudflared tunnel route dns terrafox-edge-tunnel coder.terrafoxai.com
cloudflared tunnel route dns terrafox-edge-tunnel minio.terrafoxai.com
cloudflared tunnel route dns terrafox-edge-tunnel console-minio.terrafoxai.com
cloudflared tunnel route dns terrafox-edge-tunnel mlflow.terrafoxai.com
cloudflared tunnel route dns terrafox-edge-tunnel sql.terrafoxai.com
```

---

## 🔒 Verification & External Connectivity

### Verifying Web Interfaces
Open an outside web browser and clear your authorization checkpoints to log right into your apps:
* `https://coder.terrafoxai.com`
* `https://console-minio.terrafoxai.com`
* `https://mlflow.terrafoxai.com`

### Verifying Database TCP Streams (`sql.terrafoxai.com`)
Because raw SQL database traffic cannot be processed directly by web browsers, access the data channel using either option below:

#### Method A: Cloudflare Access Routing Companion (CLI)
From any remote computer or external client workspace terminal, map the domain back to a vacant local port:
```bash
cloudflared access tcp --hostname sql.terrafoxai.com --url localhost:5432
```
*Leave this running and point your IDE database navigator directly to `localhost:5432`.*

#### Method B: Built-in Desktop App SSH Tunneling
Open your database client tools (like **DBeaver** or **pgAdmin 4**) and bypass the edge proxy entirely by using the built-in **SSH Tunnel** tab. Route your connection directly to your Fedora system profile (`sethugopalan`) over local IP address configurations.
