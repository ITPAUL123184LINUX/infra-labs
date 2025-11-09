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

<p>
Task control in Ansible means deciding <b>when</b> a task runs, <b>how many times</b> it runs,  
and <b>whether</b> it should trigger another task. These tools make automation smarter and more efficient.
</p>

<h3>Conditionals Overview</h3>
<ul>
  <li><b>loop:</b> Repeat a task for multiple items.</li>
  <li><b>when:</b> Run a task only if a condition is true.</li>
  <li><b>handlers:</b> Run only when triggered by another task that made a change.</li>
</ul>

<p><b>Plain example:</b> “Do laundry for each shirt” (<code>loop</code>),  
“only wear it if it’s cold” (<code>when</code>),  
“wash again if you spill” (<code>handler</code>).</p>

<hr/>

<h2>7.2 Writing Loops</h2>

<p>
Loops keep playbooks clean and short — one task, many executions.
</p>

<h3>1️⃣ Basic Loop</h3>
<pre><code>- name: Start some services
  service:
    name: "{{ item }}"
    state: started
  loop:
    - vsftpd
    - httpd
</code></pre>

<h3>2️⃣ Using Variables in Loops</h3>
<pre><code>vars:
  my_services:
    - httpd
    - vsftpd
tasks:
  - name: Start some services
    service:
      name: "{{ item }}"
      state: started
    loop: "{{ my_services }}"
</code></pre>

<h3>3️⃣ Using Dictionaries in Loops</h3>
<pre><code>- name: Create users using a loop
  hosts: all
  tasks:
    - name: Create users
      user:
        name: "{{ item.name }}"
        state: present
        groups: "{{ item.groups }}"
      loop:
        - { name: anna, groups: wheel }
        - { name: linda, groups: users }
        - { name: bob, groups: users }
</code></pre>

<p><b>Tip:</b> Each list item can hold multiple keys, making loops powerful for structured data.</p>

<h3>4️⃣ loop vs with_items</h3>
<ul>
  <li><b>loop</b> → modern standard</li>
  <li><b>with_items</b> → old syntax</li>
  <li><b>loop</b> replaces all <code>with_*</code> forms</li>
</ul>

<hr/>

<h2>7.3 Using When</h2>

<p>
The <b>when</b> keyword lets Ansible make choices:  
“If this is true, do it — if not, skip it.”
</p>

<h3>Example – Conditional Package Install</h3>
<pre><code>- name: when demo
  hosts: all
  vars:
    supported_distros:
      - Ubuntu
      - CentOS
      - Fedora
    mypackage: vim

  tasks:
    - name: Install package if OS is supported
      yum:
        name: "{{ mypackage }}"
        state: present
      when: ansible_distribution in supported_distros
</code></pre>

<ul>
  <li>Checks if the host OS is in the allowed list.</li>
  <li>If true → installs. If false → skips cleanly.</li>
</ul>

<h3>Common When Conditions</h3>
<pre><code>ansible_machine == "x86_64"
ansible_distribution_version == "8"
ansible_memfree_mb >= 1024
my_variable is defined
ansible_distribution in supported_distros
</code></pre>

<h3>Forcing Variable Types with Filters</h3>
<pre><code>when: vgsize | int > 5
when: runme | bool
</code></pre>

<ul>
  <li><code>| int</code> → treat value as a number</li>
  <li><code>| bool</code> → treat value as true/false</li>
</ul>

<p>
Filters don’t change data; they only tell Ansible how to interpret it.
</p>

<hr/>

<h3>Lab Summary</h3>
<ul>
  <li>Used <b>when</b> to run tasks conditionally.</li>
  <li>Tested multiple fact-based rules (OS, memory, version).</li>
  <li>Used filters to enforce correct data types.</li>
</ul>

<hr/>

<h2>7.4 Using When and Register Together</h2>

<p>
Now we combine <b>when</b> with <b>register</b> to make Ansible respond to real results.  
<code>register</code> saves what a task found or changed — like saving a report card —  
and <code>when</code> decides what to do with that info.
</p>

<h3>Example: Store and React</h3>
<pre><code>- name: Create a user and store result
  user:
    name: "{{ username }}"
  register: user_info

- name: Show register results
  debug:
    var: user_info

- name: Act only if user was changed
  debug:
    msg: "User {{ username }} was just created!"
  when: user_info.changed
</code></pre>

<h3>Testing Multiple Conditions</h3>
<pre><code>when: ansible_distribution == "CentOS" or ansible_distribution == "RedHat"
when: ansible_machine == "x86_64" and ansible_distribution == "CentOS"
when:
  - ansible_machine == "x86_64"
  - ansible_memfree_mb > 512
