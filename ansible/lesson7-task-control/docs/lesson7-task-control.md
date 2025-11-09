<h1>Lesson 7 – Using Task Control (Deep Notes)</h1>

<h2>Goal</h2>
<b>Learn how to control when, how, and under what conditions tasks run in Ansible.</b>

<h3>Control node</h3>
<ul>
  <li><b>Host:</b> <code>control</code> — runs playbooks and decides task order</li>
</ul>

<h3>Managed nodes</h3>
<ul>
  <li><b>Hosts:</b> <code>ansible1</code>, <code>ansible2</code>, <code>ansible3</code> — targets where tasks execute.</li>
</ul>

<hr/>

<h2>7.1 Understanding Conditionals</h2>
<p>Task control in Ansible means deciding <b>when</b> a task runs, <b>how many times</b> it runs, and <b>whether</b> it should trigger another task. These tools make automation smarter and more efficient.</p>

<ul>
  <li><b>loop:</b> repeat tasks</li>
  <li><b>when:</b> run tasks only if a condition is true</li>
  <li><b>handlers:</b> run only when something changes</li>
</ul>

<p><b>Analogy:</b> “Do laundry for each shirt” (<code>loop</code>), “only wear it if it’s cold” (<code>when</code>), “wash again if you spill” (<code>handler</code>).</p>

<hr/>

<h2>7.2 Writing Loops</h2>

<pre><code>- name: Start some services
  service:
    name: "{{ item }}"
    state: started
  loop:
    - vsftpd
    - httpd
</code></pre>

<p>Loops keep your playbooks short — one task can handle many items.</p>

<pre><code>vars:
  my_services:
    - httpd
    - vsftpd
tasks:
  - name: Start services dynamically
    service:
      name: "{{ item }}"
      state: started
    loop: "{{ my_services }}"
</code></pre>

<hr/>

<h2>7.3 Using When</h2>

<p><b>when</b> lets you make decisions. “If this is true, do it — otherwise skip.”</p>

<pre><code>- name: Install package only if OS supported
  yum:
    name: vim
    state: present
  when: ansible_distribution in ['RedHat','CentOS','Fedora']
</code></pre>

<p><b>Tip:</b> Use filters like <code>| int</code> and <code>| bool</code> to make sure comparisons are accurate.</p>

<hr/>

<h2>7.4 Using When and Register Together</h2>

<pre><code>- name: Create a user
  user:
    name: paul
  register: user_info

- name: Print results only if user changed
  debug:
    msg: "User created!"
  when: user_info.changed
</code></pre>

<p><b>register</b> stores results, and <b>when</b> acts on them — like saving a test score and deciding what to do next.</p>

<hr/>

<h2>7.5 Conditional Task Execution with Handlers</h2>

<p>Handlers are special tasks triggered only when a change occurs. They’re perfect for restarts and reloads.</p>

<pre><code>- name: Copy config
  copy:
    src: /tmp/index.html
    dest: /var/www/html/index.html
  notify:
    - restart_web

handlers:
  - name: restart_web
    service:
      name: httpd
      state: restarted
</code></pre>

<p><b>Tip:</b> Handlers run at the end, unless you force them early with <code>meta: flush_handlers</code>.</p>

<hr/>

<h2>7.6 Using Blocks</h2>

<p>Blocks group related tasks together, making your playbooks cleaner and stronger against errors.</p>

<pre><code>- name: Example with block, rescue, always
  block:
    - name: Remove file
      shell: rm /var/www/html/index.html
  rescue:
    - name: Recover from error
      shell: touch /tmp/rescuefile
  always:
    - name: Always log
      shell: echo "Play done" >> /tmp/play.log
</code></pre>

<p><b>block:</b> main tasks | <b>rescue:</b> backup plan | <b>always:</b> cleanup</p>

<hr/>

<h2>7.7 Managing Task Failure</h2>

<p>Failure handling gives you control over how Ansible reacts when things go wrong.</p>

<ul>
  <li><b>ignore_errors:</b> continue even if something fails</li>
  <li><b>force_handlers:</b> still run handlers after errors</li>
  <li><b>failed_when:</b> define what failure means</li>
  <li><b>fail:</b> stop intentionally with a message</li>
</ul>

<hr/>

<h2>7.8 Managing Task Failure (Advanced)</h2>

<p>Now you define your own failure logic for better reliability.</p>

<pre><code>- name: Define custom failure
  command: echo hello world
  ignore_errors: yes
  register: result
  failed_when: "'world' in result.stdout"
</code></pre>

