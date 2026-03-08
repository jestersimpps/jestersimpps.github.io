---
layout: post
title: "Building a one-click VPN for surviving the Great Firewall"
description: "I'm moving to China and the Great Firewall blocks everything. So i built a desktop app with Tauri and Rust that tunnels traffic through VLESS + Reality — undetectable"
date: 2026-03-08
author: "Jester Simpps"
categories: [Development]
tags: [Rust, Tauri, VPN, VLESS, Networking, Privacy]
image: /images/tunnel-vpn-cover.png
---

I'm moving to China for a while. and if you know anything about the internet situation there, you know the Great Firewall blocks basically everything — Google, YouTube, WhatsApp, Twitter, even ChatGPT. the usual VPN apps get detected and blocked pretty quickly too

So i built my own

A desktop app that connects you to a proxy server with one click. no config files, no manual setup, no terminal commands. just click and your traffic is encrypted, obfuscated, and routed through a server that the firewall can't distinguish from regular web traffic

## Why not just use a regular VPN

Commercial VPNs are the first thing China blocks. they know the protocols, they know the server IPs, they do deep packet inspection on everything. most VPN connections get throttled or killed within seconds

The Great Firewall is actually really sophisticated. it doesn't just block IPs — it analyzes traffic patterns, TLS fingerprints, packet timing. if your connection doesn't look like normal web browsing, it gets flagged

So i needed something that doesn't look like a VPN at all

## The trick: VLESS + Reality

This is where it gets interesting. i'm using the VLESS protocol with Reality transport — and Reality does something clever

It makes your proxy traffic look like a normal HTTPS connection to a legitimate website. in this case, the TLS handshake mimics a connection to www.microsoft.com on port 443. the firewall sees what looks like someone browsing Microsoft's website. nothing suspicious

The encryption stack:
- **Reality** handles TLS-level encryption and obfuscation
- **xtls-rprx-vision** for flow control
- **Chrome TLS fingerprint** spoofing to match real browser traffic
- **Curve25519 public key** crypto for the handshake

VLESS itself sets encryption to "none" because Reality already handles everything at the transport layer. no double encryption, no overhead, no detectable VPN signature

## The stack

- **Tauri 2** — desktop app framework (Rust backend, web frontend)
- **React 19 + TypeScript** — frontend UI
- **xray-core** — the proxy engine running VLESS + Reality
- **3x-ui** — server-side panel for managing connections

Tauri was the obvious choice. Electron would work but shipping a full Chromium for what's essentially a connect button felt excessive. Tauri gives me a native app with a tiny footprint and a Rust backend that can spawn processes and modify system routing tables

## How it works

When you click connect, a lot happens in about 2 seconds

### Auto-registration

The app ships with credentials for a 3x-ui management panel running on my server. on first connect, it generates a UUID for your device and registers itself as a client. no manual step needed

```
App launch → Generate UUID → Login to panel → Register device → Save identity
```

Every subsequent connect just checks that the registration still exists. i wanted zero friction — especially for family members who might need this too

### The proxy chain

```
Your browser
  → System SOCKS proxy (127.0.0.1:1080)
    → xray-core (local process)
      → VLESS + Reality encrypted tunnel (TCP:443)
        → Remote server (outside China)
          → Internet
```

xray-core runs as a child process. the app writes a JSON config, spawns the binary, and polls port 1080 until the SOCKS5 proxy is ready. then it tells macOS or Windows to route all traffic through that local proxy

### System integration

The trickiest part isn't the proxy — it's making the OS cooperate

On macOS, the app runs `networksetup` with admin privileges to set the system SOCKS proxy. on Windows, it writes to the registry. both require elevation

The app snapshots your network state before connecting — default gateway, interface, DNS servers. if the app crashes while connected, it restores everything on next launch automatically

## Sharing access

This was important. i'm not the only one who needs this

The app can export a VLESS URI as a QR code. scan it with v2rayNG on Android or Streisand on iOS and you get the same connection. the URI contains everything — server address, UUID, public key, Reality settings. one scan, connected

So i can set up my server once and share access with anyone who needs it. no explaining how proxy configs work, no walking people through terminal commands over WeChat

## What i'd improve

The panel credentials are hardcoded. the panel connection uses HTTP without TLS verification. config files store credentials in plaintext. for a personal tool this is fine, but i know i should fix these before sharing more broadly

The Rust code uses some `unsafe` static mutable state for process handles. it works, but an Arc/Mutex approach would be cleaner. sometimes shipping beats perfection — especially when you have a flight to catch

## The result

A 15MB desktop app that turns a $5/month VPS into a personal VPN that the Great Firewall can't detect. one click to connect, one click to disconnect. traffic looks like regular web browsing to anyone watching

Sometimes the best tool is the one nobody has to think about. and when you're about to lose access to half the internet, you build what you need
