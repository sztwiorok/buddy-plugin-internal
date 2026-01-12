---
description: Expose localhost app via Buddy tunnel
argument-hint: [port]
allowed-tools: Bash(bdy:*,lsof:*), AskUserQuestion
model: sonnet
---

Create Buddy tunnel for locally running application. Use the **tunnel skill** for tunnel procedures and options.

**Arguments:**
- `$1`: Optional port number (auto-detect if not provided)

## Workflow

### 1. Verify Authentication

Check authentication with:
```bash
bdy workspace ls
```

If fails, inform user: "Run `bdy login` in a separate terminal (AI agents cannot perform interactive login). See **bdy-auth skill** for details."

### 2. Detect Running Service

If port not provided in `$1`, detect running services on common ports.

Use tools like `lsof` to find services:
```bash
lsof -i -P | grep LISTEN
```

If multiple services found, ask user which to expose.
If no service found, ask user for the port number.

### 3. Security Configuration (MANDATORY for HTTP)

**MANDATORY:** Before creating an HTTP tunnel, you MUST ask the user about authentication using AskUserQuestionTool.

DO NOT skip this step. DO NOT proceed until user has made a choice.

Ask:
- **Question:** "Do you want to protect this HTTP tunnel with authentication?"
- **Options:**
  1. "HTTP Basic Auth (username:password)" - Will use `-a username:password` flag
  2. "Buddy Authentication" - Will use `--buddy` flag (requires Buddy account)
  3. "No authentication (public access)" - Proceed without auth

### 4. Create Tunnel

Use the **tunnel skill** procedures to create appropriate tunnel type (HTTP/TCP/TLS).

Apply the authentication option chosen in step 3:
- HTTP Basic Auth: `bdy tunnel http localhost:PORT -a username:password`
- Buddy Auth: `bdy tunnel http localhost:PORT --buddy`
- No auth: `bdy tunnel http localhost:PORT`

The tunnel skill contains additional options:
- Regional endpoints
- IP whitelisting
- Named tunnels

### 5. Display Results

Show:
- Tunnel name and type
- Local endpoint (localhost:port)
- Public URL
- How to stop (Ctrl+C)
- How to save config for reuse

## Important Notes

- Applications must bind to `0.0.0.0`, not `127.0.0.1`
- Tunnels run in foreground by default
- Use `bdy tunnel config add` to save frequent configurations

## Error Handling

If tunnel fails:
- Check app is running on specified port
- Verify app binds to 0.0.0.0
- For auth errors: direct user to run `bdy login` in separate terminal
- Check logs and tunnel output for specific errors
