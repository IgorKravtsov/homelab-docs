# HomeLab Architecture

## Overview

This document describes a self-hosted homelab architecture using:
- **GL.iNet router** as gateway with 4G failover
- **VPS as public edge** (has public IP, Home Lab does not)
- **Tailscale VPN** for VPS ↔ HomeLab connectivity
- **Proxmox** for virtualization
- **Dokploy / Traefik on VPS** for public domain routing (with Tailscale option for direct access)

## Key Principles

- **VPS as single public entry point** — HomeLab doesn't expose services directly to the internet
- **GL.iNet as portable gateway** — works with or without wired internet, 4G backup
- **Self-hosted** — complete control over infrastructure
- **Tailscale VPN** — primary VPN for VPS ↔ HomeLab connectivity (replaces WireGuard)
- **Dokploy / Traefik on VPS** — centralized routing for external users (no VPN required)

---

## Current Validated State (Apr 2026)

- **Actual GL.iNet LAN**: `192.168.8.0/24`
- **Actual router IP**: `192.168.8.1`
- **Validated VPN path**: VPS can reach `192.168.8.1` through Tailscale subnet routing
- **Validated public edge**: Dokploy / Traefik on VPS is the chosen reverse proxy layer
- **Validated test app**: `https://test-1.shatori.com` → VPS Dokploy / Traefik → Tailscale → `http://192.168.8.141:3000`

Historical notes about Nginx Proxy Manager below are superseded where they conflict with this section.

---

## Architecture Components

### GL.iNet Router (Gateway)

| Feature | Description |
|---------|-------------|
| **Model** | GL-X2000 (Spitz Plus) or GL-X3000 (Spitz AX) |
| **WAN** | Ethernet ( квартирный роутер) + 4G SIM (failover) |
| **LAN** | 1 port (need switch for multiple devices) |
| **VPN** | Tailscale client (connects to VPS) |
| **4G Failover** | Automatic switch when wired internet fails |

### VPS

| Component | Purpose |
|-----------|---------|
| Public IP | Single entry point from internet |
| Dokploy / Traefik | Reverse proxy, routing by domain, SSL |
| Tailscale client | VPN connecting VPS to HomeLab |
| Let's Encrypt | Automatic TLS certificates |

### HomeLab (Proxmox)

| VM/LXC | Services |
|--------|----------|
| **LXC: Plex** | Media server |
| **LXC: Home Assistant** | Home automation |
| **LXC: Minecraft** | Game server |
| **LXC: Other services** | Nextcloud, etc. |
| **VM: TrueNAS** | Storage (optional, separate hardware)

---

## Network Architecture

```
                            INTERNET
                               │
                               ▼
               test-1.shatori.com (DNS → VPS public IP)
                               │
                               ▼
                          [ VPS ]
                        ├─ Dokploy / Traefik
                        ├─ Let's Encrypt
                        └─ Tailscale client
                               │
                      Tailscale tunnel
                               ▼
                     [ GL.iNet Router ]
                     (WAN: wired or 4G)
                               │
            ┌──────────────────┼──────────────────┐
            │                  │                  │
            ▼                  ▼                  ▼
     [ Proxmox ]        [ Proxmox ]        [ TrueNAS ]
     ├─ Plex            ├─ HA             ├─ Storage
     ├─ Minecraft       └─ Other          └─ Backups
     └─ Other
```

### Network Details

| Segment | IP Range | Description |
|---------|----------|-------------|
| **Tailscale** | 100.x.x.x | VPN network |
| **GL.iNet LAN** | 192.168.8.x | Home Lab internal network |
| **Proxmox / Services** | 192.168.8.100-254 | VM/LXC containers and test services |

### How Access Works (No VPN on Client)

```
User browser: test-1.shatori.com
    │
    ▼
DNS resolves to VPS public IP
    │
    ▼
VPS: Dokploy / Traefik receives request
    │
    ▼
Forwards through Tailscale to GL.iNet
    │
    ▼
GL.iNet → Home Lab (service IP)
```
VPS receives request on port 80/443
    │
    ▼
VPS forwards through Tailscale tunnel
    │
    ▼
