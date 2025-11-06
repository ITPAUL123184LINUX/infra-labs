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
  <br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/docs/screenshots/lesson4-tree.PNG"/>
</p>

<hr/>

<h2>4.1 Using YAML to Write Playbooks</h2>
<p><b>Purpose:</b> Learn to describe tasks declaratively using YAML syntax instead of ansible ad-hoc commands.</p>

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

<h2>Example playbook structure</h2>
<p align="center">
  <br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/docs/screenshots/L4TEST.yml.PNG"/>
</p>


<ul>
  <li><b>Command to run:</b>
  <pre><code>ansible-playbook playbooks/L4TEST.yml</code></pre></li>
  <li><b>Expected output:</b> 
  <p align="center">
  <br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/docs/screenshots/L4TEST.outpt.PNG"/>
</p>

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

  <p align="center">
  <br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/docs/screenshots/lesson4-syntax-check.PNG"/>
</p>

  <li><b>Success Output:</b> “playbook: playbooks/01_user_mgmt.yml”</li>
  <li><b>Failure Output:</b> line/column error message identifying bad indentation or colon alignment.</li>
</ul>

---

<h2>4.4 Using Multiple-Play Playbooks</h2>
<p align="center">
  <br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/docs/screenshots/multi-play.yml.PNG"/>
</p>

<p align="center">
  <br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/docs/screenshots/multi-play.yml-otpt.PNG"/>
</p>

<ul>
  <li><b>Concept:</b> This multi-play playbook demonstrates how to execute multiple automation stages sequentially in one file. 
  Each play defines its own hosts, privilege level, and task list. The first creates a user (<code>lesson4b</code>), the second deploys a customized 
  Message of the Day banner.</li>

  <li><b>Verification Commands:</b>
  <pre><code>ansible all -m ansible.builtin.user -a 'name=lesson4b' -o
ansible all -a 'cat /etc/motd' -o
</code></pre></li>

  <li><b>Expected Output:</b> Each node has the <code>lesson4b</code> user and displays the MOTD banner text “Configured by Lesson 4 Multi-Play Example.”</li>

  <p align="center">
    <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/docs/screenshots/multi-list.verify.PNG"/>
  </p>
</ul>

---

<h2>📘 Playbook Projects (10 Playbooks)</h2>

<p>Below are the 10 playbook projects — each one links directly to its corresponding YAML file on GitHub for instant reference and execution.</p>

<!-- START: Playbook Index -->
<table>
<tr><th>#</th><th>Playbook</th><th>Description</th></tr>

<tr><td>1</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/01_user_mgmt.yml" target="_blank"><code>01_user_mgmt.yml</code></a></td><td>Users, groups, SSH keys</td></tr>

<tr><td>2</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/02_firewalld_rules.yml" target="_blank"><code>02_firewalld_rules.yml</code></a></td><td>Zones, ports, rich rules</td></tr>

<tr><td>3</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/03_package_mgmt.yml" target="_blank"><code>03_package_mgmt.yml</code></a></td><td>Install/update/remove packages</td></tr>

<tr><td>4</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/04_motd_banner.yml" target="_blank"><code>04_motd_banner.yml</code></a></td><td>Jinja2 MOTD</td></tr>

<tr><td>5</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/05_multi_service.yml" target="_blank"><code>05_multi_service.yml</code></a></td><td>Apache + MariaDB (multi-play)</td></tr>

<tr><td>6</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/06_wireguard_setup.yml" target="_blank"><code>06_wireguard_setup.yml</code></a></td><td>WireGuard VPN</td></tr>

<tr><td>7</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/07_backup_etc.yml" target="_blank"><code>07_backup_etc.yml</code></a></td><td>Timestamped /etc backup</td></tr>

<tr><td>8</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/08_monitor_agent.yml" target="_blank"><code>08_monitor_agent.yml</code></a></td><td>node_exporter</td></tr>

<tr><td>9</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/09_harden_ssh.yml" target="_blank"><code>09_harden_ssh.yml</code></a></td><td>SSH hardening</td></tr>

<tr><td>10</td><td><a href="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson4-playbooks/playbooks/10_local_repo.yml" target="_blank"><code>10_local_repo.yml</code></a></td><td>Local DNF repo</td></tr>

</table>
<!-- END: Playbook Index -->

---

