<h1>Lesson 5 – Variables (Summary Notes)</h1>

<h2>Goal</h2>
<b>Make playbooks portable and predictable by naming differences and letting scope/precedence do the work.</b>

<h2>Big Idea (in plain English)</h2>
Variables are nicknames for values. Facts are auto-filled variables about each host. We write tasks once and swap values per OS/host/group.

<h2>Where vars come from (ordered thinking)</h2>
Play/role vars → <code>group_vars/</code> → <code>host_vars/</code> → registered results → CLI <code>-e</code> (strongest).  
Also: <code>include_vars</code>/<code>vars_files</code>, <code>vars_prompt</code> (use Vault for secrets).

<h2>Practical patterns</h2>
<ul>
<li><b>Defaults:</b> <code>group_vars/all.yml</code> → <code>web_service: httpd</code>.</li>
<li><b>Host/Distro override:</b> <code>host_vars/ansible1.yml</code> → <code>web_service: apache2</code>.</li>
<li><b>Use:</b> <code>service: name={{ web_service }} state=started enabled=yes</code>.</li>
<li><b>register:</b> save a task’s output, reuse with <code>{{ result.stdout }}</code>.</li>
<li><b>CLI test:</b> <code>-e web_service=nginx</code> to override on the fly.</li>
</ul>

<h2>Good habits / gotchas</h2>
<ul>
<li>Keep logic in tasks, values in vars; avoid hard-coding.</li>
<li>Name vars by intent (<code>web_service</code>), not by distro.</li>
<li>Use Vault for any credential, token, or key.</li>
</ul>

<h2>Proof cues (add screenshots later)</h2>
<!-- cmd: ansible -m setup all | head -->
<!-- cmd: ansible-playbook playbooks/01_vars_demo.yml -v -->
<!-- cmd: ansible-playbook playbooks/01_vars_demo.yml -e "web_service=nginx" -v -->