GL.iNet receives → forwards to LAN
    │
    ▼
Service at `192.168.8.x:port`
```

### Minecraft (UDP) Flow

```
User: mc.shatori.com:25565
    │
    ▼
VPS (TCP/UDP proxy)
    │
    ▼
Tailscale → GL.iNet → Minecraft VM
```

---

## Traffic Flow

### Web Applications (HTTP/HTTPS)

| Domain | Flow |
|--------|------|
| test-1.shatori.com | VPS Dokploy / Traefik → Tailscale → GL.iNet → `192.168.8.141:3000` |
| ha.shatori.com | VPS Dokploy / Traefik → Tailscale → GL.iNet → Home Assistant |
| plex.shatori.com | VPS Dokploy / Traefik → Tailscale → GL.iNet → Plex |
| cloud.shatori.com | VPS Dokploy / Traefik → Tailscale → GL.iNet → Nextcloud |

### Minecraft (TCP/UDP)

```
mc.shatori.com:25565
    │
    ▼
VPS:25565 (nginx stream proxy)
    │
    ▼
Tailscale tunnel
    │
    ▼
GL.iNet → Minecraft VM (192.168.8.x)
```

### Key Point: Reverse Proxy on VPS

**Why Dokploy / Traefik on VPS?**
- Single public entry point with TLS termination
- No double proxy layer between internet and Home Lab
- Easy to point public domains at arbitrary internal services over Tailscale
- Matches the validated `test-1.shatori.com` setup already working in production

**VPS role:** Accept public traffic, terminate TLS, then forward to Home Lab through Tailscale subnet routing

---

## Responsibilities

### GL.iNet Router

- Gateway between internet and Home Lab
- WAN: connects to квартирный роутер or 4G SIM
- LAN: provides network for Proxmox, TrueNAS, etc.
- Tailscale client: maintains VPN connection to VPS
- 4G Failover: automatic switch when wired internet fails

### VPS

- Internet entry point (has public IP)
- Dokploy / Traefik (domain-based routing)
- Tailscale client (connects to Home Lab)
- Let's Encrypt (SSL certificates)

### Proxmox

- Virtualization host
- Runs all Home Lab services

### TrueNAS (optional, separate hardware)

- Centralized storage
- Media files, backups, documents

---

## Service Placement

### On VPS

| Service | Notes |
|---------|-------|
| **Dokploy / Traefik** | Domain-based routing, SSL termination |
| **Tailscale client** | VPN connecting to Home Lab |
| **Let's Encrypt** | Auto SSL certificates |

### In Proxmox (LXC)

| Service | Type | Notes |
|---------|------|-------|
| **Plex** | LXC | Media streaming |
| **Home Assistant** | LXC or VM | Home automation |
| **Minecraft** | LXC or VM | Game server (needs more resources) |
| **Nextcloud** | LXC | File sync and storage |
| **Other services** | LXC | As needed |

### On TrueNAS (separate hardware)

- Storage pools
- Media files (Plex library)
- Backups
- Snapshots

---

## Domain Mapping

| Domain | Service | Internal IP |
|--------|---------|-------------|
| test-1.shatori.com | Test app | 192.168.8.141:3000 |
| ha.shatori.com | Home Assistant | 192.168.8.101:8123 |
| plex.shatori.com | Plex | 192.168.8.102:32400 |
| mc.shatori.com | Minecraft | 192.168.8.103:25565 |
| cloud.shatori.com | Nextcloud | 192.168.8.104:80/443 |

All routing handled by Dokploy / Traefik on VPS.

---

## Tailscale VPN (取代 WireGuard)

Tailscale creates a **mesh VPN** between VPS, GL.iNet, and your devices. It provides:
- **Tailscale IP network**: 100.x.x.x
- **Encrypted connection**: All traffic between VPS and Home Lab is encrypted
- **Direct access**: Your laptop connects directly to Home Lab (bypasses VPS)

**Routing logic (external users via Dokploy / Traefik on VPS):**

1. User requests `ha.shatori.com`
2. DNS resolves to VPS public IP
3. VPS: Dokploy / Traefik receives request on port 80/443
4. Traefik routes to internal IP and forwards through Tailscale to GL.iNet
5. GL.iNet forwards to LAN (`192.168.8.x`)
6. Service responds

**For Minecraft (UDP):**
- VPS nginx stream proxy listens on port 25565
- Forwards UDP through Tailscale to GL.iNet
- GL.iNet forwards to Minecraft VM

---

## Tailscale (Optional — Direct Access)

### Why Tailscale

Sometimes you want **direct access** to Home Lab services without going through VPS (faster, bypasses bandwidth limits).

### How It Works

```
Your laptop/phone (Tailscale client)
       │
       ▼
   Tailscale network (100.x.x.x)
       │
       ▼
   GL.iNet (Tailscale server)
       │
       ▼
   Home Lab services (direct)