</code></pre>

<hr/>

<h2>7.5 Conditional Task Execution with Handlers</h2>

<p>
Handlers are special tasks that run <b>only</b> when something changes.  
They’re triggered by the <code>notify</code> keyword —  
for example, restarting a service after a new configuration file is copied.
</p>

<h3>Understanding Handlers</h3>
<ul>
  <li>Handlers run only if the task before them caused a change.</li>
  <li>This prevents unnecessary restarts or reboots.</li>
  <li>The main task “calls” a handler using <code>notify</code>.</li>
  <li>Handlers typically restart services, reload configs, or reboot systems.</li>
</ul>

<h3>Example – Restart Web Service After Config Change</h3>
<pre><code>- name: Copy index.html
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

<hr/>

<h3>Using Handlers More Efficiently</h3>
<ul>
  <li>Handlers normally execute <b>after all tasks</b> finish.</li>
  <li>You can force them to run early using <code>meta: flush_handlers</code>.</li>
  <li>If a task fails, handlers won’t run — unless you set <code>force_handlers: true</code>.</li>
  <li>One task can trigger multiple handlers at once.</li>
</ul>

<hr/>

<h3>Using ansible.builtin.meta Module</h3>
<p>
The <code>ansible.builtin.meta</code> module can adjust playbook flow —  
trigger handlers now, clear data, or stop a specific host mid-play.
</p>

<ul>
  <li><b>flush_handlers:</b> run all handlers immediately.</li>
  <li><b>refresh_inventory:</b> recheck inventory facts right now.</li>
  <li><b>clear_facts:</b> remove all gathered facts.</li>
  <li><b>end_host:</b> stop running the play for this host.</li>
</ul>

<hr/>

<h2>7.6 Using Blocks</h2>

<p>
Blocks let you group related tasks and manage them together.  
They make playbooks cleaner and more reliable, especially when you want to control error handling or apply conditions to several tasks at once.
</p>

<h3>Understanding Blocks</h3>
<ul>
  <li>A <b>block</b> is a logical group of tasks.</li>
  <li>It helps control how tasks execute together.</li>
  <li>You can use a single <code>when</code> to apply to the entire block.</li>
  <li>Note: <code>loop</code> can’t be used directly on blocks (only inside them).</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.6.PNG" width="850">
</p>

<hr/>

<h3>Using Blocks for Error Handling</h3>
<p>
Blocks shine when things go wrong.  
They can define what happens on success, what to do if something fails,  
and what to run every time, no matter what.
</p>

<pre><code>- name: Using blocks for error handling
  hosts: all
  tasks:
    - name: Intended to be successful
      block:
        - name: Remove a file
          shell:
            cmd: rm /var/www/html/index.html
      rescue:
        - name: Create a rescue file if removal fails
          shell:
            cmd: touch /tmp/rescuefile
      always:
        - name: Always log completion
          shell:
            cmd: echo "Play completed" >> /tmp/play.log
</code></pre>

<ul>
  <li><b>block:</b> Main tasks you want to run.</li>
  <li><b>rescue:</b> Backup plan if something fails.</li>
  <li><b>always:</b> Cleanup or final steps that always run.</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.6p1.PNG" width="850">
</p>

<hr/>

<h3>Plain-English Summary</h3>
<ul>
  <li><b>block:</b> Main section to execute.</li>
  <li><b>rescue:</b> What to do if the block fails.</li>
  <li><b>always:</b> Always runs — success or fail.</li>
  <li>One <code>when</code> can control all tasks in a block.</li>
</ul>

<p><b>Analogy:</b> Cooking dinner:  
<b>block</b> is the meal plan,  
<b>rescue</b> is ordering pizza if you burn it,  
and <b>always</b> is cleaning up afterward no matter what.</p>

<hr/>

<h3>Lab Summary – Using Blocks</h3>
<ul>
  <li>Grouped related tasks into one logical structure.</li>
  <li>Used <b>rescue</b> to handle errors automatically.</li>
  <li>Used <b>always</b> for guaranteed cleanup or notifications.</li>
</ul>

<p><b>Key takeaway:</b> Blocks make playbooks tougher — they group logic, recover from errors, and stay clean and readable.</p>

<hr/>

<h2>Next → 7.7 Combining Task Control Tools</h2>
<p>
Next, we’ll combine <b>loop</b>, <b>when</b>, <b>register</b>, <b>handlers</b>, and <b>blocks</b>  
to create adaptive, error-resistant, and efficient playbooks.
</p>

<i>End of sections 7.1–7.6 notes</i>
