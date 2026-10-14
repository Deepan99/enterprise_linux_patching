# Testing Checklist - Enterprise Linux Patching

## Pre-Test Preparation

### Environment Setup
- [ ] Navigate to project directory: `cd enterprise_linux_patching`
- [ ] Create runtime directories: `mkdir -p logs reports requests maintenance`
- [ ] Review inventory configuration: `vim inventories/production/hosts.yml`
- [ ] Update inventory with actual server IPs and SSH keys
- [ ] Ensure SSH keys have correct permissions: `chmod 600 ~/.ssh/id_rsa`

### Test Environment Selection
**RECOMMENDED**: Start with testing inventory, not production
- [ ] Use `inventories/testing/hosts.yml` for initial tests
- [ ] Or use `inventories/staging/hosts.yml` for pre-production tests
- [ ] Production should only be used after successful staging tests

---

## Phase 1: Syntax Validation

### Test 1.1: Check Ansible Configuration
```bash
cd enterprise_linux_patching
ansible --version
# Should show Ansible 2.9 or higher
```

### Test 1.2: Validate Inventory Syntax
```bash
ansible-inventory -i inventories/testing/hosts.yml --list
# Should show inventory structure without errors
```

### Test 1.3: Check Playbook Syntax
```bash
ansible-playbook playbooks/patch_linux_servers.yml --syntax-check
# Should report: "playbook: patch_linux_servers.yml"
```

### Test 1.4: Check Role Syntax
```bash
ansible-playbook playbooks/test_connectivity.yml --syntax-check
# Should report: "playbook: test_connectivity.yml"
```

---

## Phase 2: Connectivity Testing

### Test 2.1: SSH Manual Test
```bash
# Test SSH to each server manually
ssh -i ~/.ssh/id_rsa <user>@<server>
# Should connect without password prompt
```

### Test 2.2: Ansible Ping Test
```bash
ansible-playbook -i inventories/testing/hosts.yml playbooks/test_connectivity.yml
# Should ping all hosts and display system information
```

**Expected Output**:
- All hosts should show `ok` for ping task
- System information should display (OS, kernel, CPU, memory)
- No connection errors

**If Fails**:
- Check SSH key path in inventory
- Verify SSH key permissions
- Check network connectivity
- Verify sudo access

---

## Phase 3: Dry-Run Testing

### Test 3.1: Pre-Validation Dry-Run
```bash
ansible-playbook -i inventories/testing/hosts.yml \
  playbooks/patch_linux_servers_dryrun.yml \
  -e target_group=linux_servers
```

**Expected Output**:
- Pre-validation should run
- Should show CPU, memory, disk usage
- Should display "would apply patches" message
- No actual changes made

### Test 3.2: Check Validation Results
```bash
cat /tmp/patching_validation/*_pre_validation.txt
# Should show validation results for each host
```

---

## Phase 4: Small-Scale Live Test

### Test 4.1: Test on Single Server (Staging)
```bash
ansible-playbook -i inventories/staging/hosts.yml \
  playbooks/patch_linux_servers.yml \
  --limit <single-test-server> \
  -e patching_strategy=security
```

**Expected Output**:
- Pre-validation should pass
- Security patches should be applied
- If reboot required, system should reboot
- Post-validation should run
- Compliance report should be generated

### Test 4.2: Verify Post-Patch State
```bash
# Check server is accessible
ansible -i inventories/staging/hosts.yml <test-server> -m ping

# Check package updates
ansible -i inventories/staging/hosts.yml <test-server> -m shell -a "rpm -qa | tail -10"

# Check kernel version (if updated)
ansible -i inventories/staging/staging/hosts.yml <test-server> -m shell -a "uname -r"
```

---

## Phase 5: Canary Deployment Test

### Test 5.1: Test Serial Execution (1 server first)
```bash
ansible-playbook -i inventories/staging/hosts.yml \
  playbooks/patch_linux_servers.yml \
  -e patching_strategy=security
```

