
# 🛡️ Pi-hole — Network-Wide Ad Blocker

A hands-on homelab project deploying Pi-hole on a Raspberry Pi 3 B+ for network-wide ad blocking, DNS filtering, and privacy protection across an eero mesh network.

---

## 📋 Project Overview

Pi-hole is a network-level ad blocker that acts as a DNS sinkhole — intercepting DNS requests and blocking known ad-serving and tracking domains before they ever reach your devices. Unlike browser extensions, Pi-hole protects **every device on the network** including smart TVs, phones, and IoT devices.

This project documents the full end-to-end setup process including troubleshooting steps encountered during installation.

---

## 🧰 Hardware & Environment

| Component | Details |
|-----------|---------|
| Device | Raspberry Pi 3 Model B+ |
| OS | Raspberry Pi OS Lite 64-bit (Debian Trixie, headless) |
| Storage | MicroSD Card |
| Network | Metronet Fiber via eero mesh router system |
| Host Machine | Windows 11 (PowerShell / Windows Terminal) |
| Imager | Raspberry Pi Imager v2.0.7 |

---

## 🗺️ Network Architecture

```
[Metronet Fiber] → [eero Gateway Router] → [Raspberry Pi 3 B+ via Ethernet]
                          ↑
                  DNS set to Pi-hole IP
                  (xxx.xxx.x.xx)
                          ↑
              All network devices route
              DNS queries through Pi-hole
```

---

## ⚙️ Setup Process

### Step 1 — Flash the OS
- Used **Raspberry Pi Imager v2.0.7** to flash Raspberry Pi OS Lite (64-bit) to the SD card
- Configured during imaging:
  - Hostname: `[host-name]`
  - Custom username and password
  - SSH enabled
  - Localisation configured

### Step 2 — Boot & Connect
- Inserted SD card into Raspberry Pi 3 B+
- Connected via ethernet to the eero gateway router
- Powered on the Pi via micro-USB

### Step 3 — Reserve Static IP in eero App
- Located the Pi in the eero app under **Devices**
- Used **Reservations and Port Forwarding** to reserve the IP address `xxx.xxx.x.xx`
- This ensures the Pi always receives the same IP from the router

### Step 4 — SSH Into the Pi
Connected from Windows Terminal using:
```bash
ssh [hostname]@xxx.xxx.x.xx
```

### Step 5 — Update the System
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 6 — Install Pi-hole
```bash
curl -sSL https://install.pi-hole.net | bash
```
- Selected **eth0** (ethernet) as the network interface
- Selected **Cloudflare (1.1.1.1)** as the upstream DNS provider
- When warned about static IP — selected **Continue** (IP reservation handled by eero)
- Enabled anonymous statistics (optional)

### Step 7 — Configure DNS on eero
- Opened the eero app → **Settings → Network Settings → DNS**
- Set DNS to **Custom**
- Entered Pi-hole IP `xxx.xxx.x.xx` as the Primary IPv4 DNS
- Saved and allowed eero to reboot

---

## 🔧 Troubleshooting

### Issue 1 — Installer Exited at Static IP Warning
**Problem:** The Pi-hole installer exited when it detected no static IP was configured on the Pi itself.

**What went wrong:** Manually edited `/etc/dhcpcd.conf` to add a static IP configuration with an incorrect gateway address, which broke the Pi's network connection entirely.

**Fix:** Reflashed the SD card and reinstalled. This time when the installer warned about the static IP, selected **Continue** as recommended by the official Pi-hole documentation — the eero IP reservation handles static addressing at the router level, making manual configuration on the Pi unnecessary.

**Lesson learned:** Always follow the official documentation. The eero IP reservation is sufficient for maintaining a stable IP without touching the Pi's network config.

---

### Issue 2 — SSH Host Key Warning After Reflash
**Problem:** After reflashing the SD card, Windows Terminal displayed a `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED` error when trying to SSH in.

**What happened:** Reflashing generates a new SSH host key on the Pi. Windows remembered the old key and flagged the mismatch as a potential security threat.

**Fix:**
```bash
ssh-keygen -R xxx.xxx.x.xx
```
This cleared the old key from the known hosts file, allowing SSH to connect normally.

---

### Issue 3 — Two Raspberry Pi Entries in eero App
**Problem:** The Pi appeared twice in the eero device list after first boot.

**Cause:** The Pi connected to the network on both ethernet and WiFi simultaneously (WiFi credentials were configured during imaging).

**Fix:** During the reflash, WiFi credentials were intentionally left blank. This ensures the Pi only connects via ethernet, providing a more stable connection for a DNS server.

---

## 📊 Results

After installation, Pi-hole immediately began processing DNS queries across the network:

- **Total domains on blocklist:** 92,277
- **Active clients detected:** 9
- **Status:** Active ✅
- **Web interface:** `http://xxx.xxx.x.xx/admin`


## 🔐 Cybersecurity Relevance

This project demonstrates practical knowledge of:

- **DNS architecture** — understanding how DNS resolution works and how it can be intercepted at the network level
- **Network security** — protecting all devices on a network from known malicious and tracking domains
- **Linux system administration** — navigating a headless Debian-based OS via SSH
- **Network configuration** — managing IP reservations, DNS settings, and router configuration
- **Troubleshooting methodology** — diagnosing and resolving network connectivity issues caused by misconfiguration
- **Privacy hardening** — reducing device telemetry and data collection across an entire home network

---

## 🚀 Future Plans

- [ ] Set up **WireGuard VPN** to route mobile traffic through Pi-hole when away from home
- [ ] Add additional blocklists for enhanced filtering
- [ ] Set up **Unbound** as a recursive DNS resolver for full DNS privacy

---

## 📚 References

- [Pi-hole Official Documentation](https://docs.pi-hole.net/)
- [Raspberry Pi Official Pi-hole Tutorial](https://www.raspberrypi.com/tutorials/running-pi-hole-on-a-raspberry-pi/)
  
*Part of the [Gabriella-s-Home-Lab](../README.md) portfolio.*
