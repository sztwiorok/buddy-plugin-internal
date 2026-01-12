---
name: sandbox
description: Deploy and test applications in Buddy Sandbox cloud environments. Use when user asks about "deploy app", "create sandbox", "test in cloud", "isolated environment", "remote environment", "run app in sandbox", or mentions deploying, testing, or running applications in cloud sandboxes.
---

# Buddy Sandbox Deployment

On-demand cloud environments for deploying and testing applications with public HTTP/TCP endpoints.

## CRITICAL: AI Agent Requirements

> **STOP: Read this section before any sandbox operation.**

### 1. Always use `--silent` with `bdy sandbox cp`

Without `--silent`, file copy floods stdout and breaks your execution:
```bash
bdy sandbox cp --silent ./src my-app:/app    # correct
bdy sandbox cp ./src my-app:/app             # WRONG - floods output
```

### 2. Run apps in detached mode (`-d`)

Use `-d` flag for long-running processes, otherwise command blocks:
```bash
bdy sandbox exec my-app -d "npm start"       # correct - runs in background
bdy sandbox exec my-app "npm start"          # WRONG - blocks execution
```

### 3. Apps must bind to `0.0.0.0`

Applications MUST bind to `0.0.0.0`, NOT `127.0.0.1` or `localhost`.

### 4. Python on Ubuntu 24.04 requires venv

PEP 668 enforced - pip install fails without venv:
```bash
python3 -m venv venv && . venv/bin/activate && pip install -r requirements.txt
```

## Prerequisites

**Authentication Required:** Verify with `bdy workspace ls`. If fails, user must run `bdy login` in separate terminal.

## Quick Deployment Workflow

### 1. Create Sandbox

```bash
bdy sandbox create -i my-app --resources 2x4 \
  --install-commands "apt-get update && apt-get install -y nodejs npm"
```

Resources: 1x2, 2x4, 4x8, 8x16, 12x24 (CPUxRAM). Default OS: Ubuntu 24.04.

### 2. Deploy Application

```bash
bdy sandbox cp --silent ./src my-app:/app
bdy sandbox exec my-app "cd /app && npm install"
bdy sandbox exec my-app -d "cd /app && npm start"
```

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

## References

- **[references/commands.md](references/commands.md)** - Full command reference
- **[references/examples.md](references/examples.md)** - Complete deployment examples
