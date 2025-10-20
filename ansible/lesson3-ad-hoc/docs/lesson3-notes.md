<h1>Lesson 3 – Ad Hoc Commands (Deep Notes)</h1>

<p><b>Goal:</b> Build, validate, and document ad hoc Ansible commands that prove collection usage, FQCN resolution, and idempotent task behavior across managed nodes.</p>

<h3>Control node</h3>
<ul>
  <li><b>Host:</b> <code>control</code> (all Ansible commands run here)</li>
</ul>

<h3>Managed nodes</h3>
<ul>
  <li><b>Hosts:</b> <code>ansible1</code>, <code>ansible2</code>, <code>ansible3</code> – reachable via SSH as user <b>ansible</b> (key-based auth)</li>
</ul>

<h2>What I built / validated</h2>
<ul>
  <li><b>Collections in use:</b> <code>ansible.posix</code>, <code>community.general</code>, <code>community.mysql</code></li>
  <li><b>FQCN usage verified:</b> <code>community.general.timezone</code> (good replacement to show real collection usage)</li>
  <li><b>Idempotent resources on each host:</b>
    <ul>
      <li>Directories: <code>/opt/lesson3</code>, <code>/opt/lesson3/logs</code></li>
      <li>Managed file: <code>/opt/lesson3/info.txt</code></li>
      <li>User: <code>lesson3</code></li>
      <li>Service: <code>crond</code> <i>(started + enabled)</i></li>
    </ul>
  </li>
</ul>

<p align="center">
  <!-- Screenshot: repo overview -->
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-overview.PNG" />
</p>

<hr/>

<h2>Collections: Install & Search Path</h2>

<p><b>Preferred:</b> Using a project-local <code>requirements.yml</code> and local collections path keeps both the Ansible CLI and <code>ansible-navigator</code> aligned.</p>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/requirements.yml.current%20.PNG" alt="requirements.yml contents"/>
</p>

<p>
The <code>requirements.yml</code> file defines which Ansible collections my project depends on.<br>
For example:
<ul>
  <li><code>ansible.posix</code> – Core Linux system modules</li>
  <li><code>community.general</code> – Broad community-supported modules</li>
  <li><code>community.mysql</code> – MySQL and MariaDB management</li>
</ul>

Maintaining dependencies in this single YAML file lets anyone who clones my repo reproduce the same environment with one command:
</p>

<pre><code>ansible-galaxy collection install -r requirements.yml -p collections</code></pre>

<p align="center">
  <!-- Screenshot: repo overview -->
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/ansible-galaxy%20collection%20install%20-r%20requirements.yml%20-p%20collections.PNG" />
</p>

<h3>Command Breakdown</h3>
<ul>
  <li><code>ansible-galaxy</code> → Ansible’s package manager for roles and collections.</li>
  <li><code>collection install</code> → Installs all collections listed in the file.</li>
  <li><code>-r requirements.yml</code> → Reads the collection list from this YAML file.</li>
  <li><code>-p collections</code> → Installs them locally into a <code>collections/</code> folder, keeping dependencies isolated per project.</li>
</ul>

<h3>Summary</h3>
<p>
This command installs every collection defined in <code>requirements.yml</code>.<br>
<b>Why I ran it:</b> To ensure all required modules are present and available locally for consistent playbook execution.<br>
<b>Result:</b> A self-contained, portable environment that can be cloned and executed anywhere with identical behavior.
</p>
<p><b>Set the collection path in ansible.cfg if needed:</b></p>

``` bash
collections_path = ./collections:~/.ansible/collections:/usr/share/ansible/collections
```

<p align="center">
  <!-- Optional screenshot: galaxy install -->
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/collections_path.PNG" />
</p>

<hr/>

<h2>1) Prove FQCN resolution with a real community module</h2>

<pre><code>ansible all -b -m community.general.timezone -a 'name=EST' --check -o
</code></pre>

<ul>
  <li><b>Why:</b> Confirms that <b>community.general</b> collections are installed and discoverable by Ansible. This test uses <code>--check</code> to verify <i>FQCN</i> (Fully Qualified Collection Name) resolution without changing system state.</li>

  <li><b>What it does:</b> Runs a dry-run ad hoc command on <b>all managed nodes</b>, testing if the <code>community.general.timezone</code> module can set <code>EST</code> timezone. Each host returning <code>CHANGED</code> proves the module was found and would modify the timezone if applied for real.</li>

  <li><b>Options used:</b>
    <ul>
      <li><code>-b</code> → Run with elevated privileges (become/sudo).</li>
      <li><code>-m</code> → Specify which module to use (<code>community.general.timezone</code>).</li>
      <li><code>-a</code> → Provide arguments to the module (<code>name=EST</code>).</li>
      <li><code>--check</code> → Run in dry-run mode (no real changes).</li>
      <li><code>-o</code> → Display output in a single line per host (cleaner for screenshots/logs).</li>
    </ul>
  </li>

  <li><b>Acceptance:</b> Each node displays <code>CHANGED</code> in check mode, confirming successful FQCN resolution.</li>
