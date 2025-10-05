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
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson2-managed-env/docs/img/Capture.PNG" alt="Inventory File">
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
<br />
<br />
<p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson2-managed-env/docs/img/Capture.ansible.cfg.PNG" alt="Ansible.cfg">
<br />
<br />

 ### Commands I ran
 <p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson2-managed-env/docs/img/Ansible_CMDS_L2.PNG">

   - <b>ansible-inventory -i inventory.ini --graph          :   Displays the group hierarchy from the static inventory file, confirming host grouping @managed → ansible1–3.</b>

   - <b>ansible --version | sed -n '1p;/config file/ p          :   Prints Ansible version and the active config file path to verify you’re using the intended `ansible.cfg.</b>

<br />
<br />
Wait for process to complete (may take some time):  <br/>
<img src="https://i.imgur.com/JL945Ga.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Sanitization complete:  <br/>
<img src="https://i.imgur.com/K71yaM2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Observe the wiped disk:  <br/>
<img src="https://i.imgur.com/AeZkvFQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
