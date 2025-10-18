<h1>Lesson 3 – Ad Hoc Commands (Deep Notes)</h1>

### Control node
- <b>Host: control (all Ansible commands run here)</b>

<h2>Managed nodes</h2>
<b>ansible1, ansible2, ansible3</b> · <b>Reachable via SSH as user: ansible (key-based auth)</b>

<h2>What I built</h2>
<ul>
  <li><b>Verified content collections</b>: <code>ansible.posix</code>, <code>community.general</code> (via <code>requirements.yml</code> + <code>ansible-galaxy</code>)</li>
  <li><b>FQCN</b> (Fully Qualified Collection Name) usage: <code>community.general.timezone</code> (<i>note:</i> <code>community.general.ping</code> was removed)</li>
  <li><b>Idempotent resources</b> created and verified:
    <code>/opt/lesson3</code>, <code>/opt/lesson3/logs</code>,
    <code>/opt/lesson3/info.txt</code>, user <code>lesson3</code>,
    service <code>crond</code>
  </li>
  <li><b>Navigator + Execution Environments</b>: how modules are discovered/used inside EE and on the control host</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-overview-ss.PNG" />
</p>

<hr/>

<h2>Collections, FQCN & Execution Environments (EE)</h2>

- Since Ansible 2.10+, most non-core modules live in <b>collections</b>. Use the module’s <b>FQCN</b> to avoid name collisions across collections.
  - Examples: <code>ansible.builtin.service</code>, <code>ansible.posix.firewalld</code>, <code>community.general.timezone</code>
- <b>Where content comes from</b>:
  - <b>Ansible Galaxy</b> (public): <code>ansible-galaxy collection install …</code>
  - <b>Ansible Automation Platform</b> (AAP) registries / private repos
- <b>Execution Environments</b> (EE) via <code>ansible-navigator</code> bundle Python + collections in a container.
  - If you want your local collections to be visible inside Navigator, either:
    1) install them in a path Navigator searches, or
    2) pass the correct <code>-p collections</code> target when installing (see below).

<b>ansible.cfg (collection discovery)</b>  
Add/confirm a search path so both CLI and Navigator can find content:
<pre><code>[defaults]
collections_paths = ./collections:~/.ansible/collections:/usr/share/ansible/collections
</code></pre>

<b>Install collections (local project)</b>
<pre><code># Pin where collections land so Navigator finds them
ansible-galaxy collection install -r requirements.yml -p collections
</code></pre>

<b>Example requirements.yml</b> (place at project root)
<pre><code>---
collections:
  - name: community.general
  - name: ansible.posix
  # Examples of alternative sources/pinning
  # - name: my_namespace.my_collection
  #   version: "1.2.1"
  # - name: /tmp/my-collection.tar.gz
  # - name: http://www.example.local/my-collection.tar.gz
  # - name: https://github.com/ansible-collections/community.general.git
</code></pre>

<b>Navigator quick refs</b>
<pre><code># show docs for an item (inside EE)
ansible-navigator doc ansible.builtin.ping

# avoid pulling a newer container image each run
ansible-navigator --pp never doc ansible.builtin.ping
</code></pre>

<hr/>

<h2>1) Prove FQCN works (community module)</h2>

<pre><code>ansible all -b -m community.general.timezone -a 'name=UTC' --check
</code></pre>
<b>Why</b>: Confirm collection resolution + sudo works.  
<b>Options</b>: <code>-b</code>=sudo, <code>-m</code>=module, <code>-a</code>=args, <code>--check</code>=dry-run.  
<b>Expected</b>: <code>CHANGED</code> in check mode (no real change applied).

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-fqcn-ss.PNG" />
</p>

<hr/>

<h2>2) Create directories idempotently</h2>

<pre><code># create
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3 state=directory mode=0755 owner=root group=root'
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3/logs state=directory mode=0755 owner=root group=root'

