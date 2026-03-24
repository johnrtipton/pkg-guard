# pkg-guard

Local registry proxy that blocks too-young package versions from being installed.

Protects against supply-chain attacks by enforcing a minimum age threshold on package versions before they can be installed via pip, uv, or npm. Runs as a system-wide proxy — no shell wrappers, no env vars, works in cron jobs and spawned processes.

## How it works

pkg-guard sits between your package managers and upstream registries (PyPI, npm). When a package version is requested:

1. If the version was published fewer than N days ago (default: 5), it's stripped from the response
2. The package manager sees only older, vetted versions and installs the newest one that passes
3. If the best fallback version has known vulnerabilities (via OSV.dev), the age gate is bypassed to allow security updates through

## Install

```bash
# Copy the script
cp pkg-guard ~/.local/bin/pkg-guard
chmod +x ~/.local/bin/pkg-guard

# Copy config
mkdir -p ~/.config/pkg-guard
cp config.example.toml ~/.config/pkg-guard/config.toml

# Start the proxy (installs as macOS launchd service)
pkg-guard start

# Configure pip, uv, and npm to use the proxy
pkg-guard setup
```

## Usage

```bash
pkg-guard status         # Show proxy status and client config
pkg-guard log            # Tail the decision log
pkg-guard check <pkg>    # Check a PyPI package's version age
pkg-guard allow <pkg>    # Allowlist a package (skip age check)
pkg-guard deny <pkg>     # Remove from allowlist
pkg-guard stop           # Stop the proxy
pkg-guard unsetup        # Remove pip/uv/npm proxy config
```

## Configuration

Edit `~/.config/pkg-guard/config.toml`:

```toml
[server]
host = "127.0.0.1"
port = 8418

[guard]
min_age_days = 5      # Minimum package version age in days
check_vulns = true    # Bypass age gate if fallback has known vulns

[paths]
allowlist = "~/.config/pkg-guard/allowlist.txt"
log = "~/.local/share/pkg-guard/guard.log"
```

## How client config works

`pkg-guard setup` writes to system-wide config files so all invocations are covered:

- **pip**: `~/.config/pip/pip.conf` → `index-url`
- **uv**: `~/.config/uv/uv.toml` → `index-url`
- **npm**: `~/.npmrc` → `registry`

This means cron jobs, CI scripts, and subprocess-spawned installs all go through the proxy without needing environment variables.

## Requirements

- Python 3.11+ (uses `tomllib` from stdlib)
- macOS (launchd for service management)
- No external dependencies — stdlib only

## License

MIT
