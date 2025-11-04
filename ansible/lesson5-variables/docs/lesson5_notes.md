<h1>Lesson 5 – Variables (Deep Notes)</h1>

<h2>Goal</h2>
<b>Stop hard-coding. Name the moving parts once and let Ansible swap values per host/distro.</b>

<h2>Plain-English idea</h2>
A variable is a nickname for a value. I write the playbook once and point it at names like <code>web_service</code>. Ubuntu says <code>apache2</code>, RHEL says <code>httpd</code>—the playbook doesn’t care.

<h2>Facts vs variables</h2>
Facts are auto variables about the current host (OS, IP, CPU). I can use them or override with my own vars to keep logic clean.

<h2>Where vars live (what I’ll actually use)</h2>
<ul>
<li><code>group_vars/</code> for defaults the whole fleet shares.</li>
<li><code>host_vars/</code> for one-off overrides.</li>
<li><code>vars_files/include_vars</code> when I want a separate file.</li>
<li><code>register</code> to capture task output and reuse it.</li>
<li><code>vars_prompt</code> exists, but secrets go to <b>Ansible Vault</b>.</li>
</ul>

<h2>Precedence (remember this)</h2>
More specific wins. CLI <code>-e</code> overrides everything; then host_vars → group_vars → play/role defaults.

<h2>Vault rule</h2>
Anything sensitive (passwords, API tokens, private keys) lives in Vault. Period.

<h2>Proof cues (screenshots later)</h2>
<!-- cmd: ansible -m setup all | head -->
<!-- cmd: ansible-playbook playbooks/01_vars_demo.yml -v -->
<!-- cmd: ansible-playbook playbooks/01_vars_demo.yml -e "web_service=nginx" -v -->

<h2>Using Variables in a Playbook (fast patterns)</h2>
<ul>
<li><b>Inline in the play:</b> quick demo or one-off value.
<pre><code>---
- hosts: all
  vars:
    web_package: httpd
</code></pre>
</li>

<li><b>vars_files:</b> keep values in a separate file; playbook stays clean.
<pre><code>---
- hosts: all
  vars_files:
    - vars/users.yml
</code></pre>
</li>

<li><b>When to use which:</b>
  Inline = temporary/small.  
  <code>vars_files</code> = reusable or bigger sets.  
  Long-term defaults still live in <code>group_vars/</code> / <code>host_vars/</code> (stronger structure).</li>

<li><b>Naming rule:</b> name by intent (<code>web_package</code>, <code>admin_user</code>), not by OS.</li>
</ul>

<h3>Proof cues (screens later)</h3>
<!-- cmd: ansible-playbook playbooks/02_use_vars.yml --syntax-check -->
<!-- cmd: ansible-playbook playbooks/02_use_vars.yml -v -->
<!-- cmd: ansible-playbook playbooks/02_use_vars.yml -e "web_package=nginx" -v -->

<h2>Using Vars in a Play (what my screenshots show)</h2>
<ul>
<li><b>Inline vars</b> via <code>vars:</code> or load with <code>vars_files:</code>.</li>
<li><b>Jinja use:</b> <code>{{ var }}</code>. If it starts the value, use quotes: <code>"{{ web_package }}"</code>.</li>
</ul>

<pre><code>---
- hosts: all
  vars:
    user: lisa
  tasks:
    - name: create a user {{ user }}
      ansible.builtin.user:
        name: "{{ user }}"
</code></pre>

<h3>Reading the run</h3>
<ul>
<li><b>GATHERING FACTS</b> populates <code>ansible_facts</code>.</li>
<li><b>TASK create a user lisa</b> shows per-host result: <code>changed</code> when the user is created; <code>ok</code> on rerun (idempotent).</li>
<li><b>PLAY RECAP</b>: <code>ok</code>=executed fine, <code>changed</code>=state changed, <code>failed</code>=errors (should be 0).</li>
</ul>

<h2>Debug module (see what the vars resolve to)</h2>
<ul>
<li><b>Purpose:</b> print values while the play runs—perfect for sanity checks.</li>
<li><b>Usage:</b> <code>ansible.builtin.debug:</code> <code>msg: "the username is {{ user }}"</code></li>
<li><b>Quoting rule:</b> if a variable is the <i>first</i> thing in a string, wrap it in quotes:
<code>"{{ user }}"</code>.</li>
<li><b>Tip:</b> add <code>-v</code>/<code>-vv</code> for more detail during runs.</li>
</ul>

<pre><code>---
- name: create a user using a variable
  hosts: all
  vars:
    user: lisa
  tasks:
    - name: using debug
      ansible.builtin.debug:
        msg: "the username is {{ user }}"
    - name: create a user {{ user }}
      ansible.builtin.user:
        name: "{{ user }}"
