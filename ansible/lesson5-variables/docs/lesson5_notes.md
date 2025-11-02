<h1>Lesson 5 – Variables (My Notes)</h1>

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
<!-- screenshot: add image of the YAML here (user.yml) -->

<h3>Reading the run</h3>
<ul>
<li><b>GATHERING FACTS</b> populates <code>ansible_facts</code>.</li>
<li><b>TASK create a user lisa</b> shows per-host result: <code>changed</code> when the user is created; <code>ok</code> on rerun (idempotent).</li>
<li><b>PLAY RECAP</b>: <code>ok</code>=executed fine, <code>changed</code>=state changed, <code>failed</code>=errors (should be 0).</li>
</ul>
<!-- screenshot: add terminal output of the play recap -->

<h3>Screenshots (drop in docs/img/)</h3>
<!-- cmd: ansible -m setup all | head -->
<!-- save as: docs/img/lesson5-facts-head.png -->
<!-- cmd: ansible-playbook playbooks/02_use_vars.yml --syntax-check -->
<!-- save as: docs/img/lesson5-syntax-ok.png -->
<!-- cmd: ansible-playbook playbooks/02_use_vars.yml -v -->
<!-- save as: docs/img/lesson5-first-run.png -->
<!-- cmd: ansible-playbook playbooks/02_use_vars.yml -e "web_service=nginx" -v -->
<!-- save as: docs/img/lesson5-cli-override.png -->

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

<!-- screenshot (YAML): docs/img/lesson5-debug-yaml.png -->
<!-- screenshot (run output): docs/img/lesson5-debug-run.png -->

<h3>Reading the debug run</h3>
<ul>
<li><b>debug</b> prints the resolved value (proves templating works).</li>
<li><b>user task</b>: first run shows <code>changed</code>; re-run shows <code>ok</code> (idempotent).</li>
</ul>

<h3>Small summary: what this playbook does</h3>
<ul>
<li><b>vars</b>: defines <code>user: lisa</code> for the whole play.</li>
<li><b>debug</b>: prints <code>the username is {{ user }}</code> so I can see templating resolved to “lisa”.</li>
<li><b>user task</b>: creates the account using quotes
<code>name: "{{ user }}"</code> (quotes required when a var starts the value).</li>
<li><b>Idempotency</b>: first run shows <code>changed</code> on hosts where the user didn’t exist; re-run shows <code>ok</code>.</li>
<li><b>Run anatomy</b>: “GATHERING FACTS” → debug output → user creation → “PLAY RECAP” (ok/changed/failed).</li>
</ul>
<!-- screenshot (summary block shown in run): docs/img/lesson5-user-play-run.png -->
