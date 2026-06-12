# CNS 4107 Network Automation – Ansible WSL DevNet Lab
**Student:** Rehema Lembo  
**Course:** CNS 4107 Network Automation  
**Institution:** Strathmore University  
**Date:** June 12, 2026  

---

## Overview
This lab demonstrates network automation using Ansible running on Windows Subsystem for Linux (WSL) connected to a Cisco Catalyst 9000 Always-On DevNet Sandbox running IOS XE 17.15.1.

---

## Environment
| Item | Detail |
|---|---|
| Control Node | Ubuntu 24.04 on WSL2 |
| Ansible Version | core 2.21.0 |
| Python Version | 3.12.3 |
| Target Device | Cisco Catalyst 9000 (C9KV-UADP-8P) |
| IOS XE Version | 17.15.1 |
| Sandbox | Cisco DevNet Always-On Catalyst 9000 |
| Collections | cisco.ios, ansible.netcommon, community.general |

---

## What Was Done

### Part A – WSL and Ansible Setup
- Installed Ubuntu 24.04 on WSL2 via PowerShell
- Created a Python virtual environment under `~/cns4107/ansible-wsl-devnet-lab`
- Installed ansible-core 2.21.0 and dependencies
- Installed cisco.ios, ansible.netcommon, and community.general collections

### Part B – Project Structure
- Created all required project folders: inventories, group_vars, host_vars, templates, playbooks, outputs, backups, rendered, reports
- Created ansible.cfg, requirements.txt, and collections/requirements.yml

### Part C – Inventory and Variables
- Created YAML inventory pointing to the Catalyst 9000 DevNet sandbox
- Created group_vars/iosxe.yml with connection variables, NTP servers, syslog servers, loopback intent, and banner
- Created host_vars/catalyst9k.yml with host-specific metadata and report tags
- Validated inventory with ansible-inventory --graph

### Part D – Connectivity Testing
- Confirmed DNS resolution and TCP port 22 reachability
- Ran manual SSH test to the Catalyst 9000
- Successfully ran first Ansible ad hoc commands using cisco.ios.ios_command

### Part E – Playbooks, Facts and Backups
- **00_preflight.yml:** Verified inventory variables and retrieved device clock
- **01_show_commands.yml:** Retrieved show version, show ip interface brief, and filtered running config
- **02_gather_facts.yml:** Gathered structured facts and saved to outputs/catalyst9k_facts.json
- **03_backup_running_config.yml:** Backed up 19KB running configuration to backups/catalyst9k_running.cfg

### Part F – Jinja2 Templating
- Created loopbacks.j2 template using for loops and variables
- Created management_services.j2 template for NTP, syslog, and banner
- Rendered both templates locally using 04_render_config_locally.yml
- Verified rendered IOS XE CLI output before any deployment

### Part G – Dry-Run Deployment
- Created 05_check_mode_diff.yml to test configuration against the device
- Ran playbook with --check --diff flags to preview changes safely

### Part H – Resource Modules
- **07_resource_rendered_offline.yml:** Used cisco.ios.ios_interfaces in rendered state to generate CLI from structured YAML without connecting to the device
- **08_resource_gathered_readonly.yml:** Used gathered state to collect current interface state as structured JSON, saved to outputs/catalyst9k_interfaces_gathered.json

### Part I – Report Generation
- Created device_report.md.j2 Jinja2 template
- Generated reports/catalyst9k_report.md from inventory and variable data

---

## Errors Encountered and Fixes

### 1. community.general.yaml callback removed
**Error:** `The 'community.general.yaml' callback plugin has been removed`  
**Fix:** Changed `stdout_callback = yaml` to `stdout_callback = default` and added `result_format = yaml` in ansible.cfg

### 2. ansible_network_os undefined in playbook
**Error:** `'ansible_network_os' is undefined`  
**Cause:** Ansible 2.21 does not resolve connection variables from group_vars during task finalization  
**Fix:** Added connection variables explicitly in each playbook under the `vars` section

### 3. lab_allow_changes and lab_loopbacks undefined
**Error:** Variables from group_vars not available in template tasks  
**Fix:** Defined all required variables directly in each playbook's `vars` block

### 4. ios_config src path not found
**Error:** `path specified in src not found`  
**Cause:** ios_config src looks for files relative to the managed device, not localhost  
**Fix:** Used ansible.builtin.slurp to read the file on localhost and pass content directly

### 5. Duplicate option in ansible.cfg
**Error:** `option 'deprecation_warnings' already exists`  
**Cause:** Used >> append which added the option into the wrong section  
**Fix:** Rewrote ansible.cfg cleanly with all options in the correct sections

---

## Key Lessons Learned

1. **Ansible 2.21 is stricter** about variable resolution during task finalization — connection variables must be explicitly defined in the playbook `vars` block, not just in group_vars.

2. **Network devices are fundamentally different** from Linux servers in Ansible — they use `network_cli` connection plugin and cannot run Python modules directly.

3. **Jinja2 separates data from syntax** — storing intent in YAML and rendering CLI with templates makes automation reusable, reviewable, and maintainable.

4. **Resource modules go further than templates** — they can render, gather, parse, merge, and replace structured network resources without writing CLI manually.

5. **Always back up before changing** — the backup playbook saved the full running config before any dry-run was attempted.

6. **Check mode and diff mode are essential** — running with --check --diff lets you safely preview what would change on a shared sandbox.

7. **Facts are structured data** — gathered facts can be saved as JSON and used for compliance checking, documentation, and conditional automation.

---

## Deliverables Checklist
- [x] ansible.cfg
- [x] inventories/devnet.yml
- [x] group_vars/iosxe.yml
- [x] host_vars/catalyst9k.yml
- [x] playbooks/00_preflight.yml through 09_generate_report.yml
- [x] templates/loopbacks.j2
- [x] templates/management_services.j2
- [x] templates/device_report.md.j2
- [x] outputs/catalyst9k_facts.json
- [x] outputs/catalyst9k_interfaces_gathered.json
- [x] rendered/catalyst9k_loopbacks.cfg
- [x] rendered/catalyst9k_management_services.cfg
- [x] backups/catalyst9k_running.cfg
- [x] reports/catalyst9k_report.md
- [x] command_history.txt
