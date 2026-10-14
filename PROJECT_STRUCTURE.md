# Project Structure

Complete directory structure of the Enterprise Linux Patching project.

```
enterprise_linux_patching/
├── .gitignore                           # Git ignore rules
├── ansible.cfg                          # Ansible configuration
├── README.md                            # Main documentation
├── QUICKSTART.md                        # Quick start guide
├── CONFIG_EXAMPLES.md                   # Configuration examples
├── PROJECT_STRUCTURE.md                 # This file
│
├── inventories/                         # Inventory files
│   ├── production/
│   │   └── hosts.yml                    # Production server inventory
│   ├── staging/
│   │   └── hosts.yml                    # Staging server inventory
│   └── testing/
│       └── hosts.yml                    # Testing server inventory
│
├── roles/                              # Ansible roles
│   ├── pre_validation/                 # Pre-patching validation
│   │   ├── defaults/
│   │   │   └── main.yml                # Default variables
│   │   ├── handlers/
│   │   ├── meta/
│   │   ├── tasks/
│   │   │   └── main.yml                # Pre-validation tasks
│   │   ├── templates/
│   │   └── vars/
│   │
│   ├── patching/                        # Main patching role
│   │   ├── defaults/
│   │   │   └── main.yml                # Default variables
│   │   ├── handlers/
│   │   ├── meta/
│   │   ├── tasks/
│   │   │   └── main.yml                # Patching tasks
│   │   ├── templates/
│   │   └── vars/
│   │
│   ├── post_validation/                # Post-patching validation
│   │   ├── defaults/
│   │   │   └── main.yml                # Default variables
│   │   ├── handlers/
│   │   ├── meta/
│   │   ├── tasks/
│   │   │   └── main.yml                # Post-validation tasks
│   │   ├── templates/
│   │   └── vars/
│   │
│   └── rollback/                        # Rollback role
│       ├── defaults/
│       │   └── main.yml                # Default variables
│       ├── handlers/
│       ├── meta/
│       ├── tasks/
│       │   └── main.yml                # Rollback tasks
│       ├── templates/
│       └── vars/
│
├── playbooks/                          # Main playbooks
│   ├── patch_linux_servers.yml         # Main patching playbook
│   ├── patch_linux_servers_dryrun.yml  # Dry-run version
│   ├── test_connectivity.yml           # Connectivity test
│   ├── generate_compliance_report.yml  # Compliance reporting
│   │
│   ├── approval/                        # Approval workflow
│   │   ├── request_approval.yml        # Request patching approval
│   │   └── approve_request.yml        # Approve patching request
│   │
│   └── maintenance/                    # Maintenance scheduling
│       └── schedule_maintenance.yml    # Schedule maintenance window
│
├── group_vars/                         # Group variables
│   ├── all.yml                         # Global variables
│   ├── rhel_based.yml                  # RHEL-specific variables
│   └── debian_based.yml                # Debian-specific variables
│
├── host_vars/                          # Host-specific variables
│   └── example_host.yml                # Example host configuration
│
├── logs/                               # Execution logs (created at runtime)
│   └── ansible.log                     # Ansible execution log
│
├── reports/                            # Compliance reports (created at runtime)
│   ├── compliance_report_*.json       # JSON compliance reports
│   ├── compliance_report_*.html       # HTML compliance reports
│   └── audit_log_*.log                 # Audit logs
│
├── requests/                           # Approval requests (created at runtime)
│   └── *_approval.json                 # Approval request files
│
└── maintenance/                        # Maintenance schedules (created at runtime)
    ├── *_schedule.txt                  # Maintenance schedule files
    └── *_alert.txt                      # Maintenance alert files
```

## File Descriptions

### Configuration Files

- **ansible.cfg**: Main Ansible configuration with enterprise settings
- **.gitignore**: Git ignore patterns for logs, reports, and secrets

### Documentation Files

