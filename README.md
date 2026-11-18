# Salesman

SSH into this machine from anywhere, through a [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/).

The machine only makes an outbound TLS connection (with SNI `h2.cftunnel.com`, TCP 7844) to Cloudflare, so it works behind double NAT with no public IP, and through campnet, which drops connections without a host header / SNI (and with it, raw SSH and WireGuard).

Keep campnet logged in with the [auto-campnet daemon](../README.md) — the tunnel reconnects on its own whenever the network comes back.

## Prerequisites

- A domain on Cloudflare
- An SSH server on the host: `sudo apt install openssh-server`

## Setup

1. In the Cloudflare One dashboard, go to **Networks → Tunnels → Create a tunnel → Cloudflared** and name it.
2. Copy the token (the long string after `--token` in the install command shown).
3. Under **Public hostname**, add `ssh.yourdomain.com` → type **SSH**, URL `localhost:22`.
4. (Recommended) In **Access → Applications**, add a self-hosted app for `ssh.yourdomain.com` with a policy allowing only your email.

## Running

Set `TUNNEL_TOKEN` in `.env`.

```env
TUNNEL_TOKEN=yourtunneltoken
```

Optionally set `TUNNEL_TRANSPORT_PROTOCOL` (defaults to `http2`, i.e. TCP only; set to `auto` to try QUIC over UDP 7844 first).

Start

```sh
docker compose up -d
```

Check it connected

```sh
docker compose logs -f
```

Look for `Registered tunnel connection ... protocol=http2`.

Stop

```sh
docker compose down
```

The container uses host networking so it can reach the host's SSH server on `localhost:22`.

## Connecting

On your client, install [`cloudflared`](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/) and add to `~/.ssh/config`:

```
Host salesman
    HostName ssh.yourdomain.com
    ProxyCommand cloudflared access ssh --hostname %h
```

Then

```sh
ssh salesman
```

The first connection opens a browser to authenticate with Cloudflare Access.

## Hardening

Use key-only auth. On the host, in `/etc/ssh/sshd_config`:

```
PasswordAuthentication no
```

then `sudo systemctl restart ssh`.
