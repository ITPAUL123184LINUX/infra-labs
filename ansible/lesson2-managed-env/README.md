# Lesson 2 - Managed Enviorment (Inventory + SSH + sudo)

## Goal

Control node can reach ansible 1-3 as user 'ansible' with SSH keys and run sudo without a password.


## Environment

Control: control
Managed: ansible1, ansible2, ansible3
Remote user: ansible
Key: /home/ansible/.ssh/id_ed25519
Python: /usr/bin/python3
Privilege escalation: sudo (no password)

## What I did (headlines only)
- Created 'inventory.ini' with group '[managed]' and hosts.
- Wrote 'ansible.cfg' pointing to the inventory, remote_user, key, sudo, python3.
- Verified inventory graph, ping, and sudo (root_id).

## Evidence (short)
- 'ansible-inventory --graph' showed ansible1/2/3 under [managed].
- 'ansible all -m ping -o' returned "pong" from all three.
- 'ansible all -b -m command -a 'id' -o' showed 'uid=0(root)' on each host.

# Problems & fixes (today)
- SSH permission denied -- fixed by pointing 'private_key_file' in ansible.cfg.
- Wrong inventory path warnings -- fixed by running from this folder. 

## Done checklist
[X] inventory.ini present
[X] ansible.cfg present
[X] ping works 
[X] sudo (id as root) works 
