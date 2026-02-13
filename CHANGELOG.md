# Changelog

All notable changes to the TailScale_deploy project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-02-13

### Added
- **Initial release** of modular Tailscale deployment playbook
- **Automated installation** using official Tailscale installation script
- **Modular task structure**:
  - `install.yml` - Handles Tailscale installation and service setup
  - `configure.yml` - Manages authentication and connection to Tailnet
  - `status.yml` - Verifies connection and displays comprehensive status
- **Smart hostname generation**:
  - Auto-generates hostnames based on system (e.g., `raspberrypi-debian`, `myserver-ubuntu`)
  - Supports custom hostname via `tailscale_hostname` variable
- **Idempotent operations**:
  - Checks if Tailscale is already installed before installing
  - Checks if already connected before attempting connection
  - Safe to run multiple times without side effects
- **Comprehensive status reporting**:
  - Displays system hostname and IP address
  - Shows Tailscale hostname and IP (100.x.x.x)
  - Lists all connected peers in the Tailnet
  - Verifies successful connection
- **Security features**:
  - Auth keys are never logged (using `no_log: true`)
  - Supports integration with Semaphore environment variables
  - Secure handling of sensitive credentials
- **Customizable configuration**:
  - `tailscale_accept_routes` - Accept subnet routes from other nodes
  - `tailscale_accept_dns` - Accept DNS configuration from Tailscale
  - `tailscale_extra_flags` - Pass additional flags to `tailscale up`
- **Debian family support**:
  - Ubuntu (all versions)
  - Debian (all versions)
  - Raspberry Pi OS
  - Other Debian-based distributions
- **Service management**:
  - Automatically enables `tailscaled` service
  - Ensures service is running before configuration
- **Comprehensive documentation**:
  - Detailed README with usage examples
  - Troubleshooting guide
  - Security best practices
  - Semaphore integration instructions

### Features
- One-command Tailscale deployment
- Automatic service configuration
- Connection verification
- Peer discovery and listing
- Customizable hostnames
- Zero manual intervention required

### Technical Details
- Ansible role-based structure
- Modular task organization
- Variable validation and assertion
- Error handling and status checks
- Clean installation script management

### Requirements
- Ansible 2.9 or higher
- Debian-based target systems
- Sudo/root access
- Tailscale auth key (from https://login.tailscale.com/admin/settings/keys)

### Configuration
Required environment variables (set in Semaphore UI):
- `tailscale_auth_key` - Tailscale authentication key (required)
- `tailscale_api_key` - Tailscale API key (optional, reserved for future use)

Optional variables:
- `tailscale_hostname` - Custom hostname for the node
- `tailscale_accept_routes` - Accept subnet routes (default: false)
- `tailscale_accept_dns` - Accept DNS config (default: true)
- `tailscale_extra_flags` - Additional CLI flags

### Files Added
- `main.yml` - Main playbook entry point
- `roles/tailscale_deploy/defaults/main.yml` - Default variables
- `roles/tailscale_deploy/tasks/main.yml` - Main task orchestration
- `roles/tailscale_deploy/tasks/install.yml` - Installation tasks
- `roles/tailscale_deploy/tasks/configure.yml` - Configuration tasks
- `roles/tailscale_deploy/tasks/status.yml` - Status verification tasks
- `roles/tailscale_deploy/vars/main.yml` - Variable documentation
- `README.md` - Comprehensive documentation
- `CHANGELOG.md` - This file

---
