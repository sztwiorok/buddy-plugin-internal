---
name: tunnel
description: This skill should be used when the user asks to "expose localhost", "create tunnel", "share local server", "test webhooks locally", "tunnel to localhost", "make local dev accessible", "public URL for localhost", or mentions exposing local services, testing webhooks, or sharing development servers.
---

# Buddy Tunnels

Expose local services running on localhost to the internet via secure Buddy tunnels. Ideal for webhook testing, client demos, mobile device testing, and temporary service sharing.

## When to Use This Skill

Use this skill when:
- Exposing localhost web servers for webhook testing
- Sharing local development work with clients or team members
- Testing applications on mobile devices
- Demonstrating work-in-progress without deployment
- Accessing local services from remote locations
- Testing integrations that require public URLs

## Prerequisites

**Authentication Required:** Tunnels require Buddy CLI authentication. For installation and authentication setup, see the [bdy-auth](../bdy-auth/SKILL.md) skill.

**Note:** If user is not authenticated, ask them to run `bdy login` in a separate terminal (interactive login cannot be performed by AI agents) or use token-based authentication.

Verify authentication:
```bash
bdy workspace ls
```

## Quick Tunnel Workflow

### 1. Start a Tunnel

**HTTP Tunnel (most common):**
```bash
bdy tunnel http localhost:3000
```

**TCP Tunnel (databases, SSH):**
```bash
bdy tunnel tcp localhost:5432
```

**TLS Tunnel (custom certificates):**
```bash
bdy tunnel tls localhost:8443
```

### 2. Access via Public URL

After starting, Buddy provides a public URL:
```
Tunnel started: https://abc123.buddy.works → http://localhost:3000
```

Share this URL to make the local service accessible.

### 3. Stop Tunnel

Press `Ctrl+C` to stop the tunnel. The public URL becomes inactive immediately.

## Tunnel Types

### HTTP/HTTPS Tunnels

Expose web servers, APIs, and HTTP services:

```bash
# Basic HTTP tunnel
bdy tunnel http localhost:3000

# Specify protocol explicitly
bdy tunnel http http://localhost:8080
```

**Common Options:**
- `-n, --name` - Named tunnel for easy identification
- `-a, --auth user:pass` - HTTP basic authentication
- `-w, --whitelist` - IP address restrictions
- `-r, --region` - Regional endpoint (eu, us, as)

If you expose HTTP ask (using the AskUserQuestionTool) user if they want to add authentication
They can protect the tunnel with username and password or Buddy authentication.

**Example with auth:**
```bash
bdy tunnel http localhost:3000 -a username:password
bdy tunnel http localhost:3000 --buddy
```
```

### TCP Tunnels

Expose non-HTTP services (databases, game servers, SSH):

```bash
# PostgreSQL database
bdy tunnel tcp localhost:5432

# SSH server
bdy tunnel tcp localhost:22

# Custom TCP service
bdy tunnel tcp localhost:9000 -n game-server
```

### TLS Tunnels

Expose HTTPS services with custom certificates:

```bash
bdy tunnel tls localhost:8443 \
  --key /path/to/key.pem \
  --cert /path/to/cert.pem
```

## Security Options

### IP Whitelisting

Restrict access to specific IP addresses:

```bash
# Single IP
bdy tunnel http localhost:3000 -w 203.0.113.0/32

# Multiple IPs
bdy tunnel http localhost:3000 -w 203.0.113.0/24 198.51.100.0/24

# Allow all (default)
bdy tunnel http localhost:3000 -w "*"
```

### HTTP Authentication

Protect tunnels with basic authentication:

```bash
bdy tunnel http localhost:3000 -a username:password
```

### Buddy Authentication

Use Buddy account authentication:

```bash
bdy tunnel http localhost:3000 --buddy
```

Only authenticated Buddy users can access the tunnel.

## Configuration Files

Save frequently used tunnel configurations for easy reuse.

### Add Tunnel Configuration

```bash
# Save HTTP tunnel configuration
bdy tunnel config add http dev-api http://localhost:3000