- **README.md**: Comprehensive documentation with features, usage, and troubleshooting
- **QUICKSTART.md**: Quick start guide for immediate deployment
- **CONFIG_EXAMPLES.md**: Configuration examples for various scenarios
- **PROJECT_STRUCTURE.md**: This file

### Inventory Files

- **inventories/production/hosts.yml**: Production server inventory with mixed distributions
- **inventories/staging/hosts.yml**: Staging environment inventory for testing
- **inventories/testing/hosts.yml**: Testing environment inventory for development

### Role Files

#### Pre-Validation Role
- **roles/pre_validation/defaults/main.yml**: Default validation thresholds and settings
- **roles/pre_validation/tasks/main.yml**: Pre-patching validation tasks

#### Patching Role
- **roles/patching/defaults/main.yml**: Default patching strategy and behavior
- **roles/patching/tasks/main.yml**: Main patching tasks for all distributions

#### Post-Validation Role
- **roles/post_validation/defaults/main.yml**: Post-validation thresholds and checks
- **roles/post_validation/tasks/main.yml**: Post-patching validation tasks

#### Rollback Role
- **roles/rollback/defaults/main.yml**: Rollback method and behavior settings
- **roles/rollback/tasks/main.yml**: Rollback tasks for reverting changes

### Playbook Files

- **playbooks/patch_linux_servers.yml**: Main patching orchestrator playbook
- **playbooks/patch_linux_servers_dryrun.yml**: Dry-run version for testing
- **playbooks/test_connectivity.yml**: Connectivity and system information test
- **playbooks/generate_compliance_report.yml**: Compliance and audit report generator
- **playbooks/approval/request_approval.yml**: Create patching approval request
- **playbooks/approval/approve_request.yml**: Approve patching request
- **playbooks/maintenance/schedule_maintenance.yml**: Schedule maintenance window

### Variable Files

- **group_vars/all.yml**: Global variables for all hosts
- **group_vars/rhel_based.yml**: RHEL-based distribution variables
- **group_vars/debian_based.yml**: Debian-based distribution variables
- **host_vars/example_host.yml**: Example host-specific configuration

### Runtime Directories

- **logs/**: Created at runtime, contains Ansible execution logs
- **reports/**: Created at runtime, contains compliance reports
- **requests/**: Created at runtime, contains approval requests
- **maintenance/**: Created at runtime, contains maintenance schedules

## Key Features by Component

### Roles
- **Pre-Validation**: Resource checks, service validation, backup verification
- **Patching**: Multi-distribution support, conditional reboot, change logging
- **Post-Validation**: Health checks, service verification, rollback triggers
- **Rollback**: Package history, kernel rollback, backup restoration

### Playbooks
- **Approval Workflow**: Multi-stage approval process
- **Maintenance Scheduling**: Planned maintenance windows
- **Compliance Reporting**: JSON, HTML, and audit logs
- **Testing**: Connectivity tests and dry-run mode

### Variables
- **Global**: Default behavior for all hosts
- **Group**: Distribution-specific settings
- **Host**: Individual server customization

## Extension Points

### Adding New Roles
1. Create role directory under `roles/`
2. Add defaults, tasks, handlers, templates
3. Import in main playbook

### Adding New Inventories
1. Create directory under `inventories/`
2. Add hosts.yml file
3. Specify with `-i` flag

### Adding New Variables
1. Add to `group_vars/` for groups
2. Add to `host_vars/` for individual hosts
3. Override with `-e` flag at runtime

### Adding New Playbooks
1. Create playbook under `playbooks/`
2. Import required roles
3. Configure variables

## Best Practices

1. **Version Control**: Keep all configuration files in version control
2. **Environment Separation**: Use separate inventories for prod/staging/test
3. **Variable Hierarchy**: Use group_vars → host_vars → command-line overrides
4. **Documentation**: Update documentation when adding features
5. **Testing**: Always test in staging before production
6. **Logging**: Review logs after each execution
7. **Backup**: Keep backups of working configurations