```

### Services Accessible via Tailscale

| Service | Access Method |
|---------|---------------|
| Plex | Direct (bypasses VPS, maximum speed) |
| Home Assistant | Direct |
| Nextcloud | Direct |
| SSH/Console | Direct |

### Access Methods

**Option 1: By Tailscale IP**
```
http://100.64.0.1:32400 (Plex)
```

**Option 2: By MagicDNS**
```
plex.tail-scale.ts (if configured)
```

### Hybrid Access Model

| User | Access Method |
|------|---------------|
| **You** | Tailscale → Direct (fastest) |
| **Friends/Family** | public domain → VPS Dokploy / Traefik → Home Lab |

### When to Use Each

| Scenario | Use |
|----------|-----|
| **You watching Plex** | Tailscale (faster, no VPS bottleneck) |
| **Friend watching Plex** | VPS Dokploy / Traefik (no VPN needed) |
| **You accessing HA** | Either works |
| **External user accessing** | VPS Dokploy / Traefik (no VPN) |

---

## Important Notes

1. **Dokploy / Traefik on VPS** — Reliable routing, if a Home Lab service dies the public edge still stays on the VPS
2. **No VPN on client** — External users access via public domain, no Tailscale needed
3. **Tailscale optional** — Direct access for owner, bypasses VPS for faster speeds
4. **Home Lab is NOT directly exposed** — All traffic goes through VPS (which has public IP)
5. **GL.iNet provides portability** — Works with wired internet or 4G failover
6. **Services run as LXC** — Simple, efficient, no Kubernetes needed for home use

---

## Plex Deployment Options

### Three Options

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| **A: LXC** (Recommended) | Plex in container | Fast, near bare-metal, easy GPU passthrough | Less isolation than VM |
| **B: VM** | Plex in virtual machine | Full isolation | Worse GPU access, overhead |
| **C: Bare Metal** | Separate physical server | Maximum performance | No isolation, dedicated hardware |

**Recommendation:** Start with **LXC** — best balance of performance and simplicity.

---

## Starting Minimal & Scaling

### Phase 1: Start

- Proxmox (single node)
- LXC: Plex, Home Assistant, Minecraft
- TrueNAS for storage (if separate hardware)
- VPS with Dokploy / Traefik for routing

### Phase 2: Scale

- Add more LXC containers
- Add more storage
- Add backup solution
- Consider Tailscale for direct access

### Why Separate

| Combined (Same Server) | Separated |
|------------------------|-----------|
| Compute + Storage on one server | Synology/RAID for storage |
| Risk: if server fails, everything fails | Proxmox for compute only |
| Hard to scale | Easy to scale independently |

**Best Practice:** Storage (Synology/NAS) separate from Compute (Proxmox).

---

## Home Assistant OS Requirements

- **Must be VM** (not container or K8s)
- Requires direct hardware access
- Needs USB passthrough (Zigbee adapters, etc.)
- Run outside Kubernetes

---

## Starting Minimal & Scaling

### Phase 1: Start

- Proxmox
- 1 VM → Kubernetes (single node)
- 1 VM → Home Assistant OS
- Plex → LXC

### Phase 2: Scale

- Add more K8s nodes
- Separate storage to Synology
- Add backup solution

---

## Immich (Photo Management)

### Overview

Immich is a self-hosted photo management solution that can integrate with existing NAS storage.

### How It Works

Immich can use **External Library** feature:
1. Mount a folder with photos into the Immich container
2. Add it as an External Library
3. Immich scans and displays photos/videos

**Important:** Immich does NOT automatically find photos. You must explicitly:
- Mount the folder in the container
- Add it as External Library

### Three Integration Options

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| **1. Separate folder** (Recommended) | Create `media/photos` on NAS, mount to both | Clean separation, easy management | Need to manage two access points |
| **2. Shared folder** | Both Nextcloud and Immich access same `media/photos` folder | Single source of truth | Requires careful folder structure |
| **3. Nextcloud data folder** | ❌ Not recommended | - | Nextcloud has internal structure (trash, previews, versions) — bad integration point |

### Recommended Folder Structure

```
NAS Storage
└── media/
    ├── photos/        ← Shared folder for Immich + Nextcloud
    ├── videos/        ← Plex media
    ├── documents/     ← Nextcloud files
    └── backups/       ← Backup storage
