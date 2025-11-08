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
  <li><b>loop:</b> Lets you repeat a task for multiple items (like creating many users)  
      without copying the same task over and over.</li>
  <li><b>when:</b> Runs a task only if a specific condition is true — like “install nginx <i>only if</i> the host is RHEL.”</li>
  <li><b>handlers:</b> Special tasks that run <i>only when triggered</i> by another task that made a change —  
      great for things like restarting a service after a config update.</li>
</ul>

<p><b>Example in plain English:</b></p>
<ul>
  <li><code>loop</code> → “Do this for each item in my list.”</li>
  <li><code>when</code> → “Only do this if the rule I set is true.”</li>
  <li><code>handlers</code> → “If something changed, run this extra step.”</li>
</ul>

<p>
Together, these three features form the backbone of <b>task control</b> — making your playbooks smart, reactive, and clean.
</p>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.1-conditionals-overview.PNG" width="850">
</p>

<hr/>

<h3>Simple Analogy</h3>
<p>
Think of <b>loop</b> as a “for each” list in life — doing laundry for each item in a basket.  
<b>when</b> is like checking the weather before wearing a coat.  
<b>handlers</b> are like saying “if I spill something, clean it afterward.”  
Automation mirrors real-life logic — it just never forgets or gets lazy.
</p>

<hr/>

<h2>7.2 Writing Loops</h2>

<p>
Loops make Ansible efficient — you write one task and run it multiple times, automatically cycling through a list of items.
</p>

<h3>1️⃣ Basic Loop Example</h3>
<p>
The <b>loop</b> keyword repeats a task for every item in a list.  
Before Ansible 2.5, older playbooks used <code>with_items</code>, but <b>loop</b> is now the modern and preferred method.
</p>

<pre><code>- name: Start some services
  service:
    name: "{{ item }}"
    state: started
  loop:
    - vsftpd
    - httpd
</code></pre>

<p>
Here, Ansible cycles through each service name — starting <b>vsftpd</b> first, then <b>httpd</b>.  
This eliminates repetitive code and keeps playbooks short and clean.
</p>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.2-understanding-loops.PNG" width="850">
</p>

<hr/>

<h3>2️⃣ Using Variables Inside Loops</h3>
<p>
Instead of hardcoding values, define your list as a variable.  
This keeps playbooks flexible — if the list changes, you update it once.
</p>

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

<p>
Now Ansible reads the list from <code>my_services</code> and starts each service automatically.
</p>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.2-using-vars-loop.PNG" width="850">
</p>

<hr/>

<h3>3️⃣ Using Dictionaries in Loops</h3>
<p>
Loops can also process complex data — each list item can contain multiple fields like <b>name</b> and <b>groups</b>.  
This is done with dictionaries (key–value pairs).
</p>

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

<p>
Each loop item is now a dictionary — containing structured data.  
Ansible reads it like a table and uses each value automatically.
</p>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.2-using-dicts-loop.PNG" width="850"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/loopusers.ymll7.PNG" width="850">
</p>

<hr/>

<h3>4️⃣ loop vs with_items</h3>
<ul>
  <li><b>loop</b> — current standard keyword.</li>
  <li><b>with_items</b> — old style from pre-2.5 playbooks.</li>
  <li><b>with_file</b> — loop through file contents.</li>
  <li><b>with_sequence</b> — generate a list of numbers automatically.</li>
</ul>

<p>
In short: <b>loop</b> replaces all <code>with_*</code> variations.  
If you see old syntax, just mentally translate it to “loop.”
</p>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/7.2-loop-vs-with.PNG" width="850">
</p>

<hr/>

<h3>Lab Example Summary</h3>
<ul>
  <li><b>loopservices.yml</b> — starts multiple services (<code>sshd</code>, <code>crond</code>).</li>
  <li><b>loopusers.yml</b> — creates multiple users and groups dynamically.</li>
  <li>Output shows each host creating users correctly, no failures, clean recap.</li>
</ul>

<p align="center">
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/loopservices.yml.PNG" width="800"><br>
  <img src="https://github.com/ITPAUL123184LINUX/infra-labs/blob/main/ansible/lesson7-task-control/docs/img/loopusers.ymll7.PNG" width="900">
</p>

<p><b>Key takeaway:</b> Loops save time, prevent copy-paste clutter, and scale effortlessly.  
Once you master loops, playbooks start feeling like code — not repetitive scripts.</p>

<hr/>

<h2>Next → 7.3 Using When</h2>
<p>
Next, we’ll make Ansible even smarter by using <b>when</b> to run tasks only under specific conditions.
</p>

<i>End of sections 7.1–7.2 notes</i>
