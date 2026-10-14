# Enterprise Linux Patching - Ansible Project

A comprehensive, enterprise-grade Ansible solution for patching Linux servers with built-in validation, rollback capabilities, approval workflows, and compliance reporting.

## Features

- **Multi-Distribution Support**: Supports RHEL-based (RHEL, CentOS, AlmaLinux, Rocky) and Debian-based (Ubuntu, Debian) systems
- **Pre-Validation**: Comprehensive system health checks before patching
- **Flexible Patching Strategies**: Security patches, full updates, kernel-only, or custom package selection
- **Post-Validation**: Automated verification after patching
- **Rollback Capability**: Automatic and manual rollback options
- **Approval Workflow**: Multi-stage approval process for patching operations
- **Compliance Reporting**: Detailed audit logs and compliance reports (JSON, HTML)
- **Maintenance Window Support**: Schedule patching during defined maintenance windows
- **Enterprise Logging**: Comprehensive logging and audit trails

## Project Structure

```
enterprise_linux_patching/
├── ansible.cfg                      # Ansible configuration
├── README.md                        # This file
├── .gitignore                       # Git ignore rules
├── inventories/                     # Inventory files
│   ├── production/
│   │   └── hosts.yml               # Production inventory
│   ├── staging/
│   │   └── hosts.yml               # Staging inventory
│   └── testing/
│       └── hosts.yml               # Testing inventory
├── roles/                          # Ansible roles
│   ├── pre_validation/             # Pre-patching validation
│   │   ├── defaults/main.yml
│   │   └── tasks/main.yml
│   ├── patching/                   # Main patching role
│   │   ├── defaults/main.yml
│   │   └── tasks/main.yml
│   ├── post_validation/            # Post-patching validation
│   │   ├── defaults/main.yml
│   │   └── tasks/main.yml
│   └── rollback/                   # Rollback role
│       ├── defaults/main.yml
│       └── tasks/main.yml
├── playbooks/                      # Main playbooks
│   ├── patch_linux_servers.yml     # Main patching playbook
│   ├── approval/                   # Approval workflow
│   │   ├── request_approval.yml
│   │   └── approve_request.yml
│   ├── maintenance/                # Maintenance scheduling
│   │   └── schedule_maintenance.yml
│   └── generate_compliance_report.yml
├── group_vars/                     # Group variables
│   ├── all.yml                     # Global variables
│   ├── rhel_based.yml              # RHEL-specific variables
│   └── debian_based.yml            # Debian-specific variables
├── host_vars/                      # Host-specific variables
├── logs/                           # Execution logs
├── reports/                        # Compliance reports
├── requests/                       # Approval requests
└── maintenance/                   # Maintenance schedules
```

## Quick Start

### Prerequisites

- Ansible 2.9 or higher
- SSH access to target servers
- Sudo privileges on target servers
- Python 3 on control node and target servers

### Installation

1. Clone or copy the project directory:
```bash
cd enterprise_linux_patching
```

2. Configure your inventory:
```bash
# Edit inventories/production/hosts.yml with your server details
vim inventories/production/hosts.yml
```

3. Test connectivity:
```bash
ansible -i inventories/production/hosts.yml all -m ping
```

### Basic Usage

#### 1. Patch All Servers (No Approval Required)

```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml
```

#### 2. Patch Specific Group

```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml -e target_group=rhel_based
```

#### 3. With Approval Workflow

Request approval:
```bash
ansible-playbook playbooks/approval/request_approval.yml \
  -e target_group=linux_servers \
  -e strategy=security \
  -e justification="Monthly security patching"
```

Approve request:
```bash
ansible-playbook playbooks/approval/approve_request.yml -e request_id=<REQUEST_ID>
```

Execute patching:
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e target_group=linux_servers \
  -e patching_strategy=security
```

#### 4. Generate Compliance Report

```bash
ansible-playbook playbooks/generate_compliance_report.yml
```

#### 5. Schedule Maintenance Window

```bash
ansible-playbook playbooks/maintenance/schedule_maintenance.yml \
  -e target_group=linux_servers \
  -e start_time="2024-01-15 02:00" \
  -e end_time="2024-01-15 06:00" \
  -e justification="Quarterly security updates"
