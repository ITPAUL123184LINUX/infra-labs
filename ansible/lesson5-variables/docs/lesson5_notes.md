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
