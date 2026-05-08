---
title: "MinIO on a Bare Server: Docker, iptables, and the DOCKER-USER Dance"
date: 2026-05-08
tags: [minio, devops, docker, security, iptables, hosting]
---

MinIO is ridiculously simple to set up — it's practically a one-liner with Docker. The S3-compatible API, the web console, file uploads working in minutes. The easy part.

The hard part comes when you realise that **Docker port publishing ignores your firewall completely**.

## The Problem

I spun up MinIO on a bare Ubuntu server:

```bash
docker run -d -p 9000:9000 -p 9001:9001 minio/minio
```

Ports 9000 (S3 API) and 9001 (web console) were exposed to the internet. UFW was configured to only allow SSH. Didn't matter. Docker writes its NAT rules directly into iptables, bypassing UFW's INPUT chain entirely.

Every single container-published port was wide open to the internet.

## The Fix: DOCKER-USER Chain

Docker provides a dedicated iptables chain for exactly this: `DOCKER-USER`. It's evaluated before Docker's own forwarding rules, and Docker explicitly preserves any rules you add there.

Restricting the MinIO console to a single IP:

```bash
iptables -I DOCKER-USER -i eth0 -p tcp --dport 9001 ! -s 223.91.78.233 -j DROP
```

Then save it persistent:

```bash
apt install iptables-persistent
netfilter-persistent save
```

Now only that IP can reach port 9001. Everyone else gets silently dropped at the firewall.

## Making Objects Public

MinIO buckets are private by default. To serve images with direct URLs:

```bash
mc alias set local http://localhost:9000 admin "<password>"
mc anonymous set download local/images
```

Now any file in the `images` bucket is accessible at `http://host:9000/images/filename.jpg`.

## Takeaway

Docker is notoriously bad at firewall integration. Always check which ports are actually exposed vs what you *think* is exposed. A quick `ss -tlnp` or `nmap` from outside can save you a nasty surprise.

The `DOCKER-USER` chain is your friend. Use it.