# Substrate Node Update Automation

An Ansible-based automation toolkit for deploying and updating Substrate-based blockchain nodes. This project provides a streamlined way to manage node binaries across multiple environments with support for automated GitHub release deployments.

## Features

- Automated binary deployment from GitHub releases
- Support for both authenticated and unauthenticated GitHub access
- Configurable deployment paths and permissions
- Chain specification file management
- Systemd service management
- Post-deployment health checks
- Rolling updates with customizable delay
- Multiple environment support (relay chain, system chain, etc.)

## Prerequisites

- Ansible 2.9 or higher
- Target systems running systemd
- GitHub access (token optional but recommended)
- SSH access to target hosts

## Repository Structure

```
.
├── inventory/               # Inventory configuration
│   ├── avail/              # Avail network specific inventory
│   ├── bifrost/            # Bifrost network specific inventory
│   ├── encointer/          # Encointer network specific inventory
│   ├── relay_chain/        # Relay chain specific inventory
│   ├── system_chain/       # System chain specific inventory
│   └── group_vars/         # Shared group variables
├── playbooks/              # Ansible playbooks
│   ├── apt-update-all.yml  # System updates playbook
│   ├── check-version.yml   # Version verification
│   ├── deploy_binary.yml   # Main deployment playbook
│   └── restart-service.yml # Service management
└── ansible.cfg            # Ansible configuration
```

## Configuration

### Inventory Setup

Create or modify inventory files under the appropriate network directory:

```yaml
# inventory/[network]/hosts
[validators]
validator1 ansible_host=10.0.0.1
validator2 ansible_host=10.0.0.2

[full_nodes]
node1 ansible_host=10.0.0.3
```

### Group Variables

Configure group variables in `inventory/[network]/group_vars/all.yml`:

```yaml
---
# GitHub repository details
repo_org: "your-org"
repo_name: "your-repo"

# File ownership
file_owner: "node"
file_group: "node"

# Service configuration
service_name: "substrate-node"

# Binary configuration
binary:
  - binary: "node-binary.tar.gz"
    dest_path: "/usr/local/bin/"
```

## Usage

### Basic Deployment

Deploy the latest release:

```bash
ansible-playbook playbooks/deploy_binary.yml -i inventory/relay_chain/hosts
```

### Specific Version Deployment

Deploy a specific version:

```bash
ansible-playbook playbooks/deploy_binary.yml -i inventory/relay_chain/hosts -e "application_version=v1.2.3"
```

### System Updates

Update system packages:

```bash
ansible-playbook playbooks/apt-update-all.yml -i inventory/relay_chain/hosts
```

### Service Management

Restart services:

```bash
ansible-playbook playbooks/restart-service.yml -i inventory/relay_chain/hosts
```

## Playbook Details

### deploy_binary.yml

The main deployment playbook handles:

1. GitHub release version detection
2. Binary download and verification
3. Installation and permissions setting
4. Service restart
5. Health checks

Key features:

- Serial execution for rolling updates
- Configurable delay between node updates
- Automatic service restart
- Post-deployment health verification
- Support for compressed (\*.gz) binaries

### check-version.yml

Verifies installed binary versions across nodes:

- Checks current binary version
- Reports version mismatches
- Supports version comparison

### restart-service.yml

Manages service operations:

- Graceful service restart
- Status verification
- Failure handling

## Security Considerations

1. GitHub Token Usage

   - Store GitHub tokens securely using Ansible Vault
   - Use minimal required permissions for the token

2. File Permissions

   - Binaries are set with restricted permissions
   - Service runs under dedicated user account

3. SSH Access
   - Use SSH keys for authentication
   - Implement proper key rotation policies

## Troubleshooting

### Common Issues

1. GitHub Rate Limiting

   ```
   Solution: Configure GitHub token for authenticated requests
   ```

2. Service Fails to Start

   ```
   Check: systemctl status [service_name]
   Check: journalctl -u [service_name] -n 100
   ```

3. Binary Deployment Fails
   ```
   Verify: Network connectivity
   Verify: Disk space
   Verify: File permissions
   ```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
