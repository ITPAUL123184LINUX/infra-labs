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
  




<h2>Program walk-through:</h2>

<p align="center">
Launch the utility: <br/>
<img src="https://i.imgur.com/62TgaWL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Select the disk:  <br/>
<img src="https://i.imgur.com/tcTyMUE.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Enter the number of passes: <br/>
<img src="https://i.imgur.com/nCIbXbg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Confirm your selection:  <br/>
<img src="https://i.imgur.com/cdFHBiU.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
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
