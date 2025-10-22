<h1>Lesson 4 – Getting Started with Playbooks (Deep Notes)</h1>

<p><b>Goal:</b> Master playbook creation, syntax validation, and multi-play orchestration 
through both structured YAML understanding and real-world automation use cases.</p>

<h2>Environment Validation</h2>

<ul>
  <li><b>Control Node:</b> <code>control</code></li>
  <li><b>Managed Nodes:</b> <code>ansible1</code>, <code>ansible2</code>, <code>ansible3</code></li>
  <li><b>Inventory Path:</b> <code>./inventory.ini</code> (symlinked from Lesson 2)</li>
  <li><b>Ansible Config:</b> <code>./ansible.cfg</code> (symlinked from Lesson 2)</li>
</ul>

<p align="center">
  <b>Screenshot:</b> Lesson 4 structure before building any playbooks.
  <br>
  <img src="./screenshots/lesson4-structure-initial.PNG" alt="Lesson 4 Folder Layout" width="700"/>
</p>

<hr/>

<h2>4.1 Using YAML to Write Playbooks</h2>
<p><b>Purpose:</b> Learn to describe tasks declaratively using YAML syntax instead of ad-hoc commands.</p>

<ul>
  <li><b>What YAML is:</b> YAML (YAML Ain’t Markup Language) is indentation-based, human-readable data format used by Ansible to define tasks, variables, and roles.</li>
  <li><b>Core Concepts:</b> 
    <ul>
      <li><code>- name:</code> → describes the purpose of a task</li>
      <li><code>hosts:</code> → target group(s) from <code>inventory.ini</code></li>
      <li><code>become:</code> → enables privilege escalation</li>
      <li><code>tasks:</code> → ordered list of modules and their arguments</li>
    </ul>
  </li>
</ul>

<pre><code># Example playbook structure
---
- name: Create a directory on all managed nodes
  hosts: managed
  become: true
  tasks:
    - name: Ensure /opt/lesson4 exists
      ansible.builtin.file:
        path: /opt/lesson4
        state: directory
        mode: '0755'
</code></pre>

<ul>
  <li><b>Command to run:</b>
  <pre><code>ansible-playbook playbooks/01_user_mgmt.yml -v</code></pre></li>
  <li><b>Expected output:</b> 
  <code>changed=true</code> on first run; <code>ok</code> on second — proving idempotency.</li>
</ul>

---

<h2>4.2 Running Playbooks</h2>
<ul>
  <li><b>Command:</b> <code>ansible-playbook playbooks/&lt;playbook_name&gt;.yml</code></li>
  <li><b>Common flags:</b>
    <ul>
      <li><code>-C / --check</code> → dry-run mode</li>
      <li><code>-v</code> → verbose output</li>
      <li><code>--syntax-check</code> → validates YAML structure</li>
    </ul>
  </li>
  <li><b>Exam Note:</b> Always run <code>--syntax-check</code> first to avoid runtime errors.</li>
</ul>

---

<h2>4.3 Verifying Playbook Syntax</h2>
<pre><code>ansible-playbook playbooks/01_user_mgmt.yml --syntax-check
</code></pre>
<ul>
  <li><b>Success Output:</b> “playbook: playbooks/01_user_mgmt.yml”</li>
  <li><b>Failure Output:</b> line/column error message identifying bad indentation or colon alignment.</li>
</ul>

---

<h2>4.4 Using Multiple-Play Playbooks</h2>
<pre><code>---
- name: Configure users
  hosts: managed
  become: true
  tasks:
    - name: Create user lesson4a
      ansible.builtin.user:
        name: lesson4a
        state: present

- name: Deploy MOTD banner
  hosts: managed
  become: true
  tasks:
    - name: Copy banner
      ansible.builtin.copy:
        dest: /etc/motd
        content: "Managed by Lesson 4 Playbook"
</code></pre>

<ul>
  <li><b>Concept:</b> Multiple plays allow complex automation flows in one file, targeting different hosts or performing sequential roles.</li>
  <li><b>Verification:</b>
  <pre><code>ansible all -m ansible.builtin.user -a 'name=lesson4a' -o
ansible all -a 'cat /etc/motd' -o
</code></pre>
  </li>
</ul>

---

<h2>📘 Real-World Projects (10 Playbooks)</h2>

<p>Below are the 10 playbook projects that will build on Sander’s concepts, demonstrating real-world system automation, idempotency, and RHCE-grade logic.</p>

<table>
<tr><th>#</th><th>Playbook</th><th>Description</th></tr>
<tr><td>1</td><td><code>01_user_mgmt.yml</code></td><td>User and group management with SSH keys</td></tr>
<tr><td>2</td><td><code>02_firewalld_rules.yml</code></td><td>Zone, port, and rich-rule automation</td></tr>
<tr><td>3</td><td><code>03_packages_updates.yml</code></td><td>Patch management and package verification</td></tr>
<tr><td>4</td><td><code>04_motd_banner.yml</code></td><td>Dynamic /etc/motd deployment via Jinja2</td></tr>
<tr><td>5</td><td><code>05_multi_service.yml</code></td><td>Deploy Apache + MariaDB multi-play orchestration</td></tr>
<tr><td>6</td><td><code>06_wireguard_setup.yml</code></td><td>WireGuard VPN auto-configuration</td></tr>
<tr><td>7</td><td><code>07_backup_etc.yml</code></td><td>Backup /etc/ and compress with timestamp</td></tr>
<tr><td>8</td><td><code>08_monitor_agent.yml</code></td><td>Install and enable node_exporter agent</td></tr>
<tr><td>9</td><td><code>09_harden_ssh.yml</code></td><td>Enforce SSH security baselines</td></tr>
<tr><td>10</td><td><code>10_local_repo.yml</code></td><td>Create local DNF/YUM repo for offline environments</td></tr>
</table>

---

<h3>Next Step:</h3>
<p>Begin with Playbook #1 (<code>01_user_mgmt.yml</code>) — build, run, verify, and document all outputs. Capture every <code>tree -L 2</code> and <code>ansible-playbook</code> result as proof-of-work.</p>