```

## Configuration

### Global Variables (group_vars/all.yml)

Configure global patching behavior:

```yaml
patching_strategy: security              # security, full, kernel, custom
patching_reboot_required: true
patching_reboot_timeout: 600
patching_enable_rollback: true

pre_validation_cpu_threshold: 80
pre_validation_memory_threshold: 80
pre_validation_disk_threshold: 90
pre_validation_require_backup: true
```

### Distribution-Specific Variables

RHEL-based (group_vars/rhel_based.yml):
```yaml
patching_dnf_options: "-y"
patching_update_kernel: true
patching_excluded_packages:
  - kernel-firmware
```

Debian-based (group_vars/debian_based.yml):
```yaml
patching_apt_options: "-y"
patching_update_kernel: true
patching_excluded_packages:
  - linux-firmware
```

### Host-Specific Variables

Override variables per host in host_vars/<hostname>.yml:

```yaml
patching_strategy: kernel
critical_services:
  - httpd
  - postgresql
maintenance_window:
  start: "02:00"
  end: "06:00"
```

## Roles

### Pre-Validation Role

Performs comprehensive checks before patching:
- System resource validation (CPU, memory, disk)
- Service status verification
- Maintenance window check
- Backup verification
- Filesystem mount validation

### Patching Role

Executes the actual patching:
- Detects OS family automatically
- Applies patches based on strategy
- Handles kernel updates
- Conditional reboot based on needs
- Logs all changes

### Post-Validation Role

Verifies system health after patching:
- Resource validation
- Service status checks
- Port connectivity verification
- Application health checks
- Log error detection
- Triggers rollback if thresholds exceeded

### Rollback Role

Reverts patching changes:
- Package history rollback
- Kernel version rollback
- Backup restoration
- Service restart
- System reboot if needed

## Patching Strategies

### Security Patches
Only installs security updates:
```bash
-e patching_strategy=security
```

### Full Updates
Updates all packages:
```bash
-e patching_strategy=full
```

### Kernel Only
Updates only kernel packages:
```bash
-e patching_strategy=kernel
```

### Custom Packages
Updates specific packages:
```bash
-e patching_strategy=custom
-e patching_included_packages=["nginx", "postgresql"]
```

## Approval Workflow

The approval workflow ensures proper authorization before patching:

1. **Request**: Create a patching request with justification
2. **Approve**: Authorized personnel review and approve
3. **Execute**: Run patching with approved request ID
4. **Report**: Generate compliance report

## Compliance and Auditing

Compliance reports include:
- Execution timestamps
- Validation results
- Patch changes applied
- Rollback status
- Approval chain
- Compliance standards met (ISO27001, NIST, SOC2)

## Best Practices

1. **Test in Staging First**: Always test patches in staging before production
2. **Use Maintenance Windows**: Schedule patching during low-traffic periods
3. **Maintain Backups**: Ensure recent backups before patching
4. **Monitor Resources**: Set appropriate thresholds for your environment
5. **Review Logs**: Check logs after each patching cycle
6. **Document Changes**: Keep records of all patching activities
7. **Rollback Plan**: Always have a rollback plan ready

## Troubleshooting

### Patching Fails on Pre-Validation
- Check resource thresholds in group_vars/all.yml
- Verify backup exists and is recent
- Ensure maintenance window is correct

### Reboot Loop After Kernel Update
- Use rollback role to revert kernel
- Check grub configuration
- Verify kernel package compatibility

### Service Not Starting After Patch
- Check service logs in /var/log/
- Verify configuration files weren't modified
- Use rollback if needed

### Inventory Not Recognized
- Verify YAML syntax in inventory files
- Check ansible.cfg inventory path
- Test with ansible-inventory command

## Security Considerations

- Use SSH keys for authentication
- Limit sudo privileges to required commands
- Encrypt sensitive variables with Ansible Vault
- Rotate credentials regularly
- Audit access to patching playbooks
- Keep Ansible and dependencies updated

## Contributing

When adding new features:
1. Update documentation
2. Test in staging environment
3. Maintain backward compatibility
4. Add validation where appropriate
5. Update compliance reports

## License

This project is provided as-is for enterprise use.

## Support

For issues or questions:
1. Check documentation
2. Review logs in ./logs/
3. Validate configuration
4. Test with dry-run mode