```

### Access Pattern

- **Nextcloud**: Access `media/photos` for file management
- **Immich**: Mount `media/photos` as External Library
- **Both see same files**

### External Library Limitations

- **Read-only library**: Immich cannot write .xmp sidecar files
- **Metadata**: Some metadata may be lost after rescan if files moved outside Immich
- **Duplicates**: Duplicate detection is per-library, not global — same files in different libraries may appear twice

### Best Practice

1. Create dedicated folder `media/photos` on NAS
2. Grant Nextcloud access to this folder
3. Mount `media/photos` in Immich container
4. Add as External Library in Immich
5. Configure periodic rescan for new photos

---

## TrueNAS Capabilities

### What TrueNAS Does Well

| Feature | Description |
|---------|-------------|
| **Storage** | ZFS, pools, disks, RAID |
| **SMB Shares** | Network folders for Windows/macOS/Linux |
| **NFS** | Network file system for Linux |
| **iSCSI** | Block-level storage |
| **Snapshots** | Built-in ZFS snapshots |
| **Web Interface** | Web UI for administration (not file browser) |

### What TrueNAS Does NOT Do (Without Apps)

- **Cloud file portal** by default (but see WebShare below)
- **Folder sync** like Dropbox/Google Drive
- **Automatic photo backup** from phone
- **Web file browser** (limited in older versions)

### TrueNAS 26 WebShare (New)

TrueNAS 26 introduced **WebShare** — browser-based file access without SMB/NFS:
- View, upload, download, manage files from browser
- Requires configured dataset and user with access
- Not the same as admin dashboard — separate feature

### Adding Cloud Features (via Apps)

To add sync/backup capabilities, install apps on TrueNAS SCALE:

| App | Purpose |
|-----|---------|
| **Syncthing** | Folder sync between devices |
| **Nextcloud** | Personal cloud (files + sync) |
| **Immich** | Photo/video backup from phone |

---

## Nextcloud: On TrueNAS vs Proxmox

### Option 1: Nextcloud on TrueNAS

**Install via TrueNAS Apps**

✅ Good when:
- Single server, want simplicity
- Small home use case
- Don't want to manage extra VM
- OK with Nextcloud in TrueNAS ecosystem

**Considerations:**
- Less isolation from storage layer
- Updates tied to TrueNAS
- People successfully proxy it via Nginx Proxy Manager

### Option 2: Nextcloud on Proxmox (Recommended for your architecture)

**Run as VM/LXC on Proxmox**

✅ Better when:
- Building homelab around Proxmox
- Want full control over OS, PHP, DB, Redis, backups
- Prefer clean architecture: Proxmox = compute, TrueNAS = storage
- Want isolation between storage and web services

**Recommended Pattern:**
```
Proxmox (compute)
├── VM/LXC: Nextcloud
├── VM/LXC: PostgreSQL (optional, separate)
└── VM/LXC: Redis (optional, separate)