# re-run to prove idempotency (no change expected)
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3 state=directory mode=0755 owner=root group=root'
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3/logs state=directory mode=0755 owner=root group=root'

# verify
ansible all -m ansible.builtin.stat -a 'path=/opt/lesson3'
ansible all -m ansible.builtin.stat -a 'path=/opt/lesson3/logs'
</code></pre>

<b>Key args</b>: <code>state=directory</code>, <code>mode=0755</code>, <code>owner</code>, <code>group</code>.  
<b>Expected</b>: First run changed=true; second run changed=false; <code>stat</code> reports <code>isdir: true</code>, mode 0755.

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-dirs-ss.PNG" />
</p>

<hr/>

<h2>3) Manage a file + verify content via hash</h2>

<pre><code># enforce exact content + perms
ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info.txt content="Lesson 3 — FQCN + idempotency ✅
Managed by Ansible
" owner=root group=root mode=0644'

# re-run (idempotent)
ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info.txt content="Lesson 3 — FQCN + idempotency ✅
Managed by Ansible
" owner=root group=root mode=0644'

# verify same bytes on all hosts
ansible all -m ansible.builtin.command -a 'sha256sum /opt/lesson3/info.txt'
ansible all -m ansible.builtin.command -a 'cat /opt/lesson3/info.txt'
</code></pre>

<b>Expected</b>: second run has <code>changed=false</code>. SHA256 hashes identical across hosts.

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-copy-ss.PNG" />
</p>

<hr/>

<h2>4) Create user <code>lesson3</code> idempotently</h2>

<pre><code>ansible all -b -m ansible.builtin.user -a 'name=lesson3 state=present shell=/bin/bash create_home=yes'
# re-run (should be no change)
ansible all -b -m ansible.builtin.user -a 'name=lesson3 state=present shell=/bin/bash create_home=yes'

# verify
ansible all -m ansible.builtin.getent -a 'database=passwd key=lesson3'
ansible all -m ansible.builtin.stat   -a 'path=/home/lesson3'
</code></pre>

<b>Expected</b>: <code>changed=false</code> on the second run; getent shows the passwd entry; home exists.

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-user-ss.PNG" />
</p>

<hr/>

<h2>5) Ensure service <code>crond</code> is started & enabled</h2>

<pre><code>ansible all -b -m ansible.builtin.service -a 'name=crond state=started enabled=yes'
# re-run (idempotent)
ansible all -b -m ansible.builtin.service -a 'name=crond state=started enabled=yes'

# verify
ansible all -m ansible.builtin.command -a 'systemctl is-active crond'
ansible all -m ansible.builtin.command -a 'systemctl is-enabled crond'
</code></pre>

<b>Expected</b>: second run <code>changed=false</code>; outputs: <code>active</code> and <code>enabled</code>.

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-service-ss.PNG" />
</p>

<hr/>

<h2>Essential Modules & When to Use Them</h2>

- <b>ansible.builtin.ping</b> — reachability test (Python required on target)
- <b>ansible.builtin.service</b> — start/stop/enable services (<i>idempotent</i>)
- <b>ansible.builtin.copy</b> — push file content/permissions (<i>idempotent</i>)
- <b>ansible.builtin.command</b> — run a binary <i>without</i> a shell (not idempotent)
- <b>ansible.builtin.shell</b> — run through a shell (respects pipes, redirection, &amp; shell built-ins; not idempotent)
- <b>ansible.builtin.raw</b> — run a command on hosts that <i>don’t</i> have Python yet
- <b>ansible.builtin.user</b> — manage local users (<i>idempotent</i>)

<b>Golden rule</b>: Prefer the <i>most Ansible way</i> first (an idempotent module). Use <code>command</code>/<code>shell</code> only when no purpose-built module exists.

<hr/>

<h2>Idempotency — exam core</h2>

