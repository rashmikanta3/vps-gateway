# Central VPS Gateway (Reverse Proxy & SSL Router)

This repository manages the central reverse proxy and automated TLS/SSL certificate distribution for all web applications deployed on the VPS. 

It runs on **Caddy** using `network_mode: host` to route public HTTPS (`:443`) and HTTP (`:80`) traffic directly to isolated backend services listening on local loopback ports.


## Architecture Overview

               Public Traffic (Internet)
                          │
                   Ports 80 & 443
                          │
                          ▼
                 ┌──────────────────┐
                 │   vps_gateway    │  (Caddy in host network mode)
                 │  Let's Encrypt   │
                 └────────┬─────────┘
                          │
         ┌────────────────┴────────────────┐
         ▼                                 ▼
   [http://127.0.0.1:8001](http://127.0.0.1:8001)             [http://127.0.0.1:8002](http://127.0.0.1:8002)
         │                                 │
         ▼                                 ▼
┌──────────────────┐              ┌──────────────────┐
│   Arcova Proxy   │              │    Aman Proxy    │
│  (arcovahomes.in)│              │(duckdns.org/app) │
└──────────────────┘              └──────────────────┘



## Clone & Start Gateway (First Time)

git clone git@github.com:<your-username>/vps-gateway.git ~/gateway
cd ~/gateway
docker compose up -d

## 2.** Verify Certificates & Status**
## View active routing status
docker compose ps

## **Check SSL certificate retrieval logs**
docker logs --tail=50 vps_gateway

# **Zero-Downtime Configuration Reload**
## When adding new domains or editing Caddyfile, do not recreate the container. Reload the configuration in place:

cd ~/gateway
git pull origin main
docker exec -it vps_gateway caddy reload --config /etc/caddy/Caddyfile

## **Firewall Drops (Linux/OCI)**:

sudo iptables -I INPUT 1 -i docker0 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 8001 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 8002 -j ACCEPT
sudo netfilter-persistent save