TrueNAS (storage)
├── Dataset: Nextcloud data
├── Dataset: Backups
└── Snapshots
```

### Why Not Use SMB for Nextcloud Data Directory

| Mount Method | Issue |
|--------------|-------|
| SMB to TrueNAS | Not recommended for Nextcloud data directory — performance, locking issues |
| NFS | Possible but requires careful design |

**Best practice:** Nextcloud runs on Proxmox, uses NFS to mount TrueNAS dataset, or use TrueNAS scale-out functionality.

### Decision Summary

| Use Case | Recommendation |
|----------|----------------|
| Simple, one server | Nextcloud on TrueNAS |
| Proxmox-based homelab | Nextcloud in VM/LXC on Proxmox, TrueNAS as storage |

---

## Plex Access via Tailscale

### Overview

Plex in LXC on Proxmox can be accessed directly through Tailscale, bypassing the VPS entirely for maximum performance.

### Benefits

| Through VPS | Through Tailscale |
|-------------|-------------------|
| You → VPS → VPN → Plex | You → Plex direct |
| Slower | Faster |
| Double traffic | Direct connection |
| Bottleneck | No bottleneck |

### Two Setup Options

#### Option 1: Tailscale on Proxmox Host (Recommended)

```
Proxmox (Tailscale installed)
   ↓
LXC (Plex)
```

- Tailscale runs on the Proxmox host itself
- Plex runs inside LXC
- Access via Proxmox's Tailscale IP

**Setup:**
```
Tailscale IP: 100.x.x.x
Plex URL: http://100.x.x.x:32400/web
```

**Pros:**
- Simpler
- More stable
- No container modifications
- Works out of the box

#### Option 2: Tailscale Inside LXC

```
LXC (Plex + Tailscale)
```

- Plex container has its own Tailscale client

**Pros:**
- Better isolation
- Separate IP

**Cons:**
- More complex setup
- Permission issues sometimes

### How to Connect

#### Option 1: Browser
```
http://100.x.x.x:32400/web
```

#### Option 2: Plex App
- Plex app auto-discovers server through network

### Hybrid Access Model

| Service | Access Method |
|---------|---------------|
| **Plex** | Direct via Tailscale (bypasses VPS, max speed) |
| **Nextcloud** | Via VPS (Dokploy / Traefik) |
| **Home Assistant** | Via VPS (Dokploy / Traefik) |
| **Navidrome** | Via VPS (Dokploy / Traefik) |

### Why Tailscale is Better for Plex

Plex has NAT traversal and relay capabilities, but Tailscale is:
- More stable
- No Plex relay servers
- Full control
- Better throughput for streaming

### Traffic Flow Diagram

```
[You]
   ↓
[Tailscale]
   ↓
[Proxmox host]
   ↓
[Plex LXC]
```

### When to Use VPS for Plex

Use VPS (Dokploy / Traefik) for:
- External users who don't have Tailscale
- Beautiful domain with TLS (`plex.example.com`)
- Controlled access for friends/family

Use Tailscale for:
- Your direct access
- Maximum speed
- Bypassing VPS bottleneck

---

## Key Decisions Summary

### 1. Router: GL.iNet

- **Model**: GL-X2000 (Spitz Plus) or GL-X3000 (Spitz AX)
- **Why**: Supports Ethernet WAN + 4G SIM failover, LAN ports, OpenWRT based
- **Connection**: Connect to apartment router via WAN, distribute internet via LAN
- **Failover**: Automatically switches to 4G when wired internet fails

### 2. Dokploy / Traefik: On VPS

- **Why**: Public TLS and routing stay on the VPS, with no second proxy layer inside Home Lab.
- **External users flow**: public domain → VPS Dokploy / Traefik → Tailscale → Home Lab

### 3. VPN between VPS and Home Lab: Tailscale

- Tailscale on VPS (client)
- Tailscale on GL.iNet (client)
- Mesh VPN: laptop also connects directly to Home Lab

### 4. Direct Access (for owner): Tailscale

- Already included in Tailscale mesh
- No separate WireGuard needed

### 5. Access via Tailscale

- Connect to Tailscale on laptop
- Enter internal IP of service in browser:
  - Home Assistant: `http://192.168.8.101:8123`
  - Plex: `http://192.168.8.102:32400/web`
- MagicDNS can be configured optionally

---

## Two Access Modes

| User | Access Method | Route |
|------|---------------|-------|
| **External (friends)** | public domain | VPS → Dokploy / Traefik → Tailscale → Home Lab |
| **You (faster)** | Tailscale | Laptop → Tailscale → GL.iNet → Home Lab |