**Expected Behavior**:
- Should patch 1 server first (canary)
- Should wait for completion
- Should then patch 20% of remaining servers
- Should stop if >10% fail

---

## Known Limitations (For This Test)

### ⚠️ Rollback Functionality
- **Status**: NOT FUNCTIONAL
- **Workaround**: Use Pure Storage snapshots for rollback
- **Do NOT** test the rollback role - it will not actually revert changes

### ⚠️ Shell Commands vs Native Facts
- **Status**: Still using shell commands for resource validation
- **Impact**: May fail on systems with different locales or `top` versions
- **Workaround**: Test on representative systems

---

## Success Criteria

### For Testing Phase:
- [ ] All syntax checks pass
- [ ] Connectivity test succeeds on all test servers
- [ ] Dry-run completes without errors
- [ ] Single server patching completes successfully
- [ ] Pre-validation correctly stops execution if thresholds exceeded
- [ ] Serial execution patches 1 server, then 20%, then 50%
- [ ] Post-validation runs and reports status
- [ ] Compliance report generates successfully

### For Production Readiness:
- [ ] All tests pass in staging
- [ ] Pure Storage snapshot rollback procedure documented
- [ ] Rollback functionality implemented (separate project)
- [ ] Shell commands replaced with native facts
- [ ] Stakeholders trained on new serial execution behavior

---

## Failure Handling

### If Syntax Check Fails:
1. Check YAML indentation
2. Verify no typos in variable names
3. Review CRITICAL_FIXES_APPLIED.md for recent changes

### If Connectivity Fails:
1. Verify SSH key path in inventory
2. Check network connectivity
3. Verify SSH key permissions (chmod 600)
4. Test SSH manually: `ssh -i key user@server`

### If Pre-Validation Fails:
1. Check validation results in `/tmp/patching_validation/`
2. Adjust thresholds if appropriate
3. Verify maintenance window timing
4. Check backup exists

### If Patching Fails:
1. Check logs in `./logs/ansible.log`
2. Review specific error message
3. Check package manager availability
4. Verify disk space

### If Post-Validation Fails:
1. Check which validation failed
2. Review service status
3. Check application logs
4. Use Pure Storage snapshot for rollback if needed

---

## Rollback Procedure (If Test Fails)

Since automated rollback is not yet functional:

1. **Stop the playbook** (Ctrl+C if running)
2. **Identify affected servers**
3. **Revert using Pure Storage snapshot**:
   - Access Pure Storage management console
   - Select pre-patch snapshot
   - Revert to snapshot
   - Verify system state
4. **Document the failure**
5. **Report findings**

---

## Post-Test Documentation

After testing, document:

- [ ] Test date and time
- [ ] Test environment (testing/staging)
- [ ] Number of servers tested
- [ ] Test results (pass/fail)
- [ ] Any issues encountered
- [ ] Adjustments made
- [ ] Next steps required

---

## Approval for Production

Before running in production:

- [ ] All tests pass in staging
- [ ] Stakeholders review test results
- [ ] Rollback procedure verified (Pure Storage)
- [ ] Maintenance window approved
- [ ] ServiceNow change request created
- [ ] Senior SME approval obtained
- [ ] AWX job template configured (if using AWX)

---

## Quick Test Command Summary

```bash
# 1. Syntax check
ansible-playbook playbooks/patch_linux_servers.yml --syntax-check

# 2. Connectivity test
ansible-playbook -i inventories/testing/hosts.yml playbooks/test_connectivity.yml

# 3. Dry-run
ansible-playbook -i inventories/testing/hosts.yml playbooks/patch_linux_servers_dryrun.yml

# 4. Single server test
ansible-playbook -i inventories/staging/hosts.yml playbooks/patch_linux_servers.yml --limit <server>

# 5. Generate report
ansible-playbook playbooks/generate_compliance_report.yml
```
