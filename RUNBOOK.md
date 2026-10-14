# Enterprise Linux Patching - Operations Runbook

## Table of Contents
1. [Purpose](#purpose)
2. [Prerequisites](#prerequisites)
3. [Quick Reference](#quick-reference)
4. [Standard Operating Procedures](#standard-operating-procedures)
5. [Emergency Procedures](#emergency-procedures)
6. [Troubleshooting](#troubleshooting)
7. [Maintenance Tasks](#maintenance-tasks)
8. [Checklists](#checklists)

---

## Purpose

This runbook provides step-by-step procedures for managing Linux server patching in enterprise environments using the Ansible-based patching framework.

**Scope**: Production, staging, and testing Linux servers (RHEL-based and Debian-based)

**Audience**: System Administrators, DevOps Engineers, SREs

**Environment Specifics**:
- **Patching Schedule**: Quarterly
- **Approvers**: Senior SME
- **Backup Strategy**: Pure Storage
- **Monitoring/Alerting**: ELK Stack
- **Change Management**: ServiceNow (SNOW)
- **Automation Platform**: AWX

---

## Prerequisites

### System Requirements

**Control Node (where Ansible runs):**
- Ansible 2.9 or higher
- Python 3.6+
- SSH access to all target servers
- Sudo privileges on target servers
- 10GB free disk space for logs and reports

**Target Servers:**
- SSH access configured
- Sudo access for patching user
- Python 2.7+ or Python 3.5+
- Minimum 2GB free disk space for patching

### Access Requirements

- SSH key-based authentication configured
- Sudo privileges without password prompt (or configured in ansible.cfg)
- Network connectivity to all target servers
- Access to backup systems (if backup validation required)

### Initial Setup

```bash
# 1. Navigate to project directory
cd enterprise_linux_patching

# 2. Create runtime directories
mkdir -p logs reports requests maintenance

# 3. Configure inventory
vim inventories/production/hosts.yml

# 4. Test connectivity
ansible-playbook -i inventories/production/hosts.yml playbooks/test_connectivity.yml
```

---

## Quick Reference

### Common Commands

```bash
# Test connectivity to all servers
ansible-playbook -i inventories/production/hosts.yml playbooks/test_connectivity.yml

# Run dry-run (no changes)
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers_dryrun.yml

# Patch all servers in production
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml

# Patch specific group
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml -e target_group=rhel_based

# Patch with custom strategy
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml -e patching_strategy=kernel

# Generate compliance report
ansible-playbook playbooks/generate_compliance_report.yml

# Request approval
ansible-playbook playbooks/approval/request_approval.yml -e target_group=linux_servers -e justification="Monthly security updates"

# Approve request
ansible-playbook playbooks/approval/approve_request.yml -e request_id=<REQUEST_ID>

# Schedule maintenance window
ansible-playbook playbooks/maintenance/schedule_maintenance.yml -e start_time="2024-01-15 02:00" -e end_time="2024-01-15 06:00"
```

### Patching Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| `security` | Security updates only | Regular security patching |
| `full` | All package updates | Major updates, feature releases |
| `kernel` | Kernel updates only | Kernel security patches |
| `custom` | User-defined packages | Specific application updates |

### Environment Variables

```bash
# Set custom inventory
export ANSIBLE_INVENTORY=./inventories/production/hosts.yml

# Enable verbose output
export ANSIBLE_VERBOSITY=2

# Disable host key checking (for automated environments)
export ANSIBLE_HOST_KEY_CHECKING=False
```

---

## Standard Operating Procedures

### SOP-001: Quarterly Security Patching

**Frequency**: Quarterly (first Tuesday of quarter: Jan, Apr, Jul, Oct, 02:00-06:00)

**Pre-Patch Checklist:**
- [ ] Review security bulletins (CVE alerts)
- [ ] Verify Pure Storage snapshots exist (< 24 hours old)
- [ ] Confirm maintenance window approved
- [ ] Create ServiceNow change request
- [ ] Notify stakeholders 48 hours in advance
- [ ] Test in staging environment first
- [ ] Schedule job in AWX for execution

**Procedure:**

1. **Request Approval** (if required by policy)
   ```bash
   ansible-playbook playbooks/approval/request_approval.yml \
     -e target_group=linux_servers \
     -e strategy=security \
     -e justification="Quarterly security patching - CVE-2024-XXXXX"
   ```
   Note the REQUEST_ID from output

2. **Obtain Approval from Senior SME**
   - Send request ID to Senior SME
   - Update ServiceNow change request with approval status
   - Senior SME runs:
   ```bash
   ansible-playbook playbooks/approval/approve_request.yml -e request_id=<REQUEST_ID>
   ```
   - Document approval in ServiceNow

3. **Run Dry-Run Test**
   ```bash
   ansible-playbook -i inventories/production/hosts.yml \
     playbooks/patch_linux_servers_dryrun.yml \
     -e target_group=linux_servers \
     -e patching_strategy=security
   ```
   Review output for any warnings
   - Update ServiceNow change request with dry-run results

4. **Execute Patching via AWX** (Recommended)
   - Log into AWX
   - Select the patching job template
   - Set target group: `linux_servers`
   - Set patching strategy: `security`
   - Set extra variables if needed
   - Launch job
   - Monitor job in AWX UI

   **OR Execute via CLI** (for manual runs)
   ```bash
   ansible-playbook -i inventories/production/hosts.yml \
     playbooks/patch_linux_servers.yml \
     -e target_group=linux_servers \
     -e patching_strategy=security
   ```

5. **Monitor Progress**
   - Watch for pre-validation failures
   - Monitor reboot progress
   - Check post-validation results

6. **Generate Compliance Report**
   ```bash
   ansible-playbook playbooks/generate_compliance_report.yml
   ```

7. **Post-Patch Verification**
   - Review compliance report in `./reports/`
   - Check application functionality
   - Verify ELK Stack monitoring (check Kibana dashboards)
   - Query ELK for any errors in logs
   - Document any issues in ServiceNow

**Post-Patch Checklist:**
- [ ] All servers patched successfully
- [ ] No rollback triggered
- [ ] Compliance report generated
- [ ] Applications verified
- [ ] ELK Stack monitoring normal (no errors in Kibana)
- [ ] Pure Storage snapshots verified
- [ ] Stakeholders notified
- [ ] ServiceNow change request closed
- [ ] AWX job logs reviewed

**Estimated Time**: 2-4 hours (depending on server count)

---

### SOP-002: Emergency Security Patching

**Trigger**: Critical CVE (CVSS 9.0+) requiring immediate action

**Pre-Patch Checklist:**
- [ ] Security team assessment complete
- [ ] Business impact assessment done
- [ ] Senior SME approval obtained
- [ ] ServiceNow emergency change request created
- [ ] Communication plan activated
- [ ] Pure Storage snapshot verified

**Procedure:**

1. **Bypass Maintenance Window Check**
   ```bash
   ansible-playbook -i inventories/production/hosts.yml \
     playbooks/patch_linux_servers.yml \
     -e target_group=linux_servers \
     -e patching_strategy=security \
     -e pre_validation_check_maintenance_window=false \
     -e pre_validation_require_backup=false \
     -e justification="Emergency security patch CVE-2024-XXXXX"
   ```

2. **Monitor Closely**
   - Use verbose mode: `-vvv`
   - Monitor in real-time
   - Have rollback plan ready

3. **Immediate Post-Patch**
   - Verify critical services
   - Check security bulletins resolved
   - Generate compliance report

**Post-Patch Checklist:**
- [ ] Vulnerability resolved
- [ ] Systems stable
- [ ] ELK Stack shows no errors
- [ ] Pure Storage snapshot verified
- [ ] ServiceNow incident report filed
- [ ] Root cause analysis initiated
- [ ] Communication sent to stakeholders

**Estimated Time**: 1-2 hours

---

### SOP-003: Kernel Update (with Reboot)

**Frequency**: Quarterly (aligned with quarterly patching cycle)

**Pre-Patch Checklist:**
- [ ] Test kernel in staging
- [ ] Verify driver compatibility
- [ ] Plan for application downtime
- [ ] Have rollback plan ready
- [ ] Create Pure Storage snapshot before kernel update
- [ ] Notify users of planned downtime
- [ ] Create ServiceNow change request

**Procedure:**

1. **Schedule Maintenance Window**
   ```bash
   ansible-playbook playbooks/maintenance/schedule_maintenance.yml \
     -e target_group=linux_servers \
     -e start_time="2024-01-15 02:00" \
     -e end_time="2024-01-15 06:00" \
     -e justification="Quarterly kernel update"
   ```

2. **Run with Kernel Strategy**
   ```bash
   ansible-playbook -i inventories/production/hosts.yml \
     playbooks/patch_linux_servers.yml \
     -e target_group=linux_servers \
     -e patching_strategy=kernel \
     -e patching_reboot_required=true
   ```

3. **Monitor Reboot Process**
   - Watch for reboot failures
   - Verify servers come back online
   - Check post-boot functionality

**Post-Patch Checklist:**
- [ ] All servers rebooted successfully
- [ ] New kernel version verified
- [ ] Applications restarted
- [ ] ELK Stack monitoring normal
- [ ] Performance baseline checked
- [ ] Pure Storage snapshot verified
- [ ] Documentation updated
- [ ] ServiceNow change request closed

**Estimated Time**: 4-6 hours (including reboots)

---

### SOP-004: Rollback Procedure

**Trigger**: Post-validation failure or application issues after patching

**Pre-Rollback Checklist:**
- [ ] Identify cause of failure
- [ ] Document current state
- [ ] Notify stakeholders
- [ ] Notify Senior SME
- [ ] Create ServiceNow incident
- [ ] Plan rollback window
- [ ] Verify Pure Storage snapshot available

**Procedure:**

1. **Manual Rollback**
   ```bash
   ansible-playbook -i inventories/production/hosts.yml \
     playbooks/patch_linux_servers.yml \
     -e target_group=linux_servers \
     -e rollback_method=package \
     --limit <affected_server>
   ```

   **OR Restore from Pure Storage Snapshot** (if package rollback fails)
   - Access Pure Storage management console
   - Revert to pre-patch snapshot
   - Verify system state

2. **Verify Rollback**
   - Check kernel version
   - Verify package versions
   - Test application functionality
   - Check ELK Stack for errors

3. **Document Rollback**
   - Update ServiceNow change request
   - Update ServiceNow incident
   - File detailed incident report
   - Schedule remediation

**Post-Rollback Checklist:**
- [ ] System restored to previous state
- [ ] Applications functioning
- [ ] ELK Stack monitoring normal
- [ ] Pure Storage snapshot verified
- [ ] ServiceNow incident documented
- [ ] ServiceNow change request updated
- [ ] Root cause analysis initiated
- [ ] Preventive measures planned
- [ ] AWX job logs reviewed

**Estimated Time**: 1-2 hours

---

### SOP-005: Staging Environment Testing

**Frequency**: Before every production patching

**Procedure:**

1. **Configure Staging Inventory**
   ```bash
   vim inventories/staging/hosts.yml
   ```

2. **Run Full Patching in Staging**
   ```bash
   ansible-playbook -i inventories/staging/hosts.yml \
     playbooks/patch_linux_servers.yml \
     -e patching_strategy=full
   ```

3. **Verify Applications**
   - Test all critical applications
   - Check performance
   - Verify integrations
   - Check ELK Stack for errors

4. **Document Results**
   - Note any issues
   - Adjust production plan if needed
   - Update ServiceNow change request

**Estimated Time**: 1-2 hours

---

### SOP-006: AWX Job Template Configuration

**Purpose**: Configure AWX job templates for automated patching execution

**Prerequisites:**
- AWX/Ansible Tower installed and configured
- Project synced with this Ansible repository
- Credentials configured for target servers
- Inventories imported into AWX

**Procedure:**

1. **Create Job Template for Patching**
   - Navigate to AWX → Templates → Add → Job Template
   - Name: `Enterprise Linux Patching - Production`
   - Job Type: Run
   - Inventory: Select production inventory
   - Project: Select this project
   - Playbook: `playbooks/patch_linux_servers.yml`
   - Credentials: Select SSH credential
   - Privilege Escalation: Enabled
   - Save

2. **Configure Extra Variables**
   - In job template, add default extra variables:
   ```yaml
   ---
   target_group: linux_servers
   patching_strategy: security
   patching_reboot_required: true
   pre_validation_check_maintenance_window: true
   post_validation_auto_rollback_on_failure: true
   ```

3. **Create Survey for User Input**
   - Add survey questions:
     - Target Group (dropdown: linux_servers, rhel_based, debian_based)
     - Patching Strategy (dropdown: security, full, kernel, custom)
     - Reboot Required (yes/no)
     - Justification (text)
   - Save survey

4. **Set Up Scheduling**
   - Schedule job for quarterly execution
   - Set: First Tuesday of quarter at 02:00
   - Enable notifications for success/failure

5. **Configure Notifications**
   - Add Slack webhook for job completion
   - Add email for job failures
   - Configure ServiceNow integration if available

6. **Create Separate Job Templates**
   - `Dry-Run Test` → `playbooks/patch_linux_servers_dryrun.yml`
   - `Compliance Report` → `playbooks/generate_compliance_report.yml`
   - `Test Connectivity` → `playbooks/test_connectivity.yml`

**Verification:**
- [ ] Test job template with dry-run
- [ ] Verify survey works correctly
- [ ] Check notifications are sent
- [ ] Validate ServiceNow integration

---

## Emergency Procedures

### EMER-001: Patching Rollback

**Scenario**: Multiple servers fail post-validation or application not working

**Immediate Actions:**

1. **Stop Patching** (if still in progress)
   ```bash
   # Press Ctrl+C to stop playbook
   ```

2. **Assess Scope**
   ```bash
   # Check which servers are affected
   ansible -i inventories/production/hosts.yml linux_servers -m shell -a "uptime"
   ```

3. **Execute Rollback**
   ```bash
   ansible-playbook -i inventories/production/hosts.yml \
     playbooks/patch_linux_servers.yml \
     -e target_group=linux_servers \
     -e rollback_method=package
   ```

4. **Notify Team**
   - Page on-call engineer
   - Notify management
   - Alert application owners

5. **Document Incident**
   - Create incident ticket
   - Log all actions taken
   - Preserve logs

### EMER-002: Server Not Coming Back After Reboot

**Scenario**: Server doesn't return after kernel update reboot

**Immediate Actions:**

1. **Check Connection**
   ```bash
   ansible -i inventories/production/hosts.yml <server> -m ping
   ```

2. **Manual Console Access**
   - Access via console/IPMI
   - Check boot process
   - Look for errors

3. **Boot Previous Kernel**
   - In GRUB menu, select previous kernel
   - Boot system
   - Investigate issue

4. **If Unrecoverable**
   - Restore from backup
   - Document failure
   - Engage vendor support

### EMER-003: Pre-Validation Failures

**Scenario**: Pre-validation fails for multiple servers

**Immediate Actions:**

1. **Check Failure Reason**
   - Review validation results in `/tmp/patching_validation/`
   - Identify common failure pattern

2. **Adjust Thresholds if Safe**
   ```bash
   ansible-playbook -i inventories/production/hosts.yml \
     playbooks/patch_linux_servers.yml \
     -e pre_validation_cpu_threshold=90 \
     -e pre_validation_memory_threshold=90
   ```

3. **Or Skip Problematic Servers**
   ```bash
   ansible-playbook -i inventories/production/hosts.yml \
     playbooks/patch_linux_servers.yml \
     --limit "linux_servers:!problematic_servers"
   ```

4. **Document Override**
   - Justify in change management
   - Plan remediation

---

## Troubleshooting

### TSH-001: SSH Connection Failed

**Symptoms**: `UNREACHABLE!` error when running playbooks

**Possible Causes:**
- SSH key not configured
- Wrong SSH key path
- Network connectivity issue
- Firewall blocking SSH

**Resolution Steps:**

1. **Test SSH Manually**
   ```bash
   ssh -i ~/.ssh/id_rsa user@server
   ```

2. **Check SSH Key Permissions**
   ```bash
   chmod 600 ~/.ssh/id_rsa
   ```

3. **Verify Inventory Configuration**
   ```bash
   vim inventories/production/hosts.yml
   # Check ansible_host, ansible_user, ansible_ssh_private_key_file
   ```

4. **Test with Ansible**
   ```bash
   ansible -i inventories/production/hosts.yml all -m ping
   ```

### TSH-002: Sudo Permission Denied

**Symptoms**: `Permission denied` errors during patching

**Possible Causes:**
- User doesn't have sudo access
- Sudo requires password
- Sudoers configuration issue

**Resolution Steps:**

1. **Test Sudo Access**
   ```bash
   ssh user@server "sudo -v"
   ```

2. **Configure Passwordless Sudo**
   ```bash
   # On target server
   sudo visudo
   # Add: username ALL=(ALL) NOPASSWD: ALL
   ```

3. **Or Configure Ansible to Ask for Password**
   ```bash
   ansible-playbook playbook.yml --ask-become-pass
   ```

### TSH-003: Pre-Validation Failed

**Symptoms**: Pre-validation fails with resource threshold errors

**Possible Causes:**
- Server under high load
- Disk space insufficient
- Backup missing
- Outside maintenance window

**Resolution Steps:**

1. **Check Validation Results**
   ```bash
   cat /tmp/patching_validation/<server>_pre_validation.txt
   ```

2. **Investigate Specific Failure**
   - If CPU/Memory high: Check running processes
   - If Disk full: Clean up space
   - If Backup missing: Verify backup system
   - If Window issue: Check time or adjust window

3. **Adjust Thresholds if Appropriate**
   ```bash
   ansible-playbook playbook.yml \
     -e pre_validation_cpu_threshold=90
   ```

4. **Or Skip Validation for Emergency**
   ```bash
   ansible-playbook playbook.yml \
     -e pre_validation_check_maintenance_window=false
   ```

### TSH-004: Post-Validation Failed

**Symptoms**: Post-validation fails, rollback triggered

**Possible Causes:**
- Service not starting
- Application not responding
- Port not accessible
- Log errors present

**Resolution Steps:**

1. **Check Post-Validation Results**
   ```bash
   cat /tmp/patching_validation/<server>_post_validation.txt
   ```

2. **Check Service Status**
   ```bash
   ansible -i inventories/production/hosts.yml <server> -m shell -a "systemctl status <service>"
   ```

3. **Check Application Logs**
   ```bash
   ansible -i inventories/production/hosts.yml <server> -m shell -a "tail -100 /var/log/app.log"
   ```

4. **Manual Rollback if Needed**
   ```bash
   ansible-playbook playbook.yml -e rollback_method=package
   ```

### TSH-005: Playbook Timeout

**Symptoms**: Playbook hangs or times out

**Possible Causes:**
- Long-running process
- Network timeout
- Reboot taking too long
- Package download slow

**Resolution Steps:**

1. **Increase Timeout**
   ```bash
   ansible-playbook playbook.yml -e patching_reboot_timeout=1200
   ```

2. **Check Server Status**
   ```bash
   ansible -i inventories/production/hosts.yml <server> -m shell -a "uptime"
   ```

3. **Run with Verbose Output**
   ```bash
   ansible-playbook playbook.yml -vvv
   ```

4. **Run on Single Server**
   ```bash
   ansible-playbook playbook.yml --limit <server>
   ```

---

## Maintenance Tasks

### MNT-001: Monthly Maintenance

**Frequency**: Monthly

**Tasks:**
- [ ] Review and rotate logs in `./logs/`
- [ ] Archive old reports from `./reports/`
- [ ] Clean up old approval requests in `./requests/`
- [ ] Update inventory with new servers
- [ ] Remove decommissioned servers
- [ ] Review and update thresholds
- [ ] Test connectivity to all servers
- [ ] Validate SSH keys
- [ ] Update documentation

**Procedure:**

```bash
# Archive old logs
cd logs
tar -czf logs_$(date +%Y%m).tar.gz *.log
rm *.log

# Archive old reports
cd ../reports
tar -czf reports_$(date +%Y%m).tar.gz *.json *.html *.log
rm *.json *.html *.log

# Clean up old requests
cd ../requests
find . -name "*.json" -mtime +90 -delete
```

### MNT-002: Quarterly Review

**Frequency**: Quarterly

**Tasks:**
- [ ] Review patching history
- [ ] Analyze failure patterns
- [ ] Update validation thresholds based on trends
- [ ] Review and update runbook
- [ ] Train new team members
- [ ] Security audit of SSH keys
- [ ] Backup Ansible configuration
- [ ] Test disaster recovery procedures

### MNT-003: Annual Audit

**Frequency**: Annually

**Tasks:**
- [ ] Full security audit
- [ ] Compliance review
- [ ] Access review (who can run patching)
- [ ] Update documentation
- [ ] Review and update runbook
- [ ] Capacity planning
- [ ] Disaster recovery test
- [ ] Vendor contract review

---

## Checklists

### Pre-Patch Checklist

**Planning:**
- [ ] Patching window approved
- [ ] Business owner notified
- [ ] Change request submitted
- [ ] Backups verified (< 24 hours old)
- [ ] Rollback plan documented
- [ ] Communication plan activated

**Technical:**
- [ ] Inventory updated
- [ ] Connectivity tested
- [ ] Sudo access verified
- [ ] Disk space sufficient
- [ ] Maintenance window confirmed
- [ ] Staging testing completed

**Documentation:**
- [ ] Runbook reviewed
- [ ] Emergency procedures available
- [ ] Contact list updated
- [ ] Incident response team notified

### During Patching Checklist

**Monitoring:**
- [ ] Pre-validation results reviewed
- [ ] Patching progress monitored
- [ ] Reboot progress tracked
- [ ] Post-validation results checked
- [ ] Rollback status monitored

**Communication:**
- [ ] Status updates sent
- [ ] Stakeholders informed of progress
- [ ] Issues escalated promptly
- [ ] Documentation updated in real-time

### Post-Patch Checklist

**Verification:**
- [ ] All servers patched
- [ ] No rollback triggered
- [ ] Services running
- [ ] Applications functional
- [ ] Performance normal
- [ ] Monitoring clear

**Documentation:**
- [ ] Compliance report generated
- [ ] Change request closed
- [ ] Incident log updated
- [ ] Lessons learned documented
- [ ] Stakeholders notified
- [ ] Runbook updated if needed

---

## Contact Information

**Primary On-Call**: [Name] - [Phone] - [Email]
**Secondary On-Call**: [Name] - [Phone] - [Email]
**Senior SME (Approver)**: [Name] - [Phone] - [Email]
**Management**: [Name] - [Phone] - [Email]
**Security Team**: [Name] - [Phone] - [Email]
**Pure Storage Admin**: [Name] - [Phone] - [Email]
**ELK Stack Admin**: [Name] - [Phone] - [Email]
**AWX Administrator**: [Name] - [Phone] - [Email]
**ServiceNow Admin**: [Name] - [Phone] - [Email]
**Application Owners**: [List]

---

## Change History

| Date | Version | Changes | Author |
|------|---------|---------|--------|
| 2024-01-15 | 1.0 | Initial runbook creation | [Your Name] |

---

## Appendix

### A. ELK Stack Integration

**Purpose**: Monitor patching operations and detect issues using ELK Stack

**Configuration:**

1. **Log Forwarding to ELK**
   - Configure Filebeat on target servers to send logs to ELK
   - Configure Logstash to parse Ansible logs
   - Create Kibana dashboards for patching monitoring

2. **Kibana Dashboard Queries**
   - Search for patching errors:
     ```
     message: "patching" AND log_level: "ERROR"
     ```
   - Monitor service restarts:
     ```
     message: "service" AND action: "restart"
     ```
   - Track reboot events:
     ```
     message: "reboot" AND action: "system"
     ```

3. **Alerting Rules**
   - Create alert for multiple pre-validation failures
   - Create alert for post-validation failures
   - Create alert for rollback events
   - Configure alert notifications via email/Slack

**Monitoring Checklist:**
- [ ] ELK Stack receiving logs from all servers
- [ ] Kibana dashboards configured
- [ ] Alert rules tested
- [ ] Notifications working

### B. ServiceNow Integration

**Purpose**: Track patching operations in ServiceNow change management

**Integration Options:**

1. **Manual Integration**
   - Create change request before patching
   - Update change request during patching
   - Close change request after successful patching
   - Create incident for any failures

2. **AWX ServiceNow Integration**
   - Configure ServiceNow credential in AWX
   - Set up workflow to create change request
   - Configure job to update change status
   - Enable automatic incident creation on failure

**Change Request Template:**
- **Type**: Standard (for quarterly) or Emergency (for critical CVE)
- **Priority**: P3 (quarterly) or P1 (emergency)
- **Risk**: Medium (quarterly) or High (emergency)
- **Approval**: Senior SME
- **Implementation Plan**: Reference this runbook
- **Backout Plan**: Rollback procedure (SOP-004)
- **Test Plan**: Staging testing (SOP-005)

**ServiceNow Checklist:**
- [ ] Change request created before patching
- [ ] Approval obtained from Senior SME
- [ ] Change status updated during execution
- [ ] Incident created for failures
- [ ] Change closed after successful completion

### C. Pure Storage Integration

**Purpose**: Use Pure Storage snapshots for backup and rollback

**Procedure:**

1. **Pre-Patch Snapshot**
   - Create snapshot before patching
   - Name: `pre-patch-<date>-<server>`
   - Retain for 7 days minimum

2. **Rollback from Snapshot**
   - If package rollback fails
   - Revert to pre-patch snapshot
   - Verify system state
   - Update documentation

3. **Snapshot Cleanup**
   - Delete snapshots older than 30 days
   - Keep critical snapshots longer if needed

**Pure Storage Checklist:**
- [ ] Snapshot created before patching
- [ ] Snapshot verified
- [ ] Rollback procedure tested
- [ ] Cleanup schedule configured

### D. Variable Reference

See `CONFIG_EXAMPLES.md` for detailed variable configuration.

### B. Exit Codes

- `0` - Success
- `1` - General error
- `2` - SSH connection failed
- `3` - Validation failed
- `4` - Patching failed
- `5` - Rollback required

### C. Log Locations

- Ansible logs: `./logs/ansible.log`
- Validation results: `/tmp/patching_validation/`
- Compliance reports: `./reports/`
- Server patch logs: `/var/log/patching/`
