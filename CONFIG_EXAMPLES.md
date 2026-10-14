# Configuration Examples

This document provides configuration examples for different scenarios.

## Example 1: High-Criticality Database Server

### host_vars/db-server-01.yml
```yaml
---
# High-criticality database server configuration

patching_strategy: kernel
patching_reboot_required: true
patching_enable_rollback: true
patching_services_to_stop:
  - postgresql
  - mongod

pre_validation_cpu_threshold: 70
pre_validation_memory_threshold: 70
pre_validation_disk_threshold: 85
pre_validation_require_backup: true
pre_validation_backup_age_hours: 12
pre_validation_critical_services:
  - postgresql
  - mongod

post_validation_critical_services:
  - postgresql
  - mongod
post_validation_auto_rollback_on_failure: true
post_validation_rollback_threshold: 1

maintenance_window:
  start: "03:00"
  end: "05:00"
```

## Example 2: Web Server Cluster

### host_vars/web-server-01.yml
```yaml
---
# Web server configuration

patching_strategy: security
patching_reboot_required: false
patching_enable_rollback: true
patching_services_to_stop:
  - nginx
  - php-fpm

pre_validation_cpu_threshold: 85
pre_validation_memory_threshold: 85
pre_validation_disk_threshold: 90
pre_validation_require_backup: false
pre_validation_critical_services:
  - nginx

post_validation_check_applications: true
post_validation_app_health_url:
  - "http://localhost/health"
  - "http://localhost/api/status"
post_validation_app_health_expected: 200

maintenance_window:
  start: "02:00"
  end: "04:00"
```

## Example 3: Development Environment

### group_vars/dev_environment.yml
```yaml
---
# Development environment - less strict validation

patching_strategy: full
patching_reboot_required: true
patching_enable_rollback: false

pre_validation_cpu_threshold: 95
pre_validation_memory_threshold: 95
pre_validation_disk_threshold: 95
pre_validation_require_backup: false
pre_validation_check_maintenance_window: false

post_validation_auto_rollback_on_failure: false
post_validation_rollback_threshold: 5
```

## Example 4: Compliance-Heavy Environment

### group_vars/compliance_environment.yml
```yaml
---
# Strict compliance environment

patching_strategy: security
patching_reboot_required: true
patching_enable_rollback: true
patching_log_changes: true

pre_validation_cpu_threshold: 75
pre_validation_memory_threshold: 75
pre_validation_disk_threshold: 80
pre_validation_require_backup: true
pre_validation_backup_age_hours: 24
pre_validation_require_rollback_plan: true
pre_validation_check_filesystem: true
pre_validation_filesystem_mounts:
  - /boot
  - /var
  - /home
  - /data

post_validation_check_logs: true
post_validation_log_paths:
  - /var/log/messages
  - /var/log/secure
  - /var/log/audit/audit.log
post_validation_error_keywords:
  - error
  - critical
  - failed
  - security
  - unauthorized

compliance_log_enabled: true
compliance_retention_days: 2555  # 7 years
```

## Example 5: Custom Package Patching

### host_vars/app-server-01.yml
```yaml
---
# Custom package patching

patching_strategy: custom
patching_included_packages:
  - nginx
  - python3
  - nodejs
  - docker-ce
patching_excluded_packages:
  - kernel*
  - grub*
  - systemd

patching_reboot_required: false
patching_services_to_stop:
  - nginx
  - docker
```

## Example 6: Mixed Distribution Environment

### inventories/production/hosts.yml (Extended)
```yaml
---
all:
  children:
    linux_servers:
      children:
        production_web:
          hosts:
            web-prod-01:
              ansible_host: 10.0.1.10
              ansible_user: webadmin
              server_type: web
              os_family: debian
            web-prod-02:
              ansible_host: 10.0.1.11
              ansible_user: webadmin
              server_type: web
              os_family: debian
        production_db:
          hosts:
            db-prod-01:
              ansible_host: 10.0.2.10
              ansible_user: dbadmin
              server_type: database
              os_family: rhel
            db-prod-02:
              ansible_host: 10.0.2.11
              ansible_user: dbadmin
              server_type: database
              os_family: rhel
        production_app:
          hosts:
            app-prod-01:
              ansible_host: 10.0.3.10
              ansible_user: appadmin
              server_type: application
              os_family: rhel
            app-prod-02:
              ansible_host: 10.0.3.11
              ansible_user: appadmin
              server_type: application
              os_family: debian
```

## Example 7: Notification Configuration

### group_vars/all.yml (Additions)
```yaml
---
# Notification settings

enable_notifications: true
notification_email:
  - ops-team@company.com
  - security-team@company.com
notification_slack_webhook: "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
notification_pagerduty_key: "YOUR_PAGERDUTY_KEY"

# Notification triggers
notify_on_pre_validation_fail: true
notify_on_patching_complete: true
notify_on_post_validation_fail: true
notify_on_rollback: true
```

## Example 8: Approval Workflow Configuration

### group_vars/production.yml
```yaml
---
# Production approval workflow

approval_required: true
approval_approvers:
  - manager@company.com
  - lead-engineer@company.com
approval_timeout_hours: 24
approval_comment_required: true

# Rollback settings
patching_enable_rollback: true
rollback_method: package
rollback_reboot_after_rollback: true
```

## Example 9: Maintenance Window Configuration

### host_vars/batch-server-01.yml
```yaml
---
# Batch processing server - specific window

patching_strategy: full
patching_reboot_required: true

maintenance_window:
  start: "06:00"
  end: "08:00"
  timezone: "UTC"

# Skip maintenance window check for emergency patches
emergency_patch: false
```

## Example 10: Rollback Configuration

### group_vars/high_criticality.yml
```yaml
---
# Enhanced rollback for critical systems

patching_enable_rollback: true
rollback_method: snapshot
rollback_snapshot_name: "pre-patch-{{ ansible_date_time.epoch }}"
rollback_snapshot_retention: 14
rollback_stop_services: true
rollback_services_to_stop:
  - httpd
  - postgresql
  - rabbitmq-server
rollback_start_services: true
rollback_verify: true
rollback_reboot_after_rollback: true
```

## Usage Examples

### Example 1: Patch with Custom Configuration
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e target_group=production_web \
  -e patching_strategy=security \
  -e patching_reboot_required=false
```

### Example 2: Patch with Override Variables
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e target_group=production_db \
  -e pre_validation_cpu_threshold=60 \
  -e post_validation_auto_rollback_on_failure=true
```

### Example 3: Emergency Patching (Skip Validation)
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e target_group=production_app \
  -e pre_validation_check_maintenance_window=false \
  -e pre_validation_require_backup=false \
  -e justification="Emergency security patch CVE-2024-XXXX"
```

### Example 4: Rollback Specific Server
```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/patch_linux_servers.yml \
  -e target_group=production_db \
  -e rollback_method=package \
  --limit db-prod-01
```

## Best Practices

1. **Start with Conservative Thresholds**: Begin with lower resource thresholds and adjust based on your environment
2. **Test in Staging**: Always test configurations in staging before production
3. **Document Exceptions**: Keep records of any exceptions or overrides
4. **Version Control**: Store all configuration files in version control
5. **Regular Reviews**: Review and update configurations quarterly
6. **Monitor Adjustments**: Monitor system behavior after threshold changes
7. **Backup Configurations**: Keep backups of working configurations
