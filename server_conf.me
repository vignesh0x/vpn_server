# WireGuard VPN Setup Guide (Server + Client)

This guide explains how to build a complete VPN using WireGuard from scratch.
Even beginners can follow this step-by-step.

WireGuard is a fast, modern VPN protocol that is simple, secure, and easy to configure.

---

## 1. Requirements

### Server

* Linux server (Ubuntu / Debian recommended)
* Root access
* Public IP address
* WireGuard installed

### Client

* Linux / Windows / Mac / Android / iOS
* WireGuard installed

---

## 2. Install WireGuard on Server

Update packages

sudo apt update
sudo apt install wireguard -y

Check installation

wg --version

---

## 3. Generate Server Keys

WireGuard uses public-key cryptography.

Run

umask 077
wg genkey > privatekey
wg pubkey < privatekey > publickey

Check files

ls

You should see

privatekey
publickey

View keys

cat privatekey
cat publickey

Example

PrivateKey = KPkxckNEScUcwIweXnyyosfuMCqBL7aUWDfXjS4j41Y=
PublicKey = AbCdEfGhIjKlMnOpQrStUvWxYz123456

⚠ Never share the private key.

---

## 4. Create WireGuard Configuration

WireGuard configs are stored in

/etc/wireguard

Create config file

sudo nano /etc/wireguard/wg0.conf

Example server config

[Interface]
PrivateKey = SERVER_PRIVATE_KEY
Address = 172.20.0.1/16
ListenPort = 44556

Replace SERVER_PRIVATE_KEY with the key from

cat privatekey

---

## 5. Understanding the Network

Address = 172.20.0.1/16

172.20.0.1 → server VPN IP
/16 → subnet size

Subnet range

172.20.0.1 → 172.20.255.254

This allows about 65,534 possible VPN clients.

---

## 6. Configure Firewall

Check firewall

sudo ufw status

Allow WireGuard port

sudo ufw allow 44556/udp

Allow SSH so you don't lock yourself out

sudo ufw allow OpenSSH

Enable firewall

sudo ufw enable

Verify

sudo ufw status

---

## 7. Enable IP Forwarding

Edit system config

sudo nano /etc/sysctl.conf

Find this line

#net.ipv4.ip_forward=1

Uncomment it

net.ipv4.ip_forward=1

Save the file.

Apply changes

sudo sysctl -p

or

sudo sysctl -p /etc/sysctl.conf

Verify

sysctl net.ipv4.ip_forward

Expected output

net.ipv4.ip_forward = 1

---

## 8. Start WireGuard Server

Start VPN

sudo wg-quick up wg0

Check status

sudo wg

Enable auto start

sudo systemctl enable wg-quick@wg0

---

## 9. Generate Client Keys

On the client machine

umask 077
wg genkey > client_privatekey
wg pubkey < client_privatekey > client_publickey

View keys

cat client_privatekey
cat client_publickey

---

## 10. Add Client to Server

Open server config

sudo nano /etc/wireguard/wg0.conf

Add

[Peer]
PublicKey = CLIENT_PUBLIC_KEY
AllowedIPs = 172.20.0.2/32

Restart server

sudo wg-quick down wg0
sudo wg-quick up wg0

---

## 11. Client Configuration

Create file

client.conf

Example

[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 172.20.0.2/16
DNS = 1.1.1.1

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = SERVER_IP:44556
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25

Replace

CLIENT_PRIVATE_KEY
SERVER_PUBLIC_KEY
SERVER_IP

---

## 12. Start Client VPN

Linux

sudo wg-quick up client.conf

Check connection

ping 172.20.0.1

If successful, VPN is working.

---

## 13. Check VPN Status

Server command

sudo wg

Example output

interface: wg0
peer: CLIENT_PUBLIC_KEY
allowed ips: 172.20.0.2/32
latest handshake: 10 seconds ago
transfer: 10 KiB received, 20 KiB sent

---

## 14. Troubleshooting

Check WireGuard status

sudo wg

Check logs

sudo journalctl -u wg-quick@wg0

Check port

sudo ss -ulnp | grep 44556

Check firewall

sudo ufw status

---

## 15. Test Internet Through VPN

Run

curl ifconfig.me

If the IP shown equals the server IP, the VPN is working.

---

## Useful Commands

Start VPN

sudo wg-quick up wg0

Stop VPN

sudo wg-quick down wg0

Check status

sudo wg

Restart VPN

sudo systemctl restart wg-quick@wg0

---

## Security Tips

* Never share private keys
* Always keep your server updated
* Restrict SSH access
* Use strong firewall rules

---

End of Guide
