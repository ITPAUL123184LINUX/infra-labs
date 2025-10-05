# Leson 2 – Managed Environment (Deep Notes)
Control node: control                         (I run all Ansible commands here)
Managed nodes: ansible1, ansible2, ansible3   (reachable via SSH as ansible user with key)

# What I built
Inventory file at inventory.ini           
ansible.cfg in this folder to:
set default inventory to             ./inventory.ini
use remote user ansible and key       /home/ansible/.ssh/id_ed25519
enable sudo                           (become=True, become_method=sudo)
use Python 3 interpreter on targets
disable host key checking (lab only)

# inventory.ini     (inventory file)
[managed]
ansible1
ansible2
ansible3

[managed:vars]
ansible_user=ansible
ansible_python_interpreter=/usr/bin/python3

-  [managed] is a host group I’ll target in commands/playbooks.
-  Group vars apply to all hosts in managed.

# ansible.cfg
[defaults]
inventory = ./inventory.ini          					                    # default inventory (no -i needed)
remote_user = ansible                					                    # SSH user
private_key_file = /home/ansible/.ssh/id_ed25519
host_key_checking = False             					                  # LAB ONLY; set True in prod
interpreter_python = /usr/bin/python3 # use Python 3 on remotes

[privilege_escalation]
become = True
become_method = sudo
become_ask_pass = False              					                     # NOPASSWD sudoers in lab

# Commands I ran (what / why / flags)
-  All run on the control node inside ~/infra-labs/ansible/lesson2-managed-env

# 1) Visualize inventory
ansible-inventory -i inventory.ini --graph
-i        use this inventory file (optional if ansible.cfg already points to it)
--graph   prints groups/hosts tree; sanity check that Ansible sees all hosts

# @all:
  |--@managed:
  |  |--ansible1
  |  |--ansible2
  |  |--ansible3

# 2) Confirm which config Ansible is using
ansible --version | sed -n '1p;/config file/ p'
- Shows Ansible version and the active config file path (should be this folder’s ansible.cfg)

# 3) Dump effective settings (filtered)
ansible-config dump | egrep 'DEFAULT_INVENTORY|DEFAULT_REMOTE_USER|DEFAULT_PRIVATE_KEY_FILE|HOST_KEY_CHECKING|DEFAULT_INTERPRETER_PYTHON|BECOME'   
-  Proves our key defaults are applied (inventory, remote_user, key, interpreter, become)

# 4) Prove SSH/key works out-of-band (one host)
ssh -i /home/ansible/.ssh/id_ed25519 ansible@ansible1 'hostname; id'
-  If this fails, Ansible will fail. Fix SSH before proceeding. 

# 5) ICMP test to all (module = ping)
ansible all -m ping -o
all: the implicit group of all hosts
-m ping: runs Ansible’s ping module (checks Python/SSH, not ICMP)
-o: one-line output per host
 ansible1 | SUCCESS => {"changed": false, "ping": "pong"}

# 6) Verify privilege escalation (become/sudo)
ansible all -b -m command -a 'id' -o
-b: become (sudo as root)
-m command + -a 'id': run the id command
uid=0(root) gid=0(root) ... which proves passwordless sudo is working

#  Flags cheat sheet (used today)
-i <inv> — inventory path (overrides ansible.cfg)
--graph — tree view of inventory
-m <module> — which module to run (e.g., ping, command)
-a "<args>" — arguments passed to the module (for command or shell)
-b — become (sudo)
-o — one line per host

-u <user> — override remote_user (normally not needed here)
