# Quick Start Guide

This guide will help you get started with the Enterprise Linux Patching solution in minutes.

## Step 1: Initial Setup

### Clone or Navigate to Project
```bash
cd enterprise_linux_patching
```

### Create Required Directories
```bash
mkdir -p logs reports requests maintenance
```

## Step 2: Configure Inventory

Edit the production inventory with your server details:

```bash
vim inventories/production/hosts.yml
```

Replace the example IPs and SSH keys with your actual server information:

```yaml
rhel-server-01:
  ansible_host: YOUR_SERVER_IP
  ansible_user: YOUR_USERNAME
  ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

## Step 3: Test Connectivity

```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/test_connectivity.yml
```

This will:
- Ping all servers
- Gather system information
- Verify sudo access
- Display OS details

## Step 4: Run Dry-Run Test

Before actual patching, run a dry-run to see what would happen:

```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers_dryrun.yml
```

This performs all validations without making any changes.

## Step 5: Execute Patching

### Option A: Simple Patching (No Approval)

```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml
```

### Option B: Patching with Approval

1. Request approval:
```bash
ansible-playbook playbooks/approval/request_approval.yml \
  -e target_group=rhel_based \
  -e strategy=security \
  -e justification="Monthly security updates"
```

2. Note the Request ID from the output

3. Approve the request:
```bash
ansible-playbook playbooks/approval/approve_request.yml -e request_id=<REQUEST_ID>
```

4. Execute patching:
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e target_group=rhel_based \
  -e patching_strategy=security
```

## Step 6: Generate Compliance Report

```bash
ansible-playbook playbooks/generate_compliance_report.yml
```

This generates:
- JSON report in `./reports/`
- HTML report in `./reports/`
- Audit log in `./reports/`

## Common Scenarios

### Patch Only High-Criticality Servers
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e target_group=high_criticality
```

### Patch with Custom Strategy
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e patching_strategy=kernel \
  -e patching_reboot_required=true
```

### Patch Specific Server
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e target_group=rhel-server-01 \
  --limit rhel-server-01
```

### Schedule Maintenance Window
```bash
ansible-playbook playbooks/maintenance/schedule_maintenance.yml \
  -e target_group=linux_servers \
  -e start_time="2024-01-15 02:00" \
  -e end_time="2024-01-15 06:00" \
  -e justification="Quarterly security updates"
```

## Troubleshooting

### SSH Connection Failed
- Verify IP addresses in inventory
- Check SSH key permissions: `chmod 600 ~/.ssh/id_rsa`
- Test SSH manually: `ssh user@server`

### Sudo Permission Denied
- Ensure user has sudo privileges
- Check sudoers configuration
- Verify passwordless sudo: `sudo -v`

### Pre-Validation Failed
- Check resource thresholds in `group_vars/all.yml`
- Verify backup exists (if required)
- Ensure maintenance window is correct

### Patching Failed
- Check logs in `./logs/ansible.log`
- Review validation results in `/tmp/patching_validation/`
- Run dry-run to identify issues

## Next Steps

1. Customize variables in `group_vars/` for your environment
2. Set up host-specific variables in `host_vars/`
3. Configure notifications (email, Slack)
4. Integrate with your CI/CD pipeline
5. Set up automated scheduling (cron, AWX, etc.)

## Need Help?

- Review the main README.md for detailed documentation
- Check logs in `./logs/` directory
- Test with staging inventory first
- Run with `-vvv` for verbose output