</ul>

<p align="center">
  <!-- Screenshot: timezone check proof -->
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-fqcn.PNG" alt="FQCN check-mode proof (EST)" />
</p>

<hr/>



<h2>2) Create directories idempotently</h2>

```
# First run (should change)
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3a state=directory mode=0755 owner=root group=root' -o
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3a/logs state=directory mode=0755 owner=root group=root' -o
```
```
# Re-run (should be ok / no change)
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3a state=directory mode=0755 owner=root group=root' -o
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3a/logs state=directory mode=0755 owner=root group=root' -o
```
```
# Verify
ansible all -m ansible.builtin.stat -a 'path=/opt/lesson3a' -o
ansible all -m ansible.builtin.stat -a 'path=/opt/lesson3a/logs' -o
```

<p align="center">
  <!-- Screenshot: idempotent directory proof -->
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-dirs-idempotent.PNG" alt="Idempotent directory management proof" />
</p>
<ul>
  <li><b>Why:</b> I ran this command a second time to demonstrate <i>idempotency</i> — the concept that Ansible only applies changes when something is different.  
  The directories were created on the first run, so the second run should detect no changes and simply report <code>ok</code>.</li>

  <li><b>What it does:</b> Creates two directories (<code>/opt/lesson3a</code> and <code>/opt/lesson3a/logs</code>) with the correct permissions and ownership, 
  then verifies they already exist and match the expected state.</li>

  <li><b>Command breakdown:</b>
    <ul>
      <li><code>-b</code> → Run with sudo privileges (become root).</li>
      <li><code>-m ansible.builtin.file</code> → Uses Ansible’s built-in <b>file module</b>, which manages files, directories, and symlinks on the target system.</li>
      <li><code>-a</code> → Passes arguments: 
        <code>path</code> (location), 
        <code>state=directory</code> (ensures it’s a directory), 
        <code>mode=0755</code> (permissions), 
        <code>owner/group=root</code> (ownership).</li>
      <li><code>-o</code> → Shows one-line output per host for easier reading.</li>
    </ul>
  </li>

  <li><b>Acceptance:</b>
    <ul>
      <li>First run → <code>CHANGED</code> (directories created)</li>
      <li>Second run → <code>ok</code> (no changes — idempotent behavior proven)</li>
      <li><code>stat</code> confirms <code>isdir:true</code>, mode <code>0755</code>, owned by <code>root</code>.</li>
    </ul>
  </li>
</ul>

<h2>3) Manage a file & verify content via hash</h2>

```
# Step 1 — Enforce exact file content + permissions
ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info_v2.txt content="Lesson3 FQCN + idempotency (hash demo)" owner=root group=root mode=0644' -o
```
```
# Verify file integrity via SHA256 hash
ansible all -m ansible.builtin.command -a 'sha256sum /opt/lesson3/info_v2.txt' -o
```
```
# View file content on each host
ansible all -m ansible.builtin.command -a 'cat /opt/lesson3/info_v2.txt' -o
```
```
# Step 2 — Re-run to prove idempotency (no changes expected)
ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info_v2.txt content="Lesson3 FQCN + idempotency (hash demo)" owner=root group=root mode=0644' -o
```
```
# Verify hash remains unchanged (confirming idempotency)
ansible all -m ansible.builtin.command -a 'sha256sum /opt/lesson3/info_v2.txt' -o
```

<p align="center">
  <!-- Screenshot: copy + sha256 verification -->
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-copy-hash.PNG" alt="copy module idempotency + hash verification" />
</p>

<ul>
  <li><b>Why:</b> Demonstrates <i>idempotency</i> with the <code>copy</code> module — showing that identical content and permissions produce no changes when re-run. Also validates file consistency via SHA256 hash.</li>

  <li><b>What it does:</b>
    <ul>
      <li>Creates <code>/opt/lesson3/info_v2.txt</code> with defined content, permissions, and ownership.</li>
      <li>Computes SHA256 checksum across all hosts to verify identical content.</li>
      <li>Re-runs to confirm no changes — proving idempotent behavior.</li>
    </ul>
  </li>

  <li><b>Command breakdown:</b>
    <ul>
      <li><code>-b</code> → Runs with sudo privileges (become root).</li>
      <li><code>-m ansible.builtin.copy</code> → Uses Ansible’s <b>copy module</b> to create or enforce file content and permissions.</li>
      <li><code>-a</code> → Passes module arguments (<code>dest</code>, <code>content</code>, <code>owner/group</code>, <code>mode</code>).</li>
      <li><code>ansible.builtin.command</code> → Runs verification commands (<code>sha256sum</code> and <code>cat</code>) across managed nodes.</li>
      <li><code>-o</code> → Displays compact one-line output per host (clean and exam-friendly).</li>
    </ul>
  </li>

  <li><b>Acceptance:</b>
    <ul>
      <li>First run → <code>CHANGED</code> (file created and hashes identical across all nodes).</li>
      <li>Second run → <code>ok</code> (no change — idempotent behavior confirmed).</li>
      <li><code>sha256sum</code> identical on all hosts; <code>cat</code> output matches expected content.</li>
    </ul>
  </li>