</code></pre>

<h3>Small summary: what this playbook does</h3>
<ul>
<li><b>vars</b>: defines <code>user: lisa</code> for the whole play.</li>
<li><b>debug</b>: prints the resolved value (proves templating works).</li>
<li><b>user task</b>: creates the account using <code>name: "{{ user }}"</code>.</li>
<li><b>Idempotency</b>: first run → <code>changed</code>, second → <code>ok</code>.</li>
</ul>

<h2>Includes (vars_files)</h2>
<p>Keep site-specific data out of playbooks. Put it in files and include them with <code>vars_files</code> so the play stays portable.</p>

<ul>
  <li><b>Rule:</b> variables file is simple <code>key: value</code> YAML.</li>
  <li><b>Path:</b> paths are <i>relative to the playbook file</i> (example: <code>../vars/users.yml</code>).</li>
  <li><b>Type:</b> it’s a list; you can include multiple files.</li>
  <li><b>Precedence:</b> loading via <code>vars_files</code> doesn’t change normal var precedence.</li>
</ul>

<p align="center">
  <img src="docs/img/lesson5-includes-slide.png" alt="Includes slide: use vars_files, key:value" width="900"/>
</p>

<p><b>ansible-doc check</b></p>
<!-- cmd: ansible-doc -t keyword vars_files -->
<p><img src="docs/img/ansible-doc-vars_files.png" alt="ansible-doc vars_files keyword output" width="900"/></p>

<p><b>Run proof</b></p>
<!-- cmd: ansible-playbook playbooks/user.yml -v -->
<p><img src="docs/img/lesson5-vars_files-run.png" alt="vars_files play run showing ok/changed" width="900"/></p>

<hr/>

<h2>Using Host Variables</h2>

<p><b>Goal:</b> Define per-host values cleanly instead of mixing them inside the inventory.</p>

<ul>
  <li><b>Host variables</b> are unique values tied to a specific host (for example, IPs or DB names).</li>
  <li>They live under <code>host_vars/</code>, named after each hostname.</li>
  <li>Shared values live under <code>group_vars/</code>.</li>
  <li>Ansible loads both automatically—no includes needed.</li>
  <li>Inventory inline variables are <b>deprecated</b>.</li>
</ul>

<pre><code>inventory/
├── hosts.ini
├── host_vars/
│   ├── ansible1.yml
│   └── ansible2.yml
└── group_vars/
    ├── webservers.yml
    └── dbservers.yml
</code></pre>

<pre><code>ansible all -m ansible.builtin.debug -a "var=ansible_hostname" -o
</code></pre>

<p align="center">
  <img src="docs/img/lesson5-host-vars-slide.png" alt="Using Host Variables slide summary" width="900"/>
</p>

<hr/>

<h2>System Variables</h2>

<p><b>Goal:</b> Know what Ansible provides automatically—and that you <b>cannot override them</b>.</p>

<p>System variables are built in and always available. You can reference them, but never redefine them.</p>

<ul>
  <li><code>hostvars</code> — dictionary of every host’s variables (<code>hostvars['ansible1']['ansible_hostname']</code>).</li>
  <li><code>inventory_hostname</code> — full name of the host as defined in inventory.</li>
  <li><code>inventory_hostname_short</code> — short name (everything before first dot).</li>
  <li><code>groups</code> — dictionary of all inventory groups and their members.</li>
  <li><code>group_names</code> — list of groups the current host belongs to.</li>
  <li><code>ansible_check_mode</code> — true if play runs with <code>--check</code> (dry-run).</li>
  <li><code>ansible_play_batch</code> / <code>ansible_play_hosts</code> — hosts active in current play.</li>
  <li><code>ansible_version</code> — current version of Ansible running.</li>
</ul>

<p><b>Key rule:</b> These variables are read-only. Use them to query or filter data, never assign to them.</p>

<h3>Quick proof</h3>
<pre><code>ansible all -m ansible.builtin.debug -a "var=inventory_hostname" -o
ansible all -m ansible.builtin.debug -a "var=group_names" -o
ansible all -m ansible.builtin.debug -a "var=ansible_version" -o
</code></pre>

<p><b>Why:</b> This confirms the built-in variables exist for every host automatically. Perfect for conditionals, loops, and templating logic.</p>

<p align="center">
  <img src="docs/img/lesson5-system-vars-slide.png" alt="System Variables slide summary" width="900"/>
</p>

<hr/>

<h2>The <code>register</code> Keyword</h2>

<p><b>Goal:</b> Capture command or module output into a variable you can reuse later in the play.</p>