---

## Advantages

1. **Reliability**: Dokploy / Traefik on VPS — one public edge, no double proxy
2. **Portability**: GL.iNet with 4G — can move, just plug in cable
3. **Speed**: Tailscale for yourself — no VPS bottleneck
4. **Simplicity**: LXC containers, no Kubernetes

---

## Validated Dokploy / Traefik Test

The first working public test of the current edge setup is:

```text
https://test-1.shatori.com
  → VPS public IP
  → Dokploy / Traefik on VPS
  → Tailscale subnet route
  → GL.iNet LAN
  → http://192.168.8.141:3000
```

### Notes

- TLS terminates on the VPS
- Backend stays on the Home Lab LAN (`192.168.8.x`), not on a `100.x.x.x` Tailscale IP
- This confirms Dokploy / Traefik can proxy arbitrary Home Lab services over the Tailscale subnet route
- Future HTTP services should follow the same pattern unless a specific app needs raw TCP/UDP handling

### Traefik Dynamic Configuration Used for the Test

File on VPS:

```text
/etc/dokploy/traefik/dynamic/homelab-test.yml
```

```yaml
http:
  routers:
    homelab-test-http:
      rule: "Host(`test-1.shatori.com`)"
      entryPoints:
        - web
      middlewares:
        - redirect-to-https
      service: homelab-test-service

    homelab-test-https:
      rule: "Host(`test-1.shatori.com`)"
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt
      service: homelab-test-service

  middlewares:
    redirect-to-https:
      redirectScheme:
        scheme: https
        permanent: true

  services:
    homelab-test-service:
      loadBalancer:
        servers:
          - url: "http://192.168.8.141:3000"
        passHostHeader: true
```

---

## Physical Connection

### Diagram

```
                    ┌─────────────────────────────────────┐
                    │         GL.iNet Router              │
                    │  (WAN → apartment router/4G)        │
                    │  (LAN → 1 port + switch)            │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────┼──────────────────────┐
                    │              │                      │
                    ▼              ▼                      ▼
           ┌────────────┐  ┌────────────┐  ┌────────────┐
           │ TrueNAS    │  │ Proxmox    │  │ Switch     │
           │ (storage   │  │ (old       │  │ (4-8 ports)│
           │  server)   │  │  laptop)   │  └────────────┘
           └────────────┘  └────────────┘

### Connection Options

**Option 1: Wired (recommended)**

```
GL.iNet (LAN port 1) ──► Ethernet cable ──► TrueNAS
GL.iNet (LAN port 2) ──► Ethernet cable ──► Proxmox (laptop)
```

- Faster and more stable
- Required for TrueNAS (NAS must be on wire)

**Option 2: Wi-Fi (for Proxmox on laptop)**

```
GL.iNet (Wi-Fi) ──► Laptop with Proxmox
```

- Works but slower
- Good for testing initially

### What You Need for Wired Connection

| Component | What You Need |
|-----------|---------------|
| **TrueNAS** | Network card (usually built into motherboard) |
| **Proxmox** | Network card (usually built into laptop) |
| **Cables** | Ethernet cables (CAT5e or CAT6) |
| **Router** | GL.iNet with LAN ports |

### For Old Laptop

```
Laptop
   │
   ├─ Wi-Fi → Connect to GL.iNet (or apartment router)
   │
   └─ Proxmox installed
```

For now — **Wi-Fi is fine**. Later, if you want stability — buy USB Ethernet adapter (~$10-15).

### Summary

| Component | Connection |
|-----------|------------|
| **TrueNAS** | Wired via switch (to GL.iNet) |
| **Proxmox** | Wired or Wi-Fi via switch |
| **GL.iNet** | WAN → apartment router, LAN → switch → TrueNAS + Proxmox |

---

### Switch Required

All GL.iNet routers with 4G have only 1 LAN port. Add a network switch:

| Switch | Ports | Price (~) |
|--------|-------|-----------|
| **TP-Link TL-SG1008D** | 8 | €25 |
| **Netgear GS108** | 8 | €30 |
