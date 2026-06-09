# How ngrok Works: Serving localhost Without Port Forwarding

## The Problem ngrok Solves

When you run a server on your machine (e.g. `localhost:3000`), it's only reachable from your own computer. To expose it to the internet the traditional way, you'd normally have to:

- Configure **port forwarding** on your router
- Deal with a **dynamic/changing public IP**
- Open firewall ports (a security risk)
- Possibly have no control over the router at all (corporate networks, CGNAT, etc.)

ngrok removes all of this.

## How ngrok Avoids Port Forwarding

The key insight: **ngrok never needs an inbound connection to your machine.** Instead, your machine makes an *outbound* connection to ngrok's servers, and traffic flows back through that tunnel.

```
                  outbound tunnel (initiated by you)
  [ your localhost ] ───────────────────────────────►  [ ngrok cloud edge ]
       :3000                                                    ▲
                                                                │ public HTTPS
                                                                │
                                                       [ visitor on internet ]
```

Step by step:

1. You run the `ngrok` agent on your machine.
2. The agent opens a persistent **outbound** TLS connection to ngrok's edge servers. Outbound connections are almost always allowed by routers and firewalls (it's the same direction as normal web browsing), so **no port forwarding and no firewall changes are needed**.
3. ngrok assigns you a **public URL** (e.g. `https://abc123.ngrok-free.app`).
4. When a visitor hits that public URL, the request arrives at ngrok's edge, which **pushes it down the already-open tunnel** to your agent.
5. Your agent forwards the request to your local server and sends the response back up the same tunnel.

Because your machine dialed *out* and the connection stays open, ngrok can reach "into" your network without your router ever accepting an unsolicited inbound connection.

## How to Use ngrok

### 1. Install

```bash
# macOS
brew install ngrok

# Linux (snap)
snap install ngrok

# Or download the binary from ngrok.com/download
```

### 2. Authenticate (one time)

Sign up at ngrok.com, copy your authtoken from the dashboard, then:

```bash
ngrok config add-authtoken <YOUR_AUTHTOKEN>
```

### 3. Start a tunnel

If your local app runs on port 3000:

```bash
ngrok http 3000
```

You'll see output like:

```
Forwarding   https://abc123.ngrok-free.app -> http://localhost:3000
```

Share that public URL and anyone can reach your local server.

### Common variations

```bash
# Tunnel a raw TCP service (e.g. SSH, databases)
ngrok tcp 22

# Use a reserved/custom domain (paid plans)
ngrok http --domain=myapp.ngrok.app 3000

# Add basic auth in front of your tunnel
ngrok http 3000 --basic-auth="user:password"
```

The web inspector at `http://localhost:4040` lets you replay and inspect requests passing through the tunnel.

## Encryption: What's Encrypted and What Isn't

This is the part most people misunderstand. There are **two separate hops**, and they have different encryption characteristics.

```
  Visitor ──[Hop 1: HTTPS/TLS]──► ngrok edge ──[Hop 2: TLS tunnel]──► ngrok agent ──[Hop 3: plaintext?]──► your app
                                       │
                                  TLS terminates HERE
                                  (data is briefly in
                                   plaintext in memory)
```

| Hop | Path | Encrypted? |
|-----|------|------------|
| **Hop 1** | Visitor → ngrok edge | ✅ Yes — HTTPS/TLS (ngrok provides the certificate) |
| **Hop 2** | ngrok edge → your agent (the tunnel) | ✅ Yes — TLS-encrypted tunnel over the internet |
| **Hop 3** | ngrok agent → your local app | ⚠️ Usually **plaintext** (`http://localhost:3000`), but this stays inside your own machine/LAN |

The critical point: **ngrok terminates TLS at its edge by default.** That means:

- The traffic is encrypted *in transit* on both internet-facing hops.
- But at the ngrok edge, the data is **decrypted into plaintext** so ngrok can route it, apply your rules (basic auth, request inspection, etc.), and re-encrypt it for the tunnel.

## Can ngrok See Your Data?

**Yes — by default, ngrok can see your traffic in plaintext.**

Because TLS is terminated at ngrok's edge, the unencrypted contents of your HTTP requests and responses (including headers, bodies, form data, and anything not separately encrypted) pass through ngrok's servers in readable form. You are routing your traffic through a third party, so you're trusting ngrok with that data.

### How to prevent ngrok from seeing your data

If you don't want ngrok to be able to read the contents, use **end-to-end TLS / TLS passthrough**, so encryption is *not* terminated at the edge:

```bash
# TLS tunnel — ngrok forwards encrypted bytes without decrypting them.
# Your local server must present the TLS certificate.
ngrok tls --domain=myapp.example.com 443
```

With a `tls` (or raw `tcp`) tunnel, ngrok forwards the encrypted bytes blindly and **cannot read the contents** — but you lose the edge features that require seeing the request (HTTP request inspection, basic auth injection, header rewriting, etc.), and you must handle the certificate yourself.

Other mitigations:

- Put sensitive data behind **application-level encryption** before it ever leaves your app.
- Don't send real secrets/PII through free public tunnels.
- For production, prefer a setup where you control TLS termination.

## Summary

- **No port forwarding needed** because your machine makes an *outbound* connection; ngrok reaches back through that open tunnel.
- **Both internet hops are encrypted** (visitor→edge and edge→agent).
- **TLS is terminated at ngrok's edge by default**, so the data exists in plaintext there.
- **ngrok can see your data** unless you use `tls`/`tcp` passthrough tunnels or encrypt at the application layer.
