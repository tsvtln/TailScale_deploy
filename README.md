# TailScale_deploy
Ansible playbook to deploy and configure Tailscale on Debian-based systems

## Overview

This modular Ansible playbook automates the installation and configuration of Tailscale VPN on Debian-based systems (Ubuntu, Debian, Raspberry Pi OS, etc.).

## Features

**Automatic Installation** - Downloads and installs Tailscale using the official installation script  
**Authentication** - Connects to your Tailnet using an auth key  
**Custom Hostnames** - Set predictable hostnames for your nodes  
**Idempotent** - Safe to run multiple times  
**Status Verification** - Displays connection status and peer information  
**Modular Design** - Separated tasks for installation, configuration, and status checking

## Prerequisites

- Ansible 2.9 or higher
- Target systems: Debian-based distributions (Ubuntu, Debian, Raspberry Pi OS)
- Tailscale account with an auth key
- Sudo/root access on target systems

## Semaphore Configuration

### Required Environment Variables

Set these in **Semaphore UI → Environment → Variables**:

| Variable Name        | Description                                                                            | Required |
|----------------------|----------------------------------------------------------------------------------------|----------|
| `tailscale_auth_key` | Your Tailscale authentication key from https://login.tailscale.com/admin/settings/keys | **Yes**  |
| `tailscale_api_key`  | Your Tailscale API key (for future API operations)                                     | No       |

### Optional Variables

You can set these as **extra variables** in Semaphore:

| Variable Name             | Description                                    | Default                                             |
|---------------------------|------------------------------------------------|-----------------------------------------------------|
| `tailscale_hostname`      | Custom hostname for the Tailscale node         | `{{ ansible_hostname }}-{{ ansible_distribution }}` |
| `tailscale_accept_routes` | Accept subnet routes advertised by other nodes | `false`                                             |
| `tailscale_accept_dns`    | Accept DNS configuration from Tailscale        | `true`                                              |
| `tailscale_extra_flags`   | Additional flags to pass to `tailscale up`     | `""`                                                |

## Usage

### Running in Semaphore

1. **Create a new Template** in Semaphore
2. **Select this repository** as the playbook repository
3. **Set Environment variables** (see above)
4. **Select target hosts** from your inventory
5. **Run the template**

### Manual Execution

```bash
ansible-playbook -i inventory main.yml \
  -e "tailscale_auth_key=tskey-auth-xxxxx" \
  -e "tailscale_hostname=my-custom-hostname"
```

## What the Playbook Does

### 1. **Installation (`install.yml`)**
- Checks if Tailscale is already installed
- Downloads the official Tailscale installation script from `https://tailscale.com/install.sh`
- Executes the installation script
- Enables and starts the `tailscaled` service

### 2. **Configuration (`configure.yml`)**
- Determines the hostname (custom or auto-generated)
- Checks if Tailscale is already connected
- Connects to your Tailnet using: `tailscale up --authkey=<key> --hostname=<name>`
- Applies additional configuration flags

### 3. **Status Verification (`status.yml`)**
- Waits for connection to establish
- Retrieves Tailscale IP address (10.x.x.x)
- Displays connection status and peer list
- Verifies successful connection

## Example Output

```
============================================
TAILSCALE DEPLOYMENT SUMMARY
============================================
Hostname: raspberrypi
System IP: 1.2.3.4
Tailscale Hostname: raspberrypi-debian
Tailscale IP: 10.20.30.40
Status: Running

Connected Peers:
raspberrypi-debian       10.20.30.40  linux   active; direct 1.2.3.4:12345
mediaserver              10.20.30.41  linux   active; relay "fra"
laptop                   10.20.30.42  linux   idle
============================================
```

## Hostname Behavior

The playbook automatically generates hostnames based on the system:

- **Ubuntu**: `hostname-ubuntu` (e.g., `myserver-ubuntu`)
- **Debian**: `hostname-debian` (e.g., `raspberrypi-debian`)
- **Custom**: Set `tailscale_hostname` variable to override

Examples:
```yaml
# Auto-generated (default)
# raspberrypi-debian

# Custom hostname
tailscale_hostname: "raspberrypi-trixie"
# Result: raspberrypi-trixie

# Custom hostname via Semaphore extra variables
# Add to template: tailscale_hostname=my-production-server
```

## Directory Structure

```
TailScale_deploy/
├── main.yml                          # Main playbook entry point
├── README.md                         # This file
└── roles/
    └── tailscale_deploy/
        ├── defaults/
        │   └── main.yml              # Default variables
        ├── tasks/
        │   ├── main.yml              # Main task orchestration
        │   ├── install.yml           # Installation tasks
        │   ├── configure.yml         # Configuration tasks
        │   └── status.yml            # Status verification tasks
        └── vars/
            └── main.yml              # Variable documentation
```

## Troubleshooting

### Connection Issues

If Tailscale doesn't connect:

1. **Check auth key**: Ensure `tailscale_auth_key` is valid and not expired
2. **Check service**: `sudo systemctl status tailscaled`
3. **Check logs**: `sudo journalctl -u tailscaled -f`
4. **Manual test**: `sudo tailscale up` to see authentication URL

### Firewall Issues

Tailscale uses UDP port 41641 by default. Ensure:
- Outbound traffic to Tailscale servers is allowed
- UDP hole punching is not blocked by your firewall

### Re-authentication

If you need to re-authenticate or change settings:
```bash
sudo tailscale down
sudo tailscale up --authkey=<new-key> --hostname=<new-name>
```

## Security Notes

- Auth keys are not logged (using `no_log: true`)
- Use **ephemeral keys** for automated deployments
- Use **reusable keys** only for testing
- Store keys securely in Semaphore environment variables
- Rotate auth keys regularly

## Advanced Configuration

### Accept Subnet Routes

To accept subnet routes advertised by exit nodes or app connectors:

```yaml
tailscale_accept_routes: true
```

### Additional Flags

Pass additional flags to `tailscale up`:

```yaml
tailscale_extra_flags: "--ssh --advertise-tags=tag:production"
```

### API Operations

The `tailscale_api_key` variable is reserved for future API operations (e.g., managing ACLs, listing devices, etc.).

## Related Documentation

- [Tailscale Documentation](https://tailscale.com/kb/)
- [Tailscale Auth Keys](https://tailscale.com/kb/1085/auth-keys/)
- [Tailscale CLI Reference](https://tailscale.com/kb/1080/cli/)

---

All Rights Reserved.

This source code is provided for viewing purposes only.
You may not copy, modify, distribute, sublicense, or use
this code for commercial or non-commercial purposes without
explicit written permission from the author.

