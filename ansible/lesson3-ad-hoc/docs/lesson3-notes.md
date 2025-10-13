> <h1>Lesson 3 – Ad Hoc Commands (Deep Notes)</h1>
>
> <h3>Control node</h3>
<b>Host: control (all Ansible commands run here)</b>
>
> <h2>Managed nodes</h2>
<b>ansible1, ansible2, ansible3</b> · <b>Reachable via SSH as user: ansible (key-based auth)</b>
>
> <h2>What I built</h2>
<ul>
<li><b>Verified collections:</b> ansible.posix, community.general</li>
<li><b>FQCN usage:</b> community.general.timezone (since community.general.ping is removed)</li>
<li><b>Idempotent resources:</b> /opt/lesson3, /opt/lesson3/logs, /opt/lesson3/info.txt, user lesson3, service crond</li>
</ul>
>
> <p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-overview-ss.PNG">
</p>
>
>
> <h2>1) Prove FQCN works (collection module)</h2>
<pre><code>ansible all -b -m community.general.timezone -a 'name=UTC' --check</code></pre>
<b>Why:</b> Use a real community module to confirm FQCN resolution and sudo.
<b>Options:</b> <code>-b</code>=sudo, <code>-m</code>=module, <code>-a</code>=args, <code>--check</code>=dry-run.
<b>AC:</b> Each host returns <code>CHANGED</code> in check mode.
<p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-fqcn-ss.PNG">
</p>
>
>
> <h2>2) Create directories idempotently</h2>
<pre><code>ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3 state=directory mode=0755 owner=root group=root'
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3/logs state=directory mode=0755 owner=root group=root'
# re-run to prove idempotency
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3 state=directory mode=0755 owner=root group=root'
ansible all -b -m ansible.builtin.file -a 'path=/opt/lesson3/logs state=directory mode=0755 owner=root group=root'
# verify
ansible all -m ansible.builtin.stat -a 'path=/opt/lesson3'
ansible all -m ansible.builtin.stat -a 'path=/opt/lesson3/logs'
</code></pre>
<b>Why:</b> Enforce dirs + perms under /opt.
<b>Key args:</b> <code>state=directory</code>, <code>mode=0755</code>, <code>owner=root</code>, <code>group=root</code>.
<b>AC:</b> First run changed=true; second run changed=false; stat shows <code>isdir:true</code> and mode 0755.
<p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-dirs-ss.PNG">
</p>
>
>
> <h2>3) Manage a file + verify content via hash</h2>
<pre><code>ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info.txt content="Lesson 3 — FQCN + idempotency ✅
 Managed by Ansible
" owner=root group=root mode=0644'
# re-run for idempotency
ansible all -b -m ansible.builtin.copy -a 'dest=/opt/lesson3/info.txt content="Lesson 3 — FQCN + idempotency ✅
 Managed by Ansible
" owner=root group=root mode=0644'
# verification
ansible all -m ansible.builtin.command -a 'sha256sum /opt/lesson3/info.txt'
ansible all -m ansible.builtin.command -a 'cat /opt/lesson3/info.txt'
</code></pre>
<b>Why:</b> <code>copy</code> enforces byte-exact content + perms.
<b>AC:</b> Second run changed=false; identical SHA256 across hosts; file content exactly matches.
<p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-copy-ss.PNG">
</p>
>
>
> <h2>4) Create user <code>lesson3</code> idempotently</h2>
<pre><code>ansible all -b -m ansible.builtin.user -a 'name=lesson3 state=present shell=/bin/bash create_home=yes'
# re-run for idempotency
ansible all -b -m ansible.builtin.user -a 'name=lesson3 state=present shell=/bin/bash create_home=yes'
# verification
ansible all -m ansible.builtin.getent -a 'database=passwd key=lesson3'
ansible all -m ansible.builtin.stat -a 'path=/home/lesson3'
</code></pre>
<b>AC:</b> Second run changed=false; <code>getent</code> shows passwd entry; home dir exists.
<p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-user-ss.PNG">
</p>
>
> <h2>5) Ensure service <code>crond</code> started & enabled</h2>
<pre><code>ansible all -b -m ansible.builtin.service -a 'name=crond state=started enabled=yes'
# re-run to show idempotency
ansible all -b -m ansible.builtin.service -a 'name=crond state=started enabled=yes'
# verification
ansible all -m ansible.builtin.command -a 'systemctl is-active crond'
ansible all -m ansible.builtin.command -a 'systemctl is-enabled crond'
</code></pre>
<b>AC:</b> Second run changed=false; outputs are <code>active</code> and <code>enabled</code>.
<p align="center">
<img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson3-ad-hoc/docs/img/lesson3-service-ss.PNG">
</p>
>
> <h2>6) Module schema quick-reference</h2>
<pre><code>ansible-doc -s ansible.builtin.file     | head -n 25
ansible-doc -s ansible.builtin.copy     | head -n 25
ansible-doc -s ansible.builtin.user     | head -n 25
ansible-doc -s ansible.builtin.service  | head -n 25
ansible-doc -s community.general.timezone | head -n 25
</code></pre>
<b>Why:</b> Fast recall of argument names/defaults under exam pressure.
>
> <hr />
<i>End of Lesson 3 notes</i>
