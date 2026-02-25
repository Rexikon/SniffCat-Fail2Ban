<div align="center">

# SniffCat fail2ban Integration

**Automatically report banned IPs from fail2ban to [SniffCat](https://sniffcat.com)**

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![fail2ban](https://img.shields.io/badge/fail2ban-Compatible-red.svg)](https://github.com/fail2ban/fail2ban)
[![SniffCat](https://img.shields.io/badge/SniffCat-API%20v1-green.svg)](https://sniffcat.com)

</div>

---

## Overview

SniffCat-fail2ban integrates [fail2ban](https://github.com/fail2ban/fail2ban) with the [SniffCat](https://sniffcat.com) threat intelligence API. When fail2ban bans an IP address, this integration automatically reports it to SniffCat, contributing to a shared threat intelligence database.

Works with **any fail2ban jail** — SSH, Apache, Nginx, Postfix, Dovecot, and more.

### How It Works

```
Attacker → fail2ban detects abuse → sniffcat action triggered → IP reported to SniffCat API
```

1. **fail2ban** detects repeated failed attempts and bans the IP
2. **fail2ban** executes the `sniffcat` action alongside the default ban action
3. **fail2ban.sh** sends a report to the SniffCat API with the attacker's IP and jail metadata

The `sniffcat` action works **alongside** your existing ban actions — IPs are still banned normally via iptables/nftables AND reported to SniffCat.

## Requirements

- Linux server with **fail2ban** installed and running
- `curl` installed on the server
- **Root** access
- SniffCat API token — [get one here](https://sniffcat.com)

## Quick Start

```bash
sudo bash <(curl -fsSL https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main/install.sh)
```

The installer will interactively ask for your SniffCat API token and handle everything else.

## Installation

### Automatic (recommended)

Using **curl**:

```bash
sudo bash <(curl -fsSL https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main/install.sh)
```

Using **wget**:

```bash
sudo bash <(wget -qO- https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main/install.sh)
```

The installer will:

- Verify root access, dependencies, and fail2ban presence
- Ask for your SniffCat API token
- Install the reporting script to `/opt/sniffcat/`
- Install the fail2ban action to `/etc/fail2ban/action.d/`
- Create a secure config file (`chmod 600`)
- Set up error logging to `/var/log/sniffcat.log`
- Display the jail configuration instructions

### Manual

```bash
# Create installation directory
mkdir -p /opt/sniffcat

# Download the reporting script
curl -fsSL https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main/fail2ban.sh \
    -o /opt/sniffcat/fail2ban.sh
chmod 755 /opt/sniffcat/fail2ban.sh

# Download the fail2ban action
curl -fsSL https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main/sniffcat-action.conf \
    -o /etc/fail2ban/action.d/sniffcat.conf
chmod 644 /etc/fail2ban/action.d/sniffcat.conf

# Create config file with your token
cat > /opt/sniffcat/sniffcat.conf <<EOF
SNIFFCAT_TOKEN="your-token-here"
EOF
chmod 600 /opt/sniffcat/sniffcat.conf

# Create log file
touch /var/log/sniffcat.log
chmod 640 /var/log/sniffcat.log
```

## fail2ban Configuration

After installation, add the `sniffcat` action to your jails.

### Option A — Specific jail

Edit `/etc/fail2ban/jail.local` and add `sniffcat` to the desired jail:

```ini
[sshd]
enabled = true
action = %(action_)s
         sniffcat
```

### Option B — All jails globally

Add `sniffcat` to the `[DEFAULT]` section in `/etc/fail2ban/jail.local`:

```ini
[DEFAULT]
action = %(action_)s
         sniffcat
```

Then restart fail2ban:

```bash
sudo systemctl restart fail2ban
```

### Verify it works

Check that the action is loaded:

```bash
sudo fail2ban-client get sshd actions
```

You should see `sniffcat` listed among the actions.

### fail2ban Action Parameters

The action automatically passes these values from fail2ban to the reporting script:

| Parameter | fail2ban Tag  | Description                                  |
|-----------|---------------|----------------------------------------------|
| `$1`      | `<ip>`        | IP address being banned                      |
| `$2`      | `<name>`      | Jail name (e.g., `sshd`, `apache-auth`)      |
| `$3`      | `<failures>`  | Number of failed attempts that triggered ban  |

## File Structure

```
/opt/sniffcat/
├── fail2ban.sh        # Reporting script (755)
└── sniffcat.conf      # API token configuration (600)

/etc/fail2ban/action.d/
└── sniffcat.conf      # fail2ban action definition (644)

/var/log/
└── sniffcat.log       # Error log (640)
```

## Logs

Only errors are logged to `/var/log/sniffcat.log` — successful reports are silent:

```
2026-02-11 14:22:01 [SniffCat] ERROR: IP=198.51.100.23 jail=sshd failures=5 — HTTP 401: {"error":"invalid token"}
2026-02-11 15:03:44 [SniffCat] ERROR: Config file not found: /opt/sniffcat/sniffcat.conf
```

## Uninstallation

Using the uninstaller:

```bash
sudo bash <(curl -fsSL https://raw.githubusercontent.com/Rexikon/SniffCat-fail2ban/main/uninstall.sh)
```

Or manually:

```bash
rm -rf /opt/sniffcat
rm -f /etc/fail2ban/action.d/sniffcat.conf
rm -f /var/log/sniffcat.log
sudo systemctl restart fail2ban
```

> **Note:** Remember to remove the `sniffcat` action from your jails in `/etc/fail2ban/jail.local` after uninstalling.

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

## License

This project is licensed under the **GNU General Public License v3.0** — see the [LICENSE](LICENSE) file for details.
