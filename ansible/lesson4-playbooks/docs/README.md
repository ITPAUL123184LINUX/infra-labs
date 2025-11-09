# Lesson 4 – Getting Started with Playbooks

## Objective
Master how to create, validate, and run Ansible playbooks that deliver **repeatable, secure, and idempotent** automation.  
This lesson transforms raw ad-hoc commands into structured YAML workflows — the foundation of real-world infrastructure automation.

---

## Learning Focus
- Understand **YAML syntax** and indentation rules  
- Execute **multi-play playbooks** that combine different automation stages  
- Validate syntax using `--syntax-check` and `--check` (dry-run)  
- Build **modular, reusable** playbooks that handle users, services, security, and backups  

---

## Lab Environment
| Role | Hostname | Purpose |
|------|-----------|----------|
| Control Node | `control` | Runs playbooks and manages inventory |
| Managed Nodes | `ansible1`, `ansible2`, `ansible3` | Targets for automation testing |
| Config Files | `ansible.cfg`, `inventory.ini` | Symlinked from Lesson 2 for consistency |

---

## Repository Structure
```bash
lesson4-playbooks/
├── ansible.cfg
├── inventory.ini
├── playbooks/
│   ├── 01_user_mgmt.yml
│   ├── 02_firewalld_rules.yml
│   ├── 03_package_mgmt.yml
│   ├── 04_motd_banner.yml
│   ├── 05_multi_service.yml
│   ├── 06_wireguard_setup.yml
│   ├── 07_backup_etc.yml
│   ├── 08_monitor_agent.yml
│   ├── 09_harden_ssh.yml
│   └── 10_local_repo.yml
└── docs/
    └── screenshots/

