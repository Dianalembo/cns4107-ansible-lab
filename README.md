# Ansible WSL DevNet Lab
**Rehema Lembo  168297  
**Date:** June 12, 2026  


This lab demonstrates network automation using Ansible running on Windows Subsystem for Linux (WSL) connected to a Cisco Catalyst 9000 Always-On DevNet Sandbox 

## Environment
 Ubuntu 24.04 on WSL2 

the device that was target is Cisco Catalyst 9000 (C9KV-UADP-8P) running IOS XE Version | 

## lab flow
###  WSL and Ansible Setup
- we Installed Ubuntu 24.04 on WSL2 using PowerShell
- we Created a Python virtual environment under `~/cns4107/ansible-wsl-devnet-lab`
- we Installed dependencies
- we Installed cisco.ios, ansible.netcommon, and community.general collections

### Project Structure
- we Created all required project folders: inventories, group_vars, host_vars, templates, playbooks, outputs, backups, rendered, reports
- we Created ansible.cfg, requirements.txt, and collections/requirements.yml

### Inventory and Variables
- an invetory pointing to catalyst 900 (Dev sandbox) was created
- Created group_vars/iosxe.yml with connection variables, NTP servers, syslog servers, loopback intent, and banner
-we Created the YAML file host_vars/catalyst9k.yml with host-specific metadata and report tags
- the inventory was validated using ansible-inventory

### Connectivity Testing
- Confirmed DNS resolution and TCP port 22 reachability
- Ran manual SSH test to the Catalyst 9000
- Successfully ran first Ansible ad hoc commands using cisco.ios.ios_command

### Playbooks, Facts and Backups
- **00_preflight.yml:** Verified inventory variables and retrieved device clock
- **01_show_commands.yml:** Retrieved show version, show ip interface brief, and filtered running config
- **02_gather_facts.yml:** Gathered structured facts and saved to outputs/catalyst9k_facts.json
- **03_backup_running_config.yml:** Backed up 19KB running configuration to backups/catalyst9k_running.cfg

### Jinja2 Templating
- Created loopbacks.j2 template using for loops and variables
- Created management_services.j2 template for NTP, syslog, and banner
- Rendered both templates locally using 04_render_config_locally.yml
- Verified rendered IOS XE CLI output before any deployment

### Dry-Run Deployment
- Created 05_check_mode_diff.yml to test configuration against the device
- Ran playbook with --check --diff flags to preview changes safely

### Resource Modules
- **07_resource_rendered_offline.yml:** Used cisco.ios.ios_interfaces in rendered state to generate CLI from structured YAML without connecting to the device
- **08_resource_gathered_readonly.yml:** Used gathered state to collect current interface state as structured JSON, saved to outputs/catalyst9k_interfaces_gathered.json

### Report Generation
- Created device_report.md.j2 Jinja2 template
- Generated reports/catalyst9k_report.md from inventory and variable data



## Errors Encountered and Fixes

### the first error encounter is on community.general.yaml callback removed
**we receive this as an error message**`The 'community.general.yaml' callback plugin has been removed`  
**Fix:** we fixed it by Changing the `stdout_callback = yaml` to `stdout_callback = default` and adding `result_format = yaml` in ansible.cfg

### the second error was on ansible_network_os undefined in playbook
**the error message received was** `'ansible_network_os' is undefined`  
the cause of this second error was that the **Ansible 2.21 does not resolve connection variables from group_vars during task finalization**
**Fix:** we fixed it by adding connection variables explicitly in each playbook under the `vars` section

### the third error we encounter was on lab_allow_changes and lab_loopbacks undefined
**the error message we received was on the** Variables from group_vars not available in template tasks  
**we fixed by** Defining all required variables directly in each playbook's `vars` block

### the fourth error was on the ios_config src path not found
**we receive the following message as an error message** `path specified in src not found`  
**the cause of the previous errpr message was that** ios_config src looks for files relative to the managed device, not localhost  
**Fix:** Used ansible.builtin.slurp to read the file on localhost and pass content directly

### the last error we encounter was on duplicate option in ansible.cfg
**Error:** `option 'deprecation_warnings' already exists`  
**Cause:** Used >> append which added the option into the wrong section  
**Fix:** Rewrote ansible.cfg cleanly with all options in the correct sections


## Key Lessons Learned
through out this lab we learnt several lessons which are the following ones:

1. **Ansible 2.21 is stricter** on resolutions of variable during task finalization, the connection variable must be explicitly defined in the playbook `vars` block and not just in group_vars.

3. **Network devices are fundamentally different**, Linux servers in Ansible uses `network_cli` connection plugin and cannot run Python modules directly.

4. **Jinja2 separates data from syntax** the fact of storing rendering CLI and YAML with templates makes automation reusable, reviewable and maintable

7. **Resource modules go further than templates** they can render, gather, parse, merge, and replace structured network resources without writing CLI manually.

8. **Always back up before changing**  the full running config is save in the backup before any dry-run attempt.

9. **Check mode and diff mode are essential** to preview safely we run with --check --diff and this would allow to view what change on a shared sandbox.

10. **Facts are structured data** gathered facts can be saved as JSON and used for compliance checking, documentation, and conditional automation.

