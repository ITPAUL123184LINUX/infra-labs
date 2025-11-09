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

<p>
Here’s what happens:
</p>
<ul>
  <li>The first task creates a user and saves its result into <code>user_info</code>.</li>
  <li>The second shows what Ansible captured (useful for learning).</li>
  <li>The third runs <b>only</b> if the creation actually happened (<code>changed: true</code>).</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.4.PNG" width="850">
</p>

<hr/>

<h3>Testing Multiple Conditions</h3>
<p>
You can combine several rules using <code>and</code>, <code>or</code>, or parentheses to group logic.
</p>

<pre><code>when: ansible_distribution == "CentOS" or ansible_distribution == "RedHat"
when: ansible_machine == "x86_64" and ansible_distribution == "CentOS"
</code></pre>

<p>
You can also stack multiple conditions in a list —  
every line must be true for the task to run.
</p>

<pre><code>when:
  - ansible_machine == "x86_64"
  - ansible_memfree_mb > 512
</code></pre>

<p><b>Example in English:</b>  
“If this host is 64-bit <i>and</i> has more than 512 MB free memory, then run this task.”</p>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.4p1.PNG" width="850">
</p>

<hr/>

<h3>Plain-English Summary</h3>
<ul>
  <li><b>register</b> stores the outcome of a task (success, fail, changed, etc.).</li>
  <li><b>when</b> reads that outcome to decide what happens next.</li>
  <li>You can combine facts, variables, and results using <code>and</code>/<code>or</code>.</li>
  <li>Use <code>debug:</code> to explore what’s inside a registered variable.</li>
</ul>

<p><b>Analogy:</b> Think of <b>register</b> as saving the test score,  
and <b>when</b> as deciding whether you passed or need to study more.</p>

<hr/>

<h3>Lab Summary – Using When + Register</h3>
<ul>
  <li>Used <b>register</b> to capture results of a task.</li>
  <li>Used <b>when</b> to run follow-up tasks only when specific results occurred.</li>
  <li>Tested multiple conditions using <b>and</b>/<b>or</b> and lists.</li>
</ul>

<p><b>Key takeaway:</b> Together, <code>when</code> + <code>register</code> give Ansible decision-making power.  
It can now “see,” “remember,” and “act” — just like a human operator.</p>

<hr/>

<h2>Next → 7.5 Handlers and Notify</h2>
<p>
Next, we’ll dive into <b>handlers</b> — tasks that run automatically when something changes,  
like restarting a service only after configuration updates.
</p>

<i>End of sections 7.1–7.4 notes</i>