- If the current state already matches the desired state, <b>nothing should change</b>. That’s success, not failure.
- Prove it by re-running the same ad hoc command and expecting <code>changed=false</code>.
- Example demo (user):
<pre><code># create a user once, then re-run and expect no change
ansible all -b -m ansible.builtin.user -a 'name=inda state=present'
ansible all -b -m ansible.builtin.user -a 'name=inda state=present'
</code></pre>

<hr/>

<h2>Docs Mastery (fast during EX294)</h2>

- <b>ansible-doc</b> (local) — module usage & examples:
  <pre><code>ansible-doc -s ansible.builtin.file
ansible-doc -s ansible.builtin.copy
ansible-doc -s ansible.builtin.user
ansible-doc -s ansible.builtin.service
ansible-doc -s community.general.timezone
</code></pre>
- <b>ansible-navigator doc</b> (inside EE):
  <pre><code>ansible-navigator doc ansible.builtin.ping
ansible-navigator --pp never doc ansible.posix.firewalld
</code></pre>
- <b>docs.ansible.com</b> — for Ansible Core module docs; filter by collection and version when searching.

<hr/>

<h2>Lab: Working with modules (Sander)</h2>

<b>Goal</b>
<ol>
  <li>Install <code>community.crypto</code> so it’s available to both CLI and Navigator</li>
  <li>Verify reachability with <code>ping</code></li>
  <li>Run a shell pipeline across all hosts and validate output</li>
</ol>

<b>Steps</b>
<pre><code># 1) Install collection into the project path Navigator will search
ansible-galaxy collection install community.crypto -p collections
# (or include it in requirements.yml, then:)
ansible-galaxy collection install -r requirements.yml -p collections

# 2) Verify connectivity
ansible all -m ansible.builtin.ping

# 3) Run pipeline via shell (needs a shell for pipes/grep)
ansible all -m ansible.builtin.shell -a "rpm -qa | grep ssh"
</code></pre>

<b>Why <code>-p collections</code>?</b> Without it, content may land in a default path not visible to Navigator’s EE. Using a project-local <code>collections/</code> folder ensures discovery (given the <code>collections_paths</code> above).

<hr/>

<h2>Quick “Ad Hoc” Reference (what I actually ran)</h2>

<pre><code># FQCN check
ansible all -b -m community.general.timezone -a 'name=UTC' --check

# Directories
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3 state=directory mode=0755 owner=root group=root'
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3/logs state=directory mode=0755 owner=root group=root'

# File contents
ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info.txt content="Lesson 3 — FQCN + idempotency ✅
Managed by Ansible
" owner=root group=root mode=0644'

# User
ansible all -b -m ansible.builtin.user -a 'name=lesson3 state=present shell=/bin/bash create_home=yes'

# Service
ansible all -b -m ansible.builtin.service -a 'name=crond state=started enabled=yes'
</code></pre>

<hr/>

<h2>🧠 Flags & Concepts Cheat Sheet (Lesson 3)</h2>

| Item/Flag | Purpose |
|---|---|
| <code>-m &lt;module&gt;</code> | Choose module (e.g., <code>ansible.builtin.copy</code>) |
| <code>-a &lt;args&gt;</code> | Arguments to the module |
| <code>-b</code> | Become (sudo) |
| <code>--check</code> | Dry run (plan / no changes) |
| <code>-o</code> | One-line results per host |
| <code>FQCN</code> | Fully qualified name (collection.module) — avoids collisions |
| <code>ansible-galaxy collection install -p collections</code> | Install where both CLI & Navigator can find it |
| <code>collections_paths</code> | Search order for collections in <code>ansible.cfg</code> |
| <code>command</code> vs <code>shell</code> | Use <code>shell</code> for pipelines/redirects; prefer modules over either when possible |
| <code>raw</code> | Use only when Python is missing on the target |
| <code>Idempotency</code> | Re-runs should report <code>changed=false</code> when state already matches |

<hr/>

<i>End of Lesson 3 notes</i>
