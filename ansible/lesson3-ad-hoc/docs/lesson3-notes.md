<h1>Lesson 3 – Ad Hoc Commands (Deep Notes)</h1>

<h3>Control node</h3>
<ul>
  <li><b>Host:</b> <code>control</code> (all Ansible commands run here)</li>
</ul>

<h3>Managed nodes</h3>
<p><b>ansible1, ansible2, ansible3</b> – reachable via SSH as user <b>ansible</b> (key-based auth)</p>

<h2>What I built / validated</h2>
<ul>
  <li><b>Collections in use:</b> <code>ansible.posix</code>, <code>community.general</code></li>
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

<h2>Collections: install & search path</h2>

<p><b>Preferred:</b> use a project-local <code>requirements.yml</code> and a project-local collections path so
both the CLI and ansible-navigator find the same content.</p>

<p
<p align="center">
  <!-- Screenshot: repo overview -->
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/requirements.yml.PNG" />
</p>

<p>
The <code>requirements.yml</code> file defines which Ansible collections my project depends on.  
Each collection — such as <code>ansible.posix</code> (for system-level tasks) and <code>community.general</code> (for extra community modules) — extends Ansible’s core functionality.  
Storing these dependencies in a single YAML file ensures anyone cloning my repository can reproduce the same environment I have with one command:  
<code>ansible-galaxy collection install -r requirements.yml</code>.
</p>

<pre><code># Install into a project-local ./collections path (works well with navigator)
ansible-galaxy collection install -r requirements.yml -p collections
</code></pre>

<p>If needed, point Ansible to your project collections:</p>

<pre><code># In ansible.cfg for this lesson (or top-level), add:
[defaults]
collections_paths = ./collections:~/.ansible/collections:/usr/share/ansible/collections
</code></pre>

<p align="center">
  <!-- Optional screenshot: galaxy install -->
  <img src="./img/lesson3-galaxy-install-ss.png" alt="ansible-galaxy collection install output" />
</p>

<hr/>

<h2>1) Prove FQCN resolution with a real community module</h2>

<pre><code>ansible all -b -m community.general.timezone -a 'name=UTC' --check -o
</code></pre>

<ul>
  <li><b>Why:</b> Validates <i>collection</i> discovery + <i>sudo</i> without changing state.</li>
  <li><b>Options:</b> <code>-b</code> sudo, <code>-m</code> module, <code>-a</code> args, <code>--check</code> dry-run, <code>-o</code> one-line</li>
  <li><b>Acceptance:</b> each host shows <code>CHANGED</code> in check mode.</li>
</ul>

<p align="center">
  <!-- Screenshot: timezone check -->
  <img src="./img/lesson3-fqcn-ss.png" alt="FQCN check-mode proof" />
</p>

<hr/>

<h2>2) Create directories idempotently</h2>

<pre><code># First run (should change)
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3 state=directory mode=0755 owner=root group=root' -o
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3/logs state=directory mode=0755 owner=root group=root' -o

# Re-run (should be ok / no change)
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3 state=directory mode=0755 owner=root group=root' -o
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3/logs state=directory mode=0755 owner=root group=root' -o

# Verify
ansible all -m ansible.builtin.stat -a 'path=/opt/lesson3' -o
ansible all -m ansible.builtin.stat -a 'path=/opt/lesson3/logs' -o
</code></pre>

<ul>
  <li><b>Key args:</b> <code>state=directory</code>, <code>mode=0755</code>, <code>owner/group=root</code>.</li>
  <li><b>Acceptance:</b> First run <i>changed=true</i>; second run <i>changed=false</i>; <code>stat</code> shows <code>isdir:true</code>, mode 0755.</li>
</ul>

<p align="center">
  <!-- Screenshot: dirs + stat -->
  <img src="./img/lesson3-dirs-ss.png" alt="Idempotent directory management" />
</p>

<hr/>

<h2>3) Manage a file & verify content via hash</h2>

<pre><code># Enforce exact file content + perms
ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info.txt content="Lesson 3 — FQCN + idempotency
Managed by Ansible
" owner=root group=root mode=0644' -o

# Re-run to prove idempotency
ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info.txt content="Lesson 3 — FQCN + idempotency
Managed by Ansible
" owner=root group=root mode=0644' -o

# Verify content is identical on all hosts
ansible all -m ansible.builtin.command -a 'sha256sum /opt/lesson3/info.txt' -o
ansible all -m ansible.builtin.command -a 'cat /opt/lesson3/info.txt' -o
</code></pre>

<ul>
  <li><b>Why:</b> <code>copy</code> is idempotent and byte-accurate—perfect for exam proof.</li>
  <li><b>Acceptance:</b> Second run <i>changed=false</i>; identical SHA256; content matches exactly.</li>
</ul>

<p align="center">
  <!-- Screenshot: copy + sha256 -->
  <img src="./img/lesson3-copy-ss.png" alt="copy module idempotency + hash verification" />
</p>

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
