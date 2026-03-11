# Firewall Port Request — SSD Nodes VPS

## Account / Server Info

- **Account**: [your account email]
- **Server**: [your VPS hostname or IP, e.g. ssdnodes-66871ef1e08d7]
- **Server IP**: [your VPS public IP]

## Issue

I am running a self-hosted application stack on my VPS that requires several ports to be reachable from the public internet. I have configured UFW on the server and the ports are open at the OS level, but inbound connections are being blocked at the provider/network level before they reach the server.

I have confirmed that UFW is active and the rules are correct:

```
ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
5001/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
8089/tcp                   ALLOW       Anywhere
8443/tcp                   ALLOW       Anywhere
8446/tcp                   ALLOW       Anywhere
9090/tcp                   ALLOW       Anywhere
9443/tcp                   ALLOW       Anywhere
389/tcp                    ALLOW       Anywhere
636/tcp                    ALLOW       Anywhere
8888/tcp                   ALLOW       Anywhere
8554/tcp                   ALLOW       Anywhere
8322/tcp                   ALLOW       Anywhere
8189/udp                   ALLOW       Anywhere
8890/tcp                   ALLOW       Anywhere
```

External connections to these ports time out, while SSH (port 22) works fine. This indicates a firewall or security group at the infrastructure/network level is filtering traffic before it reaches the VPS.

## Ports Requested

Please open the following ports on the network/provider-level firewall for my VPS:

| Port | Protocol | Service |
|------|----------|---------|
| 80 | TCP | HTTP (Caddy web server / Let's Encrypt) |
| 443 | TCP | HTTPS (Caddy TLS reverse proxy) |
| 5001 | TCP | Admin console (HTTPS) |
| 8089 | TCP | TAK Server (TLS client connections) |
| 8443 | TCP | TAK Server (HTTPS federation) |
| 8446 | TCP | TAK Server (WebAdmin HTTPS) |
| 9090 | TCP | Authentik identity provider (HTTP) |
| 9443 | TCP | Authentik identity provider (HTTPS) |
| 389 | TCP | LDAP (Authentik outpost) |
| 636 | TCP | LDAPS (Authentik outpost) |
| 8554 | TCP | RTSP video streaming |
| 8888 | TCP | HLS video streaming |
| 8322 | TCP | SRT video streaming |
| 8890 | TCP | RTMP video streaming |
| 8189 | UDP | WebRTC video streaming |
| 5000 | TCP | Web application (HTTP) |

## Summary

All of these services are legitimate self-hosted applications running on the VPS. I need the provider-level firewall updated to allow inbound traffic on these ports so they are reachable from the internet. UFW on the server itself handles the OS-level filtering.

Thank you.
