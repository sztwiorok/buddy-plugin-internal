# Command Reference

## Table of Contents
- [Lifecycle Commands](#lifecycle-commands)
- [Execution Commands](#execution-commands)
- [File Transfer](#file-transfer)
- [Endpoint Management](#endpoint-management)
- [Snapshot Management](#snapshot-management)
- [Command Management](#command-management)

## Lifecycle Commands

```bash
bdy sandbox create -i <identifier>     # Create sandbox
bdy sandbox list                       # List sandboxes (alias: ls)
bdy sandbox get <identifier>           # Get details
bdy sandbox status <identifier>        # Get status
bdy sandbox start <identifier>         # Start stopped sandbox
bdy sandbox stop <identifier>          # Stop running sandbox
bdy sandbox restart <identifier>       # Restart sandbox
bdy sandbox destroy <identifier>       # Delete sandbox (alias: rm)
```

### Create Options

| Option | Description |
|--------|-------------|
| `-i, --identifier <id>` | Unique identifier |
| `-n, --name <name>` | Display name |
| `--os <image>` | OS image (default: ubuntu:24.04) |
| `--resources <spec>` | Resources: 1x2, 2x4, 4x8, 8x16, 12x24 (CPUxRAM) |
| `--install-commands <cmd>` | Setup commands (repeatable) |
| `--run-command <cmd>` | Startup command |
| `--snapshot <name>` | Create from snapshot |

## Execution Commands

```bash
bdy sandbox exec <identifier> "<command>"                  # Execute command
bdy sandbox exec <identifier> -d "<command>"               # Run detached
bdy sandbox exec <identifier> --runtime PYTHON "<code>"    # Different runtime
```

### Exec Options

| Option | Description |
|--------|-------------|
| `--runtime <type>` | BASH (default), JAVASCRIPT, TYPESCRIPT, PYTHON |
| `-d, --detached` | Run in background |

## File Transfer

```bash
bdy sandbox cp <source> <identifier>:<dest>         # Copy to sandbox
bdy sandbox cp ./src my-app:/app/src                # Copy directory
bdy sandbox cp --silent ./file my-app:/app/file     # Silent mode (recommended)
```

**Important:** Always use `--silent` to suppress progress output.

## Endpoint Management

```bash
bdy sandbox endpoint list <identifier>                             # List (alias: ep list)
bdy sandbox endpoint get <identifier> <name>                       # Get details
bdy sandbox endpoint add <identifier> -n <name> --port <port>      # Add
bdy sandbox endpoint update <identifier> <name> --port <new-port>  # Update
bdy sandbox endpoint delete <identifier> <name>                    # Delete
```

### Endpoint Options

| Option | Description |
|--------|-------------|
| `-n, --name <name>` | Endpoint name (required) |
| `--port <port>` | Port number (required) |
| `-t, --type <type>` | HTTP, TLS, or TCP (default: HTTP) |
| `--auth <type>` | NONE, BASIC, or BUDDY |
| `--username/--password` | For BASIC auth |
| `--whitelist <cidr>` | IP whitelist |

## Snapshot Management

```bash
bdy sandbox snapshot list <identifier>              # List (alias: snap list)
bdy sandbox snapshot get <identifier> <name>        # Get details
bdy sandbox snapshot create <identifier> -n <name>  # Create
bdy sandbox snapshot delete <identifier> <name>     # Delete
```

## Command Management

```bash
bdy sandbox command list <identifier>               # List all (alias: cmd ls)
bdy sandbox command status <identifier> <cmd-id>    # Get status
bdy sandbox command logs <identifier> <cmd-id>      # View logs
bdy sandbox command logs <identifier> --last        # Most recent logs
bdy sandbox command logs <identifier> <cmd-id> -f   # Stream logs
bdy sandbox command kill <identifier> <cmd-id>      # Kill command
```

**Tip:** Use `command list` to discover command IDs, then `logs --last` to check output.
