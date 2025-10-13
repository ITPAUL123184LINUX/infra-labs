<h1>Lesson 3 – Ad Hoc Commands (Summary)</h1>

<h2>Goal</h2>
<b>Use ad-hoc Ansible with FQCNs to enforce state across all managed hosts and PROVE idempotency, exam-style.</b>

<h2>Environment</h2>
<ul>
<li><b>Control node:</b> control (all Ansible runs here)</li>
<li><b>Managed nodes:</b> ansible1, ansible2, ansible3</li>
<li><b>Remote user:</b> ansible (SSH key) · <b>Priv-esc:</b> sudo (NOPASSWD in lab)</li>
<li><b>Collections path (ansible.cfg):</b> ./collections:~/.ansible/collections:/usr/share/ansible/collections</li>
<li><b>Collections verified:</b> ansible.posix, community.general</li>
</ul>

<h2>What I did (headlines)</h2>
<ul>
<li><b>FQCN proof:</b> ran <code>community.general.timezone</code> with <code>--check</code>.</li>
<li><b>Directories:</b> enforced <code>/opt/lesson3</code> and <code>/opt/lesson3/logs</code> (idempotent).</li>
<li><b>File management:</b> deployed <code>/opt/lesson3/info.txt</code> with exact content + perms; verified SHA256.</li>
<li><b>User:</b> created <code>lesson3</code> with home + bash; proved idempotency.</li>
<li><b>Service:</b> ensured <code>crond</code> running and enabled; second run showed no change.</li>
<li><b>Module schemas:</b> used <code>ansible-doc -s</code> to read argument specs fast.</li>
</ul>

<h2>Evidence (short)</h2>
<ul>
<li><b>FQCN:</b> <code>ansible all -b -m community.general.timezone -a 'name=UTC' --check</code> → CHANGED (dry-run).</li>
<li><b>Dirs:</b> first run <code>changed:true</code>; second run <code>changed:false</code>; <code>stat.isdir=true</code>.</li>
<li><b>File:</b> second run <code>changed:false</code>; same SHA256 across hosts; content matches.</li>
<li><b>User:</b> second run <code>changed:false</code>; <code>getent passwd lesson3</code> valid; <code>/home/lesson3</code> exists.</li>
<li><b>Service:</b> <code>systemctl is-active crond</code> → active; <code>is-enabled</code> → enabled.</li>
</ul>

<h2>Problems & fixes (today)</h2>
<ul>
<li><b>community.general.ping missing</b> in current collection. <b>Fix:</b> used <code>community.general.timezone</code> to prove FQCN.</li>
</ul>

<h2>Done checklist</h2>
<ul>
<li>[X] FQCN proven</li>
<li>[X] Idempotent directories</li>
<li>[X] Managed file + hash verification</li>
<li>[X] User creation + idempotency</li>
<li>[X] <code>crond</code> running and enabled</li>
<li>[X] <code>ansible-doc -s</code> reviewed</li>
</ul>


<p align="center">
<img src="">
</p>
