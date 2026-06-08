# ngrok: Tunneling localhost to the Public Internet

Your dev server binds to `127.0.0.1:8080` — reachable only from your own machine. ngrok gives it a public HTTPS URL that anyone (or any service, like a webhook sender or an MCP client) can reach, without you touching your router, firewall, or DNS.

## The core problem it solves

A process listening on `localhost` is invisible to the outside world for three stacked reasons:

- **It's bound to the loopback interface.** `127.0.0.1` never leaves your machine.
- **You're behind NAT.** Your machine has a private IP (`192.168.x.x`); the public internet only sees your router's IP, and the router has no rule forwarding inbound traffic to you.
- **A firewall** (yours, your ISP's, or your office's) likely blocks unsolicited inbound connections anyway.

The traditional fix — port forwarding on the router + dynamic DNS + a TLS cert — is fragile and often impossible (corporate networks, CGNAT, coffee-shop WiFi). ngrok sidesteps all of it.

## The key insight: the tunnel is outbound

ngrok never accepts an inbound connection *to* your machine. Instead, the ngrok **agent** running on your machine opens an **outbound** persistent connection to ngrok's cloud servers — and outbound connections sail through NAT and firewalls because that's exactly what they're designed to allow (it's the same path your browser uses).

Public traffic then arrives at ngrok's edge, and ngrok pushes it *down that already-open tunnel* to your agent, which hands it to your local server. The direction of the initial connection is the whole trick.

```
                                          ┌─────────────────────────┐
                                          │      ngrok edge         │
  Public client                           │  (public HTTPS URL)     │
  (browser / webhook / MCP host)          │                         │
        │                                 │                         │
        │  1. HTTPS GET                   │                         │
        │     https://abc123.ngrok.app    │                         │
        ├────────────────────────────────►│                         │
        │                                 │                         │
        │                                 │   2. push request DOWN  │
        │                                 │      the open tunnel     │
        │                                 │              │           │
        │                                 └──────────────┼──────────┘
        │                                                │
        │                              ┌─────────────────▼─────────────────┐
        │                              │  tunnel (TLS), opened OUTBOUND     │
        │                              │  by the agent at startup           │
        │                              └─────────────────┬─────────────────┘
        │                                                │
        │                              ┌─────────────────▼─────────────────┐
        │                              │   Your machine                     │
        │                              │   ┌────────────┐   ┌────────────┐  │
        │                              │   │ ngrok agent├──►│ localhost  │  │
        │                              │   │            │   │   :8080    │  │
        │                              │   └────────────┘   └────────────┘  │
        │                              │      3. forward to local server    │
        │  4. response travels back up the same tunnel                      │
        │◄─────────────────────────────────────────────────────────────────┤
                                       └────────────────────────────────────┘
```

## What actually happens, step by step

1. **You start the agent:** `ngrok http 8080`. The agent dials out to ngrok's edge and establishes a long-lived TLS tunnel. Because *you* initiated it, NAT/firewall let it through.
2. **ngrok assigns a public hostname** like `https://abc123.ngrok-free.app` (random on the free tier, or a reserved/custom domain on paid plans) and terminates TLS at its edge — so your public URL is HTTPS even though your local server speaks plain HTTP.
3. **A request hits the edge.** ngrok matches the hostname to your active tunnel and streams the raw HTTP request down to your agent.
4. **The agent replays the request** against `http://localhost:8080` as if it came from a local client.
5. **The response flows back** up the tunnel to the edge and out to the original caller.

From your local server's perspective, every request looks like it came from `127.0.0.1` (the agent). From the caller's perspective, they're talking to a normal public HTTPS site.

## Getting started

```bash
# 1. Install (macOS example; see ngrok.com/download for others)
brew install ngrok

# 2. Authenticate once (token from your ngrok dashboard)
ngrok config add-authtoken <YOUR_TOKEN>

# 3. Tunnel a local port
ngrok http 8080
```

You'll get terminal output with the public URL and a live request inspector at `http://localhost:4040` (a local web UI that shows every request/response passing through — great for debugging webhooks).

## Useful flags & features

| Need | How |
|---|---|
| **Stable URL** across restarts | Reserve a domain: `ngrok http --domain=myapp.ngrok.app 8080` (paid) |
| **Inspect/replay traffic** | Open `http://localhost:4040` — replay any request without re-triggering the sender |
| **Tunnel a raw TCP port** (SSH, DB) | `ngrok tcp 22` |
| **Basic auth** in front of the tunnel | `ngrok http --basic-auth="user:pass" 8080` |
| **Rewrite the Host header** for vhost-picky servers | `ngrok http --host-header=rewrite 8080` |
| **Config file** for multiple named tunnels | Define them in `ngrok.yml`, start with `ngrok start --all` |

## Where it's genuinely useful

- **Receiving webhooks locally** — Stripe, GitHub, Twilio, etc. need a public HTTPS callback; ngrok gives your laptop one instantly.
- **Exposing a local MCP server** to claude.ai — the connector needs a reachable HTTPS URL, and ngrok provides one without deploying anywhere. (See the OAuth note for how MCP auth then layers on top.)
- **Demoing work-in-progress** to a teammate or client without deploying.
- **Testing mobile apps / OAuth redirects** against a real HTTPS endpoint.

## Caveats & gotchas

- **Free-tier URLs are ephemeral** — a new random hostname every restart. Reserve a domain (paid) if a webhook provider has the URL hard-coded.
- **It's a public door to your machine.** Anyone with the URL can hit your local server. Add `--basic-auth`, OAuth, or ngrok's edge auth for anything sensitive.
- **Free tier shows an interstitial warning page** on first browser visit and has a request-rate cap.
- **Latency** — traffic round-trips through ngrok's edge region, so it's slower than a direct connection. Pick a region close to you (`--region`).
- **Not a production hosting solution.** It's for development, demos, and integration testing. For anything permanent, deploy properly (Cloud Run, a VPS, etc.).
- **Corporate security policies** may forbid tunneling out — check before using it on a work network.

## The one-sentence mental model

ngrok flips the connection direction: instead of the world trying (and failing) to reach *into* your NATed, firewalled machine, your machine reaches *out* to ngrok once, and ngrok becomes a public relay that pours inbound traffic back down that hole you punched.