</ul>

<hr/>

<h2>4) Create user <code>lesson3</code> idempotently</h2>

<pre><code>ansible all -b -m ansible.builtin.user -a 'name=lesson3 state=present shell=/bin/bash create_home=yes' -o
# Re-run (no change expected)
ansible all -b -m ansible.builtin.user -a 'name=lesson3 state=present shell=/bin/bash create_home=yes' -o

# Verify
ansible all -m ansible.builtin.getent -a 'database=passwd key=lesson3' -o
ansible all -m ansible.builtin.stat -a 'path=/home/lesson3' -o
</code></pre>

<p align="center">
  <!-- Screenshot: user + getent + stat -->
  <img src="./img/lesson3-user-ss.png" alt="User creation and idempotent re-run" />
</p>

<hr/>

<h2>5) Ensure service <code>crond</code> is started & enabled</h2>

<pre><code>ansible all -b -m ansible.builtin.service -a 'name=crond state=started enabled=yes' -o
# Re-run (no change expected)
ansible all -b -m ansible.builtin.service -a 'name=crond state=started enabled=yes' -o

# Verify current state
ansible all -m ansible.builtin.command -a 'systemctl is-active crond' -o
ansible all -m ansible.builtin.command -a 'systemctl is-enabled crond' -o
</code></pre>

<p align="center">
  <!-- Screenshot: service active+enabled -->
  <img src="./img/lesson3-service-ss.png" alt="Service started/enabled and idempotent" />
</p>

<hr/>

<h2>6) Module schema quick reference</h2>

<pre><code>ansible-doc -s ansible.builtin.file     | head -n 25
ansible-doc -s ansible.builtin.copy     | head -n 25
ansible-doc -s ansible.builtin.user     | head -n 25
ansible-doc -s ansible.builtin.service  | head -n 25
ansible-doc -s community.general.timezone | head -n 25
</code></pre>

<p align="center">
  <!-- Screenshot: ansible-doc summaries -->
  <img src="./img/lesson3-ansible-doc-ss.png" alt="ansible-doc schema summaries" />
</p>

<hr/>

<h2>Ansible Navigator (from Sander notes)</h2>

<ul>
  <li><b>Navigator doc:</b> <code>ansible-navigator doc -m stdout ansible.builtin.ping</code></li>
  <li><b>Use the same project collections:</b> install with <code>-p collections</code> and set <code>collections_paths</code>.</li>
  <li><b>Tip:</b> <code>--pp never</code> avoids pulling newer images during demos/exams.</li>
</ul>

<p align="center">
  <!-- Optional screenshot: navigator doc -->
  <img src="./img/lesson3-navigator-doc-ss.png" alt="ansible-navigator doc output" />
</p>

<hr/>

<h2>Idempotency: exam habits</h2>

<ul>
  <li>Prefer a specific module (<code>file</code>, <code>user</code>, <code>service</code>, <code>copy</code>) over <code>command</code>/<code>shell</code>.</li>
  <li>Prove idempotency: run twice and show <i>ok</i> on the second run.</li>
  <li>Verify state with read-only commands (<code>stat</code>, <code>getent</code>, <code>systemctl is-*</code>, hashes).</li>
  <li>Use <b>FQCN</b> (e.g., <code>community.general.timezone</code>) to avoid ambiguity.</li>
</ul>

<hr/>

<h2>Flags cheat sheet (Lesson 3)</h2>

<table>
<tr><th>Flag</th><th>Meaning</th></tr>
<tr><td><code>-m &lt;module&gt;</code></td><td>Module to run (e.g., <code>ansible.builtin.file</code>)</td></tr>
<tr><td><code>-a &lt;args&gt;</code></td><td>Arguments to the module</td></tr>
<tr><td><code>-b</code></td><td>Become (sudo)</td></tr>
<tr><td><code>-o</code></td><td>One-line output per host</td></tr>
<tr><td><code>--check</code></td><td>Dry run (plan) without changing state</td></tr>
</table>

<hr/>

<i>End of Lesson 3 notes</i>