# Save TCP tunnel configuration
bdy tunnel config add tcp postgres localhost:5432
```

### Start from Configuration

```bash
bdy tunnel start dev-api
```

Starts the tunnel using saved configuration.

### Manage Configurations

```bash
# List all configurations
bdy tunnel config get tunnels

# View specific tunnel
bdy tunnel config get tunnel dev-api

# Remove configuration
bdy tunnel config rm tunnel dev-api
```

### Global Settings

Set defaults for all tunnels:

```bash
# Set region
bdy tunnel config set region eu

# Set whitelist
bdy tunnel config set whitelist 203.0.113.0/24

# Set timeout
bdy tunnel config set timeout 300
```

## Regional Selection

Buddy operates tunnels in multiple regions:

```bash
bdy tunnel http localhost:3000 --region eu  # Europe (default)
bdy tunnel http localhost:3000 --region us  # United States
bdy tunnel http localhost:3000 --region as  # Asia-Pacific
```

Choose the region closest to users for best performance.

## Common Use Cases

### Webhook Testing

Expose local dev server for webhook testing (Stripe, GitHub, etc.):

```bash
bdy tunnel http localhost:3000 -n webhook-test
```

Use provided URL as webhook endpoint.

### Client Demos

Share work-in-progress without deployment:

```bash
bdy tunnel http localhost:8080 -n client-demo -a demo:preview
```

Share URL and credentials with client.

### Mobile Testing

Test localhost on mobile devices:

```bash
bdy tunnel http localhost:3000 -n mobile-test
```

Access tunnel URL from mobile browser.

### Database Access

Expose local database for remote access:

```bash
bdy tunnel tcp localhost:5432 -n remote-db -w your-ip/32
```

Connect using tunnel host and port.

## Important Notes

### Application Binding

Ensure applications bind to `0.0.0.0` (all interfaces), not `127.0.0.1` (localhost only):

```bash
# Good - accessible via tunnel
npm start --host 0.0.0.0

# Bad - not accessible via tunnel
npm start --host 127.0.0.1
```

### Tunnels Run in Foreground

Tunnels run in the foreground by default. To run in background, use configuration files or run in a separate terminal session.

### HTTPS Proxy

Buddy tunnels serve traffic via HTTPS. If your application generates absolute URLs, configure it to trust proxy headers.

### Connection Timeout

Tunnels have connection timeouts. Set timeout for long-running connections:

```bash
bdy tunnel http localhost:3000 --timeout 600  # 10 minutes
```

## Troubleshooting

### Connection Refused

**Symptom:** "Connection refused" when accessing tunnel

**Solutions:**
- Verify application is running on specified port
- Check application binds to `0.0.0.0`, not `127.0.0.1`
- Ensure firewall allows connections

### Tunnel Authentication Failed

**Symptom:** "Authentication failed" when starting tunnel

**Solutions:**
- Verify Buddy CLI authentication (run `bdy workspace ls`)
- Check tunnel token is configured (see [bdy-auth](../bdy-auth/SKILL.md))
- Ensure account has tunnel permissions

### Whitelist Blocking Access

**Symptom:** "Access denied" from certain IPs

**Solutions:**
- Check whitelist configuration: `bdy tunnel config get whitelist`
- Update whitelist: `bdy tunnel config set whitelist "*"`
- Verify IP format uses CIDR notation

## Additional Resources

### Reference Files

For comprehensive command reference and examples:
- **[references/commands.md](references/commands.md)** - Complete command reference with all options
- **[references/examples.md](references/examples.md)** - Practical examples for HTTP/TCP/TLS tunnels

## Next Steps

After creating tunnels:
- Use webhooks with the public tunnel URL
- Share tunnel URL with team members or clients
- Test on mobile devices using the public endpoint
- Configure saved tunnels for frequently used services
