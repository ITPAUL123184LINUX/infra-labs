<h1>Lesson 2 – Managed Environment (Deep Notes)</h1>

 ### Control node
 - <b>Host: control (all Ansible commands run here)</b> 

<h2>Managed nodes</h2>
<b>ansible1, ansible2, ansible3</b> 

<b>Reachable via SSH as user: ansible (key-based auth)</b>

<h2>What I built </h2>

- <b>Inventory file: inventory.ini</b>
- <b>Ansible config : ansible.cfg</b>
- <b>default inventory path to ./inventory.ini</b>
- <b>remote_user = ansible</b>
- <b>private_key_file = /home/ansible/.ssh/id_ed25519</b>
- <b>become=True with sudo (NOPASSWD in lab)</b>
- <b>interpreter_python = /usr/bin/python3</b>
- <b>host_key_checking = False (LAB ONLY)
</b>
  
<h2>inventory.ini</h2>

- <b>managed" is the host group I target in commands/playbooks.</b>


<p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson2-managed-env/docs/img/iventory.ini.ss.PNG">
</p>

<h2>ansible.cfg</h2>
- <b>All run on the control node inside: ~/infra-labs/ansible/lesson2-managed-env</b>

 - <b>inventory = ./inventory.ini          :   Uses the repo-local inventory so every command works from ansible/lesson2-managed-env/ without -i.</b>

 - <b>remote_user = ansible          :   Default SSH user for modules/commands; avoids typing -u ansible each time.</b>

  - <b>private_key_file = /home/ansible/.ssh/id_ed25519          :   Forces the exact key used for SSH auth; no agent confusion on shared boxes.</b>

  - <b>host_key_checking = False (LAB ONLY)          :   Skips strict host key prompts so lab runs are non-interactive; never use in prod.</b>

  - <b>interpreter_python = /usr/bin/python3          :   Pins Python location on targets so modules don’t fail on systems with multiple Pythons.</b>

   - <b>become = True          :   Elevates by default so tasks needing root don’t sporadically fail.</b>

   - <b>become_method = sudo          :   Explicitly uses sudo (standard on RHEL9); consistent with NOPASSWD lab config.</b>

   - <b>become_ask_pass = False          :   No password prompt during privilege escalation; required for unattended runs.</b>
   
   - <b>- collections_path = ./collections:~/.ansible/collections:/usr/share/ansible/collections : Specifies where Ansible should search for collections. Think of a collection as a package of reusable automation content — it can include roles, modules, plugins, and documentation. Paths are checked in order: project-level, then user-level, then system-wide.</b>
.</b>
<br />
<br />
<p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson2-managed-env/docs/img/ansible.cfg.ss.PNG">
<br />
<br />

 ### Commands I ran
 <p align="center">
<img src="https://raw.githubusercontent.com/ITPAUL123184LINUX/infra-labs/main/ansible/lesson2-managed-env/docs/img/Ansible_CMDS_L2.PNG" style="width:100%; max-width:1200px;" />
  
		
	ansible-inventory -i inventory.ini --graph

	
- <b>Displays the group hierarchy from the static inventory file, confirming host grouping @managed → ansible1–3.</b>

```bash
ansible --version | sed -n '1p;/config file/ p'
```
  - <b>Prints Ansible version and the active config file path to verify you’re using the intended `ansible.cfg.</b>
  ```bash
  ansible-config dump | egrep 'DEFAULT_INVENTORY|DEFAULT_REMOTE_USER|DEFAULT_PRIVATE_KEY_FILE|HOST_KEY_CHECKING|DEFAULT_INTERPRETER_PYTHON|BECOME'.
  ```
  - <b>This CMD proves our defaults are applied (inventory, remote_user, key, host key checking, python interpreter, become).</b>
  ```bash
  ssh -i /home/ansible/.ssh/id_ed25519 ansible@ansible1 'hostname; id'
  ```
  - <b>SSH connection was failing, I tested it manually to confirm the key and user work outside of Ansible. Ran `hostname` and `id` on ansible1.</b>
		
```bash
ansible all -m ping -o
```
- <b>Ping all managed hosts in inventory using Ansibles ping module. -o prints everything on one line.</b>

- <b> -o prints the output on one line per host.</b>

```bash
ansible all -b -m command -a 'id' -o
```

- <b>Verifies passwordless `sudo` (`become`) works across all hosts.</b>
		
---

## 🧠 Flags Cheat Sheet (Lesson 2)

| Flag         | Description                                             |
|--------------|---------------------------------------------------------|
| `-i <path>`  | Inventory file path (overrides `ansible.cfg`)           |
| `--graph`    | Inventory tree view                                     |
| `-m <module>`| Which module to run (e.g., ping, command, shell)        |
| `-a <args>`  | Arguments passed to the module                          |
| `-b`         | Become (sudo)                                           |
| `-o`         | One-line output per host                                |
| `-u <user>`  | Override `remote_user` (normally not needed here)       |

---

_End of Lesson 2 notes_
