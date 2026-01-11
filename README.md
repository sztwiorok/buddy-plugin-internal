# Buddy Plugin for Claude Code

Claude Code plugin for seamless integration with Buddy.works. Deploy applications to Buddy Sandboxes and expose localhost services via Buddy tunnels directly from your AI coding sessions.

## Features

### Skills

- **sandbox** - Deploy and test applications in Buddy Sandbox cloud environments
- **tunnel** - Expose localhost services via public Buddy tunnels
- **bdy-auth** - Install and authenticate Buddy CLI

### Commands

- `/deploy [name] [path]` - Deploy application to Buddy Sandbox from current directory
- `/expose [port]` - Create Buddy tunnel for locally running application

## Installation

### Prerequisites

1. **Claude Code CLI** - Install from [claude.ai/claude-code](https://claude.ai/claude-code)
2. **Buddy CLI (bdy)** - Installed automatically via plugin or manually:
   ```bash
   npm install -g bdy
   ```
3. **Buddy Account** - Sign up at [buddy.works](https://buddy.works)

### Install Plugin

**Option 1: One-Time Usage (with --plugin-dir flag)**

Use plugin without permanent installation:

```bash
# Clone the repository
git clone https://github.com/sztwiorok/buddy-plugin.git buddy-plugin

# Use plugin for this session only
cd buddy-plugin
cc --plugin-dir .
```

This loads the plugin for current session. You need to use `--plugin-dir` flag every time.

**Option 2: Permanent Installation (recommended)**

Install plugin permanently:

```bash
# Clone directly to Claude plugins directory
git clone https://github.com/sztwiorok/buddy-plugin.git ~/.claude/plugins/buddy

# Plugin auto-loads in all sessions
cc
```

The plugin is now permanently available in all Claude Code sessions.

## Quick Start

### 1. Authenticate with Buddy

Before using the plugin, authenticate with Buddy CLI:

```bash
# Interactive login (recommended)
bdy login

# Or use token
bdy login --token YOUR_TOKEN -w workspace -p project
```

**Note:** Run `bdy login` in a separate terminal outside of Claude Code (interactive login requires browser).

### 2. Deploy an Application

Deploy your app to Buddy Sandbox:

```bash
cc
```

Then in Claude Code:
```
/deploy
```

Claude will:
- Analyze your project (Node.js, Python, Go, etc.)
- Create Buddy Sandbox
- Deploy and start your application
- Provide public URL

### 3. Expose Localhost

Expose your local dev server:

```bash
# Start your app
npm start  # Running on localhost:3000
```

Then in Claude Code:
```
/expose
```

Claude will:
- Detect running service on port 3000
- Create Buddy tunnel
- Provide public URL for testing webhooks, demos, etc.

## Usage Examples

### Deploy Node.js Application

```bash
cc
```

```
/deploy my-api
```

Claude deploys your Node.js app with:
- Auto-detected dependencies (npm install)
- Auto-detected start command
- Auto-detected port
- Public HTTPS endpoint

### Deploy Python Flask App

```bash
cc
```

```
/deploy flask-app ./backend
```

Claude deploys Flask app from `./backend` with:
- Python venv setup (Ubuntu 24.04 compliant)
- pip install requirements.txt
- Flask app started on 0.0.0.0
- Public endpoint

### Expose Local API for Webhook Testing

```bash
# Start your API
npm run dev

# In Claude Code
cc
```

```
/expose 3000
```

Use the provided public URL for webhook configuration (Stripe, GitHub, etc.).

### Deploy Multiple Apps from Monorepo

```bash
cc
```

```
# Deploy frontend
/deploy frontend ./apps/web

# Deploy backend
/deploy backend ./apps/api
```

Each gets its own Sandbox and public URL.

## Skills Reference

### sandbox Skill

Automatically loaded when discussing or working with Buddy Sandboxes.

**Triggers:** "deploy app", "create sandbox", "test in cloud", "isolated environment"

**Provides:**
- Complete sandbox lifecycle (create, deploy, exec, destroy)
- File transfer with `bdy sandbox cp --silent`
- Endpoint management
- Command execution and logs
- Resource configuration

### tunnel Skill

Automatically loaded when discussing or working with Buddy tunnels.

**Triggers:** "expose localhost", "create tunnel", "test webhooks", "share local server"

**Provides:**
- HTTP/TCP/TLS tunnel creation
- Security options (auth, whitelist)
- Configuration management
- Regional selection
- Use cases and examples

### bdy-auth Skill

Automatically loaded when discussing Buddy CLI setup.

**Triggers:** "install bdy", "login to Buddy", "authenticate with Buddy"

**Provides:**
- Installation instructions (NPM, Homebrew, APT, Chocolatey)
- Authentication methods (interactive, token, env vars)
- Workspace and project configuration
- Troubleshooting

## Commands Reference

### /deploy

Deploy application to Buddy Sandbox.

**Syntax:**
```
/deploy [sandbox-name] [path-to-deploy]
```

**Arguments:**
- `sandbox-name` - Optional custom name (default: auto-generated)
- `path-to-deploy` - Optional path to deploy (default: current directory)

**Examples:**
```bash
/deploy                        # Deploy current directory
/deploy my-app                 # Deploy with custom name
/deploy backend ./apps/api     # Deploy subdirectory
```

**What it does:**
1. Verifies Buddy authentication
2. Analyzes project to detect type, dependencies, port
3. Creates Buddy Sandbox with appropriate resources
4. Deploys application files
5. Installs dependencies and starts app
6. Exposes public HTTP endpoint
7. Provides public URL and management commands

### /expose

Create Buddy tunnel for localhost application.

**Syntax:**
```
/expose [port]
```

**Arguments:**
- `port` - Optional port number (auto-detected if not provided)

**Examples:**
```bash
/expose           # Auto-detect running service
/expose 3000      # Expose specific port
/expose 5432      # Expose database (TCP tunnel)
```

**What it does:**
1. Verifies Buddy authentication
2. Detects running services on common ports (or uses specified port)
3. Determines tunnel type (HTTP/TCP/TLS)
4. Creates Buddy tunnel
5. Provides public URL
6. Shows how to save configuration for reuse

## Configuration

### Plugin Settings

The plugin uses `.claude/settings.local.json` for permissions:

```json
{
  "permissions": {
    "allow": [
      "Bash(bdy:*)",
      "Bash(lsof:*)",
      "Skill(plugin-dev:skill-development)"
    ]
  }
}
```

### Buddy CLI Configuration

Configure Buddy CLI globally or per-project:

**Global configuration:**
```bash
bdy login -w workspace -p project --region eu
```

**Environment variables:**
```bash
export BUDDY_TOKEN="your-token"
export BUDDY_WORKSPACE="your-workspace"
export BUDDY_PROJECT="your-project"
export BUDDY_REGION="eu"
```

## Troubleshooting

### "Not authenticated with Buddy"

**Solution:** Run `bdy login` in a separate terminal:
```bash
bdy login
```

Credentials persist across all terminal sessions, including Claude Code.

### "Command not found: bdy"

**Solution:** Install Buddy CLI:
```bash
npm install -g bdy
```

Or see [skills/bdy-auth/references/installation.md](skills/bdy-auth/references/installation.md) for other methods.

### Application not accessible via public URL

**Solution:** Ensure app binds to `0.0.0.0`, not `127.0.0.1`:

**Node.js:**
```javascript
app.listen(PORT, '0.0.0.0');
```

**Python Flask:**
```python
app.run(host='0.0.0.0', port=5000)
```

**FastAPI:**
```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

### Sandbox deployment fails

**Check logs:**
```bash
bdy sandbox command logs [sandbox-id] --last
```

**Common issues:**
- Sandbox name already exists - use different name
- Insufficient resources - try `--resources 4x8`
- Missing dependencies - check install commands

### Tunnel connection refused

**Solutions:**
- Verify app is running: `lsof -i :[port]`
- Check app binds to 0.0.0.0
- Test locally first: `curl http://localhost:[port]`

## Advanced Usage

### Deploy with Custom Resources

```bash
cc
```

```
/deploy production-api --resources 8x16
```

Deploys with 8 CPU cores and 16 GB RAM.

### Create Authenticated Tunnel

Ask Claude:
```
/expose 3000 and add basic authentication
```

Claude will add `--auth` flag to the tunnel.

### Multi-Region Deployment

Ask Claude:
```
Deploy this app to US region sandbox
```

Claude will use `--region us` when creating sandbox.

### Save Tunnel Configuration

After creating tunnel, ask Claude:
```
Save this tunnel configuration for later use
```

Claude will run:
```bash
bdy tunnel config add http [name] http://localhost:[port]
```

## Documentation

### Skills Documentation

- **[sandbox skill](skills/sandbox/SKILL.md)** - Sandbox deployment guide
- **[tunnel skill](skills/tunnel/SKILL.md)** - Tunnel creation guide
- **[bdy-auth skill](skills/bdy-auth/SKILL.md)** - CLI installation and authentication

### Reference Files

- **[Sandbox commands](skills/sandbox/references/commands.md)** - Complete sandbox CLI reference
- **[Sandbox examples](skills/sandbox/references/examples.md)** - Deployment examples
- **[Tunnel commands](skills/tunnel/references/commands.md)** - Complete tunnel CLI reference
- **[Tunnel examples](skills/tunnel/references/examples.md)** - HTTP/TCP/TLS examples
- **[Installation guide](skills/bdy-auth/references/installation.md)** - Platform-specific installation
- **[Authentication guide](skills/bdy-auth/references/authentication.md)** - Auth methods and tokens

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with Claude Code
5. Submit a pull request

## License

MIT License - See LICENSE file for details

## Support

- **Issues:** [GitHub Issues](https://github.com/sztwiorok/buddy-plugin/issues)
- **Buddy Documentation:** [buddy.works/docs](https://buddy.works/docs)
- **Claude Code:** [claude.ai/claude-code](https://claude.ai/claude-code)

## Author

Rafal Sztwiorok

---

**Built for Claude Code** - Empowering AI-assisted development with Buddy.works integration
