# Proxy Comparison: Nginx Proxy Manager vs Pangolin

## Summary

| Tool | Best For | Not Ideal For |
|------|----------|---------------|
| **Nginx Proxy Manager** | Plex, Minecraft, streaming (L4/L7) | Complex access control |
| **Pangolin** | Web apps (Nextcloud, Home Assistant, Navidrome) | Heavy streaming, raw TCP/UDP |

---

## Key Difference

### Nginx Proxy Manager

- **Type**: Pure reverse proxy (L7/L4)
- **Mechanism**: Direct transport `socket → socket`
- **Layers**: None (transparent)
- **Performance**: Minimal overhead

```
User → VPS → Plex (direct)
```

### Pangolin

- **Type**: Access platform + tunnel + identity layer
- **Mechanism**: `client → Pangolin → tunnel → resource`
- **Layers**: +1 tunnel layer, +identity layer, +resource abstraction
- **Performance**: Additional overhead

```
User → Pangolin → tunnel → Plex
```

---

## Why It Matters for Plex

### Plex Requirements

- Video streaming
- Large data volumes
- Long-lived connections
- Direct play capability
- Low latency
- High throughput
- Stable TCP

### What Pangolin Adds (Bad for Plex)

- Tunnel layer overhead
- Identity layer processing
- Resource abstraction
- Higher latency
- Reduced throughput
- Potential stream instability

### What Nginx Does (Good for Plex)

- Direct socket-to-socket
- No "resources" or "identities" or "tunnels"
- Almost transparent
- Minimal latency loss
- Full bandwidth

---

## When to Use Each

### ✅ Use Pangolin For

- Nextcloud (request-response, small data)
- Home Assistant (web-first)
- Navidrome (HTTP API)
- Internal web services

### ✅ Use Nginx/Direct For

- Plex (streaming, 4K, multiple users)
- Minecraft (raw TCP/UDP)
- Any high-throughput, latency-sensitive service

---

## Hybrid Approach (Recommended)

| Traffic Type | Solution |
|--------------|----------|
| **Web Apps** | Pangolin or Nginx Proxy |
| **Streaming** | Nginx Proxy or direct |
| **Gaming** | Direct or L4 TCP proxy |

### Best Setup

```
Web Apps → Pangolin:
  - Nextcloud
  - Home Assistant
  - Navidrome

Media/Gaming → Nginx or Direct:
  - Plex
  - Minecraft
```

---

## Why Not Pangolin for Plex

1. **Pangolin is web-first** — TCP/UDP is secondary mode
2. **Raw resources mode** has fewer features and stability
3. **Additional processing layer** adds latency
4. **Not optimized for streaming** — designed for HTTP/HTTPS

---

## When Pangolin is OK for Plex

- 1-2 users
- Performance not critical
- "Just make it work"

---

## When NOT to Use Pangolin for Plex

- Normal streaming
- 4K content
- Multiple users
- Stability matters

---

## Alternative: 100% Self-Hosted Without Tailscale

```
┌─────────────────────────────────────┐
│           Self-Hosted Stack          │
├─────────────────────────────────────┤
│  WireGuard  →  Private network      │
│  Traefik/Nginx → Web proxy          │
│  Direct access → Plex               │
└─────────────────────────────────────┘
```

This provides:
- No external dependencies
- Full control
- Optimal performance for all traffic types