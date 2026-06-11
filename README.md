# 🔒 WireGuard VPN Setup Guide (Server + Client)

A beginner-friendly guide to building your own VPN from scratch using WireGuard. No prior networking experience needed — just follow the steps!

---

## 📋 Table of Contents

- [What is WireGuard?](#what-is-wireguard)
- [Requirements](#requirements)
- [Step 1 — Generate Keys (Server)](#step-1--generate-keys-server)
- [Step 2 — Configure the Server](#step-2--configure-the-server)
- [Step 3 — Open Firewall Ports](#step-3--open-firewall-ports)
- [Step 4 — Enable IP Forwarding](#step-4--enable-ip-forwarding)
- [Step 5 — Add Client to Server Config](#step-5--add-client-to-server-config)
- [Step 6 — Generate Keys (Client)](#step-6--generate-keys-client)
- [Step 7 — Configure the Client](#step-7--configure-the-client)
- [Step 8 — Start WireGuard](#step-8--start-wireguard)
- [Quick Reference](#quick-reference)

---

## What is WireGuard?

WireGuard is a modern, fast, and simple VPN protocol. Think of it like a secure tunnel between your device and a server — all your internet traffic flows through that tunnel, encrypted and private.

---

## Requirements

| What you need | Details |
|---|---|
| A Linux server (VPS or dedicated) | Ubuntu 20.04+ recommended |
| Root or sudo access on the server | You'll need admin privileges |
| WireGuard installed | `sudo apt install wireguard` |
| A client device | Linux, Windows, macOS, Android, or iOS |

> 💡 **Install WireGuard on Ubuntu:**
> ```bash
> sudo apt update && sudo apt install wireguard -y
> ```

---

## Step 1 — Generate Keys (Server)

WireGuard uses a **public/private key pair** — like a lock and key. You generate these on the **server** first, inside a dedicated `keys` folder in your home directory.

```bash
# 1. Create a folder to store your keys (inside your home directory on the SERVER)
mkdir ~/keys

# 2. Go into that folder — generate keys HERE, not anywhere else!
cd ~/keys

# 3. Set strict permissions BEFORE generating keys — this is important!
#    umask 077 means only YOU (root) can read the key files. No other users can peek at them.
umask 077

# 4. Generate the private key, then derive the public key from it
wg genkey > privatekey
wg pubkey < privatekey > publickey
```

> 📁 **Where are the keys?** Both `privatekey` and `publickey` files are saved in `~/keys/` on your server. That's `/root/keys/` if you're logged in as root.

To view your keys:
```bash
cat ~/keys/privatekey   # Your private key — NEVER share this!
cat ~/keys/publickey    # Your public key — safe to share
```

> ⚠️ **Why `umask 077`?** Without this, Linux might create the key files with permissions that let other users on the server read them. `umask 077` locks the files down so only you can access them. Always run this before generating keys!

---

## Step 2 — Configure the Server

WireGuard config files live in `/etc/wireguard/`. You'll create a file called `wg0.conf` (think of `wg0` as the name of your VPN network interface).

```bash
sudo nano /etc/wireguard/wg0.conf
```

Paste in this config (replace the private key with your actual server private key):

```ini
[Interface]
PrivateKey = <YOUR_SERVER_PRIVATE_KEY_HERE>
Address = 172.20.0.1/16
ListenPort = 44556
```

**What each line means:**

| Line | Meaning |
|---|---|
| `PrivateKey` | The server's private key you generated in Step 1 |
| `Address = 172.20.0.1/16` | The IP address of this server on the VPN network. `172.20.0.1` is the server, `/16` means the VPN can have up to 65,534 devices |
| `ListenPort = 44556` | The UDP port WireGuard listens on. You can use any unused port |

Save and exit: `Ctrl+X`, then `Y`, then `Enter`.

---

## Step 3 — Open Firewall Ports

You need to tell your server's firewall to allow VPN traffic and keep SSH access open.

```bash
# Check current firewall status
sudo ufw status

# Allow the WireGuard VPN port (UDP)
sudo ufw allow 44556/udp

# Make sure SSH stays open so you don't lock yourself out!
sudo ufw allow OpenSSH

# Enable the firewall if it's not already on
sudo ufw enable
```

> ⚠️ **Always allow OpenSSH before enabling UFW**, or you might get locked out of your server.

---

## Step 4 — Enable IP Forwarding

By default, Linux won't forward network packets between interfaces. You need to turn this on so your VPN traffic can flow properly.

```bash
# Open the sysctl config file
sudo nano /etc/sysctl.conf
```

Find the line that says `#net.ipv4.ip_forward=1` and uncomment it (remove the `#`):

```
net.ipv4.ip_forward = 1
```

Save the file, then apply the change immediately (no reboot needed):

```bash
sudo sysctl -p /etc/sysctl.conf
```

You should see `net.ipv4.ip_forward = 1` printed in the terminal, confirming it worked.

---

## Step 5 — Add Client to Server Config

Now you need to tell the server about your client device. Every client that connects to the VPN needs a `[Peer]` block added to the server's `wg0.conf`.

> 📝 First, generate your **client keys** (see Step 6), then come back here and add the client's public key.

Open the server config again:

```bash
sudo nano /etc/wireguard/wg0.conf
```

Add a `[Peer]` block at the bottom for each client:

```ini
[Interface]
PrivateKey = <YOUR_SERVER_PRIVATE_KEY_HERE>
Address = 172.20.0.1/16
ListenPort = 44556

[Peer]
# This is your client device
PublicKey = <YOUR_CLIENT_PUBLIC_KEY_HERE>
AllowedIPs = 172.20.0.2/32
```

**What each line means:**

| Line | Meaning |
|---|---|
| `PublicKey` | The **client's** public key (generated in Step 6) |
| `AllowedIPs = 172.20.0.2/32` | The VPN IP address assigned to this client. Give each client a unique IP (`172.20.0.2`, `172.20.0.3`, etc.) |

> 💡 `/32` means exactly one IP address — this specific client only.

---

## Step 6 — Generate Keys (Client)

On your **client device**, generate a key pair the same way as the server.

**On Linux:**
```bash
# Create the keys folder on your CLIENT machine
mkdir ~/keys
cd ~/keys

# Lock down permissions before generating (same as the server!)
umask 077

wg genkey > privatekey
wg pubkey < privatekey > publickey

cat ~/keys/publickey   # Copy this — you'll need it for the server config (Step 5)
```

**On Windows/macOS/Mobile:**
The WireGuard app can generate keys for you automatically when you create a new tunnel (see Step 7).

---

## Step 7 — Configure the Client

Create a WireGuard config on your client device. This tells the client where the server is and how to connect.

**Create the file** (on Linux: `/etc/wireguard/wg0.conf`, or use the WireGuard app on other platforms):

```ini
[Interface]
PrivateKey = <YOUR_CLIENT_PRIVATE_KEY_HERE>
Address = 172.20.0.2/32
DNS = 1.1.1.1

[Peer]
# This is the server
PublicKey = <YOUR_SERVER_PUBLIC_KEY_HERE>
Endpoint = <YOUR_SERVER_PUBLIC_IP>:44556
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

**What each line means:**

| Line | Meaning |
|---|---|
| `PrivateKey` | The **client's** private key (generated in Step 6) |
| `Address = 172.20.0.2/32` | The VPN IP you assigned to this client in the server's `[Peer]` block |
| `DNS = 1.1.1.1` | DNS server to use over the VPN (Cloudflare DNS — you can change this) |
| `PublicKey` | The **server's** public key (from Step 1) |
| `Endpoint` | Your server's **real public IP address** and WireGuard port |
| `AllowedIPs = 0.0.0.0/0` | Route **all** traffic through the VPN (full tunnel mode) |
| `PersistentKeepalive = 25` | Sends a packet every 25 seconds to keep the connection alive through firewalls/NAT |

> 💡 To find your server's public IP: run `curl ifconfig.me` on your server.

---

## Step 8 — Start WireGuard

**On the server:**
```bash
# Start WireGuard now
sudo wg-quick up wg0

# Enable it to start automatically on reboot
sudo systemctl enable wg-quick@wg0

# Check that it's running
sudo wg show
```

**On the client (Linux):**
```bash
sudo wg-quick up wg0
```

**On the client (Windows/macOS/Android/iOS):**
Open the WireGuard app → Import the config file or scan the QR code → Toggle the tunnel ON.

---

## ✅ Quick Reference

```
SERVER                          CLIENT
──────────────────────          ──────────────────────
Private Key → keep secret       Private Key → keep secret
Public Key  → give to client    Public Key  → give to server

Server config adds:             Client config adds:
[Peer] with client public key   [Peer] with server public key
AllowedIPs = client VPN IP      Endpoint = server public IP:port
```

### Useful Commands

```bash
sudo wg show                  # View active WireGuard connections
sudo wg-quick up wg0          # Start the VPN interface
sudo wg-quick down wg0        # Stop the VPN interface
sudo systemctl status wg-quick@wg0   # Check service status
sudo ufw status               # Check firewall rules
```

---

## 🛠️ Troubleshooting

**Can't connect?**
- Double check that the server's public IP and port are correct in the client config
- Make sure the firewall allows UDP on port `44556` (`sudo ufw allow 44556/udp`)
- Confirm IP forwarding is enabled (`cat /proc/sys/net/ipv4/ip_forward` should output `1`)
- Make sure you put the **client's** public key in the **server** config, and the **server's** public key in the **client** config — it's a common mix-up!

**Connection drops frequently?**
- Add `PersistentKeepalive = 25` to the client's `[Peer]` block

---

> Made with Vignesh❤️ — Feel free to fork and improve this guide!
> 🌐 Website: [vignesh.zeal.ninja](http://vignesh.zeal.ninja/)
> 🌐 Wireguard: [wireguard.com](https://www.wireguard.com/)