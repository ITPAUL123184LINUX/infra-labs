Lesson 2 – Managed Environment (Deep Notes)
==========================================

Control node:
  - Host: control (all Ansible commands run here)

Managed nodes:
  - ansible1, ansible2, ansible3
  - Reachable via SSH as user: ansible (key-based auth)

What I built
------------
Inventory file: inventory.ini (describes the managed group)
Ansible config : ansible.cfg in the same folder, sets:
  - default inventory path to ./inventory.ini
  - remote_user = ansible
  - private_key_file = /home/ansible/.ssh/id_ed25519
  - become=True with sudo (NOPASSWD in lab)
  - interpreter_python = /usr/bin/python3
  - host_key_checking = False (LAB ONLY)

----------------------------------------------------------------------
inventory.ini (content)
----------------------------------------------------------------------

[managed]
ansible1
ansible2
ansible3

[managed:vars]
ansible_user=ansible
ansible_python_interpreter=/usr/bin/python3

Notes:
  - "managed" is the host group I target in commands/playbooks.
  - Group vars under [managed:vars] apply to all hosts in the group.

----------------------------------------------------------------------
ansible.cfg (content)
----------------------------------------------------------------------

[defaults]
inventory = ./inventory.ini
remote_user = ansible
private_key_file = /home/ansible/.ssh/id_ed25519
host_key_checking = False               # LAB ONLY; set True in prod
interpreter_python = /usr/bin/python3   # use Python 3 on remotes

[privilege_escalation]
become = True
become_method = sudo
become_ask_pass = False                 # NOPASSWD sudoers in lab

----------------------------------------------------------------------
Commands I ran (what / why / flags)
All run on the control node inside: ~/infra-labs/ansible/lesson2-managed-env
----------------------------------------------------------------------

1) Visualize inventory

   CMD> ansible-inventory -i inventory.ini --graph

   Why: sanity check that Ansible sees all hosts/groups.
   Flags:
     -i path    : use this inventory file (optional if ansible.cfg points here)
     --graph    : tree view of inventory

   Expected example:
     @all:
       |--@ungrouped:
       |--@managed:
       |  |--ansible1
       |  |--ansible2
       |  |--ansible3

2) Confirm which config file Ansible is using

   CMD> ansible --version | sed -n '1p;/config file/ p'

   Why: shows Ansible version and the active config file path.
        It should point to this folder’s ansible.cfg.

3) Dump effective settings (filtered to key defaults)

   CMD> ansible-config dump | egrep 'DEFAULT_INVENTORY|DEFAULT_REMOTE_USER|DEFAULT_PRIVATE_KEY_FILE|HOST_KEY_CHECKING|DEFAULT_INTERPRETER_PYTHON|BECOME'

   Why: proves our defaults are applied (inventory, remote_user, key,
        host key checking, python interpreter, become).

4) Prove SSH/key works out-of-band (one host)

   CMD> ssh -i /home/ansible/.ssh/id_ed25519 ansible@ansible1 'hostname; id'

   Why: if this fails, Ansible will fail. Fix SSH before proceeding.

5) "Ping" all hosts using Ansible’s ping module

   CMD> ansible all -m ping -o

   Notes:
     - "all" is the implicit group containing all hosts.
     - The ping module checks Python/SSH (not ICMP).
     - -o prints one line per host.

   Expected success line example:
     ansible1 | SUCCESS => {"changed": false, "ping": "pong"}

6) Verify privilege escalation (become/sudo) works

   CMD> ansible all -b -m command -a 'id' -o

   Why: proves passwordless sudo.
   Expected output includes:
     uid=0(root) gid=0(root) groups=0(root)

----------------------------------------------------------------------
Flags cheat sheet used today
----------------------------------------------------------------------

-i <path>           : inventory file path (overrides ansible.cfg)
--graph             : inventory tree view
-m <module>         : which module to run (e.g., ping, command, shell)
-a "<args>"         : arguments passed to the module
-b                  : become (sudo)
-o                  : one-line output per host
-u <user>           : override remote_user (normally not needed here)

----------------------------------------------------------------------
Troubleshooting notes
----------------------------------------------------------------------

A) "Permission denied (publickey, gssapi-keyex, gssapi-with-mic, password)"
   - Ensure the control node can SSH with the same key:
       ssh -i /home/ansible/.ssh/id_ed25519 ansible@ansible1
   - Check file perms:
       chmod 700 /home/ansible/.ssh
       chmod 600 /home/ansible/.ssh/id_ed25519
       chown -R ansible:ansible /home/ansible/.ssh
   - If using different usernames per host, set ansible_user in inventory.

B) "Failed to find interpreter" or Python errors
   - Install Python 3 on remotes or set ansible_python_interpreter
     (we set it via group vars in inventory.ini).

C) "sudo: a password is required"
   - In lab, ensure NOPASSWD for user "ansible" in sudoers, or use become_ask_pass.

D) Host key checking failures in lab
   - We disable in ansible.cfg (host_key_checking=False). In prod, leave it True.

E) Ansible can reach only localhost
   - Ensure ansible.cfg "inventory" points at ./inventory.ini OR use -i explicitly.
   - Confirm DNS or /etc/hosts entries resolve ansible1/2/3 from control.

F) GSSAPI delays or odd SSH auth negotiation
   - You can test with:
       ssh -o PreferredAuthentications=publickey -i /home/ansible/.ssh/id_ed25519 ansible@ansible1
   - Or set SSH options in ansible.cfg via "ssh_args" if needed.

----------------------------------------------------------------------
End of Lesson 2 notes (plain text)
----------------------------------------------------------------------

