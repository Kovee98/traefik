# Edge Router & GitOps Proxy

Dynamic routing and reverse proxy configuration powered by **Traefik v3**, managed via **GitOps** on a Proxmox Debian LXC.

## Automated Sync Workflow (GitOps)
```text
               ┌───────────────────────────────┐
               │    GitHub Repository Push     │
               └───────────────┬───────────────┘
                               │ HTTP POST (HMAC-SHA256)
                               ▼
  ┌─────────────────────────────────────────────────────────┐
  │                   Cloudflare Edge                       │
  └────────────────────────────┬────────────────────────────┘
                               │ Encrypted Cloudflare Tunnel
                               ▼
┌─────────────────────────────────────────────────────────────┐
│              Proxmox Debian LXC (Traefik Host)              │
│                                                             │
│   ┌───────────────────────────┐                             │
│   │ adnanh/webhook (Port 9000)│ ──executes──► sync-script   │
│   └───────────────────────────┘                    │        │
│                                                git pull     │
│                                                    ▼        │
│   ┌───────────────────────────┐             /etc/traefik/   │
│   │  Traefik Dynamic Watcher  │ ◄──watches──   dynamic/     │
│   └───────────────────────────┘                             │
└─────────────────────────────────────────────────────────────┘
```

## Network
```text
                               ┌────────────────────────┐
                               |     Public / LAN       |
                               |  *.kovalchik.cloud     |
                               └───────────┬────────────┘
                                           |
                                           ▼
                             ┌─────────────────────────────┐
                             |        Traefik Proxy        |
                             |   (:80 / :443 + TLS Term)   |
                             └──────────────┬──────────────┘
                                            |
            ┌───────────────────────────────┼──────────────────────────────┐
            |                               |                              |
            ▼                               ▼                              ▼
┌───────────────────────┐       ┌───────────────────────┐       ┌───────────────────────┐
|   Standard Web Apps   |       |   HTTPS / Self-Signed |       |     gRPC / HTTP/2     |
|  (HTTP / WebSockets)  |       |    (TLS Passthrough)  |       |       (h2c / gRPC)    |
┌───────────────────────┐       ┌───────────────────────┐       ┌───────────────────────┐
| • Media Services      |       | • Proxmox (Freya)     |       | • Omni API            |
| • Immich / Photos     |       | • Portainer           |       | • Custom Services     |
| • Arr Stack           |       | • UniFi Controller    |       |                       |
| • Internal Tools      |       |                       |       |                       |
└───────────────────────┘       └───────────────────────┘       └───────────────────────┘
```
