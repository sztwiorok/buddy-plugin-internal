---
name: bdy-sandbox
description: Deploy and test applications in Buddy Sandbox cloud environments. Use when user asks about deploying apps, testing in isolated environments, sandboxes, remote environments, exposing apps via public URL, or running apps somewhere accessible.
---

# Buddy Sandbox Deployment

On-demand cloud environments for deploying and testing applications with public HTTP/TCP endpoints.

## Prerequisites

```bash
# Interactive login (recommended)
bdy login

# Or environment variables
export BDY_API_TOKEN="your-token"
export BDY_WORKSPACE="your-workspace"
export BDY_PROJECT="your-project"
```

## Quick Deployment Workflow

### 1. Create Sandbox

```bash
bdy sandbox create -i my-app --resources 2x4 \
  --install-commands "apt-get update && apt-get install -y nodejs npm"
```

Resources: 1x2, 2x4, 4x8, 8x16, 12x24 (CPUxRAM). Default OS: Ubuntu 24.04.

### 2. Deploy Application

**Copy local files (recommended for AI agents):**
```bash
bdy sandbox cp --silent ./src my-app:/app
bdy sandbox exec my-app "cd /app && npm install"
bdy sandbox exec my-app -d "cd /app && npm start"
```

**From Git:**
```bash
bdy sandbox exec my-app "git clone https://github.com/user/repo.git /app"
bdy sandbox exec my-app "cd /app && npm install"
bdy sandbox exec my-app -d "cd /app && npm start"
```

Use `-d` flag to run in background (detached).

### 3. Expose Endpoint

```bash
bdy sandbox endpoint add my-app -n web --port 3000
```

With auth: `--auth BASIC --username admin --password secret`

### 4. Check Status

```bash
bdy sandbox endpoint list my-app        # Get public URL
bdy sandbox command logs my-app --last  # View recent logs
```

## Important Notes

**HTTPS proxy:** Endpoints are served via HTTPS. Ensure your app:
- Binds to `0.0.0.0` (not localhost)
- Trusts proxy headers if generating absolute URLs

**Python on Ubuntu 24.04:** Use venv for pip packages (PEP 668):
```bash
python3 -m venv venv && . venv/bin/activate && pip install -r requirements.txt
```

**Silent mode:** Always use `--silent` with `cp` to keep stdout clean.

## References

- **Full command reference:** See [references/commands.md](references/commands.md)
- **Complete examples:** See [references/examples.md](references/examples.md)