<p><code>register</code> doesn’t define a static variable like <code>vars</code> — it stores live data generated by a task. Think of it as “recording the result.”  
This lets me react to facts discovered during runtime instead of hardcoding values.</p>

<h3>Example</h3>

<pre><code>---
- hosts: all
  tasks:
    - name: check uptime
      ansible.builtin.command: uptime
      register: uptime_result

    - name: print the uptime output
      ansible.builtin.debug:
        msg: "Uptime info: {{ uptime_result.stdout }}"
</code></pre>

<h3>What happens</h3>
<ul>
  <li><b>First task:</b> runs the <code>uptime</code> command and saves its entire JSON-like result in <code>uptime_result</code>.</li>
  <li><b>Second task:</b> accesses <code>uptime_result.stdout</code> (standard output) and displays it with <code>debug</code>.</li>
</ul>

<h3>Useful fields in a registered variable</h3>
<ul>
  <li><code>stdout</code> — plain text output of the command.</li>
  <li><code>stderr</code> — any error output.</li>
  <li><code>rc</code> — return code (0 = success).</li>
  <li><code>changed</code> — boolean; true if Ansible detected a change.</li>
  <li><code>cmd</code> — the command line that was executed.</li>
</ul>

<h3>Why it matters for Lesson 5</h3>
<ul>
  <li><b>register</b> is dynamic: variables are created at runtime, unlike those in <code>group_vars</code> or <code>host_vars</code>.</li>
  <li>Great for proofs and conditional logic, like skipping tasks if <code>rc != 0</code> or only continuing on specific outputs.</li>
  <li>Registered data can be referenced in later plays — for example, verifying idempotency or collecting reports.</li>
</ul>

<p align="center">
  <img src="docs/img/lesson5-register-example.png" alt="Register variable example: capturing and printing command output" width="900"/>
</p>

<hr/>

<h2>Ansible Vault (Protect Sensitive Data)</h2>

<p><b>Goal:</b> Secure passwords, tokens, and private variables so they are safe in Git but still usable in playbooks.</p>

<p><b>Idea:</b> Anything confidential should never live in plain text. <code>ansible-vault</code> lets me encrypt YAML files so that even if someone opens the repo, they can’t read the secrets without the password.</p>

<h3>Vault Basics</h3>
<ul>
  <li><code>ansible-vault create secret.yml</code> — make a new encrypted file and edit it directly.</li>
  <li><code>ansible-vault edit secret.yml</code> — safely open an existing vault for changes.</li>
  <li><code>ansible-vault view secret.yml</code> — read it (still prompts for password).</li>
  <li><code>ansible-vault encrypt vars.yml</code> — encrypt an existing file.</li>
  <li><code>ansible-vault decrypt vars.yml</code> — decrypt a file (back to plain text).</li>
</ul>

<h3>Using Vaulted Files in Playbooks</h3>
<pre><code>---
- hosts: all
  vars_files:
    - secret.yml
  tasks:
    - name: show vault variable
      ansible.builtin.debug:
        msg: "The API key is {{ api_key }}"
</code></pre>

<p><b>Run command (prompts for password):</b></p>
<pre><code>ansible-playbook playbooks/secure.yml --ask-vault-pass
</code></pre>

<p><b>Alternative (for automation):</b></p>
<pre><code>ansible-playbook playbooks/secure.yml --vault-password-file ~/.vault_pass.txt
</code></pre>

<h3>Quick Facts (Exam-Ready)</h3>
<ul>
  <li>Vault protects entire files, not individual variables.</li>
  <li>You can encrypt any YAML file used in playbooks — typically <code>vars/</code> or <code>group_vars/</code> files.</li>
  <li>Multiple vaults can exist in one project, each with its own password.</li>
  <li>Vaulted files can be shared safely on GitHub if the password is stored securely elsewhere.</li>
  <li>Always re-run <code>--ask-vault-pass</code> if you see a “Decryption failed” message — it means the wrong password was supplied.</li>
  <li>Vault passwords can be rotated using <code>ansible-vault rekey file.yml</code>.</li>
</ul>

<h3>Why Vault matters for the exam</h3>
<ul>
  <li>You must know how to:
    <ul>
      <li>Create a vault file.</li>
      <li>Use it in a playbook.</li>
      <li>Decrypt or edit it as needed.</li>
    </ul>
  </li>
  <li>Remember: <code>ansible-vault create</code> + <code>--ask-vault-pass</code> are the most common exam tasks.</li>
  <li>Vault ensures your repo can be public without exposing secrets — essential for production automation and Git hygiene.</li>
</ul>

<p align="center">
  <img src="docs/img/lesson5-ansible-vault-slide.png" alt="Ansible Vault overview slide" width="900"/>
</p>


