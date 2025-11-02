# Lesson 4 – Getting Started with Playbooks
**Goal:** Playbook syntax, idempotency, multi-play.

## Playbook Index (what each teaches)
1) 01_user_mgmt.yml — users, groups, SSH keys  
2) 02_firewalld_rules.yml — zones, ports, rich rules  
3) 03_package_mgmt.yml — install/update/remove pkgs  
4) 04_motd_banner.yml — Jinja2 template to /etc/motd  
5) 05_multi_service.yml — Apache + MariaDB (multi-play)  
6) 06_wireguard_setup.yml — VPN automation  
7) 07_backup_etc.yml — tar.gz of /etc with timestamp  
8) 08_monitor_agent.yml — node_exporter agent  
9) 09_harden_ssh.yml — SSH baselines  
10) 10_local_repo.yml — offline DNF repo

## Verify & Proof
ansible-playbook playbooks/01_user_mgmt.yml --syntax-check
ansible-playbook playbooks/01_user_mgmt.yml -v
tree -L 2
