---
layout: post
title: Setting Up a Home VPN with Tailscale on pfSense
description: Simple, secure remote access with no inbound ports
date: 2025-08-14 12:00:00 -0400
categories: [Homelab, Networking]
tags: [home lab, networking, vpn, pfSense, tailscale]
author: Kyle Hartson
---

> Setting up Tailscale on pfSense gave me a secure, no-inbound-ports VPN in under an hour. This post documents my process, key configurations, and lessons learned.
{: .prompt-tip }

Before we get started, a huge thanks to these two guides which made my setup painless: 
- [Tailscale on pfSense — David Isaksson](https://davidisaksson.dev/posts/tailscale-on-pfsense/) 
- [How to Set Up Tailscale on pfSense — Flemmingss](https://flemmingss.com/how-to-set-up-tailscale-on-pfsense/)

If you're new to Tailscale, you can find their install docs [here](https://tailscale.com/kb/1347/installation).

## Why Tailscale?

I chose Tailscale for a few key reasons: 
- **No inbound firewall rules** required — it works entirely with outbound connections. 
- **Centralized authentication** via my GitHub account to control access to my tailnet. 
- **Cross-platform** support and quick deployment. 

While it's arguably less "hands-on" than rolling WireGuard manually, the speed and safety tradeoff was worth it for this stage of my homelab.

## Steps to Install & Configure

### 1. Register & Authenticate
I created a free Tailscale account using my GitHub user. This lets me manage access to my tailnet centrally and securely.

### 2. Install the pfSense Package
In **System → Package Manager**, I installed the **Tailscale** package. After installation, it appears under **VPN → Tailscale** in the web UI.

### 3. Authorize pfSense with a Pre-Auth Key
From the Tailscale **Admin Console → Settings → Keys**, I generated a **pre-authentication key**. 
I pasted this into **pfSense → VPN → Tailscale → Authentication** under *Pre-authentication key*. 
Once saved, the device appeared in the Tailscale admin console, ready for approval. 
Since it serves as an exit node and subnet router, I disabled key expiry.

### 4. Core pfSense Tailscale Settings

**Settings**
- Enable: ✔
- Listen port: default
- State directory: default
- Keep configuration: ✔ (persists through reboots)

**DNS**
- Accept DNS: ✔

**Routing**
- Advertise exit node: ✔ (lets me route traffic through pfSense — useful for whitelisted-IP scenarios) 
- Accept subnet routes: ✔ 
- Advertised routes: my internal LAN subnet 

In the Tailscale admin console, I approved both the exit node and the subnet route.

### 5. Add Remote Devices
I installed Tailscale on my laptop and iPhone, then ran on my laptop:

```bash
sudo tailscale up --accept-routes
sudo tailscale set --exit-node={TAILNET_IP_OF_PFSENSE} --exit-node-allow-lan-access=true
```

Because all connections are outbound, no new fireall rules were needed 

## Gotchas and Lessons Learned 

- An initial subnet typo blocked LAN access until fixed 
- I didn't need any NAT or packet filters for this minimal, initial setup
- I've seen reports of pfSense/Tailscale quicks, but with the settings listed it's been stable. 
