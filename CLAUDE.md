# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an Ansible role for installing and configuring OpenVPN servers with certificate-based authentication. The role supports FreeBSD and Debian systems, handles CA/certificate management, and generates client configuration files.

## Key Commands

### Testing the Role

```bash
# Run tests against lab hosts
ansible-playbook tests/test.yml

# Test certificate revocation
ansible-playbook tests/test_revoke.yml

# Reset lab environment
ansible-playbook tests/reset_lab.yml
```

### Running Specific Tags

The role supports several tags for targeted execution:
- `install` - Install OpenVPN packages
- `configure` - Configure OpenVPN server
- `ccd` - Configure client-specific settings
- `update_crl` - Update certificate revocation list
- `client` - Generate client certificates and configs
- `pull` - Pull client configs to local machine
- `revoke` - Revoke client certificates

Example:
```bash
ansible-playbook playbook.yml --tags client,pull
```

### Revoking Client Certificates

To revoke client certificates, set the `openvpn_revoke_clients` variable with a list of CNs:

```yaml
openvpn_revoke_clients:
- cn: user1
  reason: key_compromise
- cn: user2
  reason: privilege_withdrawn
```

Then run the playbook with the `revoke` tag:
```bash
ansible-playbook playbook.yml --tags revoke
```

See `examples/revoke-certificates.yml` for a complete example.

## Architecture and Structure

### Role Entry Points
- `tasks/main.yml` - Main orchestrator that includes other task files based on tags
- OS-specific variables loaded from `vars/{{ ansible_os_family }}.yml`

### Key Task Files
- `tasks/install.yml` - Package installation (OS-specific)
- `tasks/configure.yml` - Server configuration and service management
- `tasks/server-keys.yml` - CA and server certificate generation
- `tasks/client-configs.yml` - Client certificate generation and config creation

### Important Variables
- `openvpn_folder` - Base directory for OpenVPN (OS-specific)
- `openvpn_key_folder` - Certificate storage (`{{ openvpn_folder }}/keys`)
- `openvpn_client_folder` - Client config storage (`{{ openvpn_folder }}/client`)
- `openvpn_clients` - List of client configurations with CN, validity, password, and optional static IP
- `openvpn_service_name` - Service name for systemd (default: `openvpn`, use `openvpn@instance` for template units)

### Certificate Management
The role uses the `community.crypto` collection for certificate operations:
- CA certificate with configurable expiry (default: 10 years)
- Server certificate with SAN entries
- Client certificates with optional password protection
- Certificate revocation support via CRL

### Client Configuration
- Generated configs include embedded certificates and keys
- Support for static IP assignment via CCD (client-config-dir)
- Configs are pulled to `{{ playbook_dir }}/{{ inventory_hostname }}/` by default
- Password-protected client keys supported with cipher auto-selection

### OS-Specific Configuration
- **FreeBSD**: 
  - Path: `/usr/local/etc/openvpn`
  - Service: `openvpn`
- **Debian**: 
  - Path: `/etc/openvpn`
  - Service: `openvpn@openvpn`
- **RedHat**: 
  - Path: `/etc/openvpn`
  - Service: `openvpn@server`