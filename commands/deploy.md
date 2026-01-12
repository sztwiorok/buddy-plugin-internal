---
description: Deploy app to Buddy Sandbox from current directory
argument-hint: [sandbox-name] [path-to-deploy]
allowed-tools: Bash(bdy:*), Read, Glob, AskUserQuestion
model: sonnet
---

Deploy application to Buddy Sandbox.

**Arguments:**
- `$1`: Optional sandbox name
- `$2`: Optional path to deploy (default: current directory)

## Workflow

1. **Analyze project:** Understand what to deploy, dependencies, start command, port

2. **Deploy:** Follow the **sandbox skill** for deployment procedures

3. **Show results:** Sandbox ID, public URL, how to view logs