<pre><code>- name: Stop play manually
  fail:
    msg: "Critical issue found!"
  when: "'Critical' in result.stdout"
</code></pre>

<p><b>Analogy:</b> <code>ignore_errors</code> keeps driving on a flat tire,  
<code>failed_when</code> spots hidden issues,  
<code>fail</code> is your emergency brake.</p>

<hr/>

<h2>7.9 Including and Importing Files</h2>

<p>As playbooks grow, you’ll want to organize them into smaller reusable pieces using <b>includes</b> and <b>imports</b>.</p>

<h3>Understanding Inclusion</h3>
<ul>
  <li><b>Include</b> and <b>Import</b> can apply to roles, plays, or tasks.</li>
  <li><b>include</b> = <b>dynamic</b> → loaded when the playbook runs.</li>
  <li><b>import</b> = <b>static</b> → loaded before the play starts.</li>
  <li>Playbook imports must appear at the top using <code>import_playbook</code>.</li>
</ul>

<p><b>Analogy:</b> <code>include</code> is like plugging in a USB mid-run,  
while <code>import</code> is like having it already connected before starting the car.</p>

<hr/>

<h3>Including Task Files</h3>

<p>A task file is just a list of tasks — you can attach it anywhere in a playbook.</p>

<ul>
  <li><code>import_tasks</code> → static inclusion (parsed at load time)</li>
  <li><code>include_tasks</code> → dynamic inclusion (parsed during runtime)</li>
</ul>

<p>Dynamic includes have some limits:</p>
<ul>
  <li><code>ansible-playbook --list-tasks</code> won’t show them.</li>
  <li><code>ansible-playbook --start-at-task</code> won’t work with them.</li>
  <li>You can’t trigger a handler in an imported file from the main one.</li>
</ul>

<p><b>Best Practice:</b> Store task files in a dedicated folder like <code>tasks/</code> to keep everything clean and modular.</p>

<hr/>

<h3>Example – Using import_tasks vs include_tasks</h3>
<pre><code>- name: Import static tasks
  import_tasks: setup.yml

- name: Include dynamic tasks
  include_tasks: debug.yml
  when: ansible_facts['os_family'] == "RedHat"
</code></pre>

<p><b>Summary:</b>
<ul>
  <li>Use <code>import_*</code> when the structure is fixed and predictable.</li>
  <li>Use <code>include_*</code> when flexibility is needed at runtime.</li>
</ul></p>

<hr/>

<h2>Lesson 7 Lab – Running Tasks Conditionally</h2>

<p>
<b>Goal:</b> Create a playbook that writes <i>“you have a second disk”</i> if a second disk is found,  
or <i>“you have no second disk”</i> if only one disk exists.
</p>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/lesson7_lab.png" width="850">
</p>

<h3>Example Solution</h3>
<pre><code>- name: Lesson 7 Lab - Check for second disk
  hosts: all
  gather_facts: yes
  tasks:
    - name: Check number of disks
      set_fact:
        disk_count: "{{ ansible_devices | length }}"

    - name: Print if second disk exists
      debug:
        msg: "You have a second disk."
      when: disk_count > 1

    - name: Print if no second disk exists
      debug:
        msg: "You have no second disk."
      when: disk_count <= 1
</code></pre>

<h3>How It Works</h3>
<ul>
  <li><b>gather_facts:</b> collects hardware info, including disks.</li>
  <li><b>ansible_devices:</b> is a built-in variable listing all detected disks (like sda, sdb, etc.).</li>
  <li><b>set_fact:</b> counts the number of disks.</li>
  <li><b>when:</b> decides which message to show.</li>
</ul>

<p><b>Analogy:</b> It’s like checking your garage —  
if there’s more than one car, say “you’ve got a second one!”  
If not, you confirm there’s only one parked.</p>

<hr/>

<h3>Lab Summary</h3>
<ul>
  <li>Used facts to detect system hardware.</li>
  <li>Counted disks dynamically with <code>set_fact</code>.</li>
  <li>Used conditional logic (<code>when</code>) to decide what to display.</li>
</ul>

<p><b>Key takeaway:</b> Conditional tasks turn playbooks into intelligent scripts that react to real system states.</p>

<hr/>

<h2>Next → Lesson 8 – Roles and Reusability</h2>
<p>
Now that you can organize, reuse, and conditionally execute tasks,  
you’re ready to take the next step — using <b>roles</b> to structure automation for enterprise-scale projects.
</p>

<i>End of Lesson 7 notes (7.1–7.9 + Lab)</i>
