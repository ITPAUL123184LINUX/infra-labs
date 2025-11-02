<h1>Lesson 5 – Variables (Summary Notes)</h1>

<h2>Goal</h2>
<b>Make playbooks portable: name the moving parts once, reuse everywhere.</b>

<h2>Mindset</h2>
Different distros, same intent. I set <code>web_service</code> and stop chasing names.

<h2>Core</h2>
<ul>
<li><b>Variables:</b> labels used across tasks/templates.</li>
<li><b>Facts:</b> auto data about a host (OS, IP, etc.).</li>
<li><b>Precedence (simple):</b> host_vars &gt; group_vars &gt; play vars; CLI <code>-e</code> beats all.</li>
</ul>

<h2>Patterns</h2>
<ul>
<li><code>group_vars/all.yml</code> → defaults (e.g., <code>web_service: httpd</code>).</li>
<li><code>host_vars/ansible1.yml</code> → overrides (e.g., <code>web_service: apache2</code>).</li>
<li>Use: <code>service: name={{ web_service }} state=started enabled=yes</code>.</li>
<li>Secrets live in Vault (later).</li>
</ul>

<h2>Proof cues (screenshots later)</h2>
<!-- cmd: ansible -m setup all | head -->
<!-- cmd: ansible-playbook playbooks/01_vars_demo.yml -v -->
<!-- cmd: ansible-playbook playbooks/01_vars_demo.yml -e "web_service=nginx" -v -->
