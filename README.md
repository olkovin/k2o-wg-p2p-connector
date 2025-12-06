# k2o-wg-p2p-connector

[Українська версія](README.uk.md)

Automated WireGuard P2P tunnel setup between two MikroTik RouterOS devices.

## Overview

This script automates the creation of a WireGuard point-to-point tunnel between two MikroTik routers. Run the same script on both devices — keys are exchanged automatically via a temporary SSTP connection.

## Features

- **One script for both sides** — same file runs on server and client
- **Automatic key exchange** — no manual copy-paste of public keys
- **Unique tunnel ID** — each tunnel gets a timestamp-based identifier
- **Easy removal** — auto-generated removal script for clean uninstall
- **Firewall integration** — automatically opens WireGuard port on server

## Quick Start

1. Download `wg-p2p-connector.rsc` to both routers
2. Edit configuration section on both routers:
   ```routeros
   :global p2pRole "server"              # or "client"
   :global p2pServerAddress "1.2.3.4"    # public IP of server
   :global p2pSstpPass "YourPassword"    # same on both
   ```
3. On **SERVER** (router with public IP), run first:
   ```routeros
   /import wg-p2p-connector.rsc
   ```
4. On **CLIENT**, run within 5 minutes:
   ```routeros
   /import wg-p2p-connector.rsc
   ```
5. Done! Test with:
   ```routeros
   /ping 10.200.0.1   # from client
   /ping 10.200.0.2   # from server
   ```

## Configuration Reference

### Basic (required)

| Variable | Description | Example |
|----------|-------------|---------|
| `p2pRole` | Router role | `"server"` or `"client"` |
| `p2pServerAddress` | Public IP/FQDN of server | `"vpn.example.com"` |
| `p2pSstpPass` | Key exchange password | `"SecurePass123!"` |

### Main (with defaults)

| Variable | Default | Description |
|----------|---------|-------------|
| `p2pTunnelName` | `"tunnel1"` | Short tunnel identifier |
| `p2pServerWgIP` | `"10.200.0.1"` | Server WireGuard IP |
| `p2pClientWgIP` | `"10.200.0.2"` | Client WireGuard IP |
| `p2pWgNetmask` | `"/30"` | Network mask |

### Advanced (optional)

| Variable | Default | Description |
|----------|---------|-------------|
| `p2pWgPort` | `51820` | WireGuard UDP port |
| `p2pSstpPort` | `443` | SSTP TCP port for key exchange |
| `p2pSstpUser` | `"p2p-k2o-exchange"` | SSTP username |
| `p2pTimeout` | `300` | Server wait timeout (seconds) |

## How It Works

```
SERVER (run first)                     CLIENT (run second)
──────────────────                     ────────────────────
1. Generate Tunnel ID
2. Create WG interface
3. Start SSTP server
4. Wait for client...                  1. Create WG interface
                                       2. Connect via SSTP
        ◄─── SSTP tunnel ───►
                                       3. Exchange keys via SSH
5. Receive client pubkey               4. Receive server pubkey + ID
6. Add WG peer                         5. Add WG peer
7. Add firewall rule
8. Stop SSTP server                    6. Disconnect SSTP
9. Generate remove script              7. Generate remove script

        ◄─── WireGuard tunnel ───►
```

## Removal

To remove the tunnel, run on each router:
```routeros
/system script run remove-p2p-k2o-{ID}
```

The removal script name is shown at the end of deployment.

## Routing Examples

The script creates point-to-point connectivity only. For routing additional networks, see [MikroTik WireGuard documentation](https://help.mikrotik.com/docs/display/ROS/WireGuard).

**Route client LAN (192.168.88.0/24) through tunnel (on server):**
```routeros
/ip route add dst-address=192.168.88.0/24 gateway=10.200.0.2
```

**Route all traffic through tunnel (on client):**
```routeros
/ip route add dst-address=0.0.0.0/0 gateway=10.200.0.1 distance=10
```

## Manual Key Exchange

If automatic exchange fails, the script will display instructions for manual key copy-paste.

## Requirements

- MikroTik RouterOS 7.x with WireGuard support
- Server must have public IP (or port forwarding for SSTP and WG ports)
- Client must be able to reach server on SSTP port (443 by default)

## License

MIT License — see [LICENSE](LICENSE)

## Support the Project

If you find this useful:

- **GitHub**: Star this repo
- **PayPal**: [paypal.me/olekovin](https://paypal.me/olekovin)
- **Crypto**: See below

<details>
<summary>Crypto donations</summary>

- **BTC**: `bc1qxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
- **ETH**: `0xXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`
- **USDT (TRC20)**: `TXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

</details>

---

Made with help of AI by [@olekovin](https://t.me/olekovin) | [k2o.cc](https://k2o.cc)
